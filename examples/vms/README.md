# Virtual Machines

The 'kvm_vms' variable contains a list of virtual machines with their disks and interfaces.
The main purpose of this variable is to simplify the domain and storage volume creation.

A virtual machine has a limited set of configurable options which are combined with default options to create a list of domains and storage volumes.
These domains and storage volumes are then added to the existing 'kvm_domains' and 'kvm_storage_volumes' variables and further processed this way.

It is still possible to create separate domains and storage volumes when using virtual machines.

## Variables

```yaml
# Cloud Init server
# When using Cloud Init, make sure this is correct or the initial boot of VMs can fail
kvm_cloud_init_server: http://169.254.169.254/default/

# List of virtual machines
kvm_vms:
  - name: string        # Name of the virtual machine
    vcpu: int           # Number of virtual CPUs allocated to the virtual machine
    memory: string      # Amount of memory to allocate to the virtual machine, incl. unit
    efi: bool           # Enable/Disable UEFI (optional, default=true)
    cloud_init: true    # Enable/Disable Cloud Init (optional, default=true if kvm_cloud_init_server is defined)
    disks: list()       # Virtual machine disks (see below)
    interfaces: list()  # Network interfaces to attach to the virtual machine (see below)
```

## Disks

```yaml
disks:
  - pool: string  # Storage pool for disk volume
    name: string  # Name of already existing storage volume (optional, size and image are ignored)
    size: string  # Volume size
    image: string # Copied source volume for new volume (optional)
```

The disk volumes will be created together with the virtual machine, if no volume name is specified.

### Root Disk

The first defined disk will act as root volume and will be used to boot the domain.

This disk will be called <vm-name>-root.qcow2, if no volume name is specified.

### Data Disks

The other disks will be handled as data disks.

These disks will be called <vm-name>-data<nr>.qcow2, if no volume name is specified.

### Devices

All disks are virtio devices which will be called '/dev/vd[a-z]'.

## Interfaces

```yaml
interfaces:
  - bridge: string   # Name of bridge to use for the virtual machine
    vlan: int        # Optional VLAN ID (supported since Libvirt 11.0, see Linux Bridge VLAN filtering)
  - network: string  # Or name of network to use for the virtual machine
```

All interfaces are 'virtio' devices.

## UEFI

UEFI is enabled by default and has 'enrolled keys' and 'secure boot' enabled. These two parameters can not be disabled.

## Examples

Real life examples can be found in the YAML files located in this directory. 