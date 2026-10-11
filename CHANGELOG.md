# Changelog

## [1.5.0] - 2026-08-30

### New Features

- Input Validation: added role argument specs that check the provided variables before any task runs
- Storage Volumes: a dry run (--check) now lists which volumes would be created or grown
- Install: IOMMU now picks the Intel or AMD kernel flag based on the host CPU, and can be turned off with 'kvm_enable_iommu'

### Bug Fixes

- Templates: fixed malformed XML in storage volume, storage pool, network and domain templates
- Networks: a DHCP host with a 'lease' no longer produces invalid XML
- Networks: a 'dns' block no longer needs all four lists (forwarder, txt, srv, hosts); missing lists are skipped
- Networks: DNS host names are no longer also written as an invalid 'hostname' attribute
- Domains: nested dictionaries are now rendered at any depth
- Virtual Machines: no longer fail when no volumes or domains are generated
- Virtual Machines: the qcow2 'compat' setting is now placed under 'target', it was ignored under 'format'
- Storage Volumes: existing volumes are now grown automatically when their configured capacity increases; the manual 'resize' key has been removed
- Install: virt-host-validate warnings no longer abort the run; it now stops only on a real failure

### Other Changes

- Added requirements.yml so the 'community.libvirt' collection can be installed with ansible-galaxy

## [1.4.2] - 2025-05-03

### Bug Fixes

- Storage Volumes: Changed static pool name to variable
- Storage Volumes: Resize new image based volumes to match configured capacity
- Ansible Lint fixes

## [1.4.1] - 2025-02-10

### Bug Fixes

- Domains: replace '.' in domain name with '_' for interface aliases

## [1.4.0] - 2025-01-26

### New Features

- Virtual Machines: Added interface VLAN support, supported since Libvirt 11.0 for regular linux bridges (VLAN filtering)

## [1.3.0] - 2024-12-20

### New Features

- Virtual Machines: Added simplified way to create domains and storage volumes via virtual machines

## [1.2.0] - 2024-10-29

### New Features

- Domains: Added OS firmware options to enable EFI and secure boot configuration
- Domains: Added UEFI firmware examples

## [1.1.0] - 2024-10-14

### New Features

- Domains: Added SMBIOS System Information option (needed for cloud-init)
- Domains: Added cloud-init and host bridge examples
- Networks: Added host bridge and open type examples

### Other Changes

- Networks: Made ip variable optional 

## [1.0.0] - 2024-09-28

### New Features

- Install KVM and dependencies
- Libvirt network configuration
- Libvirt storage pool configuration
- Libvirt storage volume configuration
- Libvirt domain configuration
- Documentation and examples