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

At this checkpoint:

- [x] Bootable installer created
- [x] Installation media verified
- [x] Internal installation disk identified
- [x] ZFS installation completed
- [x] System rebooted from internal storage
- [x] OPNsense 26.7 console reached successfully
- [ ] Verify detected network interfaces
- [ ] Assign WAN and LAN roles
- [ ] Configure isolated lab network
- [ ] Confirm web-management access
- [ ] Update firmware/packages before production cutover

## Next Step

The next step is to verify the detected Ethernet interfaces and assign the appropriate physical ports to WAN and LAN before connecting the firewall to the existing network.
