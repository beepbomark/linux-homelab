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

IPv4 method:

IPv4 address:

Gateway:

DNS:

Observations:
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
Profile file:

Interface:

IPv4 method:

IPv4 address:

Observations:
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
2. Which property controls the IPv4 configuration method?
3. Which property controls the static IPv4 address?
4. How do you apply changes made to an existing connection profile?

### Changes Made

```text
Record the configuration changes here.
```

### Commands Used

```bash
# Record commands here.
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
Interface:

State:

IPv4 address:

Network:

Route:

NetworkManager profile:

Result:
```

### Commands Used

```bash
# Record verification commands here.
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
IPv4 connectivity:

Short hostname:

FQDN:

Observations:
```

---

## Task 7 - Verify Name Resolution

Investigate how `rhel-tester01` resolves the name of `rhel-server01`.

### Questions

1. Does `rhel-server01` resolve to the expected IPv4 address?
2. Does `rhel-server01.example.com` resolve to the expected IPv4 address?
3. Which local configuration file currently provides these mappings?
4. What is the difference between local `/etc/hosts` resolution and DNS-based resolution?

### My Findings

```text
Short hostname:

FQDN:

Resolved IPv4 address:

Resolution source:

Observations:
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
Destination:

User:

SSH connection:

Remote hostname:

Result:
```

### Commands Used

```bash
# Record commands here.
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
Source:

Destination:

Transfer result:

File ownership:

Observations:
```

### Commands Used

```bash
# Record commands here.
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
Interface:

IPv4 address:

Route:

Hostname resolution:

Network connectivity:

SSH:

Result:
```

### Commands Used

```bash
# Record verification commands here.
```

---

# Lab Result

**Status:** Not Started

## Configuration Summary

```text
Complete after finishing the lab.
```

## Verification Summary

```text
Complete after finishing the lab.
```

## What I Learned

```text
Complete after finishing the lab.
```

## Commands Practised

```bash
# Add important commands after completing the lab.
```