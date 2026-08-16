# Lab 02 - Network Configuration and Connectivity

## Objective

Configure and verify IPv4 networking on a RHEL system using NetworkManager.

This lab focuses on:

- Inspecting the existing network configuration
- Managing NetworkManager connection profiles
- Configuring static IPv4 addressing
- Verifying routing information
- Testing connectivity between RHEL systems
- Verifying hostname resolution
- Confirming that network configuration persists after reboot

---

## Lab Environment

| System | Hostname | IPv4 Address | Role |
|---|---|---|---|
| Primary Server | `rhel-server01.example.com` | `10.10.10.20/24` | Primary system |
| Test Server | `rhel-tester01.example.com` | `10.10.10.21/24` | System being configured |

Both systems are connected to:

```text
Hyper-V Switch: LAB-SWITCH
Network:        10.10.10.0/24
```

The tasks are performed primarily on:

```text
rhel-tester01.example.com
```

---

## Scenario

`rhel-tester01` requires its network connection to be reviewed and configured
using NetworkManager.

As the system administrator, inspect the current network configuration,
identify the active NetworkManager connection profile, configure the required
IPv4 settings, and verify connectivity with `rhel-server01`.

The final configuration must survive a system reboot.

---

# Tasks

## Task 1 - Inspect the Current Network Configuration

Examine the network interfaces currently available on `rhel-tester01`.

### Questions

1. Which Ethernet interface is present?
2. Is the interface currently UP or DOWN?
3. What IPv4 address is assigned to it?
4. What subnet does the address belong to?
5. What IPv6 addresses, if any, are assigned?
6. Which routes are currently configured?

### My Findings

```text
Interface: eth0

State: UP

IPv4 address: 10.10.10.21.24

IPv4 network: 10.10.10.0/24

IPv6 address(es): fe80::215:5dff:fe01:b403/64

Routes: 10.10.10.0/24 dev eth0 proto kernel scope link src 10.10.10.21 metric 100

Observations:
eth0 is UP and has the expected IPv4 address of 10.10.10.21/24.
The interface belongs to the 10.10.10.0/24 lab network.

A link-local IPv6 address is also assigned to eth0.

The routing table contains a directly connected route to the 10.10.10.0/24 network through eth0.
```

---

## Task 2 - Inspect NetworkManager

Determine how NetworkManager is managing the Ethernet interface.

### Questions

1. Which NetworkManager connection profiles exist?
2. Which profile is currently active?
3. Which interface is associated with the active profile?
4. Is IPv4 configuration automatic or manual?
5. What static IPv4 address is configured?
6. Is a gateway configured?
7. Is a DNS server configured?

### My Findings

```text
Connection profile: eth0

Interface: eth0

IPv4 method: manual

IPv4 address: 10.10.10.21/24

Gateway: None

DNS: 172.16.0.2

Observations:
The eth0 connection uses manual IPv4 configuration with the expected 10.10.10.21/24 address.

No default gateway is configured.

The connection profile contains a DNS server address of 172.16.0.2.
This address belongs to the previous network configuration and does not match the current 10.10.10.0/24 lab network.
```

--- 

## Task 3 - Locate the Persistent Connection Profile

NetworkManager stores persistent connection profiles under:

```text
/etc/NetworkManager/system-connections/
```

Locate the profile associated with the active Ethernet connection.

### Questions

1. What is the name of the connection profile file?
2. Which interface does it apply to?
3. What IPv4 method is configured?
4. What IPv4 address is stored?
5. Does the configuration correspond with the settings reported by NetworkManager?

### My Findings

```text
Profile file: eth0.nmconnection

Interface: eth0

IPv4 method: manual

IPv4 address: 10.10.10.21/24

Observations:
The persistent Network profile contains the expected manual IPv4 configuration of 10.10.10.21/24.

The configuration stored in eth0.nmconnection matches the settings reported by NetworkManager.

The profile also contains a DNS server address of 172.16.0.2, which belongs to the previous network configuration.
```

---

## Task 4 - Configure Static IPv4 Networking

Configure `rhel-tester01` with the following network identity:

```text
Hostname: rhel-tester01.example.com
IPv4:    10.10.10.21/24
Network: 10.10.10.0/24
```

Use NetworkManager from the command line.

Do not manually edit the connection profile unless necessary.

### Questions

1. Which NetworkManager command can modify an existing connection?
   > nmcli connection modify
2. Which property controls the IPv4 configuration method?
   > ipv4.method
3. Which property controls the static IPv4 address?
   > ipv4.addresses
4. How do you apply changes made to an existing connection profile?
   > nmcli connection down eth0
   > nmcli connection up eth0

### Changes Made

```text
No changes were required. The existing eth0 NetworkManager profile
was already configured with the required manual IPv4 address of
10.10.10.21/24.

The NetworkManager properties and commands required to configure
the connection were identified and reviewed.
```

### Commands Used

```bash
nmcli connection show
nmcli connection show eth0
sudo cat /etc/NetworkManager/system-connections/eth0.nmconnection
```

---

## Task 5 - Verify the IPv4 Configuration

Verify the active network configuration after applying your changes.

### Questions

1. Is `eth0` UP?
2. Does it have `10.10.10.21/24`?
3. Does the address belong to the expected `10.10.10.0/24` network?
4. Does the kernel routing table contain a route for the lab network?
5. Does the active configuration match the NetworkManager profile?

