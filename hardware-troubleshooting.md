---
layout: default
title: Hardware
description: Tips on hardware troubleshooting.
tags: system hardware debug troubleshooting configuration tuning
---

## Linux kernel

### List Linux module parameters

From `sysfs-tools`.
```sh
systool -vm amdgpu
```

### Configure Linux module parameters

1. `/etc/modprobe.d/amdgpu.conf`
1. Append to boot loader configuration line that boots the kernel.
  - `ESP/loader/entries/*.conf` for systemd-boot, `man loader.conf`
  - `/etc/default/grub` for grub, `less /etc/grub.d/README`, `info grub-mkconfig`

## Firmware and UEFI related

### SSD firmware update using systemd-boot

The firmware updater of an SSD may essentially boot Linux, as is the case with Crucial. Mount the firmware updater iso file offered by Crucial. Copy relevant files from mounted image to `/boot/` (or what your systemd-boot file structure needs).
```sh
cp --no-clobber -r mnt/EFI/BOOT/* /boot/efi/EFI/BOOT/
```
Sample firmware update entry for use with systemd-boot.
```
# /boot/efi/loader/entries/ssd_update.conf
title Crucial Firmware Update
linux /boot/vmlinuz64
initrd /boot/corepure64.gz
options libata.allow_tpm=1 quiet base loglevel=3 cde waitusb=10 consoleblank=0 superuser mse-iso rssd-fw-update rssd-fwdir=/opt/firmware
```
This entry should be modeled after `iso_mount_point/boot/isolinux/isolinux.cfg`.

Similar method can be used with grub and other common boot loaders.

### SSD identifies as disk by another manufacturer

In the case of Crucial, their disk controllers are by Silicon Motion (SM in device model). The SSD may identify as that, with no sign of the correct firmware or any partitions.

If luck is with you, this may be caused by a power spike and may fix itself when the disk does not have power for a while (an hour?). Device may not respond to SMART commands. This behaviour should fix itself as well if the model info and partitions reappear after a rest.

```
smartctl -x /dev/sdb

=== START OF INFORMATION SECTION ===
Device Model:     SMXXXXXX-10-00831000
Serial Number:    (XX)XXXXXXX-XXXXXXXX
Firmware Version: 20141211
User Capacity:    1 073 479 680 bytes [1,07 GB]
Sector Size:      512 bytes logical/physical
Device is:        Not in smartctl database 7.3/5528
ATA Version is:   ACS-2 (minor revision not indicated)
Local Time is:    Sun Jun 01 23:39:18 2026 UTC
SMART support is: Unavailable - device lacks SMART capability.
AAM feature is:   Unavailable
APM feature is:   Unavailable
Rd look-ahead is: Unavailable
Write cache is:   Unavailable
DSN feature is:   Unavailable
ATA Security is:  Unavailable
Wt Cache Reorder: Unavailable

A mandatory SMART command failed: exiting.
```

Correct info for comparison.

```
=== START OF INFORMATION SECTION ===
Model Family:     Crucial/Micron Client SSDs
Device Model:     XXXXXXXXXXXXXXX
Serial Number:    XXXXXXXXXXXX
LU WWN Device Id: X XXXXXX XXXXXXXXX
Firmware Version: XXXXXXX
User Capacity:    1 000 204 886 016 bytes [1,00 TB]
Sector Sizes:     512 bytes logical, 4096 bytes physical
Rotation Rate:    Solid State Device
Form Factor:      M.2
TRIM Command:     Available
Device is:        In smartctl database 7.3/5528
ATA Version is:   ACS-3 T13/2161-D revision 5
SATA Version is:  SATA 3.3, 6.0 Gb/s (current: 6.0 Gb/s)
Local Time is:    Sun Jun  1 20:58:27 2026 UTC
SMART support is: Available - device has SMART capability.
SMART support is: Enabled
AAM feature is:   Unavailable
APM level is:     254 (maximum performance)
Rd look-ahead is: Enabled
Write cache is:   Enabled
DSN feature is:   Unavailable
ATA Security is:  Disabled, NOT FROZEN [SEC1]
Wt Cache Reorder: Unknown

=== START OF READ SMART DATA SECTION ===
...
```
