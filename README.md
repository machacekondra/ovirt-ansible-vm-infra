oVirt Virtual Machine Infrastructure
====================================

The `oVirt.vm-infra` role manages the virtual machine infrastructure in oVirt.

Requirements
------------

 * Ansible version 2.5 or higher
 * Python SDK version 4.2 or higher

Role Variables
--------------

| Name                           | Default value |                                              |
|--------------------------------|---------------|----------------------------------------------| 
| vms                            | UNDEF         | List of dictionaries with virtual machine specifications.   |
| affinity_groups                | UNDEF         | List of dictionaries with affinity groups specifications.   |
| wait_for_ip                    | true          | If true, the playbook should wait for the virtual machine IP reported by the guest agent.  |
| debug_vm_create                | false         | If true, logs the tasks of the virtual machine being created. The log can contain passwords. |
| vm_infra_create_single_timeout | 180           | Time in seconds to wait for VM to be created and started (if state is running). |
| vm_infra_create_poll_interval  | 15            | Polling interval. Time in seconds to wait between check of state of VM.  |
| vm_infra_create_all_timeout    | vm_infra_create_single_timeout * (vms.length) | Total time to wait for all VMs to be created/started. |
| vm_infra_wait_for_ip_retries   | 5             | Number of retries to check if VM is reporting it's IP address. |
| vm_infra_wait_for_ip_delay     | 5             | Polling interval of IP address. Time in seconds to wait between check if VM reports IP address. |


The `vms` list can contain following attributes:

| Name               | Default value         |                                            |
|--------------------|-----------------------|--------------------------------------------| 
| name               | UNDEF                 | Name of the virtual machine to create.     |
| tag                | UNDEF                 | Name of the tag to assign to the virtual machine. Only administrator users can use this attribute.  |
| cloud_init         | UNDEF                 | Dictionary with values for Unix-like Virtual Machine initialization using cloud init. See [cloud_init] upstream parameter for more detailed description. |
| cloud_init_nics    | UNDEF                 | List of dictionaries representing network interafaces to be setup by cloud init. See [cloud_init_nics] upstream parameter for more detailed description. |
| profile            | UNDEF                 | Dictionary specifying the virtual machine hardware. See the table below.  |
| state              | present               | Should the Virtual Machine be stopped, present or running. Takes precedence before state value in profile. |
| nics               | UNDEF                 | List of dictionaries specifying the NICs of the virtual machine. See below for more detailed description.   |

The `profile` dictionary can contain following attributes:

| Name               | Default value         |                                              |
|--------------------|-----------------------|----------------------------------------------|
| cluster            | Default               | Name of the cluster where the virtual machine will be created. |
| clone              | No                    | If yes then the disks of the created virtual machine will be cloned and independent of the template.  This parameter is used only when state is running or present and VM didn't exist before.  |
| template           | Blank                 | Name of template that the virtual machine should be based on.   |
| template_version   | Blank                 | Version number of the template to be used for VM. By default the latest available version of the template is used.   |
| memory             | 2GiB                  | Amount of virtual machine memory.               |
| memory_guaranteed  | UNDEF                 | Amount of minimal guaranteed memory of the Virtual Machine. Prefix uses IEC 60027-2 standard (for example 1GiB, 1024MiB). <i>memory_guaranteed</i> parameter can't be lower than <i>memory</i> parameter. |
| cores              | 1                     | Number of CPU cores used by the the virtual machine.          |
| sockets            | UNDEF                 | Number of virtual CPUs sockets of the Virtual Machine.  |
| disks              | UNDEF                 | List of dictionaries specifying the additional virtual machine disks. See below for more detailed description. |
| nics               | UNDEF                 | List of dictionaries specifying the NICs of the virtual machine. See below for more detailed description.   |
| high_availability  | UNDEF                 | Whether or not the node should be set highly available. |
| storage_domain     | UNDEF                 | Name of the storage domain where all virtual machine disks should be created. Considered only when template is provided.|
| state              | present               | Should the Virtual Machine be stopped, present or running.|
| ssh_key            | UNDEF                 | SSH key to be deployed to the virtual machine. This is parameter is keep for backward compatibility and has precendece before <i>authorized_ssh_keys</i> in <i>cloud_init</i> dictionary. |
| domain             | UNDEF                 | The domain of the virtual machine. This is parameter is keep for backward compatibility and has precendece before <i>host_name</i> in <i>cloud_init</i> dictionary.|
| root_password      | UNDEF                 | The root password of the virtual machine. This is parameter is keep for backward compatibility and has precendece before <i>root_password</i> in <i>cloud_init</i> dictionary.|

The item in `disks` list of `profile` dictionary can contain following attributes:

