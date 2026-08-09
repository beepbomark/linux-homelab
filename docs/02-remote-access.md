# Remote Access Setup

## Objective

Configure reliable remote administration of `ubuntu-server01` from the Windows 11 host using SSH and Visual Studio Code Remote - SSH.

The server was initially connected through the Hyper-V Default Switch using DHCP. A dedicated Hyper-V lab network was later configured to provide predictable IP addressing and persistent remote access.

---

## Environment

| Component | Configuration |
|---|---|
| Client | Windows 11 Host |
| Server | `ubuntu-server01` |
| Server OS | Ubuntu Server 24.04.4 LTS |
| Linux User | `hanlong` |
| Hyper-V Switch | `LAB-SWITCH` |
| Lab Network | `10.10.10.0/24` |
| Windows Gateway | `10.10.10.1` |
| Ubuntu Server | `10.10.10.11` |
| Remote Protocol | SSH |
| Remote Editor | Visual Studio Code Remote - SSH |

---

## Initial SSH Setup

OpenSSH Server was installed during the Ubuntu Server installation.

The SSH service was verified using:

```bash
systemctl status ssh
```

The service showed:

```text
Active: active (running)
```

The initial IP address was identified using:

```bash
ip addr
```

A more concise view can be obtained with:

```bash
ip -br addr
```

The server was initially accessed from Windows using:

```powershell
ssh hanlong@<server-ip>
```

After authentication, the remote shell displayed:

```text
hanlong@ubuntu-server01:~$
```

This confirmed that SSH remote administration was working.

---

## Problem: DHCP Address Changed

The VM initially used the Hyper-V Default Switch.

The Default Switch automatically assigned the Ubuntu VM an IPv4 address using DHCP.

After restarting the environment, the server received a different IP address.

As a result, existing SSH and Visual Studio Code connections that referenced the previous address no longer worked.

### Observation

The server address could be checked using:

```bash
ip -br addr
```

However, manually updating the SSH configuration whenever the address changed was not a suitable long-term solution.

---

# Dedicated Hyper-V Lab Network

A dedicated Hyper-V Internal virtual switch was created:

```text
LAB-SWITCH
```

An Internal switch was selected so that:

- The Windows host can communicate with the virtual machines
- Virtual machines can communicate with each other
- The lab network remains separate from the physical network

The resulting design is:

```text
                    Internet
                       |
                 Windows 11
                       |
                      NAT
                       |
                  10.10.10.1
                       |
                  LAB-SWITCH
                 10.10.10.0/24
                       |
                ubuntu-server01
                  10.10.10.11
```

---

## Configure Windows Lab Interface

The Windows virtual network adapter was assigned:

```text
10.10.10.1/24
```

The configuration was verified using:

```powershell
Get-NetIPAddress `
    -InterfaceAlias "vEthernet (LAB-SWITCH)" `
    -AddressFamily IPv4
```

The adapter status was checked using:

```powershell
Get-NetAdapter -Name "vEthernet (LAB-SWITCH)" |
    Format-Table Name, Status, LinkSpeed
```

The adapter showed:

```text
Status: Up
```

---

## Configure NAT

Because a Hyper-V Internal switch does not automatically provide Internet access, Windows NAT was configured for the lab network.

```powershell
New-NetNat `
    -Name "LAB-NAT" `
    -InternalIPInterfaceAddressPrefix "10.10.10.0/24"
```

The NAT configuration can be verified using:

```powershell
Get-NetNat
```

Windows now acts as the gateway between the private lab network and external networks.

---

# Configure Static IP on Ubuntu

The Ubuntu VM network adapter was moved from the Hyper-V Default Switch to:

```text
LAB-SWITCH
```

Because the new lab network does not provide DHCP, a static address was configured using Netplan.

## Netplan Configuration

The existing Netplan configuration was backed up before modification.

Example:

```bash
sudo cp /etc/netplan/50-cloud-init.yaml \
/etc/netplan/50-cloud-init.yaml.bak
```

The Netplan configuration was then changed to:

```yaml
network:
  version: 2
  ethernets:
    eth0:
      dhcp4: false
      addresses:
        - 10.10.10.11/24
      routes:
        - to: default
          via: 10.10.10.1
      nameservers:
        addresses:
          - 1.1.1.1
          - 8.8.8.8
```

The configuration was validated using:

```bash
sudo netplan generate
```

It was then tested safely using:

```bash
sudo netplan try
```

After confirming the configuration, the server showed:

