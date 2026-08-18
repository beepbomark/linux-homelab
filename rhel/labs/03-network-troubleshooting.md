# Lab 03 - DNS Client Troubleshooting

## Objective

Diagnose and repair a DNS client configuration problem on a RHEL system.

This lab focuses on:

- Distinguishing IP connectivity from DNS name resolution
- Testing DNS resolution independently of network connectivity
- Inspecting `/etc/resolv.conf`
- Inspecting NetworkManager DNS configuration
- Identifying incorrect DNS client settings
- Repairing DNS configuration with `nmcli`
- Verifying DNS queries
- Confirming that the repair persists after reboot

---

## Prerequisite

This lab uses the DNS infrastructure configured in:

```text
dns-server-setup.md
```

Unlike Labs 01 and 02, the homelab now includes a functioning BIND DNS server.

---

## Lab Environment

| System | Hostname | IPv4 Address | Role |
|---|---|---|---|
| Primary Server | `rhel-server01.example.com` | `10.10.10.20/24` | BIND DNS server |
| Test Server | `rhel-tester01.example.com` | `10.10.10.21/24` | DNS client under investigation |

### Network

```text
Hyper-V Switch: LAB-SWITCH
Network:        10.10.10.0/24
DNS Server:     10.10.10.20
DNS Zone:       example.com
```

### DNS Records

The `example.com` zone contains:

```text
rhel-server01.example.com → 10.10.10.20
rhel-tester01.example.com → 10.10.10.21
```

The troubleshooting tasks are performed primarily on:

```text
rhel-tester01.example.com
```

---

## Known-Good State

Before the fault is introduced, `rhel-tester01` should have:

```text
IPv4 address: 10.10.10.21/24
DNS server:   10.10.10.20
```

The following DNS query should succeed:

```bash
dig rhel-server01.example.com
```

The expected DNS server is:

```text
SERVER: 10.10.10.20#53
```

---

# Scenario

`rhel-tester01` previously had working network connectivity and DNS name
resolution.

Users now report that network resources can still be reached using IP
addresses, but DNS-based access is not functioning correctly.

As the system administrator, determine whether the underlying network is
working, investigate the system's DNS client configuration, identify the
root cause, and restore name resolution.

Do not modify the DNS server unless evidence indicates that the server is
responsible for the problem.

---

# Tasks

## Task 1 - Verify Basic Network Configuration

Before investigating DNS, determine whether the underlying IPv4 configuration
is functioning correctly.

### Questions

1. Which Ethernet interface is being used?
2. Is the interface UP or DOWN?
3. What IPv4 address is currently assigned?
4. Does the address belong to the expected `10.10.10.0/24` network?
5. Does the expected local network route exist?

### My Findings

```text
Interface: eth0

State: UP

IPv4 address: 10.10.10.21/24

Network: 10.10.10.0/24

Route: 10.10.10.0/24 dev eth0 proto kernel scope link src 10.10.10.21 metric 100

Observations:
eth0 is UP and has the expected 10.10.10.21/24 address.
A route to the local 10.10.10.0/24 network is present.
No obvious problem has been identified with the interface,
IPv4 address, or local network route.
```

---

## Task 2 - Test IP Connectivity to the DNS Server

Test whether `rhel-tester01` can communicate directly with the DNS server
using its IPv4 address.

The DNS server is:

```text
10.10.10.20
```

### Questions

1. Can `10.10.10.20` be reached directly?
   > Yes
2. Is there packet loss?
   > No
3. What does this tell you about basic network connectivity?
   > Basic network connectivity is working because rhel-tester01 can successfully reach the DNS server at 10.10.10.20 using its IPv4 address with no packet loss.
4. If this test succeeds, should you immediately assume that DNS is working?
   > No. A successful ping to an IPv4 address confirms IP connectivity but does not verify DNS name resolution because no hostname needs to be resolved.

### My Findings

```text
Destination: 10.10.10.20

Connectivity: Successful

Packet loss: 0%

Observations:
rhel-tester01 can successfully reach the DNS server at 10.10.10.20
using its IPv4 address. This indicates that basic IP connectivity
between the two systems is functioning.
```

---

## Task 3 - Test Normal Name Resolution

Test DNS resolution using the system's normal resolver configuration.

Use the DNS-only test record:

```text
dns-test.example.com
```

This hostname exists in the `example.com` DNS zone but is not configured
in `/etc/hosts`.

### Questions

1. Can `dns-test.example.com` be resolved?
2. Does the query return `10.10.10.20`?
3. Which DNS server attempts to answer the query?
4. How does this result compare with the direct IPv4 test from Task 2?
5. What does the combination of these results suggest?

### My Findings

```text
Hostname: dns-test.example.com

Resolution result: Failed / timed out waiting for a response

Resolved address: None

DNS server used: Not yet determined

Observations:
Direct IPv4 connectivity to 10.10.10.20 works, but normal DNS
resolution for dns-test.example.com does not return a response.

This suggests that basic network connectivity is functioning, but
there may be a problem with name resolution.
```

