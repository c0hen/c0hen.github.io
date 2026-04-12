---
layout: default
title: Tools
description: Useful software and tool combos
tags: database backup postgresql tools messaging microservices docker
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

### Containers and [Docker](https://hub.docker.com)

OCI (Open Container Initiative) [low level, runtime specification](https://github.com/opencontainers/runtime-spec) and [high level, image specification](https://github.com/opencontainers/image-spec) have standardized what was initially a mess.

Containerd, Docker, [CRI-O](https://github.com/cri-o/cri-o), podman are high level since they implement at least parts of the image specification. Runc, Crun, gVisor, Firecracker implement the low level runtime specification that runs processes in the container.

Containerd architecture schema.
```
[Docker CLI] [   Docker API   ] [    Build     ]
                                                 Integrated Lifecycle
[  Compose ] [Content Trust     [Authentication] Management
              and verification]

[ Security ] [    Network     ] [   Volumes    ] Docker

[ Containerd                  [RUNC]           ] Container runtime

------------------------------------------------ OCI

[       Linux         ][        Windows        ] OS

[       Hardware      ][        Cloud          ] Infrastructure
```
[Runc](https://github.com/opencontainers/runc) is written in Go. [Crun](https://github.com/containers/crun) is an alternative that needs fewer resources and is favored by Red Hat, written in C.

Firecracker by AWS (Rust) has hardware enforced isolation via KVM. Google's gVisor has better Kubernetes and Docker integration, less overhead but less strict isolation (sandbox).

containerd offers a fully namespaced API so multiple consumers can all use a single containerd instance without conflicting with one another in a single daemon.
To inspect container
```sh
ctr namespaces
ctr leases --help
ctr -n docker tasks
```
#### Command examples

Create a default OCI configuration.
```sh
runc spec && less config.json
```
Configure runc (enter namespace)
```sh
nsenter --help
```

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
Run `uv` [using docker](https://docs.astral.sh/uv/guides/integration/docker/)
```sh
docker run --rm -it ghcr.io/astral-sh/uv:debian uv --help
```

#### Docker CLI comparable tools

Also compatible with `docker-compose`

`alias docker=podman`
```sh
podman
```
[nerdctl](https://github.com/containerd/nerdctl.git), CLI for [containerd](https://containerd.io/docs/main/getting-started/).

#### Docker helper tools

Turn docker-compose files into flowcharts with [docker-compose-viz-mermaid](https://github.com/derlin/docker-compose-viz-mermaid)
```sh
docker run --rm -v $(PWD):/data derlin/docker-compose-viz-mermaid /data/docker-compose.yml -f png
```

#### Vulnerabilities, container escape

TeamsSix container [escape check script](https://github.com/teamssix/container-escape-check/blob/main/container-escape-check.sh).
[Deepce](https://github.com/stealthcopter/deepce/blob/main/deepce.sh) Docker Enumeration, Escalation of Privileges and Container Escapes

[Genuine vulnerabilities](https://github.com/advisories/GHSA-cgrx-mc8f-2prm) exist but misconfiguration is a more likely way to escape a container.

First, figure out if you're in a container.
```sh
ls -la / | grep dockerenv
cat /proc/1/cgroup
env | grep -i kube
env | grep -i docker
```

##### `--privileged` container detection in the container

Non-privileged containers can't create network interfaces.
```sh
ip link add dummy0 type dummy
```
List disks, mount the host root file system and other nested ones as needed, chroot into the host.
```sh
fdisk -l

mount /dev/sda4 /mnt/hostroot
... snip ...

chroot /mnt/hostroot bash

id
```
```
uid=0(root) gid=0(root) groups=0(root)
```
##### Inspecting capabilities
```sh
capsh --print
cat /proc/self/status | grep Cap
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

#### Miscellaneous uses

```sh
ansible-galaxy collection list
ansible-galaxy list
ansible-galaxy search
ansible-pull --only-if-changed --verify-commit site.yml
ansible-inventory -i inventory/ --list
# count changes
ansible-playbook site.yml | grep -oE "changed=*[0-9]" | cut -d '=' -f 2
# get guest vm status using community.libvirt.virt module
ansible localhost -m virt -a "name=vm_name command=status"
# quick fact overview
ansible --inventory inventory/ srv-web -m ansible.builtin.setup
```
Loops have the default loop_var `item` but that can be renamed in case of a conflict. `loop_control` has [other uses](https://docs.ansible.com/projects/ansible/latest/playbook_guide/playbooks_loops.html#adding-controls-to-loops) and does not affect `until`.
<!-- {% raw %} -->
```yaml
community.digitalocean.digital_ocean:
  name: "{{ server }}"
  state: present
loop: "{{ servers }}"
loop_control:
  loop_var: server
  pause: 3
```
Lookup current `loop_var`:
```yaml
"{{ lookup('vars', ansible_loop_var) }}"
```
Inspect:
```yaml
{{ servers | type_debug }}
```
<!-- {% endraw %} -->

##### Test jinja2 template
```sh
ansible-playbook ansible_test_jinja2_template.yml --diff \
--extra-vars="@kvmlab/roles/web_server/defaults/main.yml"
```
```yaml
# playbook to test jinja2 template
---
- hosts: 127.0.0.1
  tasks:
  - name: Test jinja2template
    template:
      src: "kvmlab/roles/web_server/templates/nginx.conf.j2"
      dest: "test.conf"
```

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

Playbooks contain plays as top level elements, for using `roles`, `tasks` keywords.
```yaml
# site.yml
---
- name: Prepare KVM host
  hosts: kvm # ansible group name from inventory
  gather_facts: yes
  become: yes
  roles:
    - kvm_host
  post_tasks:
    - name: Print ansible_hostname
      ansible.builtin.debug:
        var: ansible_hostname
```
Ansible loads inventory sources in the order you supply them. It defines hosts, groups, and variables as it encounters them in the source files, adding the all and ungrouped groups at the end if needed.
```yaml
# inventory/test.yml
# Keep the inventory ordering!
kvm:
  hosts: localhost
```
```sh
ansible-playbook -K -i inventory/ site.yml
```

#### Modules

List of modules that only run when source changed:

- `ansible.builtin.template`
- `ansible.builtin.copy # copy acts like rsync regarding /`

List of modules that ensure state only when parameter `state` is used:
- `ansible.builtin.dnf`

#### Roles

Create your new role:
```sh
ansible-galaxy role init kvm_provision
```
[Variable precedence](https://docs.ansible.com/projects/ansible/latest/playbook_guide/playbooks_variables.html#understanding-variable-precedence)

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

#### Adding [tags](https://docs.ansible.com/projects/ansible/latest/playbook_guide/playbooks_tags.html)

Special reserved tags are `always`, `never`, `tagged`, `untagged` and `all`. Both `always` and `never` are used for tagging, others for selecting which tags to run or skip.
```yaml
ansible-playbook example.yml --list-tags
ansible-playbook example.yml --tags "configuration,packages" --list-tasks
```
##### Tag inheritance
Define the tags at the level of your play or block, or when you add a role or import a file. Ansible applies the tags down the dependency chain to all child tasks.
Fact gathering is an implicit task tagged with `always` so that runs in addition to tagged tasks by default.
