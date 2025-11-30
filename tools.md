---
layout: default
title: Tools
description: Useful software and tool combos
tags: database backup postgresql tools messaging microservices
---

* Table of contents
{:toc}

### Postgresql

- [Barman](https://pgbarman.org/) - for seamless complete database restores

#### Testing HA

[Source](https://patroni.readthedocs.io/en/latest/README.html#testing-your-ha-solution)

- Network (the network in front of your system as well as the NICs [physical or virtual] themselves)
- Disk IO
- file limits (nofile in Linux)
- RAM. Even if you have oomkiller turned off, the unavailability of RAM could cause issues.
- CPU
- Virtualization Contention (overcommitting the hypervisor)
- Any cgroup limitation (likely to be related to the above)
- kill -9 of any postgres process (except postmaster!). This is a decent simulation of a segfault.

### Bitlbee

- irssi is an extensible perl scriptable IRC client for interfacing with the bitlbee instance
- libpurple to interface bitlbee with lots of messaging protocols
- [bitlbee-mastodon](https://github.com/kensanata/bitlbee-mastodon) to connect to mastodon instances via bitlbee

### Inter Process Communication, Remote Procedure Calls

[gRPC](https://grpc.io/docs/what-is-grpc/introduction/) - message interchange format and interface definition language.
[Protocol Buffers](https://protobuf.dev/) - platform-neutral extensible mechanisms for serializing structured data, default for gRPC.

### Identity Access Management

Keycloak
