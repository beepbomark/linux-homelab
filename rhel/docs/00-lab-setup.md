# RHEL Homelab Setup

## Overview

This document covers the setup of a two-server Red Hat Enterprise Linux 9
homelab running on Microsoft Hyper-V.

The environment provides a dedicated platform for hands-on RHEL system
administration practice, including networking, package management, storage,
services, security, and troubleshooting.

---

## Lab Architecture

| Virtual Machine | Hostname | IPv4 Address | Purpose |
|---|---|---|---|
| `rhel-server01` | `rhel-server01.example.com` | `172.16.0.100/24` | Primary RHEL server |
| `rhel-tester01` | `rhel-tester01.example.com` | `172.16.0.50/24` | Secondary/test server |

### Platform

- Host OS: Windows 11
- Hypervisor: Microsoft Hyper-V
- Guest OS: Red Hat Enterprise Linux 9
- Network interface: `eth0`
- Local network: `172.16.0.0/24`

---

## 1. Primary Server

The first RHEL virtual machine was configured as:

```text
Hostname: rhel-server01.example.com
IPv4:    172.16.0.100/24
RAM:     4 GB
Disk:    40 GB
```

### Disk Layout

The system uses separate boot partitions and LVM for the main filesystems.

| Mount Point | Size | Type |
|---|---:|---|
| `/boot/efi` | 600 MiB | Standard partition |
| `/boot` | 1 GiB | Standard partition |
| `/` | 15 GiB | LVM |
| `/home` | 1 GiB | LVM |
| `swap` | 1 GiB | LVM |

Unused disk capacity was intentionally retained for future storage,
partitioning, and LVM exercises.

### Verification

```bash
hostnamectl
lsblk
free -h
ip addr
```

---

## 2. Secondary Server

A second RHEL server was created by exporting the primary VM and importing it
into Hyper-V using:

> Copy the virtual machine (create a new unique ID)

The cloned VM was renamed in Hyper-V to:

```text
rhel-tester01
```

### Change Hostname

The cloned system initially retained the identity of the primary server.

The hostname was changed with:

```bash
sudo hostnamectl set-hostname rhel-tester01.example.com
```

Verify:

```bash
hostnamectl
```

### Change IP Address

Identify the NetworkManager connection:

```bash
nmcli connection show
```

The `eth0` connection was assigned a new static IPv4 address:

```bash
sudo nmcli connection modify eth0 ipv4.addresses 172.16.0.50/24
sudo nmcli connection down eth0
sudo nmcli connection up eth0
```

Verify:

```bash
ip addr show eth0
```

Expected address:

```text
inet 172.16.0.50/24
```

---

## 3. Local Hostname Resolution

Local hostname resolution was configured using `/etc/hosts`.

On both servers:

```bash
sudo vi /etc/hosts
```

The following entries were added while retaining the existing localhost
entries:

```text
172.16.0.100   rhel-server01.example.com   rhel-server01
172.16.0.50    rhel-tester01.example.com   rhel-tester01
```

### Test Connectivity

From `rhel-server01`:

```bash
ping -c 4 rhel-tester01
```

From `rhel-tester01`:

```bash
ping -c 4 rhel-server01
```

Both servers successfully resolved the hostnames and communicated over the
local network.

---

## 4. SSH Connectivity

SSH connectivity was tested in both directions.

From `rhel-server01`:

```bash
ssh hanlong@rhel-tester01
```

From `rhel-tester01`:

```bash
ssh hanlong@rhel-server01
```

This confirmed that both systems could be administered remotely over the
lab network.

---

## 5. Local DNF Repositories

The RHEL installation DVD contains the `BaseOS` and `AppStream` repositories.

The ISO was attached to each VM using the Hyper-V virtual DVD drive.

### Create Mount Point

```bash
sudo mkdir -p /mnt/rhel9
```

Mount the virtual DVD:

```bash
sudo mount /dev/sr0 /mnt/rhel9
```

Verify:

```bash
ls /mnt/rhel9
```

The installation media contains:

```text
AppStream
BaseOS
EFI
images
isolinux
...
```

### Configure DNF

Create:

```bash
sudo vi /etc/yum.repos.d/rhel9-local.repo
```

Add:

```ini
[BaseOS]
name=RHEL 9 BaseOS
baseurl=file:///mnt/rhel9/BaseOS
enabled=1
gpgcheck=0

[AppStream]
name=RHEL 9 AppStream
baseurl=file:///mnt/rhel9/AppStream
enabled=1
gpgcheck=0
```

Refresh the DNF cache:

```bash
sudo dnf clean all
sudo dnf makecache
sudo dnf repolist
```

Both repositories should be available:

```text
RHEL 9 BaseOS
RHEL 9 AppStream
```

> `gpgcheck=0` is used for this isolated lab repository configuration.
> Package signature verification should normally remain enabled on
> production systems.

---

## 6. Persistent Installation Media Mount

Because the local DNF repositories depend on the RHEL installation media,
the virtual DVD was configured to mount automatically.

The following entry was added to `/etc/fstab`:

```text
/dev/sr0   /mnt/rhel9   iso9660   ro,nofail   0 0
```

The configuration was tested before rebooting:

```bash
sudo umount /mnt/rhel9
sudo systemctl daemon-reload
sudo mount -a
```

Verify:

```bash
findmnt /mnt/rhel9
```

Expected result:

```text
TARGET      SOURCE    FSTYPE
/mnt/rhel9  /dev/sr0  iso9660
```

Verify that the repositories remain accessible:

```bash
sudo dnf makecache
```

---

## 7. Final Environment

The completed RHEL homelab consists of:

```text
Windows 11
└── Microsoft Hyper-V
    │
    ├── rhel-server01.example.com
    │   └── 172.16.0.100/24
    │
    └── rhel-tester01.example.com
        └── 172.16.0.50/24
```

The two systems can:

- Communicate over the local network
- Resolve each other using hostnames
- Connect to each other using SSH
- Access local RHEL BaseOS and AppStream repositories
- Automatically mount the RHEL installation media

---

## Troubleshooting

### Duplicate VHDX During VM Import

During the initial VM import, Hyper-V attempted to place the cloned VHDX in
the same directory as the original virtual disk.

This caused a filename conflict because:

```text
rhel-server01.vhdx
```

already existed.

The issue was resolved by storing the cloned virtual disk in a separate
directory.

### Incorrect Local Repository URL

The local repository initially failed because of an incorrect `file://` URL.

Incorrect:

```text
file://mnt/rhel9/BaseOS
```

Correct:

```text
file:///mnt/rhel9/BaseOS
```

The third `/` represents the beginning of the absolute Linux filesystem path.

### DVD Automatically Mounted by Desktop Environment

The RHEL installation media may be automatically mounted under:

```text
/run/media/<user>/
```

For consistent repository access, the DVD is instead mounted at:

```text
/mnt/rhel9
```

using `/etc/fstab`.

### RHEL Subscription Warning

DNF may display:

```text
This system is not registered with an entitlement server.
```

The lab uses repositories from locally mounted RHEL installation media, so
the local BaseOS and AppStream repositories can function independently of
online repositories.

---

## Skills Practised

- Microsoft Hyper-V VM cloning
- RHEL installation and configuration
- Linux hostname management
- NetworkManager configuration with `nmcli`
- Static IPv4 addressing
- `/etc/hosts` hostname resolution
- SSH connectivity
- ISO9660 filesystem mounting
- Persistent mounts with `/etc/fstab`
- DNF repository configuration
- RPM/DNF package management
- Linux troubleshooting