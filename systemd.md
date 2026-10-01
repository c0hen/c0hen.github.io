---
layout: default
title: Systemd
description: Systemd services and other components
tags: systemd configuration service timer init
---

* Table of contents
{:toc}

## Systemd

### Status and messages

```sh
systemctl --user --failed status
```
Show kernel / dmesg messages with priority error or higher from last boot in json format, grep for AMD.
```sh
journalctl --priority=err --dmesg --output=json --boot=-1 --grep=AMD
journalctl -perr -k -o json -b-1 -g AMD
```
Show user logs since midnight, jump to end of messages.
```sh
journalctl --user --pager-end --since today
```
Show only warnings (level 4) for a unit.
```sh
journalctl -p4..4 -u nginx.service
```
Enable unit start on boot and restart changed unit.
```sh
systemctl --user enable mpd.service
systemctl --user daemon-reload
systemctl --user restart mpd.service
```

### Inspection, formats

```sh
systemctl --version
systemctl whoami
systemd-cat mpd.service
systemctl show mpd
systemd-analyze unit-files
```
Systemd timer, time specifications.
```
systemd-analyze calendar '*-*-* *:*:00'
```
Control group info.
```sh
systemd-cgls
systemd-cgtop
```

### Help

In addition to command named manual pages, there are many `systemd*` named pages.
```sh
man systemd-sleep
man systemd.directives
```
```sh
systemctl --help
```

### Configuration and files
```
/etc/systemd/
# System wide units
/usr/lib/systemd/system/ # upstream from distro
/etc/systemd/system/ # user defined
# Per user units
$XDG_CONFIG_HOME/systemd/user/
```

### Units

```
man systemd.unit
```
- `.service`
- `.timer`
- `.target`
- `.path`
- `.mount`
- `.automount`
- `.socket`
- `.swap`
- `.slice`
- `.scope`

#### Services

Services can be instantiated.
```sh
systemctl status nut-driver@mustek.service
```
Directives in files have valid contexts.

##### Limit memory use in a unit

```
MemoryMax=bytes
```
```sh
man systemd.resource-control
```
