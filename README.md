# Ansible Role: KVM

This role will install and configure KVM and all its dependencies.

The role can also be used to create and manage Libvirt networks, storage pools, storage volumes and domains.

**Note:** Currently, the removal of Libvirt objects has to be executed manually.

## Requirements

This role has to be executed as user 'root'.

## Tags

Following tags are supported by this role:

  * install
  * networks
  * storage_pools
  * storage_volumes
  * domains

## Variables

| Variable                                        | Type   | Required | Default | Comment                               |
|-------------------------------------------------|--------|----------|---------|---------------------------------------|
| [kvm_networks](examples/networks)               | list() | No       | []      | Libvirt networks configuration        |
| [kvm_storage_pools](examples/storage/pools)     | list() | No       | []      | Libvirt storage pools configuration   |
| [kvm_storage_volumes](examples/storage/volumes) | list() | No       | []      | Libvirt storage volumes configuration |
| [kvm_domains](examples/domains)                 | list() | No       | []      | Libvirt domains configuration         |

**Note:** Click on a variable to see more information on how to use it.

## Management

### Virsh

The 'virsh' command line tool can be used to manage and configure domains.

```shell
root # virsh
Welcome to virsh, the virtualization interactive terminal.

Type:  'help' for help with commands
       'quit' to quit

virsh # 
```

### Cockpit

Cockpit is a browser-based administration tool for Linux which also provides a package for virtual machines.

https://github.com/cockpit-project/cockpit-machines

**Note:** This module also provides virtual machine console access.
