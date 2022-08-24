Ansible Role: ansible_role_vsphere_snapshot
=========

An Ansible role that handles create/revert/removal of snapshots in VMware vSphere.


Requirements
------------

This role is dependent on the VMware community modules
Install with ansible-galaxy collection install community.vmware

Role Variables
--------------

Available variables are listed below, along with default values where applicable (see defaults/main.yml):

    ansible_role_vsphere_snapshot_datacenter: ''

vSphere Datacenter where the snapshot operations will take place.

    ansible_role_vsphere_snapshot_description: ''

Description to add to the snapshot.

    ansible_role_vsphere_snapshot_instance_uuid: ''

Instance UUID of virtual machine.

    ansible_role_vsphere_snapshot_name_prefix: ''

Prefix to add to the snapshot name.

    ansible_role_vsphere_snapshot_name_suffix: ''

Suffix to add to the snapshot name.

    ansible_role_vsphere_snapshot_port: ''

Port of the vCenter where snapshot operations will take place.

    ansible_role_vsphere_snapshot_quiesce_guest_file_system: 'false'

If set to true, the file system in the virtual machine will be quiesced if the virtual machine is powered on.

    ansible_role_vsphere_snapshot_remove_after_revert: 'true'

(Boolean) Remove snapshot after successful revert operation

    ansible_role_vsphere_snapshot_snapshot_memory: 'true'

If set to true, a memory dump of the virtual machine is included in the snapshot.

    ansible_role_vsphere_snapshot_validate_certs: ''

(Boolean) Validate SSL certificates when connecting to vCenter.

    ansible_role_vsphere_snapshot_vcenter_password: ''

Password used to connect to vCenter.

    ansible_role_vsphere_snapshot_vcenter_username: ''

Username used to connect to vCenter.

    ansible_role_vsphere_snapshot_vcenter: ''

Name of the vCenter where snapshot operations will take place.
    


Dependencies
------------

This role has a dependencie on the role ansible_role_vsphere_snapshot if vSphere snapshot functionality will be utilized.

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
        ansible_role_vsphere_snapshot_remove_after_revert: 'false'
        ansible_role_vsphere_snapshot_snapshot_memory: 'true'
        ansible_role_vsphere_snapshot_validate_certs: 'false'
        ansible_role_vsphere_snapshot_vcenter_password: 'secretpassword'
        ansible_role_vsphere_snapshot_vcenter_username: 'secretusername'
        ansible_role_vsphere_snapshot_vcenter: 'testvcenter.testdomain.com'


      roles:
         - role: ansible_role_vsphere_snapshot

License
-------

MIT

Author Information
------------------

Mattias Jonsson