```text
eth0    UP    10.10.10.11/24
```

---

## Verify Routing

The routing table was checked using:

```bash
ip route
```

The server showed a default route through the Windows host:

```text
default via 10.10.10.1 dev eth0
```

and a route for the local lab network:

```text
10.10.10.0/24 dev eth0
```

---

# Connectivity Testing

Connectivity was tested one layer at a time.

## Windows to Ubuntu

From Windows:

```powershell
ping 10.10.10.11
```

SSH connectivity was tested specifically using:

```powershell
Test-NetConnection 10.10.10.11 -Port 22
```

The test returned:

```text
TcpTestSucceeded : True
```

This confirmed that the Windows host could reach the Ubuntu SSH service.

## Ubuntu to Internet

Internet routing was tested without relying on DNS:

```bash
ping -c 4 1.1.1.1
```

The test succeeded.

DNS resolution was then tested:

```bash
ping -c 4 google.com
```

This also succeeded.

The results confirmed that:

```text
Ubuntu
  |
  +-- LAB-SWITCH
  |
  +-- Windows NAT
  |
  +-- Internet
```

was functioning correctly.

---

# Configure SSH Alias

To avoid specifying the IP address manually, the Windows OpenSSH configuration was configured with an alias.

SSH configuration:

```text
Host ubuntu-server01
    HostName 10.10.10.11
    User hanlong
```

The server can now be accessed using:

```powershell
ssh ubuntu-server01
```

instead of:

```powershell
ssh hanlong@10.10.10.11
```

---

## SSH Configuration Issue

The SSH alias initially failed with:

```text
Could not resolve hostname ubuntu-server01:
No such host is known.
```

Direct access by IP still worked:

```powershell
ssh hanlong@10.10.10.11
```

This showed that the Linux server and network were functioning correctly and narrowed the problem to the Windows SSH client configuration.

The active Windows profile was identified using:

```cmd
echo %USERPROFILE%
```

The SSH configuration needed to exist under:

```text
%USERPROFILE%\.ssh\config
```

The file had initially been saved as:

```text
config.txt
```

and was renamed to:

```text
config
```

---

## SSH Configuration Permissions

After correcting the filename, OpenSSH returned:

```text
Bad owner or permissions on ...\.ssh\config
```

The Windows ACL on the SSH configuration file was corrected.

Inheritance was removed:

```cmd
icacls "%USERPROFILE%\.ssh\config" /inheritance:r
```

Ownership was corrected where required:

```cmd
takeown /F "%USERPROFILE%\.ssh\config"
```

The active Windows user was granted access:

```cmd
icacls "%USERPROFILE%\.ssh\config" /grant:r "%USERNAME%:F"
```

The resulting SSH configuration was verified using:

```cmd
ssh -G ubuntu-server01
```

The resolved configuration correctly showed:

```text
hostname 10.10.10.11
user hanlong
```

The alias then worked successfully:

```powershell
ssh ubuntu-server01
```

---

# Visual Studio Code Remote - SSH

The Microsoft Remote - SSH extension was installed in Visual Studio Code.

VS Code was configured to use:

```text
Host ubuntu-server01
    HostName 10.10.10.11
    User hanlong
```

After connecting, Visual Studio Code installed its remote server component under:

```text
~/.vscode-server
```

The Linux filesystem can now be browsed and edited directly from Windows.

The integrated terminal runs on the remote Ubuntu server and displays:

```text
hanlong@ubuntu-server01:~$
```

---

# Final Remote Administration Workflow

The completed remote administration environment is:

```text
Windows 11
    |
    +-- Windows Terminal
    |       |
    |       +-- SSH
    |
    +-- Visual Studio Code
            |
            +-- Remote - SSH
                    |
                    v
             ubuntu-server01
               10.10.10.11
```

The Hyper-V console is now mainly used for situations where normal remote access is unavailable, including:

- Network configuration problems
- SSH failures
- Boot failures
- System recovery

---

## Result

`ubuntu-server01` now has a persistent network configuration:

```text
Hostname: ubuntu-server01
IP:       10.10.10.11/24
Gateway:  10.10.10.1
Network:  10.10.10.0/24
```

Remote administration is available through:

```powershell
ssh ubuntu-server01
```

and Visual Studio Code Remote - SSH.

The server also retains Internet connectivity through Windows NAT.

The original DHCP address-change problem has therefore been resolved by replacing the Hyper-V Default Switch with a dedicated lab network and static server addressing.