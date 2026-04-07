---
layout: default
title: KVM
description: Kernel Virtual Machine and QEMU notes
tags: kvm qemu virsh libguestfs libvirt
---

* Table of contents
{:toc}

## Kernel Virtual Machine (KVM) management tools and examples

Graphical manager - virt-manager

The qemu, libvirt and libguestfs tools are great for automation.

Default configuration directory with the xml definitions of objects is `/etc/libvirt/qemu/`.

Directing virt-inspector at a disk image shows OS info, mount points, file systems, installed applications without a need to attach the image in any way.
```sh
virt-inspector -a cloud-base-image-ubuntu.qcow2
```
List snapshots in a disk image.
```sh
qemu-img snapshot --list guest-vm.qcow2
```

No spaces allowed for ansible.builtin.command: argv! Works in Ansible.
```sh
qemu-img create -fqcow2 fedora_big.qcow2 15G
```

Get disk image virtual size in bytes.
```sh
qemu-img info --output=json fedora_small.qcow2 | jq '."virtual-size"'
```
### Help argument usage

```sh
virt-admin --help
virt-admin echo --help
virt-admin server-threadpool-set --help
```
Help topics seem to be an exception used in virt-admin
```sh
virt-admin help management
```

### Network operations using libvirt tools

```sh
virsh net-destroy --network vm-net
virsh net-undefine vm-net # --network not required
virsh net-list --all # include inactive networks
virsh domifaddr guest-vm
virsh net-info vm-net
virsh domif-setlink --domain guest-vm --interface eth0 --state up
virsh net-dumpxml vm-net
virsh dumpxml guest-vm | grep 'mac address'
virsh net-update vm-net add ip-dhcp-host '<host mac="52:54:00:6c:3c:01" name="guest-vm" ip="192.168.122.40"/>'
```

### Domain (guest vm) and disk image operations

Tools used are from libvirt, libguestfs and qemu. Default libvirt disk image pool directory is `/var/lib/libvirt/images`.

```sh
virsh pool-info virt --all
virsh pool-info -pool default
virsh pool-dumpxml default
qemu-img info guest-vm.qcow2
virsh list
virsh dominfo --domain guest-vm
virsh shutdown guest-vm
virsh destroy guest-vm
virt-customize --install cloud-init -a without-cloud-init.qcow2 -v
virt-sysprep
virt-resize # disk, including partitions and filesystems
virt-filesystems -a fedora_small.qcow2 --all --long -h
```
#### Resize a disk image

`virt-resize` resizes file systems in the image seamlessly.

```sh
virt-df -a fedora_small.qcow2
qemu-img create -f qcow2 fedora_big.qcow2 15G
truncate --reference=fedora_small.qcow2 fedora_big.qcow2
virt-resize --expand /dev/sda4 fedora_small.qcow2 fedora_big.qcow2
virt-filesystems -a fedora_big.qcow2 --all --long -h
```

#### Mount disk image as a [network block device](https://kernel.org/doc/html/latest/admin-guide/blockdev/nbd.html)

Activate the nbd module in the kernel, connect image, mount it somewhere.
```sh
modprobe nbd max_part=8
qemu-nbd --connect=/dev/nbd0 cloud-base-image-ubuntu.qcow2
gdisk /dev/nbd0 -l
mount /dev/nbd0p3 /mnt/target/
```
Do something, like change files. Then unmount, disconnect image and deactivate the nbd module.
```sh
umount /mnt/target/
qemu-nbd --disconnect /dev/nbd0
rmmod nbd
```
<!-- TODO: check, create examples
```sh
#qemu-img resize guest-vm.qcow2 15G
virsh console --safe guest-vm #serial
```
-->

## KVM bridged with physical machines

Add permanent packet forwarding to kernel conf
```sh
echo 'net.ipv4.ip_forward=1' >> /etc/sysctl.d/99-sysctl.conf
sysctl -p /etc/sysctl.d/99-sysctl.conf
```