---

## Task 4 - Query the DNS Server Directly

Determine whether the DNS server itself is functioning independently of the
client's configured resolver.

Query:

```text
10.10.10.20
```

directly for:

```text
rhel-server01.example.com
```

### Questions

1. Does the DNS server respond when queried directly?
2. What DNS response status is returned?
3. Is an answer returned?
4. What IPv4 address is contained in the answer?
5. What does this tell you about the BIND server?
6. If a direct DNS query succeeds but a normal DNS query fails, where is the
   problem most likely located?

### My Findings

```text
DNS server queried: 10.10.10.20

Query result: Successful

Status: NOERROR

Answer: dns-test.example.com → 10.10.10.20

Observations:
The DNS server responds correctly when queried directly.

This indicates that BIND on rhel-server01 is functioning and that
rhel-tester01 can reach DNS service on port 53.

Because the direct DNS query succeeds while the normal DNS query fails,
the problem is most likely with the DNS client configuration on
rhel-tester01 rather than with the DNS server or basic network connectivity.
```

---

## Task 5 - Inspect `/etc/resolv.conf`

Inspect the resolver configuration currently being used by
`rhel-tester01`.

### Questions

1. Which nameserver is listed?
2. Is the expected `10.10.10.20` DNS server present?
3. Is a search domain configured?
4. Does anything appear missing or incorrect?
5. Does the configuration explain the results observed in Tasks 3 and 4?

### My Findings

```text
Nameserver: None configured

Search domain: example.com

Expected DNS server present: No

Observations:
The resolver configuration contains the example.com search domain,
but no nameserver is configured.

The expected DNS server 10.10.10.20 is missing from /etc/resolv.conf.

This is consistent with the earlier observation that querying
10.10.10.20 directly succeeds while normal DNS resolution fails.
```

---

## Task 6 - Inspect NetworkManager DNS Configuration

Inspect the active `eth0` NetworkManager connection profile.

### Questions

1. What is the IPv4 configuration method?
2. What IPv4 address is configured?
3. What value is configured for `ipv4.dns`?
4. What DNS server appears in the active IP configuration?
5. Does the NetworkManager configuration agree with `/etc/resolv.conf`?
6. Does the DNS configuration match the expected homelab configuration?

### My Findings

```text
Connection profile: eth0

IPv4 method: manual

IPv4 address: 10.10.10.21/24

Configured DNS: None (--)

Active DNS: None

Observations:
The eth0 connection profile uses manual IPv4 configuration and has the
correct 10.10.10.21/24 address.

However, no IPv4 DNS server is configured in the NetworkManager profile,
and there is no active IP4.DNS entry.

The example.com search domain remains configured.

This agrees with /etc/resolv.conf, which contains the search domain but
does not contain a nameserver.
```

---

## Task 7 - Inspect the Persistent Configuration

Inspect the NetworkManager profile stored under:

```text
/etc/NetworkManager/system-connections/
```

Determine whether the incorrect DNS configuration is stored persistently.

### Questions

1. Which `.nmconnection` file belongs to `eth0`?
2. What DNS setting is stored in the profile?
3. Does it match the active NetworkManager configuration?
4. Would the DNS problem survive a reboot?

### My Findings

```text
Profile file: eth0.nmconnection

Stored DNS: None

Matches active configuration: Yes

Persistent after reboot: Yes

Observations:
The persistent eth0 NetworkManager profile contains the correct
10.10.10.21/24 IPv4 address and the example.com DNS search domain,
but no DNS server is configured.

This matches the active NetworkManager configuration and
/etc/resolv.conf.

Because the DNS server is missing from the persistent NetworkManager
profile, the DNS problem would remain after a reboot.
```

---

## Task 8 - Identify the Root Cause

Based on the evidence collected in Tasks 1-7, determine the root cause.

Before repairing anything, you should be able to explain:

- Whether IPv4 networking works
  > IPv4 networking is working because eth0 has the correct IP address, the local route exists, and 10.10.10.20 is reachable.
- Whether the DNS server can be reached
  > The DNS server is working because querying 10.10.10.20 directly successfully resolves dns-test.example.com.
- Whether the DNS server itself answers queries
- > Normal DNS resolution is not working because a normal query for dns-test.example.com fails.
- Whether the client is using the correct DNS server
  > The DNS client configuration is incorrect because no DNS server is configured in NetworkManager, /etc/resolv.conf, or the persistent eth0 connection profile.

### Root Cause

```text
rhel-tester01 cannot perform normal DNS resolution because no DNS server
is configured in NetworkManager, /etc/resolv.conf, or the persistent
eth0 connection profile.
```

### Evidence

```text
- eth0 is UP with the correct 10.10.10.21/24 address.
- The 10.10.10.0/24 route is present.
- rhel-server01 at 10.10.10.20 is reachable by IPv4.
- A direct DNS query to 10.10.10.20 successfully resolves
  dns-test.example.com.
- A normal DNS query for dns-test.example.com fails.
- /etc/resolv.conf contains no nameserver.
- NetworkManager has no ipv4.dns value.
- The persistent eth0.nmconnection profile contains no DNS server.
```

