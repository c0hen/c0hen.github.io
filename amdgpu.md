---
layout: default
title: Amdgpu
description: Amdgpu kernel driver, prevent overheat for older cards
tags: system hardware debug troubleshooting configuration tuning usb gpu vga frequency gaming
---

* Table of contents
{:toc}


## amdgpu hwmon interface

Read [thermal](https://docs.kernel.org/gpu/amdgpu/thermal.html) and other info provided by the amdgpu kernel driver.

```
/sys/class/drm/card0/device/hwmon/hwmon1/
```

### Enable fan speed sensors

This is a prequisite for automatic management and `fan1_target`.

```
echo 1 > /sys/class/drm/card0/device/hwmon/hwmon1/fan1_enable
```

Many of the `_enable` options disengage / override other options, check kernel logs for messages referring to such events (`journalctl -k` or `dmesg`).

Systemd service to set amdgpu /sys options.

```
#/etc/systemd/system/amdgpu-settings.service
[Unit]
Description=Set amdgpu /sys options

[Service]
Type=oneshot

ExecStart=/home/user/bin/amdgpu-settings.sh

[Install]
WantedBy=multi-user.target
```
Same after sleep.
```
#/etc/systemd/system/amdgpu-settings-after-sleep.service
[Unit]
Description=Set amdgpu /sys options after sleep
After=suspend.target hibernate.target hybrid-sleep.target suspend-then-hibernate.target

[Service]
Type=oneshot

ExecStart=/home/user/bin/amdgpu-settings.sh

[Install]
WantedBy=suspend.target hibernate.target hybrid-sleep.target suspend-then-hibernate.target
```

## Dynamic Power Management

### Get vbios and card info

```
cat /sys/class/drm/card0/device/vbios_version
```

### Feature mask to enable overdrive, prevent overheating

Older cards are by default clocked over safe limits by amdgpu.

#### Check frequencies / clocks.

```
/sys/class/drm/card0/device/pp_dpm_sclk # gpu
/sys/class/drm/card0/device/pp_dpm_mclk # memory
```

#### Enable overdrive to change frequencies.

```sh
cat /sys/module/amdgpu/parameters/ppfeaturemask
```
```
0xfff7bfff
```
Calculate the mask with the overdrive bit on.
```bash
printf '%#x' "$(( $(cat /sys/module/amdgpu/parameters/ppfeaturemask) | 0x4000 ))"
```
```
0xfff7ffff
```
Set the mask [permanently](/amdgpu/#kernel-parameters).

#### Use custom clocks

After overdrive is enabled, custom frequencies can be used.
```sh
git clone https://github.com/sibradzic/amdgpu-clocks.git
```
```sh
cd amdgpu-clocks/
sudo cp amdgpu-clocks /usr/local/bin/
sudo cp amdgpu-clocks.service /etc/systemd/system/
sudo systemctl enable amdgpu-clocks.service
sudo cp amdgpu-clocks-resume /usr/lib/systemd/system-sleep/
sudo cp ~/.config/amdgpu-custom-state.card0 /etc/default/
```
Make sure to use your card manufacturer recommended settings first to make sure everything works.
```
#/etc/default/amdgpu-custom-state.card0
OD_VDDGFX_OFFSET:
-75mV
OD_SCLK:
0: 500MHz
1: 1980MHz
FORCE_PERF_LEVEL: manual
FORCE_POWER_CAP: 270
```

## Kernel parameters

Allow recovering the device after a hang.

https://www.kernel.org/doc/html/v4.20/gpu/amdgpu.html
```
#/etc/modprobe.d/amdgpu.conf
# Enable recovery after hang. Enable overdrive, bit 14.
options amdgpu gpu_recovery=1 ppfeaturemask=0xfff7ffff
```
```sh
update-initramfs -uk all
```

Show kernel parameters of module.
```
systool -vm amdgpu
```

## Display GPU statistics using [mesa](https://docs.mesa3d.org/envvars.html)

List all names (help is one of the names).
```
GALLIUM_HUD=help glxgears
```

Use with Steam.
```
GALLIUM_HUD=shader-clock,memory-clock %command%
```

[More examples](https://manerosss.wordpress.com/2017/07/13/howto-gallium-hud/)

## USB HID device polling rate

Show polling interval. Frequency (rate) in Hz (polls in a second). Interval = frequency / 1000 (for milliseconds, one thousandth of a second). To set 1000 Hz, the interval is 1000 / 1000 = 1.

```sh
systool -m usbhid -A jspoll
```

Get info about device using using [lsusb](/hardware-troubleshooting/#lsusb).

Set [polling interval](https://wiki.archlinux.org/title/Mouse_polling_rate) for usbhid kernel driver. This uses `kbpoll` (keyboards), `mousepoll`, `jspoll` (game pads). If your device is not a Human Interface Device, other solutions may be necessary, like recompiling your driver.
```
#/etc/modprobe.d/usbhid.conf
options usbhid kbpoll=1 mousepoll=1
```
