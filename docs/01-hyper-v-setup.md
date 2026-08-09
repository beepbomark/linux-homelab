# Hyper-V Linux Lab Setup

## Objective

Build a virtualized Linux environment using Hyper-V for hands-on Linux administration practice.

## Environment

| Component | Configuration |
|---|---|
| Host OS | Windows 11 |
| Hypervisor | Hyper-V |
| Host RAM | 32 GB |
| VM Name | ubuntu-server01 |
| Guest OS | Ubuntu Server 24.04 LTS |
| VM Generation | Generation 2 |
| vCPU | 2 |
| RAM | 2 GB Dynamic Memory |
| Virtual Disk | 40 GB VHDX |
| Network | Hyper-V Default Switch |

## VM Creation

Created a Generation 2 virtual machine named `ubuntu-server01`.

The Hyper-V Default Switch was selected to provide DHCP and Internet connectivity.

## Ubuntu Installation

Installed Ubuntu Server 24.04 LTS with:

- Hostname: `ubuntu-server01`
- OpenSSH Server enabled
- Standard Ubuntu Server installation
- Default storage configuration

## Initial Configuration

Updated package information:

```bash
sudo apt update
```

Upgraded installed packages:

```bash
sudo apt upgrade -y
```

Verified network connectivity:

```bash
ping -c 4 google.com
```

Verified system information

```bash
hostname
cat /etc/os-release
ip addr
df -h
free -h
lscpu
```

## Hyper-V Checkpoing

Created a checkpoint after completing the initial installation:

```
Fresh Ubuntu 24.04
```

This provides a known-good baseline that can be restored if later lab work causes configuration problems.

## Result

Ubuntu Server was successfully deployed on Hyper-V with working network and internet connectivity.

---