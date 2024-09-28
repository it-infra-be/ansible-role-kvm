# Domains

The 'kvm_domains' variable contains a list of virtual machines to create and/or modify.

The domain configuration is based on the [Libvirt XML configuration](https://libvirt.org/formatdomain.html).

## Variables

Following Libvirt XML elements are supported:

```yaml
kvm_domains:
  - name: string      # Name of the virtual machine
    title: string     # Title of the virtual machine (OPTIONAL)
    description: string # Description of the virtual machine (OPTIONAL)
    memory: string      # Maximum amount of memory to allocate to the virtual machine, incl. unit
    maxMemory: string   # Run-time Maximum amount of memory that can be assigned to the virtual machine, incl. unit (OPTIONAL)
    currentMemory: string # Actual allocation of memory for the guest (by default same as memory), incl unit (OPTIONAL)
    memory_options: dict()    # Memory options, excl. unit (OPTIONAL)
    maxMemory_options: dict() # maxMemroy options, excl. unit (OPTIONAL)
    vcpu: int             # Maximum number of virtual CPUs allocated to the virtual machine
    vcpu_options: dict()  # Virtual CPU Options (OPTIONAL)
    vcpus: list()         # Control state of individual virtual CPUs (OPTIONAL)
    disks: list()       # Describe disks (OPTIONAL)
    filesystems: list() # Host directories and/or templates which can be accesses directly in the virtual machine (OPTIONAL) 
    interfaces: list()  # Network interfaces to attach to the virtual machine (OPTIONAL)
```

**Note:** The keys 'vcpu_options', 'memory_options' and 'mamMemory_options' are not directly available
in the Libvirt XML file. These are used to specify the XML attributes of the 'vcpu', 'memory' and 'maxMemory'
elements as these XML elements also have content.

The [domain.xml.j2](templates/domain.xml.j2) is used to convert each domain configuration to an XML file. 

## Examples

Below examples provide an overview of the common options for each supported element.
The element configurations are not limited to what is shown here.

The [Libvirt XML documentation](https://libvirt.org/formatdomain.html) contains an overview 
of all available options for the supported elements.

**Note:** Real life examples can be found in the YAML files located in this directory. 

### Memory Options

```yaml
kvm_domains:
  - name: vm1
    title: Virtual Machine 1
    description: Virtual Machine 1
    memory: 4GiB
    maxMemory: 6GiB
    currentMemory: 2GiB
    memory_options:
      dumpCore: 'off'
    maxMemory_options:
      slots: 16
    vcpu: 2
```

### vCPU Options

```yaml
kvm_domains:
  - name: vm1
    title: Virtual Machine 1
    description: Virtual Machine 1
    memory: 4GiB
    vcpu: 2
    vcpu_options:
      placement: static
      cpuset: '1-4, ^3,6'
      current: 1
    vcpus:
      - id: 0
        enabled: 'yes'
        hotpluggable: 'no'
        order: 1
      - id: 1
        enabled: 'no'
        hotpluggable: 'yes'
```

### Disks

```yaml
kvm_domains:
  - name: vm1
    memory: 2GiB
    vcpu: 2
    disks:
      - type: file
        device: cdrom
        boot:
          order: 2
        source:
          file: /isos/AlmaLinux.iso
        target:
          dev: sdx
          bus: sata
        readonly: true
      - type: volume
        device: disk
        boot:
          order: 1
        source:
          pool: virtimages
          volume: volume1.qcow2
        target:
          dev: vda
          bus: virtio
```

### Filesystems

```yaml
kvm_domains:
  - name: vm1
    memory: 2GiB
    vcpu: 2
    filesystems:
      - type: template
        source:
          name: my-vm-template
        target:
          dir: /
      - type: mount
        accessmode: mapped
        fmode: '644'
        dmode: '755'
        driver:
          type: path
        source:
          dir: /export/to/guest
        target:
          dir: /import/from/host
        readonly: true
      - type: file
        accessmode: passthrough
        driver:
          type: loop
          format: raw
        source:
          file: /export/to/guest.img
        target:
          dir: /import/from/host
        readonly: true
```

### Interfaces

```yaml
kvm_domains:
  - name: vm1
    memory: 2GiB
    vcpu: 2
    interfaces:
      - type: network
        source:
          network: default
        model:
          type: virtio
```