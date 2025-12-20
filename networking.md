---
layout: default
title: Networking
description: Networking cheatsheet
tags: iptables network tips system
---

### Commands to get a quick networking overview

```sh
route -n # lower metric route is used
ip route show
ip --json tcpmetrics s | jq '.[]|select(.dst == "127.0.0.1")'
ip --json stats show group link dev enp4s0 | jq '.'
ip address s
ip tunnel s
ss -tupan # connections
ss -st # --statistics --tcp
ip neighbour s
arp
ethtool enp4s0
ethtool -S enp4s0
tcpdump -D # --list-interfaces
iptables-save
```

### Fully flush iptables


```sh
#/bin/sh
IPT=/usr/bin/iptables
$IPT -F
$IPT -X
$IPT -t security -F
$IPT -t security -Z
$IPT -t security -X
$IPT -t raw -F
$IPT -t raw -Z
$IPT -t raw -X
$IPT -t nat -F
$IPT -t nat -Z
$IPT -t nat -X
$IPT -t mangle -F
$IPT -t mangle -Z
$IPT -t mangle -X
$IPT -P INPUT ACCEPT
$IPT -P FORWARD ACCEPT
$IPT -P OUTPUT ACCEPT
```

### Other network tools

```sh
netcat
telnet 25 127.0.0.1 # Ctrl + ] to exit back to telnet
ping -s 1400 # some connections need debugging with large packets
traceroute
ifdata -e enp4s0 # allows convenient clear queries, easy to machine parse
dig @127.0.0.1 kernel.org # bind9-dnsutils
```
