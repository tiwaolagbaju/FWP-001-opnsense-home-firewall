# OPNsense Installation

> **Security note:** This public documentation intentionally omits passwords, management addresses, MAC addresses, serial numbers, backup files, exact test-server location, and other identifying or operational details.

## Installation Target

A dedicated x86-64 mini PC was repurposed as the OPNsense firewall appliance. Existing data on the system was not required and the internal SSD was dedicated to the firewall installation.

## Installation Media

- **Initial OPNsense version:** 26.7
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

## Isolated WAN / Double-NAT Lab Validation

The WAN interface was connected to a LAN port on the existing ISP router so OPNsense could be tested without interrupting the household Internet connection. Because the lab WAN receives a private RFC1918 address from the upstream router, private-network blocking on the WAN was disabled for the lab stage only.

The WAN interface successfully received a private DHCP lease and default gateway from the upstream router. From the test workstation behind OPNsense, the following checks all passed:

- ICMP connectivity to a public Internet address
- DNS resolution of a public hostname
- HTTPS web browsing to a public site

This confirms basic WAN DHCP, routing/NAT, LAN DHCP, DNS forwarding/resolution, and firewall pass-through behavior before production cutover.

## Known-Good Configuration Backup

After the successful isolated lab validation, a full configuration backup was exported and stored privately. The backup file is intentionally excluded from the public repository because OPNsense configuration exports can contain sensitive network and firewall information.

This backup provides a recovery point before firmware updates and later security configuration changes.

## Firmware Update Checkpoint

Before performance testing and production cutover, the firewall was updated from the installation image to **OPNsense 26.7.2_2**. Post-update management access, Internet routing, DNS resolution, and web access remained functional.

## Lab Performance Validation

Three wired speed tests were performed from the test workstation behind OPNsense while the firewall remained in the isolated double-NAT lab topology.

| Test | Ping | Download | Upload |
|---|---:|---:|---:|
| 1 | 11 ms | 931.07 Mbps | 860.61 Mbps |
| 2 | 8 ms | 903.96 Mbps | 874.04 Mbps |
| 3 | 8 ms | 912.14 Mbps | 896.26 Mbps |
| **Average** | **9 ms** | **915.72 Mbps** | **876.97 Mbps** |

Compared with the pre-OPNsense baseline of approximately 936.75 Mbps download, 881.10 Mbps upload, and 7 ms latency, the lab averages were approximately:

- **Download:** 2.24% lower
- **Upload:** 0.47% lower
- **Latency:** 2 ms higher

Because this test traverses both the existing ISP router and OPNsense, it is intentionally treated as a lab validation rather than the final production benchmark. Even with the additional routing/NAT layer, throughput remained within 5% of the original wired baseline.

At this checkpoint:

- [x] Bootable installer created
- [x] Installation media verified
- [x] Internal installation disk identified
- [x] ZFS installation completed
- [x] System rebooted from internal storage
- [x] OPNsense console reached successfully
- [x] Verify detected network interfaces
- [x] Assign WAN and LAN roles
- [x] Configure isolated lab network
- [x] Confirm web-management access
- [x] Export known-good lab configuration backup
- [x] Update firmware/packages before production cutover
- [x] Validate post-update connectivity
- [x] Measure lab throughput and latency

## Next Step

The firewall has passed the initial installation, connectivity, update, backup, and lab-performance checkpoints. The next stage is to finalize the production-ready base configuration and hardening settings before moving the WAN connection from the ISP router to OPNsense.
