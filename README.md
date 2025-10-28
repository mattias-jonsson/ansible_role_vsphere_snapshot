Ansible Role: vsphere_snapshot
=========

Handles list/create/revert/removal of virtual machine snapshots in VMware vSphere.


Requirements
------------

This role is dependent on `community.vmware` collection.

Role Variables
--------------

Available variables are listed below, along with default values where applicable (see `defaults/main.yml`):



| Variable | Required | Default | Comments |
| -------- | -------- | ------- | -------- |
| `vsphere_snapshot_action` | Yes | | Snapshot operation, valid values are: `create`, `remove`, `revert`, or `list`. |
| `vsphere_snapshot_vcenter` | Yes | | Hostname or IP address of the vCenter server. |
| `vsphere_snapshot_vcenter_username` | Yes | | Username used to connect to vCenter. |
| `vsphere_snapshot_vcenter_password` | Yes | | Password used to connect to vCenter. |
| `vsphere_snapshot_datacenter` | Yes | | vSphere Datacenter where the snapshot operations will take place. |
| `vsphere_snapshot_vm_instance_uuid` | Yes | | Instance UUID of the virtual machine. |
| `vsphere_snapshot_port` | No | `443` | Port of the vCenter server (HTTPS). |
| `vsphere_snapshot_validate_certs` | No | `true` | Validate SSL certificates when connecting to vCenter. |
| `vsphere_snapshot_description` | No | | Description to add to the snapshot (used for `create` action). |
| `vsphere_snapshot_name` | Conditional | | Name of existing snapshot (required for `remove` and `revert` actions). |
| `vsphere_snapshot_name_prefix` | No | | Prefix to add to the auto-generated snapshot name. |
| `vsphere_snapshot_name_suffix` | No | | Suffix to add to the auto-generated snapshot name. |
| `vsphere_snapshot_quiesce_guest_file_system` | No | `false` | Quiesce the file system in the virtual machine when creating snapshot. Mutually exclusive with `vsphere_snapshot_snapshot_memory`. |
| `vsphere_snapshot_snapshot_memory` | No | `true` | Include a memory dump of the virtual machine in the snapshot. Mutually exclusive with `vsphere_snapshot_quiesce_guest_file_system`. |
| `vsphere_snapshot_remove_after_revert` | No | `true` | Remove snapshot after successful revert operation. |
| `vsphere_snapshot_snapshot_poweredoff` | No | `false` | Allow snapshots of powered-off virtual machines. |
| `vsphere_snapshot_tools_wait_timeout` | No | `600` | Maximum time in seconds to wait for VMware Tools to become available after reverting a snapshot (only applies to powered-on VMs). |
| `vsphere_snapshot_override_op_ready` | No | `false` | Override the guest operations ready check (use with caution). |
| `vsphere_snapshot_override_kern_crash` | No | `false` | Override the kernel crash check (use with caution). |
| `vsphere_snapshot_proxy_host` | No | | Proxy server hostname (optional). |
| `vsphere_snapshot_proxy_port` | No | | Proxy server port (optional). |


Dependencies
------------

This role has no external dependencies.

Example Playbooks
-----------------

    - hosts: servers
      gather_facts: false

      vars:
        vsphere_snapshot_vcenter: 'vcenter.example.com'
        vsphere_snapshot_vcenter_username: 'administrator@vsphere.local'
        vsphere_snapshot_vcenter_password: 'secretpassword'
        vsphere_snapshot_datacenter: 'Production-DC'
        vsphere_snapshot_port: 443
        vsphere_snapshot_validate_certs: false
        vm_name: 'web-server-01'

      tasks:
        - name: Gather VM information to get instance UUID
          vmware.vmware.guest_info:
            hostname: "{{ vsphere_snapshot_vcenter }}"
            username: "{{ vsphere_snapshot_vcenter_username }}"
            password: "{{ vsphere_snapshot_vcenter_password }}"
            schema: "vsphere"
            validate_certs: "{{ vsphere_snapshot_validate_certs }}"
            datacenter: "{{ vsphere_snapshot_datacenter }}"
            folder: "{{ vsphere_snapshot_datacenter }}/vm/production/web-servers"
            guest_name: "{{ vm_name }}"
          register: vm_info

        - name: Set VM instance UUID fact
          ansible.builtin.set_fact:
            vsphere_snapshot_vm_instance_uuid: "{{ vm_info.guests[0].config.instanceUuid }}"

        - name: Patching servers with snapshot protection
          block:
            - name: Create a snapshot before patching
              ansible.builtin.include_role:
                name: ansible_role_vsphere_snapshot
              vars:
                vsphere_snapshot_action: create
                vsphere_snapshot_description: 'Pre-patching snapshot created by Ansible'
                vsphere_snapshot_name_prefix: 'pre-patch-'
                vsphere_snapshot_quiesce_guest_file_system: true
                vsphere_snapshot_snapshot_memory: false

            - name: Patch the server
              ansible.builtin.include_role:
                name: patch_servers_role

            - name: Remove the snapshot after successful patching
              ansible.builtin.include_role:
                name: ansible_role_vsphere_snapshot
              vars:
                vsphere_snapshot_action: remove
                vsphere_snapshot_name: '{{ vsphere_snapshot_generated_name }}'

          rescue:
            - name: Revert to snapshot on failure
              ansible.builtin.include_role:
                name: ansible_role_vsphere_snapshot
              vars:
                vsphere_snapshot_action: revert
                vsphere_snapshot_name: '{{ vsphere_snapshot_generated_name }}'
                vsphere_snapshot_remove_after_revert: true