### Verification

```text
Interface: eth0

State: UP

IPv4 address: 10.10.10.21/24

Network: 10.10.10.0/24 via eth0

Route: 10.10.10.0/24 dev eth0 proto kernel scope link src 10.10.10.21 metric 100

NetworkManager profile: eth0 - manual IPv4 configuration using 10.10.10.21/24

Result: IPv4 configuration verified successfully.
```

### Commands Used

```bash
ip addr show eth0
ip route
nmcli connection show eth0
```

---

## Task 6 - Test Network Connectivity

Test communication between `rhel-tester01` and `rhel-server01`.

Perform tests using:

- IPv4 address
- Short hostname
- FQDN

### Questions

1. Can `rhel-tester01` reach `10.10.10.20`?
2. Can it reach `rhel-server01`?
3. Can it reach `rhel-server01.example.com`?
4. Are there any differences between the three tests?
5. What do the results demonstrate?

### My Findings

```text
IPv4 connectivity: Successful - 10.10.10.20 responded with 0% packet loss.

Short hostname: Successful - rhel-server01 resolved to
rhel-server01.example.com (10.10.10.20) and responded with 0% packet loss.

FQDN: Successful - rhel-server01.example.com resolved to
10.10.10.20 and responded with 0% packet loss.

Observations:
Connectivity to rhel-server01 was successful using its IPv4 address,
short hostname, and FQDN.

The results confirm that network connectivity and local hostname
resolution are functioning correctly.
```

---

## Task 7 - Verify Name Resolution

Investigate how `rhel-tester01` resolves the name of `rhel-server01`.

### Questions

1. Does `rhel-server01` resolve to the expected IPv4 address?
2. Does `rhel-server01.example.com` resolve to the expected IPv4 address?
3. Which local configuration file currently provides these mappings?
   > /etc/hosts
4. What is the difference between local `/etc/hosts` resolution and DNS-based resolution?
   > /etc/hosts provides local hostname-to-IP mappings that must be maintained on each individual system. DNS provides centralized name resolution, allowing multiple systems to query a DNS server instead of maintaining the same mappings locally.

### My Findings

```text
Short hostname: rhel-server01 resolves successfully.

FQDN: rhel-server01.example.com resolves successfully.

Resolved IPv4 address: 10.10.10.20

Resolution source: /etc/hosts

Observations:
The short hostname and FQDN both resolve to 10.10.10.20 using
entries configured locally in /etc/hosts.
```

---

## Task 8 - Test SSH Connectivity

Verify that the primary server can be accessed remotely.

From `rhel-tester01`, establish an SSH connection to:

```text
rhel-server01
```

### Questions

1. Does the hostname resolve correctly?
2. Can an SSH connection be established?
3. Which user account was used?
4. How can you verify which system you are logged into after connecting?

### My Findings

```text
Destination: rhel-server01 (10.10.10.20)

User: hanlong

SSH connection: Successful

Remote hostname: rhel-server01.example.com

Result:
Successfully established an SSH connection from rhel-tester01 to
rhel-server01. The whoami and hostname commands confirmed that the
session was running as hanlong on rhel-server01.example.com.
```

### Commands Used

```bash
ssh rhel-server01
whoami
hostname
```

---

## Task 9 - Test Secure File Copy

Use a secure remote-copy mechanism to transfer a file between the two RHEL
systems.

Copy:

```text
/etc/hosts
```

from `rhel-server01` to:

```text
/tmp/
```

on `rhel-tester01`.

Do not modify the original file.

### Questions

1. Which command provides secure file transfer over SSH?
2. What syntax is used to specify a remote file?
3. Was the file successfully copied?
4. How can you verify that the copied file exists?
5. Who owns the copied file?

### My Findings

```text
Source: rhel-server01:/etc/hosts

Destination: /tmp/hosts on rhel-tester01

Transfer result: Successful

File ownership: hanlong:hanlong

Observations:
The /etc/hosts file was successfully copied from rhel-server01 to
/tmp/hosts on rhel-tester01 using SCP.

The copied file is owned by user hanlong and group hanlong.
```

### Commands Used

```bash
scp rhel-server01:/etc/hosts /tmp/
ls -l /tmp/hosts
```

---

## Task 10 - Verify Persistence

Reboot `rhel-tester01`.

After the reboot, verify that:

1. The Ethernet interface is UP.
2. `10.10.10.21/24` remains configured.
3. The `10.10.10.0/24` route remains present.
4. Local hostname resolution still works.
5. `rhel-server01` remains reachable.
6. SSH connectivity still works.

### Post-Reboot Verification

```text
Interface: eth0 - UP

IPv4 address: 10.10.10.21/24

Route: 10.10.10.0/24 route present through eth0

Hostname resolution: rhel-server01 successfully resolved to 10.10.10.20

Network connectivity: Successful - rhel-server01 responded to ping

SSH: Successful - remote login to rhel-server01 completed

Result: Network configuration and connectivity remained functional after reboot.
```

### Commands Used

```bash
ip addr show eth0
ip route
getent hosts rhel-server01
ping -c 4 rhel-server01
ssh rhel-server01
hostname
exit
```

---
