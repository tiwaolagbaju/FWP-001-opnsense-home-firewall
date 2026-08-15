# Phase 1 — Current Network Assessment

> **Security note:** This public repository intentionally redacts or generalizes network details that could expose the home environment. Exact IP addresses, MAC addresses, hostnames, SSIDs, public IP information, serial numbers, credentials, keys, exact test-server location, and remote-access details are not published.

## ISP and Edge Equipment

- **ISP:** Verizon Fios
- **Service tier:** 1 Gig
- **ISP handoff:** Verizon Optical Network Terminal (ONT)
- **Current router:** Verizon CR1000A
- **WAN connection:** Ethernet from the Verizon ONT to the CR1000A WAN port

## Current Physical Topology

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

## Current LAN Baseline

A Windows workstation connected through the physical Ethernet interface was used to capture the current LAN configuration with `ipconfig /all`.

| Item | Public Documentation Value |
|---|---|
| Address assignment | DHCP |
| Address type | Private RFC1918 IPv4 |
| Subnet size | `/24` |
| Default gateway | Redacted |
| DHCP server | Current Verizon router |
| DNS server | Current Verizon router |

## Pre-OPNsense Performance Baseline

Three wired Internet speed tests were performed from the same workstation while the Verizon router remained the active gateway. VPN routing was excluded from the baseline.

| Test | Ping | Download | Upload |
|---|---:|---:|---:|
| 1 | 8 ms | 938.16 Mbps | 900.71 Mbps |
| 2 | 6 ms | 937.41 Mbps | 880.18 Mbps |
| 3 | 7 ms | 934.69 Mbps | 862.42 Mbps |
| **Average** | **7 ms** | **936.75 Mbps** | **881.10 Mbps** |

These measurements provide the reference point for post-migration testing. After OPNsense is deployed, the same wired workstation and test method will be used to compare throughput and latency.

## What This Tells Us

The Verizon CR1000A currently functions as the primary LAN gateway and provides DHCP and DNS services to connected clients. The wired performance baseline is close to the expected practical range for a 1 Gig Ethernet Internet connection and gives the project a measurable target for the OPNsense migration.

## Workstation Networking Notes

The test workstation contains additional virtual and VPN-related network adapters. Those adapters were excluded from the home-LAN baseline so that the assessment reflects the physical Ethernet connection only.

## Phase 1 Status

- [x] Identify ISP
- [x] Identify current router
- [x] Confirm ONT-to-router Ethernet handoff
- [x] Capture current LAN addressing and services
- [ ] Inventory connected devices and network requirements
- [x] Capture performance baseline
- [x] Define target architecture
- [ ] Define cutover and rollback plan