### Example: List all snapshots for a virtual machine

    - hosts: servers
      gather_facts: false

      vars:
        vsphere_snapshot_vcenter: 'vcenter.example.com'
        vsphere_snapshot_vcenter_username: 'administrator@vsphere.local'
        vsphere_snapshot_vcenter_password: 'secretpassword'
        vsphere_snapshot_datacenter: 'Production-DC'
        vsphere_snapshot_validate_certs: false
        vm_name: 'web-server-01'

      tasks:
        - name: Gather VM information to get instance UUID
          vmware.vmware.guest_info:
            hostname: "{{ vsphere_snapshot_vcenter }}"
            username: "{{ vsphere_snapshot_vcenter_username }}"
            password: "{{ vsphere_snapshot_vcenter_password }}"
            schema: "vsphere"
            validate_certs: "{{ vsphere_snapshot_validate_certs }}"
            datacenter: "{{ vsphere_snapshot_datacenter }}"
            folder: "{{ vsphere_snapshot_datacenter }}/vm/production/web-servers"
            guest_name: "{{ vm_name }}"
          register: vm_info

        - name: Set VM instance UUID fact
          ansible.builtin.set_fact:
            vsphere_snapshot_vm_instance_uuid: "{{ vm_info.guests[0].config.instanceUuid }}"

        - name: List all snapshots for the virtual machine
          ansible.builtin.include_role:
            name: ansible_role_vsphere_snapshot
          vars:
            vsphere_snapshot_action: list

        - name: Display snapshot information
          ansible.builtin.debug:
            msg: "{{ vsphere_snapshot_snapshots }}"

### Example: Delete the 3 newest snapshots

    - hosts: servers
      gather_facts: false

      vars:
        vsphere_snapshot_vcenter: 'vcenter.example.com'
        vsphere_snapshot_vcenter_username: 'administrator@vsphere.local'
        vsphere_snapshot_vcenter_password: 'secretpassword'
        vsphere_snapshot_datacenter: 'Production-DC'
        vsphere_snapshot_validate_certs: false
        vm_name: 'web-server-01'

      tasks:
        - name: Gather VM information to get instance UUID
          vmware.vmware.guest_info:
            hostname: "{{ vsphere_snapshot_vcenter }}"
            username: "{{ vsphere_snapshot_vcenter_username }}"
            password: "{{ vsphere_snapshot_vcenter_password }}"
            schema: "vsphere"
            validate_certs: "{{ vsphere_snapshot_validate_certs }}"
            datacenter: "{{ vsphere_snapshot_datacenter }}"
            folder: "{{ vsphere_snapshot_datacenter }}/vm/production/web-servers"
            guest_name: "{{ vm_name }}"
          register: vm_info

        - name: Set VM instance UUID fact
          ansible.builtin.set_fact:
            vsphere_snapshot_vm_instance_uuid: "{{ vm_info.guests[0].config.instanceUuid }}"

        - name: List all snapshots for the virtual machine
          ansible.builtin.include_role:
            name: ansible_role_vsphere_snapshot
          vars:
            vsphere_snapshot_action: list

        - name: Sort snapshots by age (newest first) and select top 3
          ansible.builtin.set_fact:
            newest_3_snapshots: "{{ (vsphere_snapshot_snapshots | sort(attribute='age_seconds'))[:3] }}"

        - name: Display the 3 newest snapshots to be deleted
          ansible.builtin.debug:
            msg: "Will delete: {{ snapshot.name }} (age: {{ snapshot.age_days }} days)"
          loop: "{{ newest_3_snapshots }}"
          loop_control:
            loop_var: snapshot
            label: "{{ snapshot.name }}"

        - name: Delete the 3 newest snapshots
          ansible.builtin.include_role:
            name: ansible_role_vsphere_snapshot
          vars:
            vsphere_snapshot_action: remove
            vsphere_snapshot_name: "{{ snapshot.name }}"
          loop: "{{ newest_3_snapshots }}"
          loop_control:
            loop_var: snapshot
            label: "{{ snapshot.name }}"

