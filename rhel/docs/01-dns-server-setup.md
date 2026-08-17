# RHEL Homelab - DNS Server Setup

## Overview

This document extends the original two-server RHEL 9 homelab by configuring
`rhel-server01` as a DNS server using BIND.

The original lab environment used `/etc/hosts` for local hostname resolution.
This was sufficient for the initial networking labs but does not provide a
real DNS environment for DNS client configuration and troubleshooting.

The DNS server was therefore introduced before Lab 03.

---

# 1. Updated Lab Architecture

| System | Hostname | IPv4 Address | Role |
|---|---|---|---|
| Primary Server | `rhel-server01.example.com` | `10.10.10.20/24` | DNS server / primary server |
| Test Server | `rhel-tester01.example.com` | `10.10.10.21/24` | DNS client / troubleshooting server |

### Network

```text
Network:          10.10.10.0/24
Hyper-V Switch:   LAB-SWITCH
DNS Server:       10.10.10.20
DNS Domain:       example.com
```

### Architecture

```text
                    LAB-SWITCH
                   10.10.10.0/24
                         │
            ┌────────────┴────────────┐
            │                         │
    rhel-server01              rhel-tester01
      10.10.10.20                10.10.10.21
            │                         │
        BIND/named                     │
        DNS Server ◄──── DNS queries ──┘
            │
            │ authoritative for
            ▼
        example.com
            │
            ├── rhel-server01.example.com
            │       └── 10.10.10.20
            │
            └── rhel-tester01.example.com
                    └── 10.10.10.21
```

---

# 2. DNS Concepts

## What is DNS?

DNS (Domain Name System) translates human-readable hostnames into IP
addresses.

For example:

```text
rhel-server01.example.com
            ↓
           DNS
            ↓
       10.10.10.20
```

Without name resolution, users and applications would need to use IP
addresses directly.

---

## `/etc/hosts` vs DNS

The original homelab used `/etc/hosts`:

```text
10.10.10.20   rhel-server01.example.com   rhel-server01
10.10.10.21   rhel-tester01.example.com   rhel-tester01
```

`/etc/hosts` provides local static hostname mappings.

Each machine maintains its own copy:

```text
rhel-server01 → /etc/hosts
rhel-tester01 → /etc/hosts
```

This is manageable for a very small environment but becomes difficult to
maintain as the number of systems increases.

DNS centralizes name resolution:

```text
                  DNS Server
                 10.10.10.20
                      │
              ┌───────┴───────┐
              │               │
           Client 1         Client 2
```

Clients query the DNS server instead of requiring identical hostname mappings
to be manually maintained on every system.

---

# 3. Important DNS Terminology

## DNS Server

A system that answers DNS queries.

In this lab:

```text
rhel-server01
10.10.10.20
```

provides DNS using BIND.

---

## DNS Client

A system configured to send DNS queries to a DNS server.

In this lab:

```text
rhel-tester01
10.10.10.21
```

uses:

```text
10.10.10.20
```

as its DNS server.

---

## BIND

BIND (Berkeley Internet Name Domain) is the DNS server software installed on
`rhel-server01`.

The package is:

```text
bind
```

The DNS daemon provided by BIND is:

```text
named
```

The systemd service is:

```text
named.service
```

Therefore:

```text
Package         → bind
Daemon          → named
systemd service → named.service
```

---

## Zone

A DNS zone is a portion of the DNS namespace managed by a DNS server.

The zone created for this homelab is:

```text
example.com
```

The server is authoritative for records contained within this zone.

---

## A Record

An `A` record maps a hostname to an IPv4 address.

Examples:

```text
rhel-server01.example.com → 10.10.10.20
rhel-tester01.example.com → 10.10.10.21
```

The corresponding zone records are:

```text
rhel-server01  IN  A  10.10.10.20
rhel-tester01  IN  A  10.10.10.21
```

---

## NS Record

An `NS` record identifies the authoritative DNS server for a zone.

For this lab:

```text
example.com
    ↓
rhel-server01.example.com
```

---

## SOA Record

SOA stands for Start of Authority.

It contains administrative information about the DNS zone, including:

- Primary DNS server
- Administrative contact
- Zone serial number
- Refresh interval
- Retry interval
- Expiration interval
- Cache timing information

