# virt-v2v for EL9 with upstream fixes

## In short

These are the stock EL9 `virt-v2v` packages with fixes from upstream virt-v2v and libguestfs-common added, plus two
small fixes of our own for Windows guests. They are drop-in replacements for the stock package on AlmaLinux 9 and
compatible systems. You get the fixes now, without waiting for a newer virt-v2v in EL9.

They are built and used by [V2K Migrate](https://v2kmigrate.com) for VMware-to-KVM migrations to CloudStack and
Proxmox, and shared here for anyone who runs virt-v2v on EL9.

| | |
|---|---|
| Current version | `virt-v2v-2.7.1-19.el9_7.v2k5` |
| Based on | AlmaLinux `virt-v2v-2.7.1-19.el9_7` |
| Download | [Releases](../../releases) (signed RPMs, source RPM, checksums, public key) |

## Why it matters

The stock EL9 virt-v2v (2.7.1) has bugs, most of them fixed upstream since. Without these fixes:

- a migrated **Windows Server 2025** guest can reboot forever on its first boot, because one virtio driver that always
  fails to install blocks every later step (guest agent, network setup, VMware Tools removal);
- on 64-bit Windows, the first boot does not notice that Windows is **still waiting for a restart after updates**, so
  drivers can be installed while Windows is still busy, and a driver that fails then is not retried (for example, no
  network without `netkvm`);
- on Windows Server 2025 with a virtio boot disk, Windows restarts itself in the first boot to swap in the real disk
  driver, and virtio drivers installed at that moment fail ("The Plug and Play service is not available") and stay
  missing;
- the **QEMU guest agent** install fails every time on a guest that still has a leftover "QEMU Guest Agent VSS
  Provider" service from an earlier agent;
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
| [libguestfs/virt-v2v#211](https://github.com/libguestfs/virt-v2v/pull/211) (issue [#210](https://github.com/libguestfs/virt-v2v/issues/210)) | Windows first boot: the check for a pending restart (Windows Update, component servicing) reads the 64-bit registry, so on 64-bit Windows the drivers are installed only after that restart. New in `.v2k4`. | Yes |

## Our own fixes (not upstream)

| Fix | What it fixes | Since |
|---|---|---|
| Leftover guest agent service | Before installing the QEMU guest agent, the first boot removes a "QEMU Guest Agent VSS Provider" service that has no COM+ application left (left behind by an earlier agent). Otherwise the agent installer fails with 0x80070431 every time and the guest has no agent. Same rule as QEMU commit [`55dc0aecb86b`](https://gitlab.com/qemu-project/qemu/-/commit/55dc0aecb86b7327c3b6c183001699e98ae9b553), which fixes this inside qemu-ga 110.2.4; virtio-win releases still ship 110.2.3. The installer's exit code is now logged too. | `.v2k5` |
| Drivers that hit a busy Plug and Play service | A virtio driver that fails with 0xE0000223 ("The Plug and Play service is not available") is tried once more after the next reboot. This happens on Windows Server 2025 with a virtio boot disk, when Windows restarts itself in the first boot to swap in the real disk driver. Other driver failures are reported as before. At most one extra reboot. | `.v2k5` |

Both were tested in real migrations to Proxmox VE and CloudStack (Windows Server 2016, 2019, 2022 and 2025).

The fixes are rebased onto the EL9 `rhel-9.8` branch of virt-v2v. Some needed small rewrites to fit the older code;
the behaviour matches upstream. Each patch's commit message names its upstream commit and pull request, and
[patches/README.md](patches/README.md) maps each patch file to its upstream fix.

## Install

On AlmaLinux 9 with the normal BaseOS and AppStream repositories (no EPEL or CRB needed). Download the files of the
release you want from [Releases](../../releases), then:

```
sudo rpm --import RPM-GPG-KEY-v2kmigrate
sha256sum -c SHA256SUMS --ignore-missing
rpm -K virt-v2v-2.7.1-19.el9_7.v2k5.x86_64.rpm      # "digests signatures OK"
sudo dnf install ./virt-v2v-2.7.1-19.el9_7.v2k5.x86_64.rpm
rpm -q virt-v2v                                     # virt-v2v-2.7.1-19.el9_7.v2k5.x86_64
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
(`virt-v2v.spec`, `patches/`). To rebuild: `dnf builddep virt-v2v-2.7.1-19.el9_7.v2k5.src.rpm`, then
`rpmbuild --rebuild virt-v2v-2.7.1-19.el9_7.v2k5.src.rpm`.

## Versions

Every version is a git tag (the package version, for example `v2.7.1-19.el9_7.v2k5`) and a GitHub Release. Older
releases stay downloadable. See [CHANGELOG.md](CHANGELOG.md).

## Not covered / known limits

- Community builds of upstream virt-v2v code. Not affiliated with, or supported by, Red Hat or AlmaLinux. No warranty
  (see the licence).
- Built and tested for x86_64 on AlmaLinux 9 only.
- If a Windows restart stays pending through 3 boots (it never clears), the driver step gives up so that the first
  boot can finish, and the drivers are not installed. Tip: restart Windows VMs after patching, before migrating them.
- The Windows guest agent comes from the `virtio-win` package, not from virt-v2v. qemu-ga 110.2.3 (in `virtio-win`
  1.9.61) also has a VSS backup bug fixed in 110.2.4; that is outside virt-v2v.
- VMware Tools removal can fail (exit 1603) on some Windows guests after the move off VMware; the first boot logs it
  and carries on, as the stock package does.

## Reporting problems

Open an issue in this repository for problems with these packages. A bug that is also in upstream virt-v2v belongs in
[libguestfs/virt-v2v](https://github.com/libguestfs/virt-v2v/issues).

## Licence

virt-v2v is free software under GPL-2.0-or-later and LGPL-2.0-or-later (see `COPYING`). The patches in this repository
are under the same licence.
