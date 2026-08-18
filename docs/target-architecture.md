# Target Network Architecture

> **Security note:** This public document intentionally omits exact VLAN IDs, internal subnets, hostnames, MAC addresses, management addresses, credentials, keys, remote-access details, exact room locations, cable-drop locations, and unnecessary device fingerprinting information.

## Design Goal

The long-term design separates the home network into multiple logical security zones managed by OPNsense. The purpose is to reduce lateral movement, isolate less-trusted devices, and apply firewall policy based on device role.

The first migration phase is now complete: OPNsense is already the production edge router/firewall on the direct ISP handoff. The network is intentionally operating as a stable single trusted LAN before managed switching, VLANs, and VLAN-aware wireless are introduced.

## Current Production Foundation

```text
Internet / ISP ONT
        |
        v
OPNsense Firewall / Router
        |
        v
Unmanaged Gigabit Switch
        |
        +-- Trusted wired clients
        |
        +-- Temporary Wi-Fi AP
             (former ISP router)
```

Current-state characteristics:

- OPNsense is the only production router/firewall.
- OPNsense provides DHCP and DNS to the trusted LAN.
- IPv4 and native IPv6 are active.
- The former ISP router is temporarily reused only as a downstream wireless access point with its WAN interface unused and DHCP disabled.
- The current unmanaged switch is suitable only for the present single trusted segment and is not the future VLAN core.

## Planned Segments

### Main / Trusted
Primary network for trusted personal devices such as computers, phones, and tablets.

### IoT
Separate network for smart-home and other less-trusted embedded devices. Access to trusted systems will be restricted by default and only opened where required for functionality.

### Guest
Internet-access network for visitors. Guest devices will be isolated from trusted devices, infrastructure, and internal services.

### Server / NAS
Dedicated segment for server and storage systems. Access will be limited to approved trusted clients and required services.

## Long-Term Topology

```text
Internet
   |
   v
ISP Fiber Handoff
   |
   v
OPNsense Firewall / Router
   |
   v
VLAN-capable Managed Core Switch
   |
   +-- Main / Trusted
   +-- IoT
   +-- Guest
   +-- Server / NAS
   |
   +-- VLAN-aware Wired Access Points
```

## Physical Distribution Strategy

The home already has structured Ethernet cabling from a central utility location to multiple occupied floors. The design will use that central location as the network-distribution point rather than placing the physical core beside a workstation.

The central rack is intended to house or support the firewall, future core switching, patching/cable management, and power protection where equipment dimensions allow. Existing Ethernet runs will provide wired backhaul to access points and wired edge devices on other floors.

The primary workstation will serve as the administration console while the firewall and network core remain physically separate. Management access to network infrastructure will be restricted to approved trusted/admin devices rather than exposed broadly.

## Wireless Strategy

The current production network reuses the former ISP router as a temporary wired access point. It is connected LAN-to-LAN behind OPNsense, has its own DHCP service disabled, and does not use its WAN port. Wireless clients therefore receive addressing, gateway, and DNS information from OPNsense rather than from the former router.

An additional existing Wi-Fi 6 router remains available for future reuse if needed, but it is not required for the validated current production topology.

A future wireless refresh is planned around a coordinated VLAN-aware AP platform, with the goal of approximately one access point per level. Final AP count and placement will be based on measured signal strength, throughput, roaming behavior, and dead-zone testing rather than AP count alone.

The long-term wireless platform should support multiple SSIDs mapped to 802.1Q VLANs so Trusted, IoT, and Guest wireless clients can remain in separate security zones.

## Administration Model

The primary workstation is used as the network administration hub for OPNsense and other infrastructure. The design does not require the administration workstation to be physically connected directly to the firewall; it can securely manage infrastructure across the internal network.

The current management path is limited to the trusted internal network. A dedicated management VLAN or more restrictive management-policy layer is planned for a later phase if stronger management-plane isolation is desired.

## Security Policy Direction

The intended long-term policy is based on least privilege:

- **Main / Trusted:** permitted to reach approved internal services and the Internet.
- **IoT:** Internet access permitted as required; access to trusted networks denied by default.
- **Guest:** Internet access only; access to internal networks denied.
- **Server / NAS:** reachable only from approved networks and services; unnecessary outbound access restricted where practical.
- **Infrastructure management:** restricted to approved administration devices.
- **Inter-VLAN traffic:** denied by default unless an explicit business or functional requirement exists.

## Hardware Reuse and Role Assignment

- The dedicated x86-64 mini PC is now the production OPNsense firewall host.
- The validated interface assignment is **1 GbE WAN** to the ISP handoff and **2.5 GbE LAN** toward the internal network. Production testing showed no meaningful performance penalty at the current 1 Gig Internet tier.
- The existing unmanaged Gigabit Ethernet switch remains in service for the current single trusted segment, but it will not serve as the long-term VLAN core.
- The former ISP router is retained both as a private rollback asset and as a temporary downstream access point in the current topology.
- The existing Wi-Fi 6 router remains available for future edge/AP use if needed.
- The TP-Link ER605 remains useful for lab or backup-routing scenarios and is not part of the normal production path.
- A compact 8U rack is available for central organization of compatible network equipment. Because it is a shallow 10-inch format, equipment fit and cable-clearance requirements will be verified before mounting additional gear.

## Infrastructure Requirements for the Next Phase

The next major architecture phase will require:

- A managed switch with IEEE 802.1Q VLAN support; PoE capability is preferred if it will power future wireless access points.
- VLAN-capable wireless access points capable of mapping multiple SSIDs to separate VLANs.
- Suitable UPS-backed power protection for the firewall, switch, and network core.
- Inter-VLAN firewall rules and management-plane restrictions aligned with the least-privilege model.
- A migration/testing plan that preserves the currently stable single-LAN production environment until each new segment is validated.

Exact hardware placement, addressing, VLAN identifiers, firewall rules, and management details will remain sanitized in public documentation.

## Architecture Status

- [x] OPNsense deployed as production edge router/firewall
- [x] Direct ISP Ethernet handoff validated
- [x] 1 GbE WAN / 2.5 GbE LAN assignment validated
- [x] Temporary downstream Wi-Fi AP validated
- [x] IPv4 and native IPv6 validated
- [x] Stable single trusted LAN established
- [ ] Managed VLAN-capable core switch deployed
- [ ] Trusted / IoT / Guest / Server-NAS VLANs created
- [ ] VLAN-aware AP platform deployed
- [ ] Least-privilege inter-VLAN policy implemented
- [ ] Dedicated management-plane isolation implemented
- [ ] UPS-backed core power completed
