Ansible Role: ansible_role_vsphere_snapshot
=========

Handles create/revert/removal of snapshots in VMware vSphere.


Requirements
------------

This role is dependent on `community.vmware` collection.

Role Variables
--------------

Available variables are listed below, along with default values where applicable (see `defaults/main.yml`):



| Variable | Required | Default | Comments |
| -------- | -------- | ------- | -------- |
| `ansible_role_vsphere_snapshot_action` | Yes | | Snapshot operation, valid values are: `create`, `delete` or `revert`. |
| `ansible_role_vsphere_snapshot_datacenter` | Yes | | vSphere Datacenter where the snapshot operations will take place. |
| `ansible_role_vsphere_snapshot_description` | Yes | | Description to add to the snapshot. |
| `ansible_role_vsphere_snapshot_instance_uuid` | Yes | | Instance UUID of virtual machine. |
| `ansible_role_vsphere_snapshot_name_prefix` | No | | Prefix to add to the snapshot name. |
| `ansible_role_vsphere_snapshot_name_suffix` | No | | Suffix to add to the snapshot name. |
| `ansible_role_vsphere_snapshot_port` | Yes | | Port of the vCenter where snapshot operations will take place, common value would be 443 for HTTPS. |
| `ansible_role_vsphere_snapshot_quiesce_guest_file_system` | No | false | If set to true, the file system in the virtual machine will be quiesced if the virtual machine is powered on. |
| `ansible_role_vsphere_snapshot_remove_after_revert` | No | true | (Boolean) Remove snapshot after successful revert operation |
| `ansible_role_vsphere_snapshot_snapshot_memory` | No | true | If set to true, a memory dump of the virtual machine is included in the snapshot. |
| `ansible_role_vsphere_snapshot_validate_certs` | Yes | | (Boolean) Validate SSL certificates when connecting to vCenter. |
| `ansible_role_vsphere_snapshot_vcenter_password` | Yes | | Password used to connect to vCenter. |
| `ansible_role_vsphere_snapshot_vcenter_username` | Yes | | Username used to connect to vCenter. |
| `ansible_role_vsphere_snapshot_vcenter` | Yes | | Name of the vCenter where snapshot operations will take place. |


Dependencies
------------

This role has no external dependencies.

Example Playbook
----------------

    - hosts: servers

      vars:
        ansible_role_vsphere_snapshot_datacenter: 'testdc'
        ansible_role_vsphere_snapshot_description: 'Automatic snapshot created by Ansible'
        ansible_role_vsphere_snapshot_instance_uuid: '01234567890abcdef'
        ansible_role_vsphere_snapshot_name_prefix: 'aaa_'
        ansible_role_vsphere_snapshot_name_suffix: '01234'
        ansible_role_vsphere_snapshot_port: '443'
        ansible_role_vsphere_snapshot_quiesce_guest_file_system: 'true'
        ansible_role_vsphere_snapshot_remove_after_revert: 'true'
        ansible_role_vsphere_snapshot_snapshot_memory: 'true'
        ansible_role_vsphere_snapshot_validate_certs: 'false'
        ansible_role_vsphere_snapshot_vcenter_password: 'secretpassword'
        ansible_role_vsphere_snapshot_vcenter_username: 'secretusername'
        ansible_role_vsphere_snapshot_vcenter: 'testvcenter.testdomain.com'
 
      tasks:
        - name: Patching servers block.
          block:
            - name: Include the snapshot role to create a snapshot.
              ansible.builtin.include_role:
                name: ansible_role_vsphere_snapshot
                vars:
                  ansible_role_vsphere_snapshot_action: create

            - name: Include the software update role to patch the server.
              ansible.builtin.include_role:
                name: patch_servers_role

            - name: Include the snapshot role to remove a snapshot.
              ansible.builtin.include_role:
                name: ansible_role_vsphere_snapshot
                vars:
                  ansible_role_vsphere_snapshot_action: remove

          rescue:
            - name: Include the snapshot role to revert a snapshot.
              ansible.builtin.include_role:
                name: ansible_role_vsphere_snapshot
                vars:
                  ansible_role_vsphere_snapshot_action: revert




License
-------

MIT

Author Information
------------------

Mattias Jonsson