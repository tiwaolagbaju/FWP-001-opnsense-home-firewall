# Target Network Architecture

> **Security note:** This public document intentionally omits exact VLAN IDs, internal subnets, hostnames, MAC addresses, management addresses, credentials, keys, remote-access details, exact room locations, cable-drop locations, and unnecessary device fingerprinting information.

## Design Goal

The target design separates the home network into multiple logical security zones managed by OPNsense. The purpose is to reduce lateral movement, isolate less-trusted devices, and apply firewall policy based on device role.

## Planned Segments

### Main / Trusted
Primary network for trusted personal devices such as computers, phones, and tablets.

### IoT
Separate network for smart-home and other less-trusted embedded devices. Access to trusted systems will be restricted by default and only opened where required for functionality.

### Guest
Internet-access network for visitors. Guest devices will be isolated from trusted devices, infrastructure, and internal services.

### Server / NAS
Dedicated segment for server and storage systems. Access will be limited to approved trusted clients and required services.

## High-Level Topology

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
   +-- Wired Access Points
```

## Physical Distribution Strategy

The home already has structured Ethernet cabling from a central utility location to multiple occupied floors. The design will use that central location as the network-distribution point rather than placing the physical core beside a workstation.

The central rack is intended to house or support the firewall, core switching, patching/cable management, and power protection where equipment dimensions allow. Existing Ethernet runs will provide wired backhaul to access points and wired edge devices on other floors.

The primary workstation will serve as the administration console while the firewall and network core remain physically separate. Management access to network infrastructure will be restricted to approved trusted/admin devices rather than exposed broadly.

## Initial Wireless and Workspace Design

The initial deployment will reuse two existing wireless routers as temporary wired access points:

- One access point will serve the primary workspace. Its built-in Ethernet switch ports will also provide wired connectivity to trusted workstation devices during the initial deployment.
- A second access point will provide wireless coverage near the network-core level.
- Both access points will use wired Ethernet backhaul.

During this temporary stage, devices attached to the workspace access point will remain on a single trusted network segment. Advanced per-SSID VLAN segmentation will be implemented during the later wireless refresh rather than forced onto hardware that was not designed for that role.

A future wireless refresh is planned around a coordinated VLAN-aware AP platform, with the goal of approximately one access point per level. Final AP count and placement will be based on measured signal strength, throughput, roaming behavior, and dead-zone testing rather than AP count alone.

The long-term wireless platform should support multiple SSIDs mapped to 802.1Q VLANs so Trusted, IoT, and Guest wireless clients can remain in separate security zones.

## Administration Model

The primary workstation will be used as the network administration hub for OPNsense and other infrastructure. The design does not require the administration workstation to be physically connected directly to the firewall; it can securely manage infrastructure across the internal network.

Initial management policy will allow administration only from explicitly approved trusted/admin devices. A dedicated management VLAN may be introduced later if the environment grows or if stronger management-plane isolation is desired.

## Security Policy Direction

The intended policy is based on least privilege:

- **Main / Trusted:** permitted to reach approved internal services and the Internet.
- **IoT:** Internet access permitted as required; access to trusted networks denied by default.
- **Guest:** Internet access only; access to internal networks denied.
- **Server / NAS:** reachable only from approved networks and services; unnecessary outbound access restricted where practical.
- **Infrastructure management:** restricted to approved administration devices.
- **Inter-VLAN traffic:** denied by default unless an explicit business or functional requirement exists.

## Existing Hardware Reuse Plan

The current equipment will be reused where it fits the security design rather than replacing hardware unnecessarily.

- A dedicated x86-64 mini PC is planned as the OPNsense firewall host. It provides two physical Ethernet interfaces: one Intel-based 2.5 GbE interface and one Realtek-based 1 GbE interface. Both interface families are supported by FreeBSD, the operating-system base used by OPNsense. Final WAN/LAN port assignment will be chosen after confirming the subscribed Internet speed and validating both links under load.
- The existing unmanaged Gigabit Ethernet switch can remain in service as an edge switch for devices that all belong to one network segment, but it will not serve as the core VLAN switch.
- An existing Wi-Fi 6 router will be repurposed temporarily as the primary-workspace access point and local wired edge switch for trusted devices.
- The ISP-provided router will be retained for rollback and evaluated as a temporary secondary access point near the core location.
- An existing wired VPN router is useful for lab testing or backup routing but is not planned to remain in the primary production path once OPNsense becomes the edge firewall/router.
- A compact 8U rack is available for central organization of compatible network equipment. Because it is a shallow 10-inch format, equipment fit and cable-clearance requirements will be verified before mounting.

## Infrastructure Requirements

The final design will require:

- An OPNsense-compatible x86-64 system with reliable separate WAN and LAN connectivity.
- A managed switch with IEEE 802.1Q VLAN support; PoE capability is preferred if it will power the future wireless access points.
- VLAN-capable wireless access points capable of mapping multiple SSIDs to separate VLANs.
- Suitable power protection for the network core.

Exact hardware placement, addressing, VLAN identifiers, firewall rules, and management details will remain sanitized in public documentation.
