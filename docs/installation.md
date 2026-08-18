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

The validated production assignment is:

- **LAN:** `igc0` — preserves the faster 2.5 GbE interface for the internal network and future managed switching
- **WAN:** `re0` — matches the current 1 Gig WAN requirement

No LAGG or production VLAN interfaces were required for the initial migration phase. VLANs are planned for the later segmentation phase after the stable single-LAN deployment.

## Initial LAN Validation

A directly connected test workstation successfully received a DHCP lease from the OPNsense LAN interface and reached the OPNsense web-management interface over HTTPS. This confirmed that the LAN interface, DHCP service, and local management path were functioning before the WAN side was connected.

## Isolated WAN / Double-NAT Lab Validation

The WAN interface was connected to a LAN port on the existing ISP router so OPNsense could be tested without interrupting the household Internet connection. Because the lab WAN received a private RFC1918 address from the upstream router, private-network blocking on the WAN was disabled for the lab stage only.

The WAN interface successfully received a private DHCP lease and default gateway from the upstream router. From the test workstation behind OPNsense, the following checks all passed:

- ICMP connectivity to a public Internet address
- DNS resolution of a public hostname
- HTTPS web browsing to a public site

This confirmed basic WAN DHCP, routing/NAT, LAN DHCP, DNS forwarding/resolution, and firewall pass-through behavior before production cutover.

## Known-Good Configuration Backup

After the successful isolated lab validation, a full configuration backup was exported and stored privately. The backup file is intentionally excluded from the public repository because OPNsense configuration exports can contain sensitive network and firewall information.

This created a recovery point before firmware updates and later security configuration changes.

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

Because this test traversed both the existing ISP router and OPNsense, it was treated as a lab validation rather than the final production benchmark. Even with the additional routing/NAT layer, throughput remained within 5% of the original wired baseline.

## Production WAN Performance Validation

After the ISP Ethernet handoff was moved directly to OPNsense and the firewall obtained a public WAN lease, three additional wired speed tests were performed from the trusted test workstation.

| Test | Ping | Download | Upload |
|---|---:|---:|---:|
| 1 | 5 ms | 938.19 Mbps | 924.27 Mbps |
| 2 | 8 ms | 937.68 Mbps | 912.77 Mbps |
| 3 | 8 ms | 937.05 Mbps | 922.28 Mbps |
| **Average** | **7 ms** | **937.64 Mbps** | **919.77 Mbps** |

Compared with the original pre-OPNsense wired baseline of approximately 936.75 Mbps download, 881.10 Mbps upload, and 7 ms latency:

- **Download:** approximately 0.10% higher
- **Upload:** approximately 4.39% higher
- **Latency:** unchanged on average

The production test therefore met the project's performance target with no meaningful throughput or latency penalty from replacing the ISP router with OPNsense.

## Automatic Power Recovery

The mini PC firmware was configured with **AC power loss control = Always on**. This ensures the firewall appliance automatically powers back on after utility power is restored rather than requiring a manual power-button press.

A controlled power-restore test was completed successfully. After AC power was removed and restored, the mini PC powered itself back on without a manual power-button press and OPNsense booted normally.

## Production Installation Status

The installation phase is complete. The appliance has progressed from bench installation through lab validation and into production operation on the direct ISP handoff.

Completed checkpoints:

- [x] Bootable installer created
- [x] Installation media verified
- [x] Internal installation disk identified
- [x] ZFS installation completed
- [x] System rebooted from internal storage
- [x] OPNsense console reached successfully
- [x] Network interfaces detected and assigned
- [x] Isolated lab network configured
- [x] Web-management access confirmed
- [x] Known-good lab configuration backup exported
- [x] Firmware/packages updated before production cutover
- [x] Post-update connectivity validated
- [x] Lab throughput and latency measured
- [x] Automatic startup after AC power restoration configured
- [x] Automatic power recovery validated
- [x] Direct public WAN connectivity validated
- [x] Production throughput and latency measured
- [x] Temporary downstream wireless access point validated
- [x] Native IPv6 validated after production cutover
- [x] Post-cutover configuration backup exported

## Final Installation Outcome

The dedicated mini PC is now operating as the production OPNsense edge firewall/router. The validated interface assignment, direct ISP handoff, power-recovery behavior, and measured throughput all met the project requirements. Security hardening, DNS enforcement, IPv6 validation, WAN exposure testing, and rollback details are documented separately in the other project files.
