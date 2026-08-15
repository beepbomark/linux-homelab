# Lab 01 - Network Troubleshooting

## Objective

Diagnose and repair an invalid IPv4 network configuration on a RHEL system.

This lab focuses on:

- Testing network connectivity
- Inspecting interface configuration
- Examining NetworkManager connection profiles
- Identifying incorrect IPv4 settings
- Restoring network connectivity
- Verifying persistent configuration

---

## Lab Environment

| System | Hostname | IPv4 Address | Role |
|---|---|---|---|
| Primary Server | `rhel-server01.example.com` | `10.10.10.20/24` | Known-good remote system |
| Test Server | `rhel-tester01.example.com` | `10.10.10.21/24` | System under investigation |

The troubleshooting tasks are performed primarily on:

```text
rhel-tester01.example.com
```

The systems are connected through the Hyper-V `LAB-SWITCH` on:

```text
10.10.10.0/24
```

---

## Scenario

`rhel-tester01` previously had working network connectivity.

Its network configuration has been altered and the server can no longer
communicate correctly with other systems on the lab network.

As the system administrator, diagnose the networking problem and restore the
server to its original working configuration.

Do not reinstall the operating system or recreate the virtual machine.

---

# Tasks

## Task 1 - Test Network Connectivity

Test connectivity between `rhel-tester01` and the known-good
`rhel-server01`.

Test connectivity using:

- Short hostname
- FQDN
- IPv4 address

### Questions

1. Can `rhel-tester01` reach `rhel-server01`?
2. Can `rhel-tester01` reach `rhel-server01.example.com`?
3. Can it reach `10.10.10.20` directly?
4. Are the results different when using a hostname versus an IPv4 address?
5. What do the results suggest about the problem?

### My Findings

```text
Short hostname: Failed - Network is unreachable

FQDN: Failed - Network is unreachable

IPv4 address: Failed - Network is unreacble

Observations:
Both the short hostname and FQDN successfully resolved to 10.10.10.20, but the destination network could not be reached.

Direct communication using the IPv4 address also failed, indicating that the problem is related to network connectivity rather than hostname resolution.
```

---

## Task 2 - Inspect the Network Interface

Examine the current network interfaces and their IPv4 configuration.

### Questions

1. Which network interface is being used?
2. Is the interface UP or DOWN?
3. What IPv4 address is currently assigned?
4. What subnet is the interface connected to?
5. Does the IPv4 configuration match the expected `10.10.10.0/24` lab network?
6. Are there any obvious abnormalities in the interface configuration?

### My Findings

```text
Interface: eth0

State: UP

IPv4 address: 192.168.50.21/24

Subnet: 192.168.50.0/24

Observations: 
The eth0 interface is UP, but its IPv4 address belongs to the 192.168.50.0/24 network. The expected lab network is 10.10.10.0/24

This indicates that the interface has an incorrect IPv4 configuration.
```

---

## Task 3 - Inspect NetworkManager Configuration

Review the persistent NetworkManager connection configuration.

NetworkManager connection profiles are stored under:

```text
/etc/NetworkManager/system-connections/
```

### Questions

1. Which NetworkManager connection profile belongs to the active interface?
2. What IPv4 address is configured in the profile?
3. What IPv4 configuration method is being used?
4. Is a default gateway configured?
5. Does the persistent configuration match the currently active configuration?
6. Do you notice any incorrect or missing settings?

### My Findings

```text
Connection profile: eth0

IPv4 method: manual

Configured IPv4 address: 192.169.50.21/24

Configured gateway: None

Observations:
The persistent NetworkManager profile contains the incorrect 192.168.50.21/24 address.

The expected lab network is 10.10.10.0/24, so the saved configuration does not match the intended network.

No default gateway is configured.
```

---

## Task 4 - Identify the Root Cause

Based on the evidence collected in Tasks 1-3, determine why networking is
not functioning correctly.

Do not repair the configuration until you can explain the fault.

### Root Cause

