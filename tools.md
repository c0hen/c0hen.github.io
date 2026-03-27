---
layout: default
title: Tools
description: Useful software and tool combos
tags: database backup postgresql tools messaging microservices terraform docker
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

### Network security

Tcpdump, Wireshark

Intrusion Detection, Prevention System (IDS, IPS)

[Suricata](https://docs.suricata.io)
[Snort](https://docs.snort.org)

[SysAdmin, Audit, Network, Security tools list](https://www.sans.org/tools)

### [Docker](https://hub.docker.com)

[Docker CLI reference](https://docs.docker.com/reference/cli/docker/container/exec/)
```sh
docker run --help
docker run hello-world
docker pull rockylinux/rockylinux:10-ubi-micro
docker images
docker history rockylinux/rockylinux:10-ubi-micro
docker inspect minikube
docker exec --tty minikube sh -c 'uname -a'
```

### [Terraform](https://registry.terraform.io/browse/providers)

```sh
terraform -install-autocomplete
```

Alias to run terraform in a docker container.
```sh
alias terraformz='docker run --rm -it -w $PWD -v $PWD:$PWD hashicorp/terraform:latest'
```
The `providers` helps to troubleshoot bind mounts.
```sh
docker run --rm -it -w $PWD -v $PWD:$PWD hashicorp/terraform:latest providers
```

Define infra, initialize, validate.
```sh
terraform init
terraform fmt
terraform validate
terraform plan
```

Create infrastructure, inspect.
```sh
terraform apply
terraform show
terraform state list
```

### [Ansible](https://galaxy.ansible.com/ui/collections/)

#### Playbooks

```sh
ansible-playbook --syntax-check kvm_provision.yaml
ansible-lint kvm_provision.yaml # recursive check descending to roles
ansible-playbook --ask-become-pass kvm_provision.yaml --extra-vars vm=web01
ansible-playbook -K kvm_provision.yaml -e vm=web01
```

Error `YAML parsing failed: Colons in unquoted values must be followed by a non-space character.`

is likely caused by an indentation error.
