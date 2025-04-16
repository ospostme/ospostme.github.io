---
title: Homelab Enviroment Setup
date: 2025-04-14 00:00:00 Z
---

# Hardware List

| Hardware type        | Vendor                     | number |          |
| -------------------- | -------------------------- | ------ | -------- |
| CPU                  | Intel(R) Xeon(R) Gold 6138 | 2      | 2.00 GHz |
| NVMe SSD             | SAMSUNG                    | 4      | 256G     |
| NVme SSD controller  | ?                          | 1      | NA       |
| SATA SSD             | Intel                      | 4      | 480G     |
| SAS HDD              | TOSHIBA                    | 3      | 6T       |
| MegaRAID SAS 9364-8i | ?                          | 1      | NA       |
| DDR4 2666MHz         | SAMSUNG                    | 4      | 64G      |
| Tesla P100           | NVIDIA                     | 1      | 16G      |
| X722 10GBASE-T       | Intel                      | 1      | NA       |
| NF5280M5             | Inspur                     | NA     | NA       |

## Storage Verify

Hardware collected from second-hand markets, it's necessary to verify that if
they are qualified as they claimed, especially the storage.

> The smartmontools package contains two utility programs (smartctl and smartd)
> to control and monitor storage systems using the Self-Monitoring, Analysis and
> Reporting Technology System (SMART) built into most modern ATA/SATA, SCSI/SAS
> and NVMe disks.
> SMART provides data about the health and reliability of storage drives like
> HDDs and SSDs. This data, called SMART attributes, includes metrics like
> temperature, power-on hours, and error rates, which can help predict potential
> failures.

```cmd
sudo apt-get install smartmontools
sudo lshw -class disk
  *-disk:0
       description: SCSI Disk
       product: MR9362-8i      ---->(1) Driver get from Raid device
       vendor: LSI                      4 Intel 480 SATA SSD Raid 5
       physical id: 2.0.0
       bus info: scsi@0:2.0.0
       logical name: /dev/sda
       version: 4.22
       serial: 00555eb15d222f4b2f3084820bb00506
       size: 1339GiB (1438GB)
       capabilities: gpt-1.00 partitioned partitioned:gpt
       configuration: ansiversion=5 guid=df54c6a9-067c-4132-aa63-46c4e31c94db logicalsectorsize=512 sectorsize=4096
  *-disk:1
       description: SCSI Disk
       product: MR9362-8i      ---->(2) 3 TOSHIBA 6T HDD  Raid 5
       vendor: LSI
       physical id: 2.1.0
       bus info: scsi@0:2.1.0
       logical name: /dev/sdb
       version: 4.22
       serial: 00a14af514c0de4c2f3084820bb00506
       size: 10TiB (12TB)
       capabilities: gpt-1.00 partitioned partitioned:gpt
       configuration: ansiversion=5 guid=5faf45aa-64c4-4384-9c30-96d678f9c934 logicalsectorsize=512 sectorsize=4096
  *-namespace:0
       description: NVMe disk
       physical id: 0
       logical name: hwmon3
  *-namespace:1
       description: NVMe disk
       physical id: 2
       logical name: /dev/ng0n1
  *-namespace:2
       description: NVMe disk
       physical id: 1
       bus info: nvme@0:1
       logical name: /dev/nvme0n1
       size: 238GiB (256GB)
       configuration: logicalsectorsize=512 sectorsize=512 wwid=eui.0025388191b46aac
  *-namespace:0
       description: NVMe disk
       physical id: 0
       logical name: hwmon1
  *-namespace:1
       description: NVMe disk
       physical id: 2
       logical name: /dev/ng1n1
  *-namespace:2
       description: NVMe disk
       physical id: 1
       bus info: nvme@1:1
       logical name: /dev/nvme1n1
       size: 238GiB (256GB)
       configuration: logicalsectorsize=512 sectorsize=512 wwid=eui.0025388b91014a37
  *-namespace:0
       description: NVMe disk
       physical id: 0
       logical name: hwmon0
  *-namespace:1
       description: NVMe disk
       physical id: 2
       logical name: /dev/ng2n1
  *-namespace:2
       description: NVMe disk
       physical id: 1
       bus info: nvme@2:1
       logical name: /dev/nvme2n1
       size: 238GiB (256GB)
       configuration: logicalsectorsize=512 sectorsize=512 wwid=eui.002538879101723a
  *-namespace:0
       description: NVMe disk
       physical id: 0
       logical name: hwmon2
  *-namespace:1
       description: NVMe disk
       physical id: 2
       logical name: /dev/ng3n1
  *-namespace:2
       description: NVMe disk
       physical id: 1
       bus info: nvme@3:1
       logical name: /dev/nvme3n1
       size: 238GiB (256GB)
       configuration: logicalsectorsize=512 sectorsize=512 wwid=eui.0025388981bd05b8

```

### SMART information of visible drivers

```cmd
ospost@rabbit:~/HW$ sudo smartctl -a /dev/nvme0
smartctl 7.2 2020-12-30 r5155 [x86_64-linux-5.15.0-105-generic] (local build)
Copyright (C) 2002-20, Bruce Allen, Christian Franke, www.smartmontools.org

=== START OF INFORMATION SECTION ===
Model Number:                       SAMSUNG MZVLB256HAHQ-000H1
Serial Number:                      S425NX0M146105
Firmware Version:                   EXD70H1Q
PCI Vendor/Subsystem ID:            0x144d
IEEE OUI Identifier:                0x002538
Total NVM Capacity:                 256,060,514,304 [256 GB]
Unallocated NVM Capacity:           0
Controller ID:                      4
NVMe Version:                       1.2
Number of Namespaces:               1
Namespace 1 Size/Capacity:          256,060,514,304 [256 GB]
Namespace 1 Utilization:            71,870,574,592 [71.8 GB]
Namespace 1 Formatted LBA Size:     512
Namespace 1 IEEE EUI-64:            002538 8191b46aac
Local Time is:                      Mon Apr 14 11:34:54 2025 UTC
Firmware Updates (0x16):            3 Slots, no Reset required
Optional Admin Commands (0x0017):   Security Format Frmw_DL Self_Test
Optional NVM Commands (0x001f):     Comp Wr_Unc DS_Mngmt Wr_Zero Sav/Sel_Feat
Log Page Attributes (0x03):         S/H_per_NS Cmd_Eff_Lg
Maximum Data Transfer Size:         512 Pages
Warning  Comp. Temp. Threshold:     81 Celsius
Critical Comp. Temp. Threshold:     82 Celsius

Supported Power States
St Op     Max   Active     Idle   RL RT WL WT  Ent_Lat  Ex_Lat
 0 +     7.02W       -        -    0  0  0  0        0       0
 1 +     6.30W       -        -    1  1  1  1        0       0
 2 +     3.50W       -        -    2  2  2  2        0       0
 3 -   0.0760W       -        -    3  3  3  3      210    1200
 4 -   0.0050W       -        -    4  4  4  4     2000    8000

Supported LBA Sizes (NSID 0x1)
Id Fmt  Data  Metadt  Rel_Perf
 0 +     512       0         0
 1 -    4096       0         0

=== START OF SMART DATA SECTION ===
SMART overall-health self-assessment test result: PASSED

SMART/Health Information (NVMe Log 0x02)
Critical Warning:                   0x00
Temperature:                        30 Celsius
Available Spare:                    100%
Available Spare Threshold:          5%
Percentage Used:                    15%
Data Units Read:                    42,610,675 [21.8 TB]
Data Units Written:                 81,336,635 [41.6 TB]
Host Read Commands:                 888,736,152
Host Write Commands:                2,273,241,624
Controller Busy Time:               17,274
Power Cycles:                       415
Power On Hours:                     16,886
Unsafe Shutdowns:                   40
Media and Data Integrity Errors:    0
Error Information Log Entries:      3,939
Warning  Comp. Temperature Time:    0
Critical Comp. Temperature Time:    0
Temperature Sensor 1:               30 Celsius
Temperature Sensor 2:               54 Celsius

Error Information (NVMe Log 0x01, 16 of 256 entries)
Num   ErrCount  SQId   CmdId  Status  PELoc          LBA  NSID    VS
  0       3939     0  0xb004  0x4004      -            0     0     -


```

### SMART information of invisible drivers behind raid controller

