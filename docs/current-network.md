# Phase 1 — Pre-Migration Network Assessment

> **Security note:** This public repository intentionally redacts or generalizes network details that could expose the home environment. Exact IP addresses, MAC addresses, hostnames, SSIDs, public IP information, serial numbers, credentials, keys, exact test-server location, and remote-access details are not published.

> **Status note:** This document is intentionally historical. It records the network before OPNsense replaced the ISP router and serves as the baseline for the completed migration.

## ISP and Edge Equipment

- **ISP:** Verizon Fios
- **Service tier:** 1 Gig
- **ISP handoff:** Verizon Optical Network Terminal (ONT)
- **Pre-migration router:** Verizon CR1000A
- **Pre-migration WAN connection:** Ethernet from the Verizon ONT to the CR1000A WAN port

## Pre-Migration Physical Topology

```text
Verizon Fios
     |
     | Fiber
     v
Verizon ONT
     |
     | Ethernet
     v
Verizon CR1000A
     |
     +-- Wi-Fi
     |
     +-- Ethernet
          |
          v
      Home Devices
```

## Pre-Migration LAN Baseline

A Windows workstation connected through the physical Ethernet interface was used to capture the LAN configuration with `ipconfig /all`.

| Item | Public Documentation Value |
|---|---|
| Address assignment | DHCP |
| Address type | Private RFC1918 IPv4 |
| Subnet size | `/24` |
| Default gateway | Redacted |
| DHCP server | Verizon CR1000A |
| DNS server | Verizon CR1000A |

The workstation also contained virtual and VPN-related adapters. Those were excluded so the baseline represented the physical home-LAN path only.

## Pre-OPNsense Performance Baseline

Three wired Internet speed tests were performed from the same workstation while the Verizon router remained the active gateway. VPN routing was excluded from the baseline.

| Test | Ping | Download | Upload |
|---|---:|---:|---:|
| 1 | 8 ms | 938.16 Mbps | 900.71 Mbps |
| 2 | 6 ms | 937.41 Mbps | 880.18 Mbps |
| 3 | 7 ms | 934.69 Mbps | 862.42 Mbps |
| **Average** | **7 ms** | **936.75 Mbps** | **881.10 Mbps** |

These measurements became the reference point for the later OPNsense lab and production tests. The final direct-OPNsense production average was approximately 937.64 Mbps download, 919.77 Mbps upload, and 7 ms latency, so the migration met the project's no-material-performance-loss target.

## Baseline Findings

Before the migration, the CR1000A was the primary LAN gateway and supplied routing, DHCP, DNS, wired connectivity, and Wi-Fi service. The 1 Gig wired baseline was already performing near the practical limit of a Gigabit Ethernet WAN path, making throughput preservation an important success criterion for the replacement firewall.

The assessment also established several design requirements for the project:

- preserve the Ethernet ONT handoff
- maintain near-gigabit wired performance
- preserve a fast rollback path during cutover
- keep wireless service available during the transition
- centralize routing, DHCP, DNS, and firewall policy on OPNsense
- prepare for later segmentation into Trusted, IoT, Guest, and Server/NAS zones

A full device-by-device inventory was not required for the initial single-LAN migration and remains more relevant to the future VLAN-segmentation phase.

## Phase 1 Status

- [x] Identify ISP and service tier
- [x] Identify pre-migration router
- [x] Confirm ONT-to-router Ethernet handoff
- [x] Capture baseline LAN addressing and services
- [x] Capture performance baseline
- [x] Define project network requirements
- [x] Define target architecture
- [x] Define cutover and rollback plan
- [x] Preserve the original environment for rollback

## Post-Migration Reference

The current production topology is documented in the main [`README.md`](../README.md), with detailed cutover and final validation results in [`cutover-rollback.md`](cutover-rollback.md). This file remains the historical baseline used to compare the completed OPNsense deployment against the original ISP-router environment.
