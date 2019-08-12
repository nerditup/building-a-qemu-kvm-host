# Building a QEMU KVM Host [WIP]
How to build a QEMU KVM host.

## Table of Contents

<!-- vim-markdown-toc GFM -->

* [Prepare the Hard Disk](#prepare-the-hard-disk)
* [Secure the Server](#secure-the-server)
    * [Automatic Security Updates](#automatic-security-updates)
    * [User Account with sudo Privileges](#user-account-with-sudo-privileges)
    * [SSH](#ssh)
    * [4096-bit SSH Authentication Key-pair](#4096-bit-ssh-authentication-key-pair)
    * [Remote SSH Keys](#remote-ssh-keys)
    * [Firewall](#firewall)
* [Mount the Storage RAID](#mount-the-storage-raid)
* [Bridge the Host Network](#bridget-the-host-network)
* [Install QEMU](#install-qemu)
* [Add User to the KVM Group](#add-user-to-the-kvm-group)
* [Create the VM Disk](#create-the-vm-disk)
* [Create the TAP Device](#create-the-tap-device)
* [Sharing Storage Between the Host and Guests](#sharing-storage-between-the-host-and-guest)
    * [Block Storage](#block-storage)
    * [File/NFS Storage](#file-nfs-storage)
    * [Object Storage](#object-storage)
* [References](#references)

<!-- vim-markdown-toc -->

## Prepare the Hard Disk

## Secure the Server

### Automatic Security Updates

### User Account with `sudo` Privileges

### SSH

### 4096-bit SSH Authentication Key-pair

### Remote SSH Keys

### Firewall

```
# apt install nftables
```

Put the following configuration in `/etc/nftables.conf`

```
flush ruleset

table inet filter {
        chain input {
                type filter hook input priority 0;

                # accept any localhost traffic
                iif lo accept

                # accept traffic originated from us
                ct state established,related accept

                # activate the following line to accept common local services
                tcp dport { 22 } ct state new accept

                # accept neighbour discovery otherwise IPv6 connectivity breaks
                ip6 nexthdr icmpv6 icmpv6 type { nd-neighbor-solicit,  nd-router-advert, nd-neighbor-advert } accept

                # count and drop any other traffic
                counter drop
        }
}
```

## Mount the Storage RAID

To mount an `md` RAID block device at boot, the filesystem must be configured in `/etc/fstab`. The following command is used to get the UUID of the filesystem on `md0`,

```
$ lsblk -no UUID /dev/md0 
99418827-5bf7-4390-94b2-d4bd67ec2cee
```

Example `/etc/fstab` configuration,

```
UUID=99418827-5bf7-4390-94b2-d4bd67ec2cee /srv/storage xfs defaults 0 0
```

## Bridge the Host Network

```
# ip addr flush dev enp2s0
# ip link add br0 type bridge
# ip link set enp2s0 master br0
# ip link set dev br0 up
# ip link set dev enp2s0 up
# systemctl restart networking
```

## Install QEMU

```
# apt --no-install-recommends install qemu-kvm
```

## Add User to KVM Group

```
# usermod -a -G kvm $(whoami)
```

## Create the VM Disk

## Create the TAP Device

```
# ip tuntap add dev tap0 mode tap user $(whoami)
# ip link set tap0 master br0
# ip link set dev tap0 up
```

## Sharing Storage Between Host and Guest

### Block Storage

### File/NFS Storage

### Object Storage

## References

- \[1\]: https://wiki.debian.org/
- \[2\]: https://wiki.archlinux.org/ 