Add a bridge manually with brctl (bridge-utils in debian).
```sh
brctl addbr br0
brctl stp br0 off
# forward delay
brctl setfd br0 0
# lower value = higher priority
brctl setbridgeprio br0 0
```
Need to add vnet0 to bridge_ports after startup of VM to reach VM
```sh
brtcl addif br0 vnet0
```

Permanent: edit host's /etc/network/interfaces

```sh
#set up a bridge and give it a static ip
auto br0
iface br0 inet static
        address 192.168.1.2
        netmask 255.255.255.0
        network 192.168.1.0
        broadcast 192.168.1.255
        gateway 192.168.1.1
        bridge_ports eth0
# need to add vnet0 to bridge_ports after startup of VM to reach VM
# brtcl addif br0 vnet0
        bridge_stp off
        bridge_fd 0
# bridge_maxwait only? needed for networking.service
        bridge_maxwait 0
        nameserver 8.8.8.8
        #post-up /etc/network/if-up.d/tc_br0.sh
```
and bring it up for testing using ifup, in which case:

check that gateway is not mentioned twice in /etc/network/interfaces

careful, may need to flush host IP, routes, etc to bring up the bridge on the host with ifup on debian.

```sh
ip addr flush dev eth0
ifup br0
```

Contents of my traffic shaping script tc_br0.sh, commented out in the above example interfaces file. Make sure to make it executable if planning to use it.
Using Fair Queueing Controlled Delay. Some reasoning behind this from the [Arch Linux wiki.](https://wiki.archlinux.org/index.php/advanced_traffic_control)
```sh
#!/bin/sh
#tc qdisc del root dev br0
#tc qdisc show dev br0
/sbin/tc qdisc add dev br0 root fq_codel
```

## virsh (debian package libvirt-clients)

### List VMs

```sh
virsh list --all
```

### For offline configuration

[Source](https://serverfault.com/questions/403561/change-amount-of-ram-and-cpu-cores-in-kvm)

#### edit the XML in VM config:
```sh
virsh edit vm_name
```

#### or individual, easily scriptable commands:

##### To increase the number of CPUs

```sh
virsh setvcpus vm_name vcpu_count --config --maximum
virsh setvcpus vm_name vcpu_count --config
```

##### To increase the memory size

```sh
virsh setmaxmem vm_name memsize --config
virsh setmem vm_name memsize --config
```

### For online configuration

You can set the vCPU and memory while the VM is running with `--current` instead of `--config`, but the new numbers have to be within the maximum values already set. You can not set these maximum numbers while the VM is running. You will have to shutdown the VM.
```sh
virsh shutdown vm_name
```
Change the max memory or CPU count and start it again.
```sh
virsh start vm_name
```

## Creating a VM

### vm-install

Menu driven CLI utility

### virsh

Use 
```sh
--location
```
to use 
```sh
--extra-args
```
or

use 
```sh
--cdrom=/home/isos/debian9.iso
```

```sh
# virt-install \
--name debian9 \
--description "VM with debian 9" \
--os-type=linux \
--os-variant=debian9 \
--ram=2048 \
--vcpus=2 \
--disk path=/var/lib/libvirt/images/debian9.img,bus=virtio,size=10 \
--graphics none \
--location http://httpredir.debian.org/debian/dists/stretch/main/installer-amd64/ \
--network bridge=br0 \
--console pty,target_type=serial \
--extra-args 'console=ttyS0,115200n8 serial'
```

### Clone a VM
```sh
virt-clone --connect=qemu://example.com/system -o vm_old_source -n vm_new_dest --auto-clone
```

## Connect to VM Console without network or X

To connect to the console of the virtual machine use the following command. You can use “ctrl + ]” to exit out of the VM console.

```sh
virsh console vm_name
```

Setting up access to a VM’s console is no different than a physical server, where you simply add the proper kernel boot parameters to the VM.

To add serial console on a Debian 9 VM (Grub2), edit

```sh
#/etc/default/grub
GRUB_CMDLINE_LINUX_DEFAULT="console=ttyS0 console=tty0,115200n8 serial"
GRUB_TERMINAL=serial
GRUB_SERIAL_COMMAND="serial --unit=0 --speed=115200 --word=8 --parity=no --stop=1"
```
run
```sh
update-grub
```
and reboot the VM.

Default console speed is 9600 baud


[Minicom](https://salsa.debian.org/minicom-team/minicom) is an easy to use serial communication program.

```sh
minicom -D /dev/ttyS0
```

## Automated VM deployment of debian with preseed.cfg

This Debian 9 example is deprecated but latest preseed examples exist on [debian.org](https://www.debian.org/releases/stable/example-preseed.txt).

virtual.sh

```sh
virt-install --connect=qemu:///system \
	--name debian9 \
	--description "debian9 test KVM" \
	--os-type=linux \
	--os-variant=debian9 \
	--ram=4096 \
	--vcpus=4 \
	--graphics none \
	--location http://httpredir.debian.org/debian/dists/stretch/main/installer-amd64/ \
	--network bridge=br0 \
	--initrd-inject=/home/user1/virt/preseed.cfg \
	--extra-args 'auto' \
	--disk=pool=virt,size=20,format=qcow2,bus=virtio
#	--extra-args 'console=ttyS0,115200n8 serial'
# graphics options: spice; none; vnc,password=foobar
### The name preseed.cfg is mandatory for debian installer d-i to pick it up
```

preseed.cfg

```sh

#### Contents of the preconfiguration file (for stretch)
### Localization
# Preseeding only locale sets language, country and locale.
#d-i debian-installer/locale string en_US

# The values can also be preseeded individually for greater flexibility.
d-i debian-installer/language string en
d-i debian-installer/country string EE
d-i debian-installer/locale string en_GB.UTF-8
# Optionally specify additional locales to be generated.
d-i localechooser/supported-locales multiselect en_US.UTF-8, et_EE.UTF-8

# Keyboard selection.
#d-i keyboard-configuration/xkb-keymap select et
d-i keyboard-configuration/xkb-keymap select us(dvorak)
#d-i keyboard-configuration/variant select standard
d-i keyboard-configuration/toggle select No toggling
#d-i	keyboard-configuration/variantcode	string	dvorak

### Network configuration
# Disable network configuration entirely. This is useful for cdrom
# installations on non-networked devices where the network questions,
# warning and long timeouts are a nuisance.
#d-i netcfg/enable boolean false

# netcfg will choose an interface that has link if possible. This makes it
# skip displaying a list if there is more than one interface.
d-i netcfg/choose_interface select auto

# To pick a particular interface instead:
#d-i netcfg/choose_interface select eth1

# To set a different link detection timeout (default is 3 seconds).
# Values are interpreted as seconds.
#d-i netcfg/link_wait_timeout string 10

# If you have a slow dhcp server and the installer times out waiting for
# it, this might be useful.
#d-i netcfg/dhcp_timeout string 60
#d-i netcfg/dhcpv6_timeout string 60

# If you prefer to configure the network manually, uncomment this line and
# the static network configuration below.
d-i netcfg/disable_autoconfig boolean true

# If you want the preconfiguration file to work on systems both with and
# without a dhcp server, uncomment these lines and the static network
# configuration below.
#d-i netcfg/dhcp_failed note
#d-i netcfg/dhcp_options select Configure network manually

# Static network configuration.
#
# IPv4 example
d-i netcfg/get_ipaddress string 192.168.1.23
d-i netcfg/get_netmask string 255.255.255.0
d-i netcfg/get_gateway string 192.168.1.1
d-i netcfg/get_nameservers string 192.168.1.1
d-i netcfg/confirm_static boolean true
#
# IPv6 example
#d-i netcfg/get_ipaddress string fc00::2
#d-i netcfg/get_netmask string ffff:ffff:ffff:ffff::
#d-i netcfg/get_gateway string fc00::1
#d-i netcfg/get_nameservers string fc00::1
#d-i netcfg/confirm_static boolean true

# Any hostname and domain names assigned from dhcp take precedence over
# values set here. However, setting the values still prevents the questions
# from being shown, even if values come from dhcp.
d-i netcfg/get_hostname string debian9
d-i netcfg/get_domain string site.lan

# If you want to force a hostname, regardless of what either the DHCP
# server returns or what the reverse DNS entry for the IP is, uncomment
# and adjust the following line.
#d-i netcfg/hostname string somehost

# Disable that annoying WEP key dialog.
d-i netcfg/wireless_wep string
# The wacky dhcp hostname that some ISPs use as a password of sorts.
#d-i netcfg/dhcp_hostname string radish

# If non-free firmware is needed for the network or other hardware, you can
# configure the installer to always try to load it, without prompting. Or
# change to false to disable asking.
#d-i hw-detect/load_firmware boolean true

### Network console
# Use the following settings if you wish to make use of the network-console
# component for remote installation over SSH. This only makes sense if you
# intend to perform the remainder of the installation manually.
#d-i anna/choose_modules string network-console
#d-i network-console/authorized_keys_url string http://10.0.0.1/openssh-key
#d-i network-console/password password r00tme
#d-i network-console/password-again password r00tme

### Mirror settings
# If you select ftp, the mirror/country string does not need to be set.
#d-i mirror/protocol string ftp
d-i mirror/country string manual
d-i mirror/http/hostname string ftp.se.debian.org
d-i mirror/http/directory string /debian
d-i mirror/http/proxy string

# Suite to install.
#d-i mirror/suite string testing
# Suite to use for loading installer components (optional).
#d-i mirror/udeb/suite string testing

### Account setup
# Skip creation of a root account (normal user account will be able to
# use sudo).
d-i passwd/root-login boolean false
# Alternatively, to skip creation of a normal user account.
#d-i passwd/make-user boolean false

# Root password, either in clear text
#d-i passwd/root-password password r00tme
#d-i passwd/root-password-again password r00tme
# or encrypted using a crypt(3)  hash.
#d-i passwd/root-password-crypted password [crypt(3) hash]

# To create a normal user account.
d-i passwd/user-fullname string siteadmin
d-i passwd/username string siteadmin
# Normal user's password, either in clear text
#d-i passwd/user-password password insecure
#d-i passwd/user-password-again password insecure
# or encrypted using a crypt(3) hash.
d-i passwd/user-password-crypted password $6$KmsFpIlvPKeDVp5I$vU5/ydWNa93q2sxkgxQqjFhBBvAOgKhRgcbNiP07wrM3bEcjGsu7qkLSlO0vXo3JMMM4NcEPRAaIVdzJeRJj2.
# Create the first user with the specified UID instead of the default.
#d-i passwd/user-uid string 1010

# The user account will be added to some standard initial groups. To
# override that, use this.
#d-i passwd/user-default-groups string audio cdrom video

### Clock and time zone setup
# Controls whether or not the hardware clock is set to UTC.
d-i clock-setup/utc boolean true

# You may set this to any valid setting for $TZ; see the contents of
# /usr/share/zoneinfo/ for valid values.
d-i time/zone string Europe/Tallinn

# Controls whether to use NTP to set the clock during the install
d-i clock-setup/ntp boolean true
# NTP server to use. The default is almost always fine here.
#d-i clock-setup/ntp-server string ntp.example.com

### Partitioning
## Partitioning example
# If the system has free space you can choose to only partition that space.
# This is only honoured if partman-auto/method (below) is not set.
#d-i partman-auto/init_automatically_partition select biggest_free

# Alternatively, you may specify a disk to partition. If the system has only
# one disk the installer will default to using that, but otherwise the device
# name must be given in traditional, non-devfs format (so e.g. /dev/sda
# and not e.g. /dev/discs/disc0/disc).
# For example, to use the first SCSI/SATA hard disk:
#d-i partman-auto/disk string /dev/sda
# In addition, you'll need to specify the method to use.
# The presently available methods are:
# - regular: use the usual partition types for your architecture
# - lvm:     use LVM to partition the disk
# - crypto:  use LVM within an encrypted partition
d-i partman-auto/method string lvm

# If one of the disks that are going to be automatically partitioned
# contains an old LVM configuration, the user will normally receive a
# warning. This can be preseeded away...
d-i partman-lvm/device_remove_lvm boolean true
# The same applies to pre-existing software RAID array:
d-i partman-md/device_remove_md boolean true
# And the same goes for the confirmation to write the lvm partitions.
d-i partman-lvm/confirm boolean true
d-i partman-lvm/confirm_nooverwrite boolean true

# You can choose one of the three predefined partitioning recipes:
# - atomic: all files in one partition
# - home:   separate /home partition
# - multi:  separate /home, /var, and /tmp partitions
#d-i partman-auto/choose_recipe select atomic

# Or provide a recipe of your own...
# If you have a way to get a recipe file into the d-i environment, you can
# just point at it.
#d-i partman-auto/expert_recipe_file string /hd-media/recipe

# If not, you can put an entire recipe into the preconfiguration file in one
# (logical) line. This example creates a small /boot partition, suitable
# swap, and uses the rest of the space for the root partition:
d-i partman-auto-lvm/new_vg_name string site_vg

d-i partman-auto/expert_recipe string                         \
      boot-root ::                                            \
              512 1000000000 512 ext4                                 \
                      $primary{ }                             \
                      $bootable{ }                            \
                      method{ format } format{ }              \
                      use_filesystem{ } filesystem{ ext4 }    \
                      options/noatime{ noatime }              \
                      mountpoint{ /boot }                     \
                      label{ site_boot }                       \
              .                                               \
              6144 10240 10240 ext4                           \
                      $lvmok{ }                               \
                      method{ format } format{ }              \
                      use_filesystem{ } filesystem{ ext4 }    \
                      reserved_for_root{ 5 }                  \
                      options/relatime{ relatime }            \
                      mountpoint{ / }                         \
                      label{ site_root }                       \
              .                                               \
              2048 1024 4096 ext4                             \
                      $lvmok{ }                               \
                      method{ format } format{ }              \
                      use_filesystem{ } filesystem{ ext4 }    \
                      reserved_for_root{ 5 }                  \
                      options/relatime{ relatime }            \
                      mountpoint{ /var/log }                  \
                      label{ site_var_log }                    \
              .                                               \
#              2048 2048 102400 ext4                            \
#                      $lvmok{ }                               \
#                      method{ format } format{ }              \
#                      use_filesystem{ } filesystem{ ext4 }    \
#                      mountpoint{ /var/lib/mysql }                      \
#                      reserved_for_root{ 2 }                  \
#                      label{ site_var_lib_mysql }                    \
#              .                                               \
#              1024 2048 102400 ext4                            \
#                      $lvmok{ }                               \
#                      method{ format } format{ }              \
#                      use_filesystem{ } filesystem{ ext4 }    \
#                      mountpoint{ /srv }                      \
#                      reserved_for_root{ 1 }                  \
#                      label{ site_srv }                    \
#              .                                               \
#              2048 2048 100% linux-swap                       \
              2048 2048 2048 linux-swap                       \
                      $lvmok{ }                               \
                      method{ swap } format{ }                \
                      label{ swap01 }                         \
              .

# The full recipe format is documented in the file partman-auto-recipe.txt
# included in the 'debian-installer' package or available from D-I source
# repository. This also documents how to specify settings such as file
# system labels, volume group names and which physical devices to include
# in a volume group.

# This makes partman automatically partition without confirmation, provided
# that you told it what to do using one of the methods above.
d-i partman-partitioning/confirm_write_new_label boolean true
d-i partman/choose_partition select finish
d-i partman/confirm boolean true
d-i partman/confirm_nooverwrite boolean true

# When disk encryption is enabled, skip wiping the partitions beforehand.
#d-i partman-auto-crypto/erase_disks boolean false

## Partitioning using RAID
# The method should be set to "raid".
#d-i partman-auto/method string raid
# Specify the disks to be partitioned. They will all get the same layout,
# so this will only work if the disks are the same size.
#d-i partman-auto/disk string /dev/sda /dev/sdb

# Next you need to specify the physical partitions that will be used. 
#d-i partman-auto/expert_recipe string \
#      multiraid ::                                         \
#              1000 5000 4000 raid                          \
#                      $primary{ } method{ raid }           \
#              .                                            \
#              64 512 300% raid                             \
#                      method{ raid }                       \
#              .                                            \
#              500 10000 1000000000 raid                    \
#                      method{ raid }                       \
#              .

# Last you need to specify how the previously defined partitions will be
# used in the RAID setup. Remember to use the correct partition numbers
# for logical partitions. RAID levels 0, 1, 5, 6 and 10 are supported;
# devices are separated using "#".
# Parameters are:
# <raidtype> <devcount> <sparecount> <fstype> <mountpoint> \
#          <devices> <sparedevices>

#d-i partman-auto-raid/recipe string \
#    1 2 0 ext3 /                    \
#          /dev/sda1#/dev/sdb1       \
#    .                               \
#    1 2 0 swap -                    \
#          /dev/sda5#/dev/sdb5       \
#    .                               \
#    0 2 0 ext3 /home                \
#          /dev/sda6#/dev/sdb6       \
#    .

# For additional information see the file partman-auto-raid-recipe.txt
# included in the 'debian-installer' package or available from D-I source
# repository.

# This makes partman automatically partition without confirmation.
d-i partman-md/confirm boolean true
d-i partman-partitioning/confirm_write_new_label boolean true
d-i partman/choose_partition select finish
d-i partman/confirm boolean true
d-i partman/confirm_nooverwrite boolean true

## Controlling how partitions are mounted
# The default is to mount by UUID, but you can also choose "traditional" to
# use traditional device names, or "label" to try filesystem labels before
# falling back to UUIDs.
#d-i partman/mount_style select uuid

### Base system installation
# Configure APT to not install recommended packages by default. Use of this
# option can result in an incomplete system and should only be used by very
# experienced users.
#d-i base-installer/install-recommends boolean false

# The kernel image (meta) package to be installed; "none" can be used if no
# kernel is to be installed.
#d-i base-installer/kernel/image string linux-image-686
d-i base-installer/kernel/image string linux-image-amd64

### Apt setup
# You can choose to install non-free and contrib software.
#d-i apt-setup/non-free boolean true
#d-i apt-setup/contrib boolean true
# Uncomment this if you don't want to use a network mirror.
#d-i apt-setup/use_mirror boolean false
# Select which update services to use; define the mirrors to be used.
# Values shown below are the normal defaults.
#d-i apt-setup/services-select multiselect security, updates
d-i apt-setup/services-select multiselect security, updates, nonfree, contrib
#d-i apt-setup/security_host string security.debian.org

# Additional repositories, local[0-9] available
#d-i apt-setup/local0/repository string \
#       http://local.server/debian stable main
#d-i apt-setup/local0/comment string local server
# Enable deb-src lines
#d-i apt-setup/local0/source boolean true
# URL to the public key of the local repository; you must provide a key or
# apt will complain about the unauthenticated repository and so the
# sources.list line will be left commented out
#d-i apt-setup/local0/key string http://local.server/key

# By default the installer requires that repositories be authenticated
# using a known gpg key. This setting can be used to disable that
# authentication. Warning: Insecure, not recommended.
#d-i debian-installer/allow_unauthenticated boolean true

# Uncomment this to add multiarch configuration for i386
#d-i apt-setup/multiarch string i386


### Package selection
#tasksel tasksel/first multiselect standard, web-server, kde-desktop
tasksel tasksel/first multiselect standard

### keyboard for console configuration
# console-data
# don't touch
#console-common  console-data/keymap/policy      select  Don't touch keymap
#console-data    console-data/keymap/policy      select  Don't touch keymap
# select
console-common  console-data/keymap/policy      select  Select keymap from arch list
console-data    console-data/keymap/policy      select  Select keymap from arch list
#
# keyboard layouts
####console-data    console-data/keymap/qwerty/estonian/standard  select
####console-data    console-data/keymap/dvorak/us/standard/keymap  select
####console-data	console-data/keymap/dvorak/layout	select	
###console-data	console-data/keymap/layout	select dvorak
###console-data	console-data/keymap/dvorak/layout	select	standard

##qwerty ET
#console-data console-data/keymap/qwerty/layout select ET estonian
#console-data console-data/keymap/family select qwerty
#console-common console-data/keymap/family select qwerty

#dvorak US
console-data console-data/keymap/dvorak/layout select US american
console-data console-data/keymap/family select dvorak
console-common console-data/keymap/family select dvorak


# Individual additional packages to install
d-i pkgsel/include string openssh-server sudo aptitude tmux mc vim tmux wget w3m rsync ncdu bzip2 zip mlocate net-tools ntpdate dnsutils iftop iotop ethtool debian-goodies console-data debconf-utils exim4-daemon-light
# Whether to upgrade packages after debootstrap.
# Allowed values: none, safe-upgrade, full-upgrade
d-i pkgsel/upgrade select safe-upgrade

# Some versions of the installer can report back on what software you have
# installed, and what software you use. The default is not to report back,
# but sending reports helps the project determine what software is most
# popular and include it on CDs.
popularity-contest popularity-contest/participate boolean false

### Boot loader installation
# Grub is the default boot loader (for x86). If you want lilo installed
# instead, uncomment this:
#d-i grub-installer/skip boolean true
# To also skip installing lilo, and install no bootloader, uncomment this
# too:
#d-i lilo-installer/skip boolean true


# This is fairly safe to set, it makes grub install automatically to the MBR
# if no other operating system is detected on the machine.
d-i grub-installer/only_debian boolean true

# This one makes grub-installer install to the MBR if it also finds some other
# OS, which is less safe as it might not be able to boot that other OS.
d-i grub-installer/with_other_os boolean false

# Due notably to potential USB sticks, the location of the MBR can not be
# determined safely in general, so this needs to be specified:
#d-i grub-installer/bootdev  string /dev/sda
# To install to the first device (assuming it is not a USB stick):
d-i grub-installer/bootdev  string default

# Alternatively, if you want to install to a location other than the mbr,
# uncomment and edit these lines:
#d-i grub-installer/only_debian boolean false
#d-i grub-installer/with_other_os boolean false
#d-i grub-installer/bootdev  string (hd0,1)
# To install grub to multiple disks:
#d-i grub-installer/bootdev  string (hd0,1) (hd1,1) (hd2,1)

# Optional password for grub, either in clear text
#d-i grub-installer/password password r00tme
#d-i grub-installer/password-again password r00tme
# or encrypted using an MD5 hash, see grub-md5-crypt(8).
#d-i grub-installer/password-crypted password [MD5 hash]

# Use the following option to add additional boot parameters for the
# installed system (if supported by the bootloader installer).
# Note: options passed to the installer will be added automatically.
#d-i debian-installer/add-kernel-opts string nousb

### Finishing up the installation
# During installations from serial console, the regular virtual consoles
# (VT1-VT6) are normally disabled in /etc/inittab. Uncomment the next
# line to prevent this.
#d-i finish-install/keep-consoles boolean true

### skip some annoying installation status notes

# Avoid that last message about the install being complete.
d-i finish-install/reboot_in_progress note
# Avoid the introductory message.
base-config base-config/intro note 
# Avoid the final message.
base-config base-config/login note 


# This will prevent the installer from ejecting the CD during the reboot,
# which is useful in some situations.
#d-i cdrom-detect/eject boolean false

# This is how to make the installer shutdown when finished, but not
# reboot into the installed system.
#d-i debian-installer/exit/halt boolean true
# This will power off the machine instead of just halting it.
#d-i debian-installer/exit/poweroff boolean true

### Preseeding other packages
# Depending on what software you choose to install, or if things go wrong
# during the installation process, it's possible that other questions may
# be asked. You can preseed those too, of course. To get a list of every
# possible question that could be asked during an install, do an
# installation, and then run these commands:
#   debconf-get-selections --installer > file
#   debconf-get-selections >> file

###### Mailer configuration.

# During a normal install, exim asks only two questions. Here's how to
# avoid even those. More complicated preseeding is possible.
#exim4-config  exim4/dc_eximconfig_configtype  select no configuration at this time
# It's a good idea to set this to whatever user account you choose to
# create. Leaving the value blank results in postmaster mail going to
# /var/mail/mail.
exim4-config  exim4/dc_postmaster   string siteadmin
# Use the single /etc/exim4/exim4.conf.template conf format / don't use split conf.
#To update conf: # man update-exim4.conf
exim4-config	exim4/use_split_config	boolean	false

exim4-config	exim4/dc_localdelivery	select	mbox format in /var/mail/
## Reconfigure exim4-config instead of this package
#exim4-base	exim4-base/drec	error	
#exim4-config	exim4/no_config	boolean	true
## Reconfigure exim4-config instead of this package
#exim4	exim4/drec	error	
#exim4-config	exim4/dc_relay_domains	string	
#exim4-config	exim4/dc_minimaldns	boolean	false
#exim4-config	exim4/dc_readhost	string	
#exim4-config	exim4/mailname	string	debian9.local
#exim4-config	exim4/dc_relay_nets	string	
#exim4-base	exim4/purge_spool	boolean	false
exim4-config	exim4/dc_local_interfaces	string	127.0.0.1 ; ::1
#exim4-config	exim4/hide_mailname	boolean	
## Reconfigure exim4-config instead of this package
#exim4-daemon-light	exim4-daemon-light/drec	error	
exim4-config	exim4/dc_eximconfig_configtype	select	local delivery only; not on a network
#exim4-config	exim4/dc_smarthost	string	
#exim4-config	exim4/dc_other_hostnames	string	debian9.local
exim4-config	exim4/dc_postmaster	string	siteadmin


#### Advanced options
### Running custom commands during the installation
# d-i preseeding is inherently not secure. Nothing in the installer checks
# for attempts at buffer overflows or other exploits of the values of a
# preconfiguration file like this one. Only use preconfiguration files from
# trusted locations! To drive that home, and because it's generally useful,
# here's a way to run any shell command you'd like inside the installer,
# automatically.

# This first command is run as early as possible, just after
# preseeding is read.
#d-i preseed/early_command string anna-install some-udeb
# This command is run immediately before the partitioner starts. It may be
# useful to apply dynamic partitioner preseeding that depends on the state
# of the disks (which may not be visible when preseed/early_command runs).
#d-i partman/early_command \
#       string debconf-set partman-auto/disk "$(list-devices disk | head -n1)"
# This command is run just before the install finishes, but when there is
# still a usable /target directory. You can chroot to /target and use it
# directly, or use the apt-install and in-target commands to easily install
# packages and run commands in the target system.
#d-i preseed/late_command string apt-install zsh; in-target chsh -s /bin/zsh

### There can be only one!... preseed late_command
d-i preseed/late_command string wget ftp://ftpsecure:foobar@192.168.1.10/debian-postinstall.sh -O /target/root/debian-postinstall.sh; chown root:root /target/root/debian-postinstall.sh; chmod 750 /target/root/debian-postinstall.sh
```
