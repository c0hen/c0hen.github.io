---
layout: default
title: Udev rules network device names
description: Create custom udev rules
tags: udev network debian deprecated
---
### Debian 9 network name example is deprecated, here to show udev rule creation

Create custom, static, machine independent network device names to simplify firewalling and scripting.

Udev network device names by default change to BIOS/UEFI location based names in Debian Stretch (9).

```sh
/usr/share/doc/udev/README.Debian.gz
```

Read Debian's documents and if you depend on device names staying the same (firewall, scripts etc.), you can follow the advice to set up /etc/udev/rules.d/76-netnames.rules and then (re)move empty /etc/systemd/network/99-default.link if you have it (in case you upgraded).

```sh
#/etc/udev/rules.d/76-netnames.rules
#   The name of the rules file needs to have a prefix smaller than "80" so that
#   it runs before /lib/udev/rules.d/80-net-setup-link.rules, and should have a
#   prefix bigger than "75" so that it runs after 75-net-description.rules and
#   thus you can use matches on ID_VENDOR and similar properties.
## identify by vendor/model ID
#SUBSYSTEM=="net", ACTION=="add", ENV{ID_VENDOR_ID}=="0x8086", \
#    ENV{ID_MODEL_ID}=="0x1502", NAME="eth-intel-gb"
#
## USB device by path
## get ID_PATH if not present yet
#ENV{ID_PATH}=="", IMPORT{builtin}="path_id"
#SUBSYSTEM=="net", ACTION=="add", ENV{ID_PATH}=="*-usb-0:3:1*", NAME="eth-blue-hub"
# identify device by MAC address
SUBSYSTEM=="net", ACTION=="add", ATTR{address}=="ea:e9:8a:aa:3f:ab", NAME="lan"
SUBSYSTEM=="net", ACTION=="add", ATTR{address}=="aa:a2:b3:27:dd:af", NAME="wan"
```

To make the changes take effect:

```sh
sudo update-initramfs -u
```