### Example: Delete snapshots older than 7 days

    - hosts: servers
      gather_facts: false

      vars:
        vsphere_snapshot_vcenter: 'vcenter.example.com'
        vsphere_snapshot_vcenter_username: 'administrator@vsphere.local'
        vsphere_snapshot_vcenter_password: 'secretpassword'
        vsphere_snapshot_datacenter: 'Production-DC'
        vsphere_snapshot_validate_certs: false
        vm_name: 'web-server-01'
        max_snapshot_age_days: 7

      tasks:
        - name: Gather VM information to get instance UUID
          vmware.vmware.guest_info:
            hostname: "{{ vsphere_snapshot_vcenter }}"
            username: "{{ vsphere_snapshot_vcenter_username }}"
            password: "{{ vsphere_snapshot_vcenter_password }}"
            schema: "vsphere"
            validate_certs: "{{ vsphere_snapshot_validate_certs }}"
            datacenter: "{{ vsphere_snapshot_datacenter }}"
            folder: "{{ vsphere_snapshot_datacenter }}/vm/production/web-servers"
            guest_name: "{{ vm_name }}"
          register: vm_info

        - name: Set VM instance UUID fact
          ansible.builtin.set_fact:
            vsphere_snapshot_vm_instance_uuid: "{{ vm_info.guests[0].config.instanceUuid }}"

        - name: List all snapshots for the virtual machine
          ansible.builtin.include_role:
            name: ansible_role_vsphere_snapshot
          vars:
            vsphere_snapshot_action: list

        - name: Display snapshots older than {{ max_snapshot_age_days }} days
          ansible.builtin.debug:
            msg: "Found {{ old_snapshots | length }} snapshots older than {{ max_snapshot_age_days }} days"
          vars:
            old_snapshots: "{{ vsphere_snapshot_snapshots | selectattr('age_days', '>', max_snapshot_age_days) | list }}"

        - name: Delete snapshots older than {{ max_snapshot_age_days }} days
          ansible.builtin.include_role:
            name: ansible_role_vsphere_snapshot
          vars:
            vsphere_snapshot_action: remove
            vsphere_snapshot_name: "{{ snapshot.name }}"
          loop: "{{ vsphere_snapshot_snapshots | selectattr('age_days', '>', max_snapshot_age_days) | list }}"
          loop_control:
            loop_var: snapshot
            label: "{{ snapshot.name }} ({{ snapshot.age_days }} days old)"

### Return Values

When using the `list` action, the role populates the following variable:

- `vsphere_snapshot_snapshots`: A list of dictionaries containing snapshot information including:
  - `name`: Snapshot name
  - `description`: Snapshot description
  - `creation_time`: When the snapshot was created
  - `state`: Snapshot state
  - `age_days`: Age of snapshot in days
  - `age_hours`: Age of snapshot in hours
  - `age_minutes`: Age of snapshot in minutes
  - `age_seconds`: Age of snapshot in seconds

When using the `create` action, the role populates:

- `vsphere_snapshot_generated_name`: The auto-generated name of the created snapshot
- `vsphere_snapshot_created`: Boolean indicating if a snapshot was created

When using the `remove` action, the role populates:

- `vsphere_snapshot_removed`: Boolean indicating if the snapshot was successfully removed

When using the `revert` action, the role populates:

- `vsphere_snapshot_reverted`: Boolean indicating if the snapshot was successfully reverted to


License
-------

MIT

Author Information
------------------

Mattias Jonsson
