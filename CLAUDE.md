# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

An Ansible role (`it_infra_be.kvm`) that installs KVM/libvirt on Enterprise Linux 9 and
declaratively manages libvirt networks, storage pools, storage volumes, and domains.
There is no playbook, no test suite, and no CI config in the repo — it is consumed as a
Galaxy role by other projects.

- Target platform: EL9 only. `min_ansible_version` is 2.15.
- Must run as `root`.
- Requires the `community.libvirt` collection on the controller.
- Object **removal is not implemented** — deleting a network/pool/volume/domain is a manual `virsh` operation.

## Commands

```sh
# Lint (the CHANGELOG shows ansible-lint is the accepted linter; noqa directives exist in tasks/)
ansible-lint

# Exercise the role — write a throwaway playbook that includes it, then:
ansible-playbook -i localhost, -c local test.yml --tags vms
```

Run subsets with tags: `install`, `networks`, `storage_pools`, `storage_volumes`,
`domains`, `vms`. The `vms` tag also pulls in the `storage_volumes` and `domains` task files.

## Architecture

### Entry order (`tasks/main.yml`)

`install` → `networks` → `storage_pools` → `vms` → `storage_volumes` → `domains`.

`tasks/vms.yml` must run before `storage_volumes`/`domains` because it *expands* the
high-level `kvm_vms` input into the low-level lists those task files consume (see below).

### Two input levels

Low-level vars map almost 1:1 onto libvirt XML:
`kvm_networks`, `kvm_storage_pools`, `kvm_storage_volumes`, `kvm_domains`.

`kvm_vms` is sugar. `tasks/vms.yml` renders `templates/vm_storage_volumes.yml.j2` and
`templates/vm_domains.yml.j2`, parses them with `from_yaml`, and **appends** the results
onto `kvm_storage_volumes` / `kvm_domains` via `set_fact`. Everything downstream only ever
sees the low-level lists.

`kvm_vms` conventions: first disk becomes `<vm>-root.qcow2` (boot order 1, target `vda`);
extra disks become `<vm>-data<N>.qcow2` (`vdb`, `vdc`, …). `disk.image` is copied into
the new volume (`virsh vol-create-from`, a full copy, not a backing image) then resized to
`disk.size`. EFI + secure boot and
cloud-init (nocloud-net SMBIOS serial) are **on by default**, disabled per-VM with
`efi: false` / `cloud_init: false`. Cloud-init injection also requires the
`kvm_cloud_init_server` var to be set.

### The dict-to-XML templating pattern (central idea)

Each task passes the loop `item` dict straight into a Jinja2 template as `template_vars`,
and the template walks the dict generically to emit XML: **scalar values become XML
attributes, nested mappings become nested elements, keys become tag/attribute names.**
So the YAML you write in the example files is essentially libvirt XML expressed as a dict.

- `templates/domain.xml.j2` — has a reusable `dict_to_xml(name, data, ignore_list)` macro
  that recurses to any nesting depth. `ignore_list` names keys that need bespoke handling
  and leaves the root tag open for the caller to close (e.g. `readonly`, `shareable` on
  disks; `alias` on interfaces, where the template synthesizes `ua-<name>-nic<N>` aliases
  with `.` replaced by `_`).
- `templates/network.xml.j2`, `storage_pool.xml.j2`, `storage_volume.xml.j2` — hand-rolled
  recursive walkers (2–3 levels deep) rather than the macro, plus explicit blocks for
  repeated/ordered elements (`ip`, `dns`, `dhcp`, `route`, `source`).

When adding support for a new libvirt XML element, prefer extending the generic walk;
only add an explicit block when ordering, repetition, or a valueless tag requires it.
The libvirt schema URL is cited at the top of each template.

### Idempotency

- Networks, pools, domains: `community.libvirt` `virt_net` / `virt_pool` / `virt` with the
  `define` → `active` → `autostart` triad. `define` is idempotent.
- Storage volumes (`tasks/storage_volumes.yml`): the module can't do what's needed, so this
  task gathers `virt_pool` facts and skips creation when
  `item.name in (ansible_libvirt_pools[item.pool]['volumes'] | default([]))` (the
  `default` covers a pool that does not exist yet, e.g. one defined in the same `--check`
  run), then shells out to `virsh
  vol-create` / `vol-create-from` / `vol-resize`. Volume XML is written to a tempfile that
  is deleted afterward. For existing volumes, `virsh vol-info --bytes` gives the current
  capacity; `vol-resize` runs only when the configured `capacity` (converted to bytes with
  libvirt's unit rules: `G`/`GiB` = 1024-based, `GB` = 1000-based; not `human_to_bytes`,
  which treats `GB` as 1024-based)
  is larger, so the resize is idempotent and shrinking is never attempted.

### Check mode

`--check` previews networks, pools and domains normally (those modules support it). The
storage-volume `virsh` calls are `command` tasks, which Ansible skips in a dry run, so
`tasks/storage_volumes.yml` instead prints `create:` / `grow:` lines (two `debug` tasks
gated on `ansible_check_mode`) showing what it would have done. `virsh vol-info` carries
`check_mode: false` so the size comparison still runs during a dry run.

### install.yml specifics

Installs `qemu-kvm libvirt virt-install virt-viewer`, enables the modular `virt*d`
sockets (RHEL 9 style, not monolithic `libvirtd`), and runs `virt-host-validate`. Unless
`kvm_enable_iommu` is set to false, it appends `<vendor>_iommu=on iommu=pt` to
`GRUB_CMDLINE_LINUX` (vendor read from `/proc/cpuinfo`) and triggers the `update grub`
handler (`grub2-mkconfig ... --update-bls-cmdline`). The validate task captures the
result (`failed_when: false`, `check_mode: false`) and a following
`debug` prints it and fails only on a real `FAIL` line or a return code of 2+, so a
plain `WARN` (return code 1) no longer aborts the run.

## Input schema

`meta/argument_specs.yml` (entry point `main`) validates the role's variables before any
task runs — Ansible prepends the check automatically. It constrains only the keys the task
files and templates dereference by name; the free-form nested structures the templates walk
generically (network `forward`/`dns`/`source`, domain `disks`/`interfaces`, ...) are
validated only as `dict` / `list of dict` so valid libvirt XML is never rejected. The keys
directly on a list entry are a closed set, though: an unknown key there (e.g. the removed
`resize` on a volume) stops the run with "Unsupported parameters". Validation
sees the user-supplied values, before `tasks/vms.yml` expands `kvm_vms`. Keep this file in
sync whenever you change what a template accepts.

## Examples as documentation

`examples/{networks,storage/pools,storage/volumes,domains,vms}/` hold annotated var files
that double as the reference for each input's schema. Update the relevant example (and its
`README.md`) whenever you change what a template accepts. Keep `CHANGELOG.md` current —
releases are tagged `vX.Y.Z` and the changelog is the source of version history.
