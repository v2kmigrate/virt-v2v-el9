# virt-v2v for EL9 with upstream fixes

## In short

These are the stock EL9 `virt-v2v` packages with fixes from upstream virt-v2v and libguestfs-common added. They are
drop-in replacements for the stock package on AlmaLinux 9 and compatible systems. You get the fixes now, without
waiting for a newer virt-v2v in EL9.

They are built and used by [V2K Migrate](https://v2kmigrate.com) for VMware-to-KVM migrations to CloudStack and
Proxmox, and shared here for anyone who runs virt-v2v on EL9.

| | |
|---|---|
| Current version | `virt-v2v-2.7.1-19.el9_7.v2k3` |
| Based on | AlmaLinux `virt-v2v-2.7.1-19.el9_7` |
| Download | [Releases](../../releases) (signed RPMs, source RPM, checksums, public key) |

## Why it matters

The stock EL9 virt-v2v (2.7.1) has bugs that upstream has fixed since. Without these fixes:

- a migrated **Windows Server 2025** guest can reboot forever on its first boot, because one virtio driver that always
  fails to install blocks every later step (guest agent, network setup, VMware Tools removal);
- migrated **Debian 11, 12 and 13** guests with UEFI boot can fail to boot;
- there is no way to switch off the **fstrim** step, which can take a long time on large disks or slow storage.

## Upstream fixes included

| Upstream fix | What it fixes | Merged upstream |
|---|---|---|
| virt-v2v commit [`25163945`](https://github.com/libguestfs/virt-v2v/commit/25163945) (first proposed as [PR #121](https://github.com/libguestfs/virt-v2v/pull/121)) | Adds `--no-fstrim` to `virt-v2v`, `virt-v2v-in-place` and `virt-v2v-inspector`, to skip trimming free space at the end of the conversion. | Yes, as that commit (virt-v2v 2.12.0); the PR itself was closed |
| [libguestfs/virt-v2v#126](https://github.com/libguestfs/virt-v2v/pull/126) | Debian 12 guests with UEFI boot: adds the rule that fixes the UEFI bootloader after conversion. | Yes |
| [libguestfs/virt-v2v#195](https://github.com/libguestfs/virt-v2v/pull/195) | Same, for Debian 11. | Yes |
| [libguestfs/virt-v2v#201](https://github.com/libguestfs/virt-v2v/pull/201) | Same, for Debian 13. | Yes |
| [libguestfs/virt-v2v#207](https://github.com/libguestfs/virt-v2v/pull/207) (issue [#206](https://github.com/libguestfs/virt-v2v/issues/206)) | Debian UEFI first-boot script: keeps the `\EFI\BOOT` fallback loader unless `efibootmgr` succeeded, works without `sudo`, and finds the EFI partition itself, so the guest still boots after its first boot. Needed together with the three Debian rules above. This package carries an equivalent version of the fix, written before #207 was merged. | Yes |
| [libguestfs/virt-v2v#202](https://github.com/libguestfs/virt-v2v/pull/202) (issue [#178](https://github.com/libguestfs/virt-v2v/issues/178)) | Windows first boot: the virtio driver install no longer stops on one failing driver. It installs every driver, lists the ones that failed, and the first boot carries on. This stops the endless reboot loop seen with Windows Server 2025 and `smbus.inf`. | Yes |
| [libguestfs/libguestfs-common#54](https://github.com/libguestfs/libguestfs-common/pull/54) | Windows first boot: a step that keeps asking for a retry is tried at most 3 times, then skipped, so later steps still run. | Yes |

The fixes are rebased onto the EL9 `rhel-9.8` branch of virt-v2v. Some needed small rewrites to fit the older code;
the behaviour matches upstream. Each patch's commit message names its upstream commit and pull request, and
[patches/README.md](patches/README.md) maps each patch file to its upstream fix.

## Install

On AlmaLinux 9 with the normal BaseOS and AppStream repositories (no EPEL or CRB needed). Download the files of the
release you want from [Releases](../../releases), then:

```
sudo rpm --import RPM-GPG-KEY-v2kmigrate
sha256sum -c SHA256SUMS --ignore-missing
rpm -K virt-v2v-2.7.1-19.el9_7.v2k3.x86_64.rpm      # "digests signatures OK"
sudo dnf install ./virt-v2v-2.7.1-19.el9_7.v2k3.x86_64.rpm
rpm -q virt-v2v                                     # virt-v2v-2.7.1-19.el9_7.v2k3.x86_64
virt-v2v --help | grep no-fstrim
```

It installs as an upgrade over the stock `virt-v2v-2.7.1-19.el9_7`. To go back: `sudo dnf downgrade virt-v2v`.
Optional: `virt-v2v-bash-completion` (Tab completion of virt-v2v options in bash).

## Verify

The packages and `SHA256SUMS` are signed with the V2K Migrate package key, the same key as on packages.v2kmigrate.com:

- `V2K Migrate Packages <packages@v2kmigrate.com>`, RSA 4096
- fingerprint `7DD7 2D9D B1D8 3F05 0853  D6C8 B1A0 4AAE 39CA ECD0`
- in this repository and in every release as `RPM-GPG-KEY-v2kmigrate`; also at
  https://packages.v2kmigrate.com/cloudstack/public-key.asc

To check the checksum file itself: `gpg --import RPM-GPG-KEY-v2kmigrate`, then `gpg --verify SHA256SUMS.asc SHA256SUMS`.

## Source

The source RPM is in every release. It is AlmaLinux's source package with the patches in this repository added
(`virt-v2v.spec`, `patches/`). To rebuild: `dnf builddep virt-v2v-2.7.1-19.el9_7.v2k3.src.rpm`, then
`rpmbuild --rebuild virt-v2v-2.7.1-19.el9_7.v2k3.src.rpm`.

## Versions

Every version is a git tag (the package version, for example `v2.7.1-19.el9_7.v2k3`) and a GitHub Release. Older
releases stay downloadable. See [CHANGELOG.md](CHANGELOG.md).

## Not covered / known limits

- Community builds of upstream virt-v2v code. Not affiliated with, or supported by, Red Hat or AlmaLinux. No warranty
  (see the licence).
- Built and tested for x86_64 on AlmaLinux 9 only.
- Not in this version: [libguestfs/virt-v2v#211](https://github.com/libguestfs/virt-v2v/pull/211) (issue
  [#210](https://github.com/libguestfs/virt-v2v/issues/210)). Before installing drivers, the Windows first boot is
  meant to wait if Windows still has a restart pending after updates. On 64-bit Windows it reads the wrong part of the
  registry, so it does not notice. The stock package has the same problem. Tip: restart Windows VMs after patching,
  before migrating them.
- The Windows guest agent comes from the `virtio-win` package, not from virt-v2v. With `virtio-win` 1.9.61 (qemu-ga
  110.2.3), the agent install fails on a guest that still has a leftover "QEMU Guest Agent VSS Provider" service from
  an earlier agent. Upstream fixed this in qemu-ga 110.2.4 (QEMU commit `55dc0aecb86b`). Workaround: if the source VM
  has that service but no QEMU guest agent installed, delete the service before migrating
  (`sc delete "QEMU Guest Agent VSS Provider"`).

## Reporting problems

Open an issue in this repository for problems with these packages. A bug that is also in upstream virt-v2v belongs in
[libguestfs/virt-v2v](https://github.com/libguestfs/virt-v2v/issues).

## Licence

virt-v2v is free software under GPL-2.0-or-later and LGPL-2.0-or-later (see `COPYING`). The patches in this repository
are under the same licence.
