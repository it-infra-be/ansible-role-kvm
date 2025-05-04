# Changelog

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