[SMART data be read from HDDs connected to LSI MegaRAID controllers](https://www.broadcom.com/support/knowledgebase/1211161499892/can-smart-data-be-read-from-hdds-connected-to-lsi-megaraid-contr)
[Smartmontools with MegaRAID Controller](https://www.thomas-krenn.com/en/wiki/Smartmontools_with_MegaRAID_Controller)

```cmd
ospost@rabbit:~/HW/stocli$ cat /proc/scsi/scsi
Attached devices:
Host: scsi0 Channel: 02 Id: 00 Lun: 00
  Vendor: LSI      Model: MR9362-8i        Rev: 4.22
  Type:   Direct-Access                    ANSI  SCSI revision: 05
Host: scsi0 Channel: 02 Id: 01 Lun: 00
  Vendor: LSI      Model: MR9362-8i        Rev: 4.22
  Type:   Direct-Access                    ANSI  SCSI revision: 05

```

There are two RAID volumes (Vendor: LSI)

```cmd
ospost@rabbit:~/HW/stocli$ sudo /opt/MegaRAID/storcli/storcli64 /c0 /eall /sall show
Controller = 0
Status = Success
Description = Show Drive Information Succeeded.


Drive Information :
=================

----------------------------------------------------------------------------
EID:Slt DID State DG       Size Intf Med SED PI SeSz Model               Sp
----------------------------------------------------------------------------
22:1     25 Onln   0 446.625 GB SATA SSD N   N  512B INTEL SSDSC2BB480G4 U
22:2     24 Onln   0 446.625 GB SATA SSD N   N  512B INTEL SSDSC2BB480G4 U
22:7     39 Onln   1   5.457 TB SAS  HDD N   N  512B MG04SCA60EE         U
22:8     40 Onln   1   5.457 TB SAS  HDD N   N  512B MG04SCA60EE         U
22:9     41 Onln   1   5.457 TB SAS  HDD N   N  512B MG04SCA60EE         U
22:10    28 Onln   0 446.625 GB SATA SSD N   N  512B INTEL SSDSC2BB480G4 U
22:15    38 Onln   0 446.625 GB SATA SSD N   N  512B INTEL SSDSC2BB480G4 U
----------------------------------------------------------------------------

EID-Enclosure Device ID|Slt-Slot No.|DID-Device ID|DG-DriveGroup
DHS-Dedicated Hot Spare|UGood-Unconfigured Good|GHS-Global Hotspare
UBad-Unconfigured Bad|Onln-Online|Offln-Offline|Intf-Interface
Med-Media Type|SED-Self Encryptive Drive|PI-Protection Info
SeSz-Sector Size|Sp-Spun|U-Up|D-Down|T-Transition|F-Foreign
UGUnsp-Unsupported|UGShld-UnConfigured shielded|HSPShld-Hotspare shielded
CFShld-Configured shielded

```

There are seven hard drivers, with device ID 25, 24, 39, 40, 41, 28, 38 on RAID controller

```cmd

sudo smartctl -a -d megaraid,25  /dev/sda
sudo smartctl -a -d megaraid,24  /dev/sda
sudo smartctl -a -d megaraid,28  /dev/sda
sudo smartctl -a -d megaraid,38  /dev/sda
sudo smartctl -a -d megaraid,39  /dev/sdb
sudo smartctl -a -d megaraid,40  /dev/sdb
sudo smartctl -a -d megaraid,41  /dev/sdb

```

Speed test

```
ospost@rabbit:~/HW$ sudo hdparm -Tt /dev/sda

/dev/sda:
 Timing cached reads:   15072 MB in  1.99 seconds = 7585.64 MB/sec
 Timing buffered disk reads: 4716 MB in  3.00 seconds = 1571.46 MB/sec
ospost@rabbit:~/HW$ sudo hdparm -Tt /dev/sdb

/dev/sdb:
 Timing cached reads:   15294 MB in  1.99 seconds = 7696.85 MB/sec
 Timing buffered disk reads: 206 MB in  3.02 seconds =  68.24 MB/sec
ospost@rabbit:~/HW$ sudo hdparm -Tt /dev/vgfast/lvfast

/dev/vgfast/lvfast:
 Timing cached reads:   15352 MB in  1.99 seconds = 7726.92 MB/sec
 Timing buffered disk reads: 5578 MB in  3.00 seconds = 1858.20 MB/sec
ospost@rabbit:~/HW$ sudo hdparm -Tt /dev/vgcache/lvslow

/dev/vgcache/lvslow:
 Timing cached reads:   16060 MB in  1.99 seconds = 8085.37 MB/sec
 Timing buffered disk reads: 284 MB in  3.10 seconds =  91.50 MB/sec


TBD Speed test with dd, fio

```

## Physical/Logical Drivers

```cmd

ospost@rabbit:~/HW$ lsblk
NAME                               MAJ:MIN RM   SIZE RO TYPE  MOUNTPOINTS
loop0                                7:0    0  63.9M  1 loop  /snap/core20/2318
loop1                                7:1    0     4K  1 loop  /snap/bare/5
loop2                                7:2    0  63.7M  1 loop  /snap/core20/2496
loop3                                7:3    0  73.9M  1 loop  /snap/core22/1908
loop4                                7:4    0  73.9M  1 loop  /snap/core22/1802
loop5                                7:5    0 258.3M  1 loop  /snap/firefox/5947
loop6                                7:6    0   242M  1 loop  /snap/firefox/6019
loop7                                7:7    0  91.7M  1 loop  /snap/gtk-common-themes/1535
loop8                                7:8    0   516M  1 loop  /snap/gnome-42-2204/202
loop9                                7:9    0  89.4M  1 loop  /snap/lxd/31333
loop10                               7:10   0  44.4M  1 loop  /snap/snapd/23545
loop11                               7:11   0    87M  1 loop  /snap/lxd/29351
loop12                               7:12   0  44.4M  1 loop  /snap/snapd/23771
sda                                  8:0    0   1.3T  0 disk
├─sda1                               8:1    0     1G  0 part  /boot/efi
├─sda2                               8:2    0     2G  0 part  /boot
└─sda3                               8:3    0   1.3T  0 part
  ├─ubuntu--vg-ubuntu--lv          253:0    0   100G  0 lvm   /
  └─ubuntu--vg-lv--0               253:1    0   1.2T  0 lvm   /home
sdb                                  8:16   0  10.9T  0 disk
├─sdb1                               8:17   0     5T  0 part
│ └─vgcache-lvslow_corig           253:6    0   4.9T  0 lvm
│   └─vgcache-lvslow               253:7    0   4.9T  0 lvm   /mnt/slowcache
├─sdb2                               8:18   0     2T  0 part
└─sdb3                               8:19   0   3.9T  0 part
  └─vgbackup-lvbackup              253:3    0     3T  0 lvm   /mnt/backup
nvme2n1                            259:0    0 238.5G  0 disk
└─md127                              9:127  0 953.4G  0 raid0
  ├─md127p1                        259:4    0   200G  0 part
  │ ├─vgcache-lv_cache_cpool_cdata 253:4    0   100G  0 lvm
  │ │ └─vgcache-lvslow             253:7    0   4.9T  0 lvm   /mnt/slowcache
  │ └─vgcache-lv_cache_cpool_cmeta 253:5    0   100M  0 lvm
  │   └─vgcache-lvslow             253:7    0   4.9T  0 lvm   /mnt/slowcache
  └─md127p2                        259:5    0 753.4G  0 part
    └─vgfast-lvfast                253:2    0   753G  0 lvm   /mnt/fast
nvme1n1                            259:1    0 238.5G  0 disk
└─md127                              9:127  0 953.4G  0 raid0
  ├─md127p1                        259:4    0   200G  0 part
  │ ├─vgcache-lv_cache_cpool_cdata 253:4    0   100G  0 lvm
  │ │ └─vgcache-lvslow             253:7    0   4.9T  0 lvm   /mnt/slowcache
  │ └─vgcache-lv_cache_cpool_cmeta 253:5    0   100M  0 lvm
  │   └─vgcache-lvslow             253:7    0   4.9T  0 lvm   /mnt/slowcache
  └─md127p2                        259:5    0 753.4G  0 part
    └─vgfast-lvfast                253:2    0   753G  0 lvm   /mnt/fast
nvme3n1                            259:2    0 238.5G  0 disk
└─md127                              9:127  0 953.4G  0 raid0
  ├─md127p1                        259:4    0   200G  0 part
  │ ├─vgcache-lv_cache_cpool_cdata 253:4    0   100G  0 lvm
  │ │ └─vgcache-lvslow             253:7    0   4.9T  0 lvm   /mnt/slowcache
  │ └─vgcache-lv_cache_cpool_cmeta 253:5    0   100M  0 lvm
  │   └─vgcache-lvslow             253:7    0   4.9T  0 lvm   /mnt/slowcache
  └─md127p2                        259:5    0 753.4G  0 part
    └─vgfast-lvfast                253:2    0   753G  0 lvm   /mnt/fast
nvme0n1                            259:3    0 238.5G  0 disk
└─md127                              9:127  0 953.4G  0 raid0
  ├─md127p1                        259:4    0   200G  0 part
  │ ├─vgcache-lv_cache_cpool_cdata 253:4    0   100G  0 lvm
  │ │ └─vgcache-lvslow             253:7    0   4.9T  0 lvm   /mnt/slowcache
  │ └─vgcache-lv_cache_cpool_cmeta 253:5    0   100M  0 lvm
  │   └─vgcache-lvslow             253:7    0   4.9T  0 lvm   /mnt/slowcache
  └─md127p2                        259:5    0 753.4G  0 part
    └─vgfast-lvfast                253:2    0   753G  0 lvm   /mnt/fast

```

### Software raid

```
sudo mdadm --create --verbose /dev/md0 --level=0 --raid-devices=4 /dev/nvme0 /dev/nvme1 /dev/nvme2 /dev/nvme3

ospost@rabbit:~/HW$ cat /proc/mdstat
Personalities : [raid0] [linear] [multipath] [raid1] [raid6] [raid5] [raid4] [raid10]
md127 : active raid0 nvme0n1[0] nvme1n1[1] nvme3n1[3] nvme2n1[2]
      999706624 blocks super 1.2 512k chunks

```

### layout

1. sda FOUR Intel 480G SATA SSD drivers construct RAID 5 group (raid controller)

2. sdb THREE TOSHIBA 6T SAS HDD drivers construct RAID 5 group (raid controller)

- create slow volume group for backup purpose
- slow HDD volume combine with fast SSD cache pool to create cache LVM

3. md127 FOUR NVMe SSD construct RAID 5 group (software raid)

- create fast volume group for high speed
- create small SSD fast cache pool

### physical/logical volumes

LVM (Logical Volume Management) uses Physical Volumes (PVs), Volume Groups (VGs),
and Logical Volumes (LVs)

Physical Volumes (PVs):
These are the raw storage devices recognized by LVM, such as individual hard
drives, partitions, or virtual disks.

```
sudo pvcreate /dev/xxx
sudo pvs
ospost@rabbit:~/HW$ sudo pvs
  PV           VG        Fmt  Attr PSize    PFree
  /dev/md127p1 vgcache   lvm2 a--  <200.00g  <99.90g
  /dev/md127p2 vgfast    lvm2 a--   753.39g  400.00m
  /dev/sda3    ubuntu-vg lvm2 a--    <1.31t       0
  /dev/sdb1    vgcache   lvm2 a--    <5.00t <102.30g
  /dev/sdb2    vgbackup  lvm2 a--     2.00t    2.00t
  /dev/sdb3    vgbackup  lvm2 a--    <3.92t <937.00g

sudo pvdisplay

```

Volume Groups (VGs):
VGs aggregate multiple PVs into a single, virtual pool of storage. This allows
LVM to manage a larger amount of space and provides flexibility in how storage is
allocated

```
sudo vgcreate vg_name pv1 pv2 ...

sudo vgcreate vg1 /dev/sdb1

ospost@rabbit:~/HW$ sudo vgs
  VG        #PV #LV #SN Attr   VSize   VFree
  ubuntu-vg   1   2   0 wz--n-  <1.31t       0
  vgbackup    2   1   0 wz--n-  <5.92t   <2.92t
  vgcache     2   1   0 wz--n-  <5.20t <202.20g
  vgfast      1   1   0 wz--n- 753.39g  400.00m

sudo vgdisplay

```

Logical Volumes (LVs):
LVs are created within a VG and represent the actual partitions that will hold
filesystems. Unlike traditional partitions, LVs can span multiple physical devices
within the VG, offering dynamic resizing and allocation capabilities.

```
sudo lvcreate vgfast -n lvfast

ospost@rabbit:~/HW$ sudo lvs
  LV        VG        Attr       LSize   Pool             Origin         Data%  Meta%  Move Log Cpy%Sync Convert
  lv-0      ubuntu-vg -wi-ao----  <1.21t
  ubuntu-lv ubuntu-vg -wi-ao---- 100.00g
  lvbackup  vgbackup  -wi-ao----   3.00t
  lvslow    vgcache   Cwi-aoC---   4.90t [lv_cache_cpool] [lvslow_corig] 50.67  9.62            0.00
  lvfast    vgfast    -wi-ao---- 753.00g

sudo lvdisplay

```

LV created on top of (within) VG, which consist of one or more PV. LV not
binding with physical driver directly, offering dynamic resizing allocation
capability.

TDB snapshots ..

### Fast disk cache for slow disk

To create a fast SSD cache for a slower HDD in Ubuntu, you can use tools like
bcache, which acts as a block layer cache, or LVM caching which creates a logical
volume cache.

LVM caching utilizes a small, fast logical volume (LV) on the SSD to cache
frequently accessed data from a larger, slower LV on the HDD. You can limit the
cache size on the SSD and create a larger LV on the HDD, with the cached data
residing on the SSD.

[LVM caching](https://manpages.ubuntu.com/manpages/xenial/man7/lvmcache.7.html)

```
sudo vgcreate vgcache /dev/sdb1 /dev/md127p1

sudo lvcreate -L 4.9T -n lvslow vgcache /dev/sdb1
sudo lvcreate -L 100G -n lv_cache vgcache /dev/md127p1
sudo lvcreate -L 100M -n lv_cache_meta vgcache /dev/md127p1

sudo lvconvert --type cache-pool --cachemode writethrough --poolmetadata vgcache/lv_cache_meta vgcache/lv_cache
lvs
sduo lvs -a -o +devices
sudo lvconvert --type cache --cachepool vgcache/lv_cache vgcache/lvslow

```

# KVM

Kernel-based Virtual Machine (KVM) is an open source virtualization technology
for Linux operating systems. With KVM, Linux can function as a hypervisor that
runs multiple, isolated virtual machines (VMs).

## libvirt and virsh

The libvirt project provides an API for managing virtualization platforms. Within
libvirt, virsh is a command-line utility for creating, starting, listing, and
stopping VMs, as well as entering a virtualization shell.

## Virtual Machine Manager

Virtual Machine Manager (known as VMM or virt-manager) provides a desktop
interface for VMs, and is available for major Linux distributions.

## Web consoles

VM administrators can choose to manage their VMs using web-based interfaces.
For example, Cockpit offers a solution that lets users manage VMs from a web interface.

## Networking

KVM networking involves connecting virtual machines (VMs) to a network, whether
it's the host network, a virtual network, or a combination of both.
It's achieved by creating virtual network interfaces (vNICs) within the guest VM
and mapping them to the host's network infrastructure. This can be done using
bridges, NAT (Network Address Translation), or other mechanisms.

```text


                         ┌┐ physical Interface eno1
                         ││
             ┌───────────└┘──────────────┐
             │    ovs virtual switch     │  switch name :ovsbr0
             │                           │
             │                           │
             │           ┌┐              │
             └───────────││──────────────┘
                         ││
                         ││ ovs internal port ovsbr0
                         └┘ host network ipaddress 192.168.124.13
             ──┌──┐─────┌──┐──────┌──┐────
               │  │     │  │      │  │
               │  │     │  │      │  │ KVM eth0
               │  │     │  │      │  │ libvirt vagrant-libvirt NAT
               │  │     │  │      └──┘            192.168.121.0/24
               │  │     │  │
               │  │     │  │ KVM eth1
               │  │     └──┘ libvirt bridge network ovs (host network)
               │  │                                   192.168.124.0/24
               │  │
               └──┘ libvirt default NAT 192.168.122.0/24

```

### Bridged Networking

#### Linux Bridge

[Linux Bridge](https://developers.redhat.com/articles/2022/04/06/introduction-linux-bridging-commands-and-features#spanning_tree_protocol)
VMs can be assigned an IP address on the same network as the host, allowing them
to communicate directly with other devices on the network. This requires setting
up a bridge on the host, which acts as a virtual switch.

A Linux bridge is a kernel module that behaves like a network switch, forwarding
packets between interfaces that are connected to it. It's usually used for
forwarding packets on routers, on gateways, or between VMs and network namespaces
on a host. It cannot perform routing or filtering based on IP addresses or
higher-level protocols. It also lacks advanced features such as QoS, tunneling,
mirroring, etc. Moreover, it may not perform well in high-bandwidth or
high-traffic situations, as it can become overwhelmed or crash.

- general-purpose bridging mechanism within the Linux kernel.
- Configuration via Netlink
- VLAN filter
- VxLAN tunnel mapping
- Primarily operates at Layer 2 (MAC address level)
- Lacking features like routing or advanced traffic management at higher layers.

#### Open vSwitch

Open vSwitch (OVS) is a newer and more sophisticated solution that is targeted at
large-scale and complex virtualization environments. It is a multilayer virtual
switch that can operate at both Layer 2 and Layer 3 of the OSI model. It supports
many advanced features such as GRE, VXLAN, Geneve, MPLS, BGP, NetFlow, sFlow, etc.
It also integrates well with various management and orchestration tools such as
OpenStack, Kubernetes, Docker, etc.

OVS has many advantages over a Linux bridge in terms of functionality and
flexibility. However, it also has some drawbacks. It is more complicated to
configure and use, and it requires more resources and dependencies on the host
machine. It may also introduce some overhead or latency due to its complex
processing logic.

[Opne vSwitch](https://blog.scottlowe.org/2012/08/17/installing-kvm-and-open-vswitch-on-ubuntu/)
[Bridge with netplan](https://www.core27.co/post/bridge-networks-for-kvm-on-ubuntu-2204-server)
[ovs for KVM](https://cloudspinx.com/configure-open-vswitch-bridge-on-kvm-for-virtual-machines/)

```cmd
ovs-vsctl add-br ovsbr0
ovs-vsctl add-port ovsbr0 eno1

ospost@rabbit:/etc/netplan$ sudo ovs-vsctl show
374f99aa-d614-43b0-9c2f-50193eb5f061
    Bridge ovsbr0
        fail_mode: standalone
        Port eno1
            Interface eno1
        Port ovsbr0
            Interface ovsbr0
                type: internal
    ovs_version: "2.17.9"

ospost@rabbit:/etc/netplan$ sudo cat /etc/netplan/50-cloud-init.yaml
# This file is generated from information provided by the datasource.  Changes
# to it will not persist across an instance reboot.  To disable cloud-init's
# network configuration capabilities, write a file
# /etc/cloud/cloud.cfg.d/99-disable-network-config.cfg with the following:
# network: {config: disabled}
network:
    ethernets:
        eno1:
            dhcp4: no
    bridges:
        ovsbr0:
            interfaces:
              - eno1
            openvswitch:
              fail-mode: standalone
            dhcp4: yes

    version: 2

```

#### libvirt Bridge

Specifically designed for connecting VMs to the network within the libvirt
virtualization environment.

```cmd
ospost@rabbit:~/workspace/k8s/mini$ virsh net-list
 Name              State    Autostart   Persistent
----------------------------------------------------
 default           active   no          yes
 ovs               active   yes         yes
 vagrant-libvirt   active   no          yes
```

- default: libvirt NAT network
- ovs: manually created bridge to host ovs network
- vagrant-libvirt: NAT network create by vagrant

```cmd
ospost@rabbit:~/workspace/k8s/mini$
ospost@rabbit:~/workspace/k8s/mini$ virsh net-dumpxml default
<network>
  <name>default</name>
  <uuid>e6fafb5f-f0b5-42da-8b29-e27ba073a8fe</uuid>
  <forward mode='nat'>
    <nat>
      <port start='1024' end='65535'/>
    </nat>
  </forward>
  <bridge name='virbr0' stp='on' delay='0'/>
  <mac address='52:54:00:c5:b7:ad'/>
  <ip address='192.168.122.1' netmask='255.255.255.0'>
    <dhcp>
      <range start='192.168.122.2' end='192.168.122.254'/>
    </dhcp>
  </ip>
</network>

ospost@rabbit:~/workspace/k8s/mini$ virsh net-dumpxml ovs
<network>
  <name>ovs</name>
  <uuid>426e85c9-0a3f-4e8d-b718-9e0dbecfe544</uuid>
  <forward mode='bridge'/>
  <bridge name='ovsbr0'/>
  <virtualport type='openvswitch'/>
</network>

ospost@rabbit:~/workspace/k8s/mini$ virsh net-dumpxml vagrant-libvirt
<network connections='5' ipv6='yes'>
  <name>vagrant-libvirt</name>
  <uuid>557a8397-2fa1-4f77-a93d-d1b5cd0392f5</uuid>
  <forward mode='nat'>
    <nat>
      <port start='1024' end='65535'/>
    </nat>
  </forward>
  <bridge name='virbr1' stp='on' delay='0'/>
  <mac address='52:54:00:b3:95:7d'/>
  <ip address='192.168.121.1' netmask='255.255.255.0'>
    <dhcp>
      <range start='192.168.121.1' end='192.168.121.254'/>
    </dhcp>
  </ip>
</network>

```

[libvirt-networking](https://jamielinux.com/docs/libvirt-networking-handbook/bridged-network.html)

### NAT (Network Address Translation)

### Host-Only Networking

### Virtual Networks

## tools

```cmd
virsh net-list
virsh net-create
virsh net-autostart xxx
virsh net-dumpxml xxx
virsh list
virsh edit --domain xxx

```

# GPU

> TBD GPU time slicing

## KVM GPU passthrough

GPU passthrough is a virtualization technique that allows a virtual machine (VM)
to directly access and utilize a physical GPU on the host system, bypassing the
host's operating system and drivers. This enables the VM to have nearly native
GPU performance for tasks like gaming, video editing, or running applications
that require significant GPU processing power.

[GPU passthrough](https://documentation.suse.com/sles/15-SP6/html/SLES-all/app-gpu-passthru.html)
[Ubuntu GPU passthrough](https://askubuntu.com/questions/1406888/ubuntu-22-04-gpu-passthrough-qemu)
[GPU Passthrough for Beginners](https://github.com/Andrew-Willms/GPU-Passthrough-On-Ubuntu-22.04.2-for-Beginners)
[Cloud](http://cloudpods.org/blog/nvidia-gpu-passthrough-record/)

- Check that CPU virtualization is enabled `dmesg | grep VT-d`
- Enable IOMMU `GRUB_CMDLINE_LINUX_DEFAULT="quiet splash intel_iommu=on"`
- re-generate grub
- Blacklist the Nouveau driver
- Configure VFIO and isolate the GPU used for pass-through. vfio takeover GPU
  device mangement
- config KVM with vfio-pci devices
- KVM guest GPU driver installation

IOMMU refers to the chipset device that maps virtual addresses to physical addresses on your I/O devices (i.e. GPU, disk, etc.)

In order to configure GPU passthrough you need to determine the PCI address(es)
of your GPU and any other devices you wish to pass to your VM.

```cmd

ospost@rabbit:~/HW$ sudo lspci -nn | grep -i nvidia
3b:00.0 3D controller [0302]: NVIDIA Corporation GP100GL [Tesla P100 PCIe 16GB] [10de:15f8] (rev a1)

pci address => 3b:00.0

```

## vGPU

GPU passthrough will bind entire GPU to VM, NVIDIA vGPU software creates virtual GPUs
that can be shared across multiple virtual machines. To freely allocate GPU
resources in KVM based k8s cluster, we have to use vGPU to split the physical
GPU properly.

## P100 vGPU support for Linux hosted KVM matrix

[vGPU Grid Download](https://archive.org/download/NVIDIA-VGPU-Driver-Archive/NVIDIA-GRID-vGPU-Linux-KVM-Drivers/)

| vGPU        | Release | Branch | vGPU Branch Type  | Latest Release | Release Date  | EOL Date      |
| ----------- | ------- | ------ | ----------------- | -------------- | ------------- | ------------- |
| NVIDIA vGPU | 16      | R535   | Long-Term Support | 16.9           | January 2025  | July 2026     |
| NVIDIA vGPU | 15      | R525   | EOL Production    | 15.4           | October 2023  | December 2023 |
| NVIDIA vGPU | 14      | R510   | EOL Production    | 14.4           | December 2022 | February 2023 |

| vGPU | Linux vGPU Manager | Windows vGPU Manager | Linux Driver | Windows Driver | Release Date  |
| ---- | ------------------ | -------------------- | ------------ | -------------- | ------------- |
| 16.9 | 535.230.02         | 539.14               | 535.230.02   | 539.19         | January 2025  |
| 16.8 | 535.216.01         | 538.95               | 535.216.01   | 538.95         | October 2024  |
| 16.7 | 535.183.04         | 538.67               | 535.183.06   | 538.78         | July 2024     |
| ...  | ...                | ...                  | ...          | ...            | ...           |
| 16.2 | 535.129.03         | 537.70               | 535.129.03   | 537.70         | October 2023  |
| 16.1 | 535.104.06         | 537.13               | 535.104.05   | 537.13         | August 2023   |
| 16.0 | 535.54.06          | 536.22               | 535.54.03    | 536.25         | July 2023     |
| 15.4 | 525.147.01         | 529.19               | 525.147.05   | 529.19         | October 2023  |
| 15.3 | 525.125.03         | 529.06               | 525.125.06   | 529.11         | June 2023     |
| 15.2 | 525.105.14         | 528.89               | 525.105.17   | 528.89         | March 2023    |
| 15.1 | 525.85.07          | 528.24               | 525.85.05    | 528.24         | January 2023  |
| 15.0 | 525.60.12          | 527.41               | 525.60.13    | 527.41         | December 2022 |
| 14.4 | 510.108.03         | NA                   | 510.108.03   | 514.08         | December 2022 |

## ubuntu kernel/vGPU driver version

P100 vGPU manger available in R510/R525/R535, as the error of "gpl-only symbol"
kernel error during install, choose the oldest one.

> TBD use the latest 16.9 vGPU manager, recompile kernel should resolve the
> 'qpl-only' error

## KVM vGPU setting

[vGPU manager](https://cloud-atlas.readthedocs.io/zh-cn/latest/kvm/vgpu/install_vgpu_manager.html)
[Ubuntu vGPU config](https://docs.nvidia.com/vgpu/latest/grid-vgpu-user-guide/index.html#ubuntu-install-configure-vgpu)

- Install vGPU Manager for Linux KVM on hypervisor host
- Verify kernel module
- Getting domain and bus, device, function of physical GPU
- Getting full-identifier of GPU
- check VGPU mode
- check supported mdev types
- Create vGPU device instance
- Create vGPU configuration, add in corresponding VM

```cmd

ospost@rabbit:~/workspace/AI/nvidia_vGPU_510/Host_Drivers$ chmod +x ./NVIDIA-Linux-x86_64-510.108.03-vgpu-kvm.run
ospost@rabbit:~/workspace/AI/nvidia_vGPU_510/Host_Drivers$ sudo bash ./NVIDIA-Linux-x86_64-510.108.03-vgpu-kvm.run


ospost@rabbit:~/workspace/AI/nvidia_vGPU_510/Host_Drivers$ lsmod | grep vfio
nvidia_vgpu_vfio       57344  26
mdev                   28672  3 nvidia_vgpu_vfio


ospost@rabbit:~/workspace/AI/nvidia_vGPU_510/Host_Drivers$ sudo lspci | grep -i nvidia
3b:00.0 3D controller: NVIDIA Corporation GP100GL [Tesla P100 PCIe 16GB] (rev a1)
ospost@rabbit:~/workspace/AI/nvidia_vGPU_510/Host_Drivers$ lspci -vvvnnn -s 3b:00.0  | grep -i kernel
        Kernel driver in use: nvidia
        Kernel modules: nvidiafb, nvidia_vgpu_vfio, nvidia

virsh nodedev-list --cap pci | grep


ospost@rabbit:~/workspace/AI/nvidia_vGPU_510/Host_Drivers$ virsh nodedev-list --cap pci | grep 3b_00_0
pci_0000_3b_00_0


ospost@rabbit:~/workspace/AI/nvidia_vGPU_510/Host_Drivers$ virsh nodedev-dumpxml pci_0000_3b_00_0 | egrep 'domain|bus|slot|function'
    <domain>0</domain>
    <bus>59</bus>
    <slot>0</slot>
    <function>0</function>
      <address domain='0x0000' bus='0x3b' slot='0x00' function='0x0'/>
      <address domain='0x0000' bus='0x3a' slot='0x00' function='0x0'/>


ospost@rabbit:~/workspace/AI/nvidia_vGPU_510/Host_Drivers$ nvidia-smi -q | grep VGPU
        Virtualization Mode               : Host VGPU
        Host VGPU Mode                    : Non SR-IOV



ospost@rabbit:/sys/class/mdev_bus/0000:3b:00.0/mdev_supported_types$ mdevctl types
0000:3b:00.0
  nvidia-160
    Available instances: 0
    Device API: vfio-pci
    Name: GRID P100-2B
    Description: num_heads=4, frl_config=45, framebuffer=2048M, max_resolution=5120x2880, max_instance=8
  nvidia-211
    Available instances: 0
    Device API: vfio-pci
    Name: GRID P100-2B4
    Description: num_heads=4, frl_config=45, framebuffer=2048M, max_resolution=5120x2880, max_instance=8
  nvidia-244
    Available instances: 0
    Device API: vfio-pci
    Name: GRID P100-1B4
    Description: num_heads=4, frl_config=45, framebuffer=1024M, max_resolution=5120x2880, max_instance=16
  nvidia-293
    Available instances: 0
    Device API: vfio-pci
    Name: GRID P100-4C
    Description: num_heads=1, frl_config=60, framebuffer=4096M, max_resolution=4096x2160, max_instance=4
  nvidia-294
    Available instances: 0
    Device API: vfio-pci
    Name: GRID P100-8C
    Description: num_heads=1, frl_config=60, framebuffer=8192M, max_resolution=4096x2160, max_instance=2
  nvidia-295
    Available instances: 0
    Device API: vfio-pci
    Name: GRID P100-16C
    Description: num_heads=1, frl_config=60, framebuffer=16384M, max_resolution=4096x2160, max_instance=1
  nvidia-83
    Available instances: 0
    Device API: vfio-pci
    Name: GRID P100-1Q
    Description: num_heads=4, frl_config=60, framebuffer=1024M, max_resolution=5120x2880, max_instance=16
  nvidia-84
    Available instances: 0
    Device API: vfio-pci
    Name: GRID P100-2Q
    Description: num_heads=4, frl_config=60, framebuffer=2048M, max_resolution=7680x4320, max_instance=8
  nvidia-85
    Available instances: 0
    Device API: vfio-pci
    Name: GRID P100-4Q
    Description: num_heads=4, frl_config=60, framebuffer=4096M, max_resolution=7680x4320, max_instance=4
  nvidia-86
    Available instances: 0
    Device API: vfio-pci
    Name: GRID P100-8Q
    Description: num_heads=4, frl_config=60, framebuffer=8192M, max_resolution=7680x4320, max_instance=2
  nvidia-87
    Available instances: 0
    Device API: vfio-pci
    Name: GRID P100-16Q
    Description: num_heads=4, frl_config=60, framebuffer=16384M, max_resolution=7680x4320, max_instance=1
  nvidia-88
    Available instances: 0
    Device API: vfio-pci
    Name: GRID P100-1A
    Description: num_heads=1, frl_config=60, framebuffer=1024M, max_resolution=1280x1024, max_instance=16
  nvidia-89
    Available instances: 0
    Device API: vfio-pci
    Name: GRID P100-2A
    Description: num_heads=1, frl_config=60, framebuffer=2048M, max_resolution=1280x1024, max_instance=8
  nvidia-90
    Available instances: 0
    Device API: vfio-pci
    Name: GRID P100-4A
    Description: num_heads=1, frl_config=60, framebuffer=4096M, max_resolution=1280x1024, max_instance=4
  nvidia-91
    Available instances: 0
    Device API: vfio-pci
    Name: GRID P100-8A
    Description: num_heads=1, frl_config=60, framebuffer=8192M, max_resolution=1280x1024, max_instance=2
  nvidia-92
    Available instances: 0
    Device API: vfio-pci
    Name: GRID P100-16A
    Description: num_heads=1, frl_config=60, framebuffer=16384M, max_resolution=1280x1024, max_instance=1
  nvidia-93
    Available instances: 0
    Device API: vfio-pci
    Name: GRID P100-1B
    Description: num_heads=4, frl_config=45, framebuffer=1024M, max_resolution=5120x2880, max_instance=16


UUID=`uuidgen`
echo "$UUID" > nvidia-86/create
UUID=`uuidgen`
echo "$UUID" > nvidia-86/create



ospost@rabbit:/sys/class/mdev_bus/0000:3b:00.0/mdev_supported_types/nvidia-86$ ls -lh /sys/bus/mdev/devices/
total 0
lrwxrwxrwx 1 root root 0 Apr 15 01:39 3131f5fd-7c04-4a61-92c1-0c37f9547b6b -> ../../../devices/pci0000:3a/0000:3a:00.0/0000:3b:00.0/3131f5fd-7c04-4a61-92c1-0c37f9547b6b
lrwxrwxrwx 1 root root 0 Apr 15 01:39 e55bf865-6983-4d53-967f-51e8a3bb0f57 -> ../../../devices/pci0000:3a/0000:3a:00.0/0000:3b:00.0/e55bf865-6983-4d53-967f-51e8a3bb0f57

ospost@rabbit:/sys/class/mdev_bus/0000:3b:00.0/mdev_supported_types/nvidia-86$ mdevctl list
e55bf865-6983-4d53-967f-51e8a3bb0f57 0000:3b:00.0 nvidia-294 (defined)
3131f5fd-7c04-4a61-92c1-0c37f9547b6b 0000:3b:00.0 nvidia-294 (defined)



ospost@rabbit:/sys/class/mdev_bus/0000:3b:00.0/mdev_supported_types/nvidia-86$ virsh nodedev-dumpxml pci_0000_3b_00_0
<device>
  <name>pci_0000_3b_00_0</name>
  <path>/sys/devices/pci0000:3a/0000:3a:00.0/0000:3b:00.0</path>
  <parent>pci_0000_3a_00_0</parent>
  <driver>
    <name>nvidia</name>
  </driver>
  <capability type='pci'>
    <class>0x030200</class>
    <domain>0</domain>
    <bus>59</bus>
    <slot>0</slot>
    <function>0</function>
    <product id='0x15f8'>GP100GL [Tesla P100 PCIe 16GB]</product>
    <vendor id='0x10de'>NVIDIA Corporation</vendor>
    <capability type='mdev_types'>
      <type id='nvidia-88'>
        <name>GRID P100-1A</name>
        <deviceAPI>vfio-pci</deviceAPI>
        <availableInstances>0</availableInstances>
      </type>
      <type id='nvidia-211'>
        <name>GRID P100-2B4</name>
        <deviceAPI>vfio-pci</deviceAPI>
        <availableInstances>0</availableInstances>
      </type>
      <type id='nvidia-86'>
        <name>GRID P100-8Q</name>
        <deviceAPI>vfio-pci</deviceAPI>
        <availableInstances>0</availableInstances>
      </type>
      <type id='nvidia-84'>
        <name>GRID P100-2Q</name>
        <deviceAPI>vfio-pci</deviceAPI>
        <availableInstances>0</availableInstances>
      </type>
      <type id='nvidia-294'>
        <name>GRID P100-8C</name>
        <deviceAPI>vfio-pci</deviceAPI>
        <availableInstances>0</availableInstances>
      </type>
      <type id='nvidia-92'>
        <name>GRID P100-16A</name>
        <deviceAPI>vfio-pci</deviceAPI>
        <availableInstances>0</availableInstances>
      </type>
      <type id='nvidia-244'>
        <name>GRID P100-1B4</name>
        <deviceAPI>vfio-pci</deviceAPI>
        <availableInstances>0</availableInstances>
      </type>
      <type id='nvidia-90'>
        <name>GRID P100-4A</name>
        <deviceAPI>vfio-pci</deviceAPI>
        <availableInstances>0</availableInstances>
      </type>
      <type id='nvidia-89'>
        <name>GRID P100-2A</name>
        <deviceAPI>vfio-pci</deviceAPI>
        <availableInstances>0</availableInstances>
      </type>
      <type id='nvidia-87'>
        <name>GRID P100-16Q</name>
        <deviceAPI>vfio-pci</deviceAPI>
        <availableInstances>0</availableInstances>
      </type>
      <type id='nvidia-85'>
        <name>GRID P100-4Q</name>
        <deviceAPI>vfio-pci</deviceAPI>
        <availableInstances>0</availableInstances>
      </type>
      <type id='nvidia-295'>
        <name>GRID P100-16C</name>
        <deviceAPI>vfio-pci</deviceAPI>
        <availableInstances>0</availableInstances>
      </type>
      <type id='nvidia-93'>
        <name>GRID P100-1B</name>
        <deviceAPI>vfio-pci</deviceAPI>
        <availableInstances>0</availableInstances>
      </type>
      <type id='nvidia-83'>
        <name>GRID P100-1Q</name>
        <deviceAPI>vfio-pci</deviceAPI>
        <availableInstances>0</availableInstances>
      </type>
      <type id='nvidia-293'>
        <name>GRID P100-4C</name>
        <deviceAPI>vfio-pci</deviceAPI>
        <availableInstances>0</availableInstances>
      </type>
      <type id='nvidia-160'>
        <name>GRID P100-2B</name>
        <deviceAPI>vfio-pci</deviceAPI>
        <availableInstances>0</availableInstances>
      </type>
      <type id='nvidia-91'>
        <name>GRID P100-8A</name>
        <deviceAPI>vfio-pci</deviceAPI>
        <availableInstances>0</availableInstances>
      </type>
    </capability>
    <iommuGroup number='35'>
      <address domain='0x0000' bus='0x3b' slot='0x00' function='0x0'/>
      <address domain='0x0000' bus='0x3a' slot='0x00' function='0x0'/>
    </iommuGroup>
    <numa node='0'/>
    <pci-express>
      <link validity='cap' port='0' speed='8' width='16'/>
      <link validity='sta' speed='8' width='16'/>
    </pci-express>
  </capability>
</device>



ospost@rabbit:/sys/class/mdev_bus/0000:3b:00.0/mdev_supported_types/nvidia-86$ cat ~/HW/vgpu1
<device>
    <parent>pci_0000_3b_00_0</parent>
    <capability type="mdev">
        <type id="nvidia-294"/>
        <uuid>e55bf865-6983-4d53-967f-51e8a3bb0f57</uuid>
    </capability>
</device>
ospost@rabbit:/sys/class/mdev_bus/0000:3b:00.0/mdev_supported_types/nvidia-86$ cat ~/HW/vgpu2
<device>
    <parent>pci_0000_3b_00_0</parent>
    <capability type="mdev">
        <type id="nvidia-294"/>
        <uuid>3131f5fd-7c04-4a61-92c1-0c37f9547b6b</uuid>
    </capability>
</device>



ospost@rabbit:/sys/class/mdev_bus/0000:3b:00.0/mdev_supported_types/nvidia-86$ virsh nodedev-define ~/HW/vgpu1
ospost@rabbit:/sys/class/mdev_bus/0000:3b:00.0/mdev_supported_types/nvidia-86$ virsh nodedev-define ~/HW/vgpu2


ospost@rabbit:/sys/class/mdev_bus/0000:3b:00.0/mdev_supported_types/nvidia-86$ virsh nodedev-list --cap mdev
mdev_3131f5fd_7c04_4a61_92c1_0c37f9547b6b_0000_3b_00_0
mdev_e55bf865_6983_4d53_967f_51e8a3bb0f57_0000_3b_00_0



ospost@rabbit:/sys/class/mdev_bus/0000:3b:00.0/mdev_supported_types/nvidia-86$ cat ~/HW/vgpu1_virt
    <hostdev mode='subsystem' type='mdev' managed='no' model='vfio-pci' display='off'>
      <source>
        <address uuid='e55bf865-6983-4d53-967f-51e8a3bb0f57'/>
      </source>
      <address type='pci' domain='0x0000' bus='0x00' slot='0x07' function='0x0'/>
    </hostdev>

ospost@rabbit:/sys/class/mdev_bus/0000:3b:00.0/mdev_supported_types/nvidia-86$ cat ~/HW/vgpu2_virt
    <hostdev mode='subsystem' type='mdev' managed='no' model='vfio-pci' display='off'>
      <source>
        <address uuid='3131f5fd-7c04-4a61-92c1-0c37f9547b6b'/>
      </source>
      <address type='pci' domain='0x0000' bus='0x00' slot='0x07' function='0x0'/>
    </hostdev>


```

## vGPU License Server

[vGPU License Server](https://github.com/fenghan0430/How-to-use-vGPU)
[Nvidia DLS License Server](https://github.com/GreenDamTan/fastapi-dls_mirror)

vGPU Software Compatibility Matrix:

550.127.05
550.90.07
550.90.07
550.54.15
550.54.14
535.216.01
535.183.06
535.183.01
535.161.08
535.161.07
535.154.05
535.129.03
535.104.05
535.54.03
525.147.05
510.108.03

# Quick Reference for basic tools involved

## Vagrant

Vagrant is a HashiCorp tool that simplifies creating and managing developer
environments. It bridges the gap between the host machine (your local computer)
and the guest machine (the virtual environment) to ensure seamless integration.
By using configuration files called Vagrantfile, Vagrant automates the setup and
configuration process, enabling teams to focus on development rather than
environment management.

- Vagrant ensures consistent and reproducible environments
- Vagrant environments are portable and shareable, letting teams replicate setups
  efficiently across projects and infrastructure
- Vagrant works with providers like VirtualBox, VMware, Docker, and more, giving
  you the flexibility to tailor environments to your infrastructure and application
  requirements.
- Configure port forwarding lets you access services running on the guest machine
  directly from the host machine.

> Forward ports for Terramino (8081 for frontend, 8080 for backend)
> config.vm.network "forwarded_port", guest: 8080, host: 8080
> config.vm.network "forwarded_port", guest: 8081, host: 8081

- folder synchronization/NFS mount
- multi-machine environments for cluster

### Development environment setup/Provision

```cmd

ospost@rabbit:~/workspace/k8s/mini$ vagrant up
ospost@rabbit:~/workspace/k8s/mini$ vagrant up --no-provision brother sister
ospost@rabbit:~/workspace/k8s/mini$ vagrant halt
ospost@rabbit:~/workspace/k8s/mini$ vagrant destroy

ospost@rabbit:~/workspace/k8s/mini$ vagrant status
[fog][WARNING] Unrecognized arguments: libvirt_ip_command
[fog][WARNING] Unrecognized arguments: libvirt_ip_command
[fog][WARNING] Unrecognized arguments: libvirt_ip_command
[fog][WARNING] Unrecognized arguments: libvirt_ip_command
[fog][WARNING] Unrecognized arguments: libvirt_ip_command
Current machine states:

master1                   running (libvirt)
master2                   running (libvirt)
master3                   running (libvirt)
brother                   running (libvirt)
sister                    running (libvirt)

This environment represents multiple VMs. The VMs are all listed
above with their current state. For more information about a specific
VM, run `vagrant status NAME`.
ospost@rabbit:~/workspace/k8s/mini$ vagrant ssh master
The machine with the name 'master' was not found configured for
this Vagrant environment.
ospost@rabbit:~/workspace/k8s/mini$ vagrant ssh master1
[fog][WARNING] Unrecognized arguments: libvirt_ip_command

Last login: Tue Apr 15 03:02:06 2025 from 192.168.121.1
vagrant@node1:~$


ospost@rabbit:~/workspace/k8s/mini$ ls
common_provision.sh  docker_toolkit.sh  network  test  Vagrantfile  vGPU_cuda_toolkit_cudnn.sh
ospost@rabbit:~/workspace/k8s/mini$ cat Vagrantfile
# -*- mode: ruby -*-
# vi: set ft=ruby :

#Script for creating kvm (Kernel native) vm's for kubernetes
#best be doing this on a fresh box lest things get cattywampus
#need a bridge set up under br0

#Net work prefix in which a single digit is appended
#ex 192.168.1.5 will have a master at 192.168.1.50 and workers starting from 192.168.1.51
#CONTROL_NETWORK_PREFIX="192.168.124.20"
#COMPUTE_NETWORK_PREFIX="192.168.124.21"

#make sure libvirt, qemu, kvm etc are installed
#make sure other hypervisers such as virtualbox are not
ENV['VAGRANT_DEFAULT_PROVIDER'] = 'libvirt'
IMAGE_NAME = "generic/ubuntu2204"

#right now NUM_NODES must be under 9
MASTER_NODES = 3
#WORKER_NODES = 3

Vagrant.configure("2") do |config|
    config.ssh.keys_only = false
    config.vm.synced_folder "./network", "/network"
    config.vm.provider :libvirt do |libvirt|
        libvirt.cpus = 8
        libvirt.memory = 16384
        libvirt.driver = "kvm"
    #config.vm.synced_folder "/mnt/slowcache/NFS/vagrant/", "/home/vagrant/workspace",
    #    type: "nfs",
    #    nfs_version: 4.1,
    #    nfs_udp: false,
    #    mount_options: ['actimeo=0']
    #    #linux__nfs_options: ['rw','no_subtree_check','all_squash','sync','anonuid=65534','anongid=65534']
    #config.nfs.map_uid=65534
    #config.nfs.map_gid=65534
    end

    (1..MASTER_NODES).each do |i|
        config.vm.define "master#{i}" do |master|
            master.vm.box = IMAGE_NAME
                master.vm.network :public_network,
                :dev => "ovsbr0",
                :ovs => true
            master.vm.hostname = "master#{i}"
            master.vm.provision "shell", inline: <<-SHELL
                sudo cp /network/m#{i}-50-vagrant.yaml /etc/netplan/50-vagrant.yaml
                sudo netplan apply
            SHELL
        end
    end



    config.vm.define "brother" do |brother|
        brother.vm.provider :libvirt do |libvirt|
            libvirt.cpus = 16
            libvirt.memory = 131072
            libvirt.driver = "kvm"
        end
        brother.vm.box = IMAGE_NAME
        brother.vm.network :public_network,
            :dev => "ovsbr0",
            :ovs => true
        brother.vm.hostname = "brother"
        brother.vm.provision "shell", inline: <<-SHELL
            sudo cp /network/n1-50-vagrant.yaml /etc/netplan/50-vagrant.yaml
            sudo netplan apply
        SHELL

        brother.vm.provision "shell", path: "vGPU_cuda_toolkit_cudnn.sh"
        brother.vm.provision "shell", path: "docker_toolkit.sh"

    end

    config.vm.define "sister" do |sister|
        sister.vm.provider :libvirt do |libvirt|
            libvirt.cpus = 16
            libvirt.memory = 65536
            libvirt.driver = "kvm"
        end
        sister.vm.box = IMAGE_NAME
        sister.vm.network :public_network,
            :dev => "ovsbr0",
            :ovs => true
        sister.vm.hostname = "sister"
        sister.vm.provision "shell", inline: <<-SHELL
            sudo cp /network/n2-50-vagrant.yaml /etc/netplan/50-vagrant.yaml
            sudo netplan apply
        SHELL
        sister.vm.provision "shell", path: "vGPU_cuda_toolkit_cudnn.sh"
        sister.vm.provision "shell", path: "docker_toolkit.sh"

    end

    config.vm.provision "shell", path: "common_provision.sh"
end

```

## kubespary

K8S deploy tools, using Ansible as its substrate for provisioning and orchestration

- python environment and Ansible install
- cluster hosts config
- group variables refine in all/all.yml, k8s_cluster/k8s-cluster.yml
  k8s_cluster/addons.yml
- install with Ansible playbook

```cmd

ospost $ conda activate fuck
(fuck) ospost $
Wed Apr 16 00:07:50 CST 2025 ospost@M2.local:/Users/ospost/workspace/k8s-kubeflow/1.28.14
(fuck) ospost $ ls
cert             csi-driver-nfs   image-keeper     juju             kubespray-venv   nfs-csi-config   proxy_config.yml
command          harbor.tar.gz    ingress          kubespray        metallb          nvidia           temp
Wed Apr 16 00:07:51 CST 2025 ospost@M2.local:/Users/ospost/workspace/k8s-kubeflow/1.28.14
(fuck) ospost $ source kubespray-venv/bin/activate
Wed Apr 16 00:08:03 CST 2025 ospost@M2.local:/Users/ospost/workspace/k8s-kubeflow/1.28.14
(kubespray-venv) (fuck) ospost $ ls
cert             csi-driver-nfs   image-keeper     juju             kubespray-venv   nfs-csi-config   proxy_config.yml
command          harbor.tar.gz    ingress          kubespray        metallb          nvidia           temp
Wed Apr 16 00:08:14 CST 2025 ospost@M2.local:/Users/ospost/workspace/k8s-kubeflow/1.28.14
(kubespray-venv) (fuck) ospost $ cd kubespray
Wed Apr 16 00:08:17 CST 2025 ospost@M2.local:/Users/ospost/workspace/k8s-kubeflow/1.28.14/kubespray
(kubespray-venv) (fuck) ospost(release-2.24) $ ansible-playbook -i inventory/mini/hosts.yaml --become --become-user=root cluster.yml

```

## docker

## containerd

## Proxy

```text

                            ┌─────┐ ┌──────┐ ┌──────┐
                            │ Git │ │ porn │ │google│
                            └─────┘ └──────┘ └──────┘
                             ▲         ▲         ▲
                 ┌───────────┴─────────┴─────────┘
                 │                ▲             ▲
               ┌─┴──────┐      ┌──┴──────┐     ┌┴─────────┐
               │ server │      │ server2 │     │  server3 │
               └────────┘      └─────────┘     └──────────┘
                ▲   ▲             ▲               ▲
    ┌───────────┼───┼───────────┐ │               │
    │cloudflare │   │           │ │    ┌──────────┘
    │      ┌────┴─┐ │           │ │    │
    │ dns  │ proxy│ │           │ │    │
    │      └──────┘ │           │ │    │
    │         ▲  ┌────────────┐ │ │    │
    │         │  │ page worker│ │ │    │
    │         │  │ docker.io  │ │ │    │
    │         │  │ quay.io    │ │ │    │
    │         │  │  ...       │ │ │    │
    │         │  └────────────┘ │ │    │
    │         │     ▲           │ │    │
    │         │     │           │ │    │
    └─────────┼─────┼───────────┘ │    │
              │     │             │    │
              │     │             │    │
              │     │             │    │
            ┌──────────────────────────────┐
            │      KVM host proxy client   │
            │                              │
            │          balancer            │
            │                              │
            │          haproxy             │
            │            ▲                 │
            └────────────┼─────────────────┘
                         │
        ┌───────────────┐│
        │  ┌────────────│─┐
        │  │ ┌──────────│───┐
        │  │ │ Guest VMs│ │ │
        └───────────────┘ │ │
           └─│────────────┘ │
             └──────────────┘


```

## NFS

> no_root_squash:
>
> > By default, NFS shares change the root user to the nfsnobody
> > user, an unprivileged user account. In this way, all root-created files are
> > owned by nfsnobody, which prevents uploading of programs with the setuid bit
> > set. If no_root_squash is used, remote root users are able to change any file
> > on the shared file system and leave trojaned applications for other users to
> > inadvertently execute.
>
> all_squash:
>
> > all user mapping to nobody:nogroup
>
> some container by default will encounter privilege problem with NFS volumes,
> for example mysql may not work with all_squash as the init script want to change
> password file to root only, while jupiter-torch-full init script doesn't work
> with root_squash NFS volume.
>
> k8s fsGroup may help, need to change pod yaml file

```text

ospost@rabbit:~$ cat /etc/exports
# /etc/exports: the access control list for filesystems which may be exported
#		to NFS clients.  See exports(5).
#
# Example for NFSv2 and NFSv3:
# /srv/homes       hostname1(rw,sync,no_subtree_check) hostname2(ro,sync,no_subtree_check)
#
# Example for NFSv4:
# /srv/nfs4        gss/krb5i(rw,sync,fsid=0,crossmnt,no_subtree_check)
# /srv/nfs4/homes  gss/krb5i(rw,sync,no_subtree_check)
#

#/mnt/slowcache/NFS    	  	gss/krb5i(rw,sync,fsid=0,crossmnt,no_subtree_check)
#/mnt/slowcache/NFS/K8S        	gss/krb5i(rw,sync,no_subtree_check)

#/mnt/slowcache/NFS/K8S        	*(rw,sync,no_subtree_check,no_root_squash)
/mnt/slowcache/NFS/K8S        	*(rw,sync,no_subtree_check,no_root_squash)
/home/NFS/K8S        		*(rw,sync,no_subtree_check,no_root_squash)
/mnt/slowcache/NFS/k8snobody  	*(rw,sync,all_squash,anonuid=65534,anongid=65534,no_subtree_check)
/home/NFS/k8snobody  		*(rw,sync,all_squash,anonuid=65534,anongid=65534,no_subtree_check)
/mnt/slowcache/NFS/vagrant    	*(rw,sync,all_squash,anonuid=65534,anongid=65534,no_subtree_check)

```

## k8s nfs csi driver

Requires existing and already configured NFSv3 or NFSv4 server, it supports
dynamic provisioning of Persistent Volumes via Persistent Volume Claims by
creating a new sub directory under NFS server

```
curl -skSL https://raw.githubusercontent.com/kubernetes-csi/csi-driver-nfs/v4.10.0/deploy/install-driver.sh | bash -s v4.10.0

ospost $ cat fast-storageclass-nfs.yaml
---
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: fast-nfs-csi-no-squash
  annotations:
    storageclass.kubernetes.io/is-default-class: "true"
provisioner: nfs.csi.k8s.io
parameters:
  #server: nfs-server.default.svc.cluster.local
  server: 192.168.121.1
  share: /home/NFS/K8S
  # csi.storage.k8s.io/provisioner-secret is only needed for providing mountOptions in DeleteVolume
  # csi.storage.k8s.io/provisioner-secret-name: "mount-options"
  # csi.storage.k8s.io/provisioner-secret-namespace: "default"
reclaimPolicy: Delete
volumeBindingMode: Immediate
allowVolumeExpansion: true
mountOptions:
  - nfsvers=4.1
Wed Apr 16 00:15:17 CST 2025 ospost@M2.local:/Users/ospost/workspace/k8s-kubeflow/1.28.14/nfs-csi-config
ospost $ cat fast-storageclass-nfs-nobody.yaml
---
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: fast-nfs-csi-squash
  #  annotations:
  #  storageclass.kubernetes.io/is-default-class: "true"
provisioner: nfs.csi.k8s.io
parameters:
  #server: nfs-server.default.svc.cluster.local
  server: 192.168.121.1
  share: /home/NFS/k8snobody
  # csi.storage.k8s.io/provisioner-secret is only needed for providing mountOptions in DeleteVolume
  # csi.storage.k8s.io/provisioner-secret-name: "mount-options"
  # csi.storage.k8s.io/provisioner-secret-namespace: "default"
reclaimPolicy: Delete
volumeBindingMode: Immediate
allowVolumeExpansion: true
mountOptions:
  - nfsvers=4.1
Wed Apr 16 00:15:23 CST 2025 ospost@M2.local:/Users/ospost/workspace/k8s-kubeflow/1.28.14/nfs-csi-config
ospost $ cat storageclass-nfs
storageclass-nfs-nobody.yaml  storageclass-nfs.yaml
ospost $ cat storageclass-nfs.yaml
---
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: nfs-csi-no-squash
    #annotations:
    #storageclass.kubernetes.io/is-default-class: "true"
provisioner: nfs.csi.k8s.io
parameters:
  server: nfs-server.default.svc.cluster.local
  server: 192.168.121.1
  share: /mnt/slowcache/NFS/K8S
  # csi.storage.k8s.io/provisioner-secret is only needed for providing mountOptions in DeleteVolume
  # csi.storage.k8s.io/provisioner-secret-name: "mount-options"
  # csi.storage.k8s.io/provisioner-secret-namespace: "default"
reclaimPolicy: Delete
volumeBindingMode: Immediate
allowVolumeExpansion: true
mountOptions:
  - nfsvers=4.1
Wed Apr 16 00:15:40 CST 2025 ospost@M2.local:/Users/ospost/workspace/k8s-kubeflow/1.28.14/nfs-csi-config
ospost $ cat storageclass-nfs-nobody.yaml
---
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: nfs-csi-squash
  #  annotations:
  #  storageclass.kubernetes.io/is-default-class: "true"
provisioner: nfs.csi.k8s.io
parameters:
  #server: nfs-server.default.svc.cluster.local
  server: 192.168.121.1
  share: /mnt/slowcache/NFS/k8snobody
  # csi.storage.k8s.io/provisioner-secret is only needed for providing mountOptions in DeleteVolume
  # csi.storage.k8s.io/provisioner-secret-name: "mount-options"
  # csi.storage.k8s.io/provisioner-secret-namespace: "default"
reclaimPolicy: Delete
volumeBindingMode: Immediate
allowVolumeExpansion: true
mountOptions:
  - nfsvers=4.1

ospost $ kubectl apply -f fast-storageclass-nfs.yaml
ospost $ kubectl apply -f fast-storageclass-nfs-nobody.yaml
ospost $ kubectl apply -f storageclass-nfs.yaml
ospost $ kubectl apply -f storageclass-nfs-nobody.yaml

ospost $ kubectl get sc
NAME                               PROVISIONER      RECLAIMPOLICY   VOLUMEBINDINGMODE   ALLOWVOLUMEEXPANSION   AGE
fast-nfs-csi-no-squash (default)   nfs.csi.k8s.io   Delete          Immediate           true                   3d11h
fast-nfs-csi-squash                nfs.csi.k8s.io   Delete          Immediate           true                   3d11h
nfs-csi-no-squash                  nfs.csi.k8s.io   Delete          Immediate           true                   3d11h
nfs-csi-squash                     nfs.csi.k8s.io   Delete          Immediate           true                   3d11h
```

## nvida device plugin

The NVIDIA device plugin for Kubernetes is a Daemonset that allows you to automatically:

- Expose the number of GPUs on each nodes of your cluster
- Keep track of the health of your GPUs
- Run GPU enabled containers in your Kubernetes cluster.

> gpu node need to set default runtime as nvidia
>
> > sudo nvidia-ctk runtime configure --runtime=docker --set-as-default
> > sudo systemctl restart docker
> >
> > sudo nvidia-ctk runtime configure --runtime=containerd --set-as-default
> > sudo systemctl restart containerd.service

```cmd
ospost $ cat nvidia-device-plugin.yml
# Copyright (c) 2019, NVIDIA CORPORATION.  All rights reserved.
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: nvidia-device-plugin-daemonset
  namespace: kube-system
spec:
  selector:
    matchLabels:
      name: nvidia-device-plugin-ds
  updateStrategy:
    type: RollingUpdate
  template:
    metadata:
      labels:
        name: nvidia-device-plugin-ds
    spec:
      tolerations:
      - key: nvidia.com/gpu
        operator: Exists
        effect: NoSchedule
      # Mark this pod as a critical add-on; when enabled, the critical add-on
      # scheduler reserves resources for critical add-on pods so that they can
      # be rescheduled after a failure.
      # See https://kubernetes.io/docs/tasks/administer-cluster/guaranteed-scheduling-critical-addon-pods/
      priorityClassName: "system-node-critical"
      containers:
      - image: nvcr.io/nvidia/k8s-device-plugin:v0.17.1
        name: nvidia-device-plugin-ctr
        env:
          - name: FAIL_ON_INIT_ERROR
            value: "false"
        securityContext:
          allowPrivilegeEscalation: false
          capabilities:
            drop: ["ALL"]
        volumeMounts:
        - name: device-plugin
          mountPath: /var/lib/kubelet/device-plugins
      volumes:
      - name: device-plugin
        hostPath:
          path: /var/lib/kubelet/device-plugins
Wed Apr 16 00:19:09 CST 2025 ospost@M2.local:/Users/ospost/workspace/k8s-kubeflow/1.28.14/nvidia
ospost $ kubectl apply -f nvidia-device-plugin.yml

```

## helm

Helm helps you manage Kubernetes applications — Helm Charts help you define,
install, and upgrade even the most complex Kubernetes application.

kubspary 2.27 with k8s 1.31 works fine with device plugin installed by helm
BUT kubespary 2.24 with k8s 1.28 only works with static device plugin daemonset.

```cmd
helm repo add nvdp https://nvidia.github.io/k8s-device-plugin
helm repo list
helm repo update; helm search repo nvdp --devel

kubectl apply -f nfs-csi.yaml
kubectl delete -f pvc-nfs-csi-dynamic.yaml
kubectl apply -f pvc-nfs-csi-dynamic.yaml
kubectl get sc

helm upgrade -i nvdp nvdp/nvidia-device-plugin   --namespace nvidia-device-plugin   --create-namespace   --version 0.17.1 --set runtimeClassName="nvidia" --set fail-on-init-error=false  --set deviceDiscoveryStrategy=nvml --set gfd.enabled=true

```

## charts

A chart is a collection of files that describe a related set of Kubernetes
resources. A single chart might be used to deploy something simple, like a
memcached pod, or something complex, like a full web app stack with HTTP servers,
databases, caches, and so on.

Charts are created as files laid out in a particular directory tree. They can be
packaged into versioned archives to be deployed.

## kubeflow

Kubeflow makes artificial intelligence and machine learning simple, portable,
and scalable. We are an ecosystem of Kubernetes based components for each stage
in the AI/ML Lifecycle

The Kubeflow Platform refers to the full suite of Kubeflow components bundled
together with additional integration and management tools. Using Kubeflow as a
platform means deploying a comprehensive ML toolkit for the entire ML lifecycle.

## kubeflow manifest

The Kubeflow manifests are a collection of community maintained manifests to
install Kubeflow in popular Kubernetes clusters

## charmed kubeflow

Canonical distributions of Kubeflow

```cmd

juju add-k8s mini
juju bootstrap mini
juju add-model kubeflow
juju deploy kubeflow --trust --channel=1.9/stable
juju deploy mlflow --channel=2.15/stable --trust
juju config dex-auth static-username=ospost
juju config dex-auth static-password=
juju deploy resource-dispatcher --channel 2.0/stable --trust

juju integrate mlflow-server:pod-defaults resource-dispatcher:pod-defaults
juju integrate mlflow-minio:object-storage kserve-controller:object-storage
juju integrate kserve-controller:service-accounts resource-dispatcher:service-accounts
juju integrate kserve-controller:secrets resource-dispatcher:secrets
juju integrate mlflow-server:ingress istio-pilot:ingress
juju integrate mlflow-server:dashboard-links kubeflow-dashboard:links

```

## Argo CD (Continuous Delivery)

Argo CD is an extremely widely-used tool that helps you programmatically manage
the applications deployed on a Kubernetes cluster. Argo CD is a declarative,
GitOps continuous delivery tool for Kubernetes.

Argo CD is implemented as a Kubernetes controller which continuously monitors
running applications and compares the current, live state against the desired
target state (as specified in the Git repo)

- Application
  The main config for Argo CD is the Application, a Kubernetes custom resource
  that specifies Kubernetes manifests for Argo CD to deploy and manage (typically
  from a git repository).

> Argo/Argo CD could play an very important role in development Cycle
> worth to put effort on it.

## deployKF

> compare to charmed kubeflow, deployDF document is much better

deployKF provides centralized configs, seamless in-place upgrades, and the
flexibility to run on any Kubernetes cluster, including self-hosted and cloud services.

deployKF requires ArgoCD for managing the platform. With integration with git.

```text

                               ┌────────────────────┐ @2 commit manifest to git
                               │   Git repository   │ ◄────────────────┐
                               └────────────────────┘                  │
                                 ▲                                     │
                                 │                                     │
                                 │ @3 get manifest from git            │
                                 │                                     │
     ┌───────────────────────────┼───┐                                 │
     │                  ┌────────┴┐  │                                 │
     │ k8s cluster      │ ArgoCD  │  │                                 │
     │                  └─────────┘  │                                 │
     └───────────────────────────────┘                                 │
                        ▲▲ ▲                                           │
        ┌───────────────┘│ │                 ┌──────────────┐          │
        │   ┌────────────┘ │                 │deployKF value│          │
        │   │              │                 └──────────────┘          │
        │   │              │                  │ @1 create manifests    │
        │   │              │                  │ from deployKF values   │
        │   │              │                  ▼                        │
        │   │              │                 ┌──────────────┐          │
                                             │ KF manifests │ ─────────┘
                                             └──────────────┘
        @4 retrieve the applications
           compoments described in the manifests
```

```cmd
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

kubectl port-forward svc/argocd-server -n argocd 8081:443
argocd admin initial-password -n argocd
DKF_CLI_VERSION="0.1.2"
DKF_CLI_ARCH=$(uname -m | sed -e 's/x86_64/amd64/')
DFK_CLI_DEST=/usr/local/bin/deploykf
sudo curl -fL "https://github.com/deploykf/cli/releases/download/v${DKF_CLI_VERSION}/deploykf-darwin-${DKF_CLI_ARCH}" -o "${DFK_CLI_DE
ST}"

git clone https://github.com/ospostme/kf.git
cd kf
curl -fL -o "sample-values-0.1.5.yaml"   "https://raw.githubusercontent.com/deployKF/deployKF/v0.1.5/sample-values.yaml"

deploykf generate     --source-version "0.1.5"     --values ./sample-values-0.1.5.yaml     --values ./custom-overrides.yaml     --outp
ut-dir ./GENERATOR_OUTPUT

kubectl apply --filename GENERATOR_OUTPUT/app-of-apps.yaml

```

## Harbor

Harbor is an open source registry, help to consistently and securely manage
artifacts across cloud native compute platforms like Kubernetes and Docker.

> Install Harbor on a Kubernetes cluster with helm is more much easy. To serve
> different targets, like docker desktop, k8s at anytime, choose standalone
> deployment.

### Harbor env

- Install harbor on KVM host. Docker compose used to control the Harbor
  containers. External storage and ssl configuration customized within config
  template.
- Get issued cert with acme.sh (Need domain).
- Add hosts records to mapping domain to local address
- create pull though cache for docker.io, quay.io, registry.k8s.io, ghcr.io,
  then any new pull request will add corresponding image into harbor for later
  usage.
- Customized containerd certs.d, mapping docker/query/k8s/ghcr registry to
  harbor local registry, then k8s pull request just as normal but served by proxy
  though cache.
- TDB create other kind of projects for those registries doesn't support pull
  though cache.

### containerd integration

```text

```

## kuik kube-image-keeper

kube-image-keeper is container image caching system for Kubernetes.

When a pod is created, kuik's mutating webhook rewrites its images on the fly to
point to the local caching registry, adding a localhost:{port}/ prefix (the port
is 7439 by default, and is configurable). This means that you don't need to
modify/rewrite the source registry url of your manifest/helm chart used to
deploy your solution, kuik will take care of it.

3 ways to tell kuik which pods it should manage

- label kube-image-keeper.enix.io/image-caching-policy=ignore
- Helm value controllers.webhook.ignoredNamespaces
- Helm value controllers.webhook.objectSelector.matchExpressions (match all by
  default)

> Easy install and make it fly with k8s cluster.  
> not suitable for k8s install
> cache ignore control is complicated.

## k9s

# High level summary of the steps to home lab env setup

- Determine suitable vGPU driver

- KVM cluster setup with Vagrant

- k8s cluster install with kubespary

- Adjust contained configuration and verify

- nfs driver/vGPU driver

- metallb address pool, enable load balancer

- charmed kubeflow/Mflow install with Juju

- Jupiter Lab customize for dive into deep learning
