---
layout: post
title: "Experimenting with NVMe over TCP"
---

## What is NVMe over TCP

NVMe over TCP is like iSCSI, a standard for network-based storage. It
provides block-level access to NVMe drives over TCP/IP, having much
higher performance than iSCSI.

iSCSI was originally designed for SCSI drives, i.e. *SSD* (Slow
Spinning Disks). It's an old protocol that does not suit the needs of
today's SSDs.

The kernel Linux has native support for NVMe over TCP.

## My setup

```
Target:
Motherboard: Supermicro H12SSL-NT
CPU: AMD EPYC 7282
RAM: plenty
Storage: 4 x 7.68TB Kioxia CD6 U.3 NVMe SSD, ZFS RAIDz1
Network: Onboard Broadcom 10GbE
OS: Debian 12 (bookworm)

Initiator:
Motherboard: a good one
CPU: AMD Ryzen 9 5950X
RAM: plenty
Network: TP-link AQC107 10GbE
OS: Arch Linux, kernel 6.3.y
```

They are connected to the switch ports of my home router (OpenWRT
x86_64 on Ryzen 3 3100 with Intel X710 10GbE, promiscuous mode on).

MTU is set to 9000.

### Configure target

- Install `nvme-cli` package from the distro's package manager, if not
  already installed.

- Load `nvmet-tcp` kernel module if it is not loaded at boot:

```
modprobe nvmet-tcp
```

Alternatively, let the kernel load `nvmet-tcp` kernel module at boot:

```
echo "nvmet_tcp" > /etc/modules-load.d/nvmet_tcp.conf
```

- Create and configure an NVMe target subsystem, let's call it "mysub":

```
mkdir /sys/kernel/config/nvmet/subsystems/mysub
cd /sys/kernel/config/nvmet/subsystems/mysub
echo 1 > attr_allow_any_host
```

If the kernel module `nvmet-tcp` is loaded, directory
`/sys/kernel/config/nvmet` should exist.

If running as sudo, `echo 1 > attr_allow_any_host` might fail. In this
case, `su root` or try `echo 1 | sudo tee -a attr_allow_any_host >
/dev/null` instead.

- Before creating a namespace for the target, use `lsblk` or `nvme
  list` to find out the name of the NVMe device to be attached to the
  target (i.e. /dev/nvme0n1).

- Next, create and configure a namespace `1`:

```
mkdir /sys/kernel/config/nvmet/subsystems/mysub/namespace/1
cd /sys/kernel/config/nvmet/subsystems/mysub/namespace/1
echo -n /dev/nvme0n1 > device_path
echo 1 > enable
```

- Configure a port `1` for an initiator to connect:

```
mkdir /sys/kernel/config/nvmet/ports/1
cd /sys/kernel/config/nvmet/ports/1
echo "ipv4" > addr_adrfam
echo "tcp" > addr_trtype
echo 4420 > addr_trsvcid
echo 192.168.0.231 > addr_traddr
```

Here, our target has a static IPv4 address `192.168.0.231`. 4420 is
the common port number used for NVMe-oF connection.

- Create a symbolic link from the `mysub` subsystem to the port `1`

```
ln -s /sys/kernel/config/nvmet/subsystems/mysub/ /sys/kernel/config/nvmet/ports/1/subsystems/mysub
```

Using `dmesg | grep nvme_tcp`, we can see in the kernel log that the
port is enabled.

### Configure initiator

- Install `nvme-cli` package from the distro's package manager, if not
  already installed.

- Load `nvme-tcp` kernel module (***NOT*** `nvmet-tcp`) if it is not
  loaded at boot:

```
modprobe nvme-tcp
```

Alternatively, let the kernel load `nvme-tcp` kernel module at boot:

```
echo "nvme_tcp" > /etc/modules-load.d/nvme_tcp.conf
```

- Attempt to discover a remote target. This step can be skipped.

```
nvme discover -t tcp -a 192.168.0.231 -s 4420
```

- Connect the NVMe device:

```
nvme connect -t tcp -a 192.168.0.231 -s 4420 -n mysub
```

The subsystem NQN argument `-n mysub` can be replaced by a host NQN,
which is stored in the target's `/etc/nvme/hostnqn`. The command will
look like this:

```
nvme connect -t tcp -a 192.168.0.231 -s 4420 --hostnqn=nqn.2014-08.org.nvmexpress:uuid:1b4e28ba-2fa1-11d2-883f-0016d3ccabcd
```

- Check `nvme list` or `lsblk` for the attached device. To disconnect,
  first unmount the mounted NVMe device, then run `nvme disconnect`:

```
nvme disconnect /dev/nvme2n1 -n mysub
```

### Performance

Definitely faster than NFS, Samba, or iSCSI.

After formatting the zvol to ext4, I was able to use the package
`fio-3.34` to test the I/O performance. Here is the test profile I
used:

```
[global]
bs=4k
ioengine=libaio
iodepth=32
size=1g
numjobs=16
direct=1
refill_buffers=1
runtime=60
directory=/mnt
group_reporting=1
filename=ssd.test.file

[rand-read]
rw=randread
stonewall

[rand-write]
rw=randwrite
stonewall
```

I got about 650MB/s sequential read/write and about 90K IOPS random
read/write, slightly better than a typical SATA SSD. The numbers were
not great, probably because of the suboptimal zpool setup. Mounting
the zvol on the target directly, I only got about 1.5GB/s sequential
read. Also, the ethernet adapter at the initiator's side, AQC107, is
known to be not very stable and sometimes overheat (and no support for
RDMA). It would be better if I had a 25Gb or 40Gb network adapter.

### Reference

- [NVMe over TCP を試す](https://note.com/ipoc/n/n6bca8a1faa7a)

- [How to setup NVMe/TCP with NVME-oF using KVM and
  QEMU](https://futurewei-cloud.github.io/ARM-Datacenter/qemu/nvme-of-tcp-vms/)

- [NVMe over TCP Test
  Report](https://mellanox.my.site.com/mellanoxcommunity/s/article/NVMe-over-TCP-Test-Report)