---

## Task 9 - Repair the DNS Client Configuration

Restore the expected DNS client configuration on `rhel-tester01`.

The correct DNS server is:

```text
10.10.10.20
```

Use NetworkManager to make the repair.

Do not manually edit `/etc/resolv.conf` as the primary repair method.

### Questions

1. Which NetworkManager property controls IPv4 DNS servers?
   > ipv4.dns
2. Which command can modify an existing connection profile?
   > nmcli connection modify eth0
3. How can the updated connection profile be applied?
   > sudo nmcli connection modify eth0 ipv4.dns 10.10.10.20
   > sudo nmcli connection down eth0
   > sudo nmcli connection up eth0
4. How can you confirm that NetworkManager accepted the new DNS setting?
   > nmcli connection show eth0

### Changes Made

```text
Changes Made:

Restored 10.10.10.20 as the IPv4 DNS server in the eth0
NetworkManager connection profile and reactivated the connection.
```

---

## Task 10 - Verify the Repair

After repairing the DNS configuration, verify that:

- `eth0` still has `10.10.10.21/24`
- The local network route still exists
- `10.10.10.20` remains reachable
- `/etc/resolv.conf` contains the expected resolver
- NetworkManager reports the expected DNS server
- Normal DNS queries succeed
- The DNS response contains the correct IPv4 address

### Verification

```text
Interface: eth0 - UP

IPv4 address: 10.10.10.21/24

Route: 10.10.10.0/24 via eth0

DNS server connectivity:
Successful. 10.10.10.20 responded with 0% packet loss.

Resolver configuration:
10.10.10.20 is configured as the nameserver in /etc/resolv.conf.

NetworkManager DNS:
10.10.10.20

DNS resolution:
Successful. dig returned status NOERROR.

Resolved address:
dns-test.example.com → 10.10.10.20

Result:
DNS client configuration was successfully restored. rhel-tester01 can
reach the DNS server and resolve the DNS-only test record using its
normal resolver configuration.
```

### Commands Used

```bash
ip addr show eth0
ip route
ping -c 4 10.10.10.20
cat /etc/resolv.conf
nmcli connection show eth0
dig dns-test.example.com
```

---

## Task 11 - Verify Name-Based Connectivity

After restoring DNS, verify that the system can use the FQDN rather than
requiring the IPv4 address directly.

Test:

```text
rhel-server01.example.com
```

### Questions

1. Does the FQDN resolve to `10.10.10.20`?
2. Can the server be reached using its FQDN?
3. Can an SSH connection be established using its FQDN?
4. What does this demonstrate?

### My Findings

```text
FQDN resolution:
Successful. dns-test.example.com resolved to 10.10.10.20.

Network connectivity:
Successful.

SSH:
Successful. An SSH connection was established using
dns-test.example.com.

Observations:
The DNS-only hostname resolved to 10.10.10.20 and was successfully
used to establish an SSH connection to rhel-server01.

Running hostname on the remote system returned
rhel-server01.example.com, confirming that the connection reached the
expected server.
```

---

## Task 12 - Verify Persistence

Reboot `rhel-tester01`.

After rebooting, verify that:

1. `eth0` remains UP.
2. `10.10.10.21/24` remains configured.
3. The `10.10.10.0/24` route remains present.
4. `10.10.10.20` remains configured as the DNS server.
5. `/etc/resolv.conf` contains the expected resolver.
6. Normal DNS queries still succeed.
7. `rhel-server01.example.com` remains reachable.
8. SSH using the FQDN still works.

### Post-Reboot Verification

```text
Interface: eth0 - UP

IPv4 address: 10.10.10.21/24

Route:
10.10.10.0/24 dev eth0 proto kernel scope link
src 10.10.10.21 metric 100

DNS configuration:
10.10.10.20 configured through NetworkManager.

Resolver configuration:
/etc/resolv.conf contains:
nameserver 10.10.10.20

DNS resolution:
Successful. dns-test.example.com resolves to 10.10.10.20.

Network connectivity:
Successful. dns-test.example.com is reachable with 0% packet loss.

SSH:
Successful. SSH connection to dns-test.example.com established.

Result:
The repaired DNS configuration persisted after reboot and normal
DNS-based connectivity continues to function.
```

---

## Commands Practised

```bash
ip addr show eth0
ip route
ping -c 4 10.10.10.20

dig dns-test.example.com
dig @10.10.10.20 dns-test.example.com

cat /etc/resolv.conf

nmcli connection show eth0

ls -l /etc/NetworkManager/system-connections/*.nmconnection
sudo cat /etc/NetworkManager/system-connections/eth0.nmconnection

sudo nmcli connection modify eth0 ipv4.dns 10.10.10.20
sudo nmcli connection down eth0
sudo nmcli connection up eth0

ssh dns-test.example.com

sudo reboot
```