# Patches

These are the patches added to AlmaLinux's `virt-v2v-2.7.1-19.el9_7` source package, in the order the spec file
applies them. They are `git format-patch` files, exactly as in the source RPM. Each backport's commit message names its
upstream commit and pull request; the two downstream patches say so in theirs.

| File | Upstream fix |
|---|---|
| `9001-Add-no-fstrim-option-to-disable-fstrim-during-conver.patch` | virt-v2v commit [`25163945`](https://github.com/libguestfs/virt-v2v/commit/25163945) (first proposed as [#121](https://github.com/libguestfs/virt-v2v/pull/121)) |
| `9002-convert-convert_linux.ml-Add-debian-12-UEFI.patch` | [libguestfs/virt-v2v#126](https://github.com/libguestfs/virt-v2v/pull/126) |
| `9003-Add-UEFI-bootloader-fix-rule-for-Debian-11.patch` | [libguestfs/virt-v2v#195](https://github.com/libguestfs/virt-v2v/pull/195) |
| `9004-Add-UEFI-bootloader-fix-rule-for-Debian-13.patch` | [libguestfs/virt-v2v#201](https://github.com/libguestfs/virt-v2v/pull/201) |
| `9005-convert-convert_linux.ml-Keep-the-UEFI-fallback-unle.patch` | equivalent of [libguestfs/virt-v2v#207](https://github.com/libguestfs/virt-v2v/pull/207), written before it was merged |
| `9006-convert_windows-do-not-abort-the-pnputil-firstboot-s.patch` | [libguestfs/virt-v2v#202](https://github.com/libguestfs/virt-v2v/pull/202), rewritten for the rhel-9.8 script |
| `9007-mlcustomize-firstboot-bound-the-retries-of-a-Windows.patch` | [libguestfs/libguestfs-common#54](https://github.com/libguestfs/libguestfs-common/pull/54), rewritten for the rhel-9.8 common submodule |
| `9008-convert_windows-check-for-a-pending-reboot-in-the-64.patch` | [libguestfs/virt-v2v#211](https://github.com/libguestfs/virt-v2v/pull/211), applied by hand to the rhel-9.8 script (from v2k4) |
| `9009-mlcustomize-inject_virtio_win-remove-a-leftover-VSS-.patch` | **downstream**: removes a leftover "QEMU Guest Agent VSS Provider" service without its COM+ application before the agent install; same rule as QEMU commit `55dc0aecb86b` (from v2k5) |
| `9010-convert_windows-retry-drivers-once-when-Plug-and-Pla.patch` | **downstream**: retries drivers once that failed because Plug and Play was busy (0xE0000223) (from v2k5) |