```text
rhel-tester01 is configured with an incorrect static IPv4 address.  
It's eth0 connection profile uses 192.168.50.21/24 instead of the expected 10.10.10.21/24.

Because the interface is on the wrong subnet, it cannot communicate with rhel-server01 on the 10.10.10.0/24 lab network.
```

---

## Task 5 - Repair the Configuration

Restore the correct network configuration.

The expected network identity of the test server is:

```text
Hostname: rhel-tester01.example.com
IPv4:    10.10.10.21/24
Network: 10.10.10.0/24
```

Restore the NetworkManager connection profile so that the server can
communicate correctly with the other systems on the lab network.

### Changes Made

```text
Corrected the static IPv4 address assigned to the eth0 NetworkManager connection profile from 192.168.50.21/24 to 10.10.10.21/24.

The connection was then deactivated and reactivated to apply the corrected network configuration
```

### Commands Used

```bash
nmcli connection modify eth0 ipv4.addresses 10.10.10.21/24
nmcli connection down eth0
nmcli connection up eth0
```

---

## Task 6 - Verify the Repair

After repairing the configuration, verify that:

- The network interface is UP.
- The interface has the correct IPv4 address.
- The expected network route exists.
- `rhel-server01` is reachable by IPv4 address.
- `rhel-server01` is reachable by short hostname.
- `rhel-server01` is reachable by FQDN.
- SSH connectivity works.

### Verification

```text
Interface: UP

IPv4 address: 10.10.10.21/24

Ping by IPv4: Successful

Ping by hostname: Successful

Ping by FQDN: Successful

SSH: Successful

Result:  
Network connectivity successfully restored.
```

### Commands Used

```bash
ip addr show eth0
ip route
ping -c 4 10.10.10.20
ping -c 4 rhel-server01
ping -c 4 rhel-server01.example.com
ssh hanlong@rhel-server01
```

---

## Task 7 - Verify Persistence

Reboot `rhel-tester01`.

After rebooting, verify that:

1. The network interface returns to the UP state.
2. The correct `10.10.10.21/24` IPv4 address remains configured.
3. The expected network route remains present.
4. Hostname resolution still works.
5. `rhel-server01` remains reachable.
6. SSH connectivity still works.

### Post-Reboot Verification

```text
IPv4 address: 10.10.10.21/24 remained configured after reboot.

Route: The 10.10.10.0/24 network route remained present.

Hostname resolution: rhel-server01 and rhel-server01.example.com resolved successfully.

Network connectivity: Successful.

SSH: Successful.

Result: The repaired network configuration persisted after reboot.
```

---

# Lab Result

## Root Cause

```text
The eth0 NetworkManager connection profile was configured with the incorrect
static IPv4 address 192.168.50.21/24.

The expected lab network was 10.10.10.0/24, with rhel-tester01 expected to
use 10.10.10.21/24.

Although the interface was UP, the incorrect IPv4 address placed the system
on the wrong subnet and prevented communication with rhel-server01.
```

## Resolution

```text
The static IPv4 address in the eth0 NetworkManager connection profile was
changed from 192.168.50.21/24 to 10.10.10.21/24.

The connection was deactivated and reactivated to apply the corrected
configuration. Connectivity was verified and the system was rebooted to
confirm that the configuration persisted.
```

## What I Learned

```text
An interface being UP does not necessarily mean that network connectivity is
working.

I learned to troubleshoot the problem systematically by first testing
connectivity, then inspecting the active interface configuration, and finally
checking the persistent NetworkManager connection profile.

I also learned that a system configured with an IPv4 address from the wrong
subnet cannot directly communicate with systems on the intended local subnet.

After modifying a NetworkManager connection profile, the connection can be
deactivated and reactivated to apply the new configuration. Reboot testing
can then confirm that the repair is persistent.
```

## Commands Practised

```bash
ping -c 4 <destination>

ip addr show
ip addr show eth0
ip route

nmcli connection show
nmcli connection show eth0
nmcli -f ipv4.method,ipv4.addresses,ipv4.gateway connection show eth0

nmcli connection modify eth0 ipv4.addresses 10.10.10.21/24
nmcli connection down eth0
nmcli connection up eth0

ssh hanlong@rhel-server01

sudo reboot
```