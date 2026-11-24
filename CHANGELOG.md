# Changelog

Newest first. Each version is a git tag and a GitHub Release with signed packages.

## v2.7.1-19.el9_7.v2k3 (2026-10-29)

Based on AlmaLinux `virt-v2v-2.7.1-19.el9_7`.

- Windows first boot: the virtio driver install no longer stops on one failing driver; it installs every driver,
  lists the failed ones and carries on (upstream virt-v2v #202, issue #178).
- Windows first boot: a step that keeps asking for a retry is tried at most 3 times, then skipped
  (upstream libguestfs-common #54).
- Debian 11, 12 and 13 with UEFI boot: bootloader fix rules (upstream virt-v2v #195, #126, #201), and the first-boot
  script keeps the UEFI fallback loader unless `efibootmgr` succeeded (same fix as upstream virt-v2v #207).
- New option `--no-fstrim` for `virt-v2v`, `virt-v2v-in-place` and `virt-v2v-inspector` (upstream commit `25163945`).