---

## DNS Port

DNS normally uses:

```text
Port: 53
Protocol: UDP and TCP
```

The RHEL firewall therefore needs to allow the `dns` service.

---

# 4. Install BIND

The BIND packages were available through the locally configured RHEL 9
AppStream repository.

Availability was checked with:

```bash
dnf list bind
```

The packages were installed with:

```bash
sudo dnf install bind bind-utils
```

### Packages

`bind` provides the BIND DNS server.

`bind-utils` provides DNS troubleshooting utilities such as:

```text
dig
host
nslookup
```

Verify installation:

```bash
rpm -q bind bind-utils
```

---

# 5. Initial Service State

The BIND systemd service was inspected with:

```bash
systemctl status named
```

Before configuration, the service was:

```text
Loaded: loaded
Active: inactive (dead)
Enabled: disabled
```

The service was intentionally not started until its configuration had been
completed and validated.

---

# 6. Configure BIND

The primary BIND configuration file is:

```text
/etc/named.conf
```

A backup was created before modification:

```bash
sudo cp /etc/named.conf /etc/named.conf.bak
```

---

## Default Configuration

The original configuration included:

```text
listen-on port 53 { 127.0.0.1; };
allow-query     { localhost; };
```

This meant that BIND would listen only on the local loopback address and
would only accept queries from the local system.

This would prevent `rhel-tester01` from using the server.

---

## Lab Configuration

The configuration was changed to:

```text
listen-on port 53 { 127.0.0.1; 10.10.10.20; };
allow-query     { localhost; 10.10.10.0/24; };
```

### Meaning

```text
listen-on
    ↓
Which IPv4 addresses should named listen on?

127.0.0.1
10.10.10.20
```

and:

```text
allow-query
    ↓
Which systems are allowed to send DNS queries?

localhost
10.10.10.0/24
```

This allows systems on the homelab network to query the DNS server.

---

# 7. Validate BIND Configuration

Before starting BIND, the configuration was checked with:

```bash
sudo named-checkconf
```

No output indicates that the configuration syntax is valid.

This should be performed after modifying:

```text
/etc/named.conf
```

and before restarting `named`.

---

# 8. Configure the DNS Zone

The following zone was added to `/etc/named.conf`:

```conf
zone "example.com" IN {
    type master;
    file "example.com.zone";
};
```

This tells BIND that:

```text
Zone:      example.com
Type:      master
Zone file: /var/named/example.com.zone
```

---

# 9. Create the Zone File

The zone file was created at:

```text
/var/named/example.com.zone
```

Configuration:

```dns
$TTL 86400
@   IN  SOA rhel-server01.example.com. admin.example.com. (
        2026081701
        3600
        1800
        604800
        86400
)

    IN  NS  rhel-server01.example.com.

rhel-server01  IN  A   10.10.10.20
rhel-tester01  IN  A   10.10.10.21
```

---

# 10. Understanding the Zone File

## `$TTL`

```text
$TTL 86400
```

TTL means Time To Live.

It controls how long DNS information may be cached.

```text
86400 seconds = 24 hours
```

---

## SOA

```text
@ IN SOA rhel-server01.example.com. admin.example.com.
```

`@` represents the current zone:

```text
example.com
```

`rhel-server01.example.com.` identifies the primary authoritative server.

`admin.example.com.` represents the administrative contact in DNS notation.

---

## Serial Number

```text
2026081701
```

The serial number identifies the version of the zone.

A common convention is:

```text
YYYYMMDDNN
```

For example:

```text
2026081701
│       └─ revision 01
└───────── 2026-08-17
```

The serial should be incremented when zone data is changed.

This becomes especially important when secondary DNS servers are introduced,
because they use the serial number to determine whether the zone has changed.

---

## NS Record

```text
IN NS rhel-server01.example.com.
```

This identifies `rhel-server01.example.com` as an authoritative nameserver
for the zone.

---

## A Records

```text
rhel-server01  IN  A  10.10.10.20
rhel-tester01  IN  A  10.10.10.21
```

These provide:

```text
rhel-server01.example.com → 10.10.10.20
rhel-tester01.example.com → 10.10.10.21
```

