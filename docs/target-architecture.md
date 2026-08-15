# Target Network Architecture

> **Security note:** This public document intentionally omits exact VLAN IDs, internal subnets, hostnames, MAC addresses, management addresses, credentials, keys, and remote-access details.

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
Verizon ONT
   |
   v
OPNsense Firewall / Router
   |
   v
VLAN-capable Managed Switch
   |
   +-- Main / Trusted
   +-- IoT
   +-- Guest
   +-- Server / NAS
   |
   +-- VLAN-capable Wireless Access Point
```

## Security Policy Direction

The intended policy is based on least privilege:

- **Main / Trusted:** permitted to reach approved internal services and the Internet.
- **IoT:** Internet access permitted as required; access to trusted networks denied by default.
- **Guest:** Internet access only; access to internal networks denied.
- **Server / NAS:** reachable only from approved networks and services; unnecessary outbound access restricted where practical.
- **Inter-VLAN traffic:** denied by default unless an explicit business or functional requirement exists.

## Infrastructure Requirements

The final design will require:

- An OPNsense-compatible system with separate WAN and LAN connectivity.
- A managed switch with IEEE 802.1Q VLAN support.
- A VLAN-capable wireless access point if multiple wireless SSIDs are mapped to separate network segments.

Exact hardware, addressing, VLAN identifiers, and firewall rules will be selected and tested later in the project and will remain sanitized in public documentation.
