# Storage Volumes

The 'kvm_storage_volumes' variable contains a list of storage volumes to create and/or resize.

The storage volume configuration is based on the [Libvirt XML configuration](https://libvirt.org/formatstorage.html#storage-volume-xml).

## Variables

Following Libvirt XML elements are supported:

```yaml
kvm_storage_volumes:
  - name: string # Name of storage volume
    pool: string # Pool name to place storage volume in
    from: string # Pathname of image to copy content from (OPTIONAL)
    resize: bool # Inform when a volume needs to be resized with a newly configured capacity (OPTIONAL) 
    allocation: string  # Total storage allocation for the volume, incl. unit (OPTIONAL)
    capacity: string    # Total storage capacity for the volume, incl. unit
    target: dict()  # Describes mapping of the storage volume in the hosts filesystem (OPTIONAL)
    backingStore: {}  # Describe backing store for the storage volumes (OPTIONAL)
```

**Note:**  The 'pool' and 'from' keys are not part of the Libvirt XML file.
These keys are needed during storage volume creation.

**Note:**  The 'resize' key is not part of the Libvirt XML file.
This key is used to enforce a storage volume resize.

The [storage_volumes.xml.j2](templates/storage_volumes.xml.j2) is used to convert each storage volume configuration to an XML file. 

## Examples

Below examples provide an overview of the common options for each supported element.
The element configurations are not limited to what is shown here.

The [Libvirt XML documentation](https://libvirt.org/formatstorage.html#storage-volume-xml) contains an overview 
of all available options for the supported elements.

**Note:** Real life examples can be found in the YAML files located in this directory. 

### Target

```yaml
kvm_storage_volumes:
  - name: volume1.raw
    pool: virtimages
    from: /var/lib/virt/images/image.qcow2
    allocation: 0B
    capacity: 1T
    target:
      path: /var/lib/virt/images/volume1.raw
      permissions:
        owner: 107
        group: 107
        mode: 0744
        label: virt_image_t
```

### Backing Store

```yaml
kvm_storage_volumes:
  - name: volume1.qcow2
    pool: virtimages
    capacity: 1T
    target:
      format:
        type: qcow2
        compat: 1.1
    backingStore:
      path: /var/lib/virt/images/image.qcow2
      format:
        type: qcow2
```

### Resize

```yaml
kvm_storage_volumes:
  - name: volume1.raw
    pool: virtimages
    resize: true  # Can be removed or disabled again when the storage volume is resized
    capacity: 1T
```

**Note:** It is not possible to shrink a volume.