---

# 11. Validate the Zone

The zone file was checked before starting the DNS service:

```bash
sudo named-checkzone example.com /var/named/example.com.zone
```

Successful validation returned:

```text
zone example.com/IN: loaded serial 2026081701
OK
```

The main configuration was checked again:

```bash
sudo named-checkconf
```

---

# 12. Zone File Permissions and SELinux

The zone file was assigned appropriate ownership and permissions:

```bash
sudo chown root:named /var/named/example.com.zone
sudo chmod 640 /var/named/example.com.zone
```

SELinux file context was restored with:

```bash
sudo restorecon -v /var/named/example.com.zone
```

This is important on RHEL because correct traditional Linux permissions do
not necessarily mean that SELinux will permit a service to access a file.

---

# 13. Start the DNS Service

BIND was started and enabled at boot:

```bash
sudo systemctl enable --now named
```

Verify:

```bash
systemctl status named
```

Expected:

```text
Active: active (running)
```

`enable --now` performs two operations:

```text
--now  → start the service immediately
enable → start the service automatically during future boots
```

---

# 14. Configure the Firewall

DNS traffic was allowed through `firewalld`:

```bash
sudo firewall-cmd --permanent --add-service=dns
sudo firewall-cmd --reload
```

Verify:

```bash
sudo firewall-cmd --list-services
```

The output should include:

```text
dns
```

This permits DNS traffic through the RHEL firewall.

---

# 15. Test DNS Locally

DNS was first tested directly from `rhel-server01`.

```bash
dig @10.10.10.20 rhel-server01.example.com
```

and:

```bash
dig @10.10.10.20 rhel-tester01.example.com
```

Successful responses returned:

```text
rhel-server01.example.com → 10.10.10.20
rhel-tester01.example.com → 10.10.10.21
```

Important fields in `dig` output include:

```text
status: NOERROR
ANSWER: 1
SERVER: 10.10.10.20#53
```

The `aa` flag indicates an authoritative answer:

```text
flags: qr aa ...
          ↑
    Authoritative Answer
```

---

# 16. Test DNS from Another System

Before configuring the client's default resolver, DNS connectivity was tested
directly from `rhel-tester01`:

```bash
dig @10.10.10.20 rhel-server01.example.com
```

This explicitly tells `dig` to query:

```text
10.10.10.20
```

A successful result proved that:

- `rhel-tester01` could reach `rhel-server01`
- TCP/IP connectivity was functioning
- Port 53 was accessible
- `firewalld` permitted DNS
- BIND accepted queries from the lab network
- The `example.com` zone could answer the query

---

# 17. Configure the DNS Client

`rhel-tester01` previously contained a stale DNS setting:

```text
172.16.0.2
```

This belonged to an earlier network configuration and was no longer
appropriate for the `10.10.10.0/24` homelab.

The NetworkManager connection was changed to use:

```text
10.10.10.20
```

with:

```bash
sudo nmcli connection modify eth0 ipv4.dns 10.10.10.20
```

The connection was then reactivated:

```bash
sudo nmcli connection down eth0
sudo nmcli connection up eth0
```

---

# 18. Verify Client DNS Configuration

NetworkManager can be inspected with:

```bash
nmcli connection show eth0
```

Relevant fields include:

```text
ipv4.dns
IP4.DNS
```

The resolver configuration can also be inspected with:

```bash
cat /etc/resolv.conf
```

The expected DNS server is:

```text
10.10.10.20
```

---

# 19. Test the Default Resolver

A final test was performed without explicitly specifying the DNS server:

```bash
dig rhel-server01.example.com
```

The response included:

```text
status: NOERROR
ANSWER: 1
SERVER: 10.10.10.20#53
```

and:

```text
rhel-server01.example.com → 10.10.10.20
```

This is different from:

```bash
dig @10.10.10.20 rhel-server01.example.com
```

The `@10.10.10.20` form explicitly selects a DNS server.

Without `@10.10.10.20`, `dig` uses the system's configured resolver.

Therefore, the successful test confirmed that `rhel-tester01` was correctly
configured to use `rhel-server01` as its DNS server.

---

# 20. Important Troubleshooting Distinction

One of the most important concepts for troubleshooting is separating
network connectivity from name resolution.

