# Storage Pools

The 'kvm_storage_pools' variable contains a list of storage pools to create and/or modify.

The storage pool configuration is based on the [Libvirt XML configuration](https://libvirt.org/formatstorage.html#storage-pool-xml).

## Variables

Following Libvirt XML elements are supported:

```yaml
kvm_storage_pools:
  - name: string # Name of the storage pool
    type: string # Type of storage pool
    features: dict() # Control copy-on-write state for BTRFS (OPTIONAL)
    source: {}  # Source of the storage pool (OPTIONAL)
    target: {}  # Describes mapping of the storage pool into the host filesystem
```

The [storage_pools.xml.j2](templates/storage_pools.xml.j2) is used to convert each storage pool configuration to an XML file. 

## Examples

Below examples provide an overview of the common options for each supported element.
The element configurations are not limited to what is shown here.

The [Libvirt XML documentation](https://libvirt.org/formatstorage.html#storage-pool-xml) contains an overview 
of all available options for the supported elements.

**Note:** Real life examples can be found in the YAML files located in this directory. 

### Features

```yaml
kvm_storage_pools:
  - name: virtimages
    type: dir
    features:
      cow:
        state: 'no'
    target:
      path: /var/lib/virt/images
```

### Source

```yaml
kvm_storage_pools:
  - name: dir-virtimages
    type: dir
    target:
      path: /var/lib/virt/images
  - name: iscsi-virtimages
    type: iscsi
    source:
      host:
        name: iscsi.example.com
      device:
        path: iqn.2013-06.com.example:iscsi-pool
      auth:
        type: chap
        username: myuser
        secret:
          usage: libvirtiscsi
    target:
      path: /dev/disk/by-path
  - name: nfs-virtimages
    type: netfs
    source:
      host:
        name: nfs.example.com
      dir:
        path: /var/lib/libvirt/images
      format:
        type: nfs
      protocol:
        ver: 3
    target:
      path: /dev/disk/by-path
```

### Target

```yaml
kvm_storage_pools:
  - name: virtimages
    type: dir
    target:
      path: /virtimages
      permissions:
        owner: 107
        group: 107
        mode: '0744'
        label: virt_image_t
```