| Name               | Default value  |                                              |
|--------------------|----------------|----------------------------------------------| 
| size               | UNDEF          | The size of the additional disk. |
| name               | UNDEF          | The name of the additional disk.  |
| storage_domain     | UNDEF          | The name of storage domain where disk should be created. |
| interface          | UNDEF          | The interface of the disk. |
| format             | UNDEF          | Specify format of the disk.  <ul><li>cow - If set, the disk will by created as sparse disk, so space will be allocated for the volume as needed. This format is also known as thin provisioned disks</li><li>raw - If set, disk space will be allocated right away. This format is also known as preallocated disks.</li></ul> |
| bootable           | UNDEF          | True if the disk should be bootable. |

The item in `nics` list of `profile` dictionary can contain following attributes:

| Name               | Default value  |                                              |
|--------------------|----------------|----------------------------------------------| 
| name               | UNDEF          | The name of the network interface.           |
| interface          | UNDEF          | Type of the network interface.               |
| mac_address        | UNDEF          | Custom MAC address of the network interface, by default it's obtained from MAC pool. |
| network            | UNDEF          | Logical network which the VM network interface should use. If network is not specified, then Empty network is used. |
| profile            | UNDEF          | Virtual network interface profile to be attached to VM network interface. |

The `affinity_groups` list can contain following attributes:

| Name               | Default value       |                                              |
|--------------------|---------------------|----------------------------------------------|
| cluster            | UNDEF (Required)    |  Name of the cluster of the affinity group.  |
| description        | UNDEF               |  Human readable description.                 |
| host_enforcing     | false               |  <ul><li>true - VM cannot start on host if it does not satisfy the `host_rule`.</li><li>false - VM will follow `host_rule` with soft enforcement.</li></ul>|
| host_rule          | UNDEF               |  <ul><li>positive - VM's in this group must run on this host.</li> <li>negative - VM's in this group may not run on this host</li></ul> |
| hosts              | UNDEF               |  List of host names assigned to this group.  |
| name               | UNDEF (Required)    |  Name of affinity group.                     |
| state              | UNDEF               |  Whether group should be present or absent.  |
| vm_enforcing       | false               |  <ul><li>true - VM cannot start if it cannot satisfy the `vm_rule`.</li><li>false - VM will follow `vm_rule` with soft enforcement.</li></ul> |
| vm_rule            | UNDEF               |  <ul><li>positive - all vms in this group try to run on the same host.</li><li>negative - all vms in this group try to run on separate hosts.</li><li>disabled - this affinity group does not take effect.</li></ul> |
| vms                | UNDEF               |  List of VM's to be assigned to this affinity group. |
| wait               | true                |  If true, the module will wait for the desired state. |

Dependencies
------------

None.

Example Playbook
----------------

```yaml
---
- name: oVirt infra
  hosts: localhost
  connection: local
  gather_facts: false

  vars_files:
    # Contains encrypted `engine_password` varibale using ansible-vault
    - passwords.yml

  vars:
    engine_url: https://ovirt-engine.example.com/ovirt-engine/api
    engine_user: admin@internal
    engine_cafile: /etc/pki/ovirt-engine/ca.pem

    httpd_vm:
      cluster: production
      domain: example.com
      template: rhel7
      memory: 2GiB
      cores: 2
      ssh_key: ssh-rsa AAA...LGx user@fqdn
      disks:
        - size: 10GiB
          name: data
          storage_domain: mynfsstorage
          interface: virtio

    db_vm:
      cluster: production
      domain: example.com
      template: rhel7
      memory: 4GiB
      cores: 1
      ssh_key: ssh-rsa AAA...LGx user@fqdn
      disks:
        - size: 50GiB
          name: data
          storage_domain: mynfsstorage
          interface: virtio
      nics:
        - name: ovirtmgmt
          network: ovirtmgmt
          profile: ovirtmgmt

    vms:
      - name: postgresql-vm-0
        tag: postgresql_vm
        profile: "{{ db_vm }}"
      - name: postgresql-vm-1
        tag: postgresql_vm
        profile: "{{ db_vm }}"
      - name: apache-vm
        tag: httpd_vm
        profile: "{{ httpd_vm }}"

    affinity_groups:
      - name: db-ag
        cluster: production
        vm_enforcing: true
        vm_rule: negative
        vms:
          - postgresql-vm-0
          - postgresql-vm-1

  roles:
    - oVirt.vm-infra
```

[![asciicast](https://asciinema.org/a/111662.png)](https://asciinema.org/a/111662)
[cloud_init]: http://docs.ansible.com/ansible/latest/ovirt_vms_module.html#parameters
[cloud_init_nics]: http://docs.ansible.com/ansible/latest/ovirt_vms_module.html#parameters

License
-------

Apache License 2.0