For example:

```text
ping 10.10.10.20
        │
        ├── FAIL
        │
        └── Investigate networking first

ping 10.10.10.20
        │
        └── SUCCESS
                │
                ▼
DNS lookup fails
                │
                └── Investigate name resolution
```

If an IP address is reachable but a hostname cannot be resolved, the
underlying network may be functioning correctly while DNS is not.

This distinction forms the basis of the DNS troubleshooting lab.

---

# 21. Important Files

| File | Purpose |
|---|---|
| `/etc/named.conf` | Main BIND server configuration |
| `/var/named/example.com.zone` | DNS records for the `example.com` zone |
| `/etc/resolv.conf` | Resolver configuration used by a DNS client |
| `/etc/NetworkManager/system-connections/eth0.nmconnection` | Persistent NetworkManager connection configuration |
| `/etc/hosts` | Local static hostname mappings |

---

# 22. Important Commands

## Server Configuration

```bash
dnf list bind
sudo dnf install bind bind-utils

systemctl status named
sudo systemctl enable --now named

sudo named-checkconf
sudo named-checkzone example.com /var/named/example.com.zone
```

## Firewall

```bash
sudo firewall-cmd --permanent --add-service=dns
sudo firewall-cmd --reload
sudo firewall-cmd --list-services
```

## DNS Testing

```bash
dig @10.10.10.20 rhel-server01.example.com
dig @10.10.10.20 rhel-tester01.example.com

dig rhel-server01.example.com

getent hosts rhel-server01.example.com
```

## Client Configuration

```bash
nmcli connection show eth0

sudo nmcli connection modify eth0 ipv4.dns 10.10.10.20

sudo nmcli connection down eth0
sudo nmcli connection up eth0

cat /etc/resolv.conf
```

---

# 23. Key Points to Remember

1. BIND is the DNS software package; `named` is its DNS daemon.

2. DNS normally uses port 53.

3. `/etc/named.conf` controls the BIND server configuration.

4. DNS records for this lab are stored in the `example.com` zone.

5. An `A` record maps a hostname to an IPv4 address.

6. An `NS` record identifies an authoritative nameserver.

7. The SOA record contains administrative and zone-management information.

8. Increment the zone serial when modifying zone data.

9. Run `named-checkconf` after modifying BIND configuration.

10. Run `named-checkzone` after modifying a zone file.

11. Validate configuration before restarting a production service.

12. `firewalld` must permit DNS traffic from clients.

13. NetworkManager should be used to persist DNS client configuration.

14. `/etc/resolv.conf` shows the resolver configuration being used by the
    system, but it should not normally be treated as the primary persistent
    NetworkManager configuration.

15. `dig @SERVER NAME` explicitly queries a particular DNS server.

16. `dig NAME` uses the system's configured DNS resolver.

17. Successful communication by IP but failure by hostname strongly suggests
    investigating name resolution.

18. `/etc/hosts` and DNS can both provide hostname resolution. When
    troubleshooting DNS, be aware that an `/etc/hosts` entry may allow a
    hostname to resolve even when DNS itself is broken.

---

# 24. Updated Homelab State

The homelab now consists of:

```text
Windows 11
└── Microsoft Hyper-V
    │
    └── LAB-SWITCH
        │
        ├── rhel-server01.example.com
        │   ├── 10.10.10.20/24
        │   ├── BIND/named
        │   ├── DNS: TCP/UDP 53
        │   └── Authoritative for example.com
        │
        └── rhel-tester01.example.com
            ├── 10.10.10.21/24
            └── DNS Server: 10.10.10.20
```

This updated environment is the baseline used for Lab 03 and subsequent
DNS-related exercises.

---

# Skills Practised

- Installing RHEL packages with DNF
- Managing services with systemd
- Configuring BIND
- Understanding DNS zones
- Creating DNS A records
- Understanding NS and SOA records
- Validating BIND configuration
- Managing file permissions
- Working with SELinux file contexts
- Configuring `firewalld`
- Configuring DNS clients with NetworkManager
- Using `dig`
- Understanding `/etc/resolv.conf`
- Distinguishing IP connectivity from name resolution
- Troubleshooting DNS client/server communication