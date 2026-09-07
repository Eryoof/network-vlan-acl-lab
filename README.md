# network-vlan-acl-lab

A simulated enterprise-style network built in **Cisco Packet Tracer**, designed to segment two departments into isolated VLANs, route traffic between them, and enforce targeted access control to protect a sensitive host.

> **Note:** This is a simulation/learning project built with Cisco Packet Tracer, not a production deployment.

## Overview

The network represents a small building with two departments — **IT** and **Marketing** — each on its own VLAN. A single router handles inter-VLAN routing using the Router-on-a-Stick method, and an Extended ACL restricts one department from reaching a specific protected host in the other.

## Topology

```
        PC0        PC1
          \         /
           \       /
            Switch0 --- Router1
           /       \
          /         \
        PC2        PC3
```

| Device | Role | VLAN | IP Address |
|---|---|---|---|
| PC0 | IT | 10 | 192.168.10.10 |
| PC1 | IT (protected host) | 10 | 192.168.10.11 |
| PC2 | Marketing | 20 | 192.168.20.10 |
| PC3 | Marketing | 20 | 192.168.20.11 |
| Router1 (Fa0/0.10) | IT gateway | 10 | 192.168.10.1 |
| Router1 (Fa0/0.20) | Marketing gateway | 20 | 192.168.20.1 |

*(See `/screenshots` for the full topology diagram.)*

## Tools Used

- Cisco Packet Tracer
- Cisco IOS CLI (Router & Switch configuration)

## Part 1 — VLAN Segmentation & Inter-VLAN Routing

**Goal:** Isolate the two departments into separate logical networks while still allowing controlled communication between them.

### Switch Configuration (VLANs)

```
vlan 10
name IT
exit
vlan 20
name Marketing
exit

interface fastethernet0/1
switchport mode access
switchport access vlan 10
exit

interface fastethernet0/2
switchport mode access
switchport access vlan 10
exit

interface fastethernet0/3
switchport mode access
switchport access vlan 20
exit

interface fastethernet0/4
switchport mode access
switchport access vlan 20
exit

interface fastethernet0/5
switchport mode trunk
exit
```

### Router Configuration (Router-on-a-Stick)

```
interface fastethernet0/0
no ip address
no shutdown
exit

interface fastethernet0/0.10
encapsulation dot1Q 10
ip address 192.168.10.1 255.255.255.0
exit

interface fastethernet0/0.20
encapsulation dot1Q 20
ip address 192.168.20.1 255.255.255.0
exit
```

### Testing

| Test | Result |
|---|---|
| PC0 → PC1 (same VLAN) | ✅ Success |
| PC0 → PC2 (different VLAN, via router) | ✅ Success (0% packet loss) |

## Part 2 — Access Control (ACL)

**Goal:** Prevent the Marketing department from reaching one specific host in IT (PC1), without affecting any other traffic.

### ACL Configuration

```
access-list 100 deny ip any host 192.168.10.11
access-list 100 permit ip any any

interface fastethernet0/0.20
ip access-group 100 in
```

- The `deny` rule blocks any traffic destined for PC1 specifically.
- The `permit any any` rule is required — without it, the router's implicit deny would block all other traffic too.
- The ACL is applied inbound on the Marketing gateway interface, so it only filters traffic coming from Marketing.

### Testing

| Direction | Target | Result |
|---|---|---|
| Marketing → PC1 (protected) | 192.168.10.11 | ❌ Blocked (100% loss, "Destination host unreachable") |
| Marketing → PC0 (unprotected) | 192.168.10.10 | ✅ Success (0% loss) |
| IT → PC1 (same VLAN, no router involved) | 192.168.10.11 | ✅ Success |
| PC1 → Marketing | 192.168.20.11 | ❌ Blocked (return traffic filtered) |

### Key Insight

The ACL is **stateless** — it filters based on the direction and destination of each packet independently, not on which side initiated the conversation. This is why traffic initiated *by* the protected host (PC1) toward Marketing still fails: the reply traffic re-enters the router through the same filtered interface and gets blocked.

## What This Demonstrates

- VLAN segmentation for logical network isolation
- Inter-VLAN routing using Router-on-a-Stick
- Extended ACLs for targeted, host-level access control
- Systematic connectivity testing to validate network design

## Screenshots

See the `/screenshots` folder for the full topology and ping test results.

