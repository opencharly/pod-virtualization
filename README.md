# pod-virtualization

The `virtualization` candy of the OpenCharly candy library, as a standalone repo
(the candy de-submodule cutover, kind-prefixed naming). It provides the
QEMU/KVM/libvirt virtualization stack for both supervisord and systemd init
systems.

## What it provides

Installs the QEMU/KVM emulators, `qemu-img`, and the libvirt daemons. The daemon
set is **distro-divergent**: Fedora and Arch ship the split modular daemons
(`virtqemud` + `virtnetworkd`), while Debian and Ubuntu build libvirt without the
split and ship only the monolithic `libvirtd` (which serves both the qemu and
network drivers).

| Property | Value |
|---|---|
| Requires | `layer-supervisord` |
| Services (Fedora/Arch) | `virtqemud` (priority 5), `virtnetworkd` (priority 6) |
| Services (Debian/Ubuntu) | `libvirtd` (priority 5) — monolithic; serves both drivers |
| Devices | `/dev/kvm` (declared by the consumer box or `container-nesting`) |

Each daemon ships as **mixed `service:` entries** — a `use_packaged: <unit>.socket`
form rendered on systemd targets and a custom `exec:` form rendered on
supervisord targets — selected by a per-entry `distro:` filter. One candy serves
container/pod deploys AND host/bootc/VM deploys with no `<name>-host` sibling.

In session mode (`qemu:///session`), the daemons and clients work at uid 1000
with only `/dev/kvm` passthrough — no `CAP_SYS_ADMIN`, no root escalation.

## How to use it

Typically pulled in via the `charly` toolchain candy:

```yaml
charly-arch:
  candy:
    - ...
    - charly                # the full toolchain — pulls virtualization
    - container-nesting     # donates /dev/fuse + /dev/net/tun (VMs need /dev/kvm)
```

## Verification

The candy's `check:` plan asserts `/usr/bin/virsh`, `/usr/bin/qemu-system-x86_64`,
`/usr/bin/qemu-img`, and the libvirt qemu driver package (via `package_map:`), and
— at deploy scope — `virsh -c qemu:///session list --all` exiting 0.

## Layout

- `charly.yml` — the `virtualization:` candy entity (description, `distro`,
  `service`, `plan`) plus its `skill:` entity.
- `.github/workflows/tag-on-merge.yml` — CalVer tag + `CHANGELOG/` on merge.
- `CHANGELOG/` — per-CalVer history.
- `README.md` — this user overview.

## Related

- Owning skill: `/charly-infrastructure:virtualization` — the canonical
  mixed-`service:` polymorphism worked example and the rootless
  `qemu:///session` model.
- `/charly-tools:charly` — the full toolchain that pulls this candy into
  charly-toolchain boxes.
- `/charly-distros:container-nesting` — pairs with this candy for boxes needing
  both nested containers and nested VMs.
- [`opencharly/charly`](https://github.com/opencharly/charly) — the charly CLI and image builder
