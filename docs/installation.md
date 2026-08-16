# OPNsense Installation

> **Security note:** This public documentation intentionally omits passwords, management addresses, MAC addresses, serial numbers, and other identifying or operational details.

## Installation Target

A dedicated x86-64 mini PC was repurposed as the OPNsense firewall appliance. Existing data on the system was not required and the internal SSD was dedicated to the firewall installation.

## Installation Media

- **OPNsense version:** 26.7
- **Architecture:** amd64
- **Installer type:** VGA USB image
- **Filesystem:** ZFS
- **Storage layout:** Single-disk stripe

The installation image checksum was verified before the USB installer was created.

## Installation Result

OPNsense 26.7 installed successfully to the internal SSD and booted from local storage after the installer USB was removed.

## Network Interface Assignment

Both onboard Ethernet interfaces were detected successfully by OPNsense:

- `igc0` — Intel-based 2.5 GbE interface
- `re0` — Realtek-based 1 GbE interface

For the current 1 Gig Internet service, the initial assignment is:

- **LAN:** `igc0` — preserves the faster 2.5 GbE interface for the internal network and future managed switching
- **WAN:** `re0` — matches the present 1 Gig WAN requirement

No LAGG, VLAN, or optional physical interface was configured during the initial console setup. VLANs will be added later after the basic WAN/LAN lab configuration is validated.

## Initial LAN Validation

A directly connected test workstation successfully received a DHCP lease from the OPNsense LAN interface and reached the OPNsense web-management interface over HTTPS. This confirmed that the LAN interface, DHCP service, and local management path were functioning before the WAN side was connected.

At this checkpoint:

- [x] Bootable installer created
- [x] Installation media verified
- [x] Internal installation disk identified
- [x] ZFS installation completed
- [x] System rebooted from internal storage
- [x] OPNsense 26.7 console reached successfully
- [x] Verify detected network interfaces
- [x] Assign WAN and LAN roles
- [ ] Configure isolated lab network
- [x] Confirm web-management access
- [ ] Update firmware/packages before production cutover

## Next Step

The next step is to connect the OPNsense WAN interface to the existing router for an isolated double-NAT lab test, verify that the WAN receives a private DHCP address, and confirm Internet access from the test workstation before touching the ISP handoff.
