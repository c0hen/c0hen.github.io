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

#### Configuration

Generate config defaults, all commented.
```sh
ansible-config init --disabled > ansible.cfg
ansible-config init --disabled -t all > ansible_full.cfg
```
Specify and install requirements.
```sh
ansible-galaxy install -r requirements.yml # roles
ansible-galaxy collection install -r requirements.yml # collections
```
```yaml
#requirements.yml
---
roles:
  - src: https://my.scm.com/my-ansible-roles/role1.git
    scm: git
    version: master
    name: role1

collections:
# simple notation
  - community.libvirt
```

Specify requirements for roles in `role1/meta/main.yml` using the same notation as in `requirements.yml`

#### Playbooks

```sh
ansible-playbook --syntax-check kvm_provision.yml
yamllint -d relaxed kvm_provision.yml
ansible-lint kvm_provision.yml # recursive check descending to roles
ansible-playbook --ask-become-pass kvm_provision.yml --extra-vars vm=web01
ansible-playbook -K kvm_provision.yml -e vm=web01 -e net=br0
```

Ansible-playbook error `YAML parsing failed: Colons in unquoted values must be followed by a non-space character.`

is likely caused by an indentation error.

#### Roles

Create your new role:
```sh
ansible-galaxy role init kvm_provision
```

##### Include role conditionally:
###### In a task
```yaml
# roles/webapp/tasks/main.yml
---
- name: Include monitoring role conditionally
  ansible.builtin.include_role:
    name: monitoring
  when: webapp_enable_monitoring | default(true) | bool
```
###### In `meta/main.yml` using a feature flag
<!-- {% raw %} -->
```yaml
# roles/webapp/meta/main.yml
dependencies:
  - role: monitoring
    vars:
      monitoring_enabled: "{{ webapp_enable_monitoring | default(true) }}"
```
<!-- {% endraw %} -->
```yaml
# roles/monitoring/tasks/main.yml
# Skip all tasks if monitoring is disabled
---
- name: Install monitoring agent
  ansible.builtin.dnf:
    name: monitoring-agent
    state: present
  when: monitoring_enabled | bool
```

#### Documentation using [docsible](https://docsible.github.io/learn/templating/)

```sh
docsible --role roles/kvm_provision/ --playbook kvm_provision.yml --no-backup --graph
```

#### Secrets

Debug output can also include secret information despite no_log settings being enabled.
Put the encrypt_string result in a vars file like `vault.yml` containing secrets to see clearly which secrets are which. Add `vault.yml` to `.gitignore`

```sh
ansible-vault create secrets_file.enc # no secret name recorded
ansible-vault encrypt_string 'supersecret1' --name 'vault_root_pass' >> vault.yml
ansible-playbook -i inventory.ini -e @vault.yml --vault-password-file password_file kvm_provision.yml
```

Example lookup from Hashicorp Vault
<!-- {% raw %} -->
```yaml
- name: Ensure API key is present in config file
      ansible.builtin.lineinfile:
        path: /etc/app/configuration.ini
        line: "API_KEY={{ lookup('hashi_vault', 'secret=config-secrets/data/app/api-key:data token=s.FOmpGEHjzSdxGixLNi0AkdA7 url=http://localhost:8201')['key'] }}"
```
<!-- {% endraw %} -->
