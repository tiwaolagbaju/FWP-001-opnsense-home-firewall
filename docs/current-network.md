# Phase 1 — Current Network Assessment

> **Security note:** This public repository intentionally redacts or generalizes network details that could expose the home environment. Exact IP addresses, MAC addresses, hostnames, SSIDs, public IP information, serial numbers, credentials, keys, and remote-access details are not published.

## ISP and Edge Equipment

- **ISP:** Verizon Fios
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

## What This Tells Us

The Verizon CR1000A currently functions as the primary LAN gateway and provides DHCP and DNS services to connected clients. This establishes a baseline for comparing the existing consumer-router design with the future OPNsense deployment.

## Workstation Networking Notes

The test workstation contains additional virtual and VPN-related network adapters. Those adapters were excluded from the home-LAN baseline so that the assessment reflects the physical Ethernet connection only.

## Phase 1 Status

- [x] Identify ISP
- [x] Identify current router
- [x] Confirm ONT-to-router Ethernet handoff
- [x] Capture current LAN addressing and services
- [ ] Inventory connected devices and network requirements
- [ ] Capture performance baseline
- [ ] Define target architecture
- [ ] Define cutover and rollback plan
