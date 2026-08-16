# Cutover and Rollback Plan

> **Security note:** This public plan intentionally omits exact IP addresses, management URLs, credentials, SSIDs, MAC addresses, cable-drop locations, and other operational details that could expose the home environment.

## Objective

Replace the ISP-provided router as the primary edge router/firewall with OPNsense while preserving a fast, low-risk rollback path.

## Change Strategy

The migration will be performed in controlled stages rather than changing the entire network at once.

### Stage 1 — Preserve the Existing Environment

- Export and securely store a backup of the ISP router configuration.
- Keep the ISP router available and unmodified enough to resume normal routing if needed.
- Record the current wired performance baseline.
- Label WAN and LAN cables before moving them.
- Reserve a future management address for the ISP router outside the OPNsense DHCP pool before converting it to a temporary access point.

### Stage 2 — Build and Validate OPNsense in the Lab

- Install OPNsense on the dedicated x86-64 mini PC.
- Assign separate WAN and LAN physical interfaces.
- Confirm console and web-management access from a directly connected test client.
- Update OPNsense before production use.
- Confirm DHCP, DNS, routing, firewall, DNS hardening, and DNS-enforcement functionality in an isolated test setup.
- Export a full OPNsense configuration backup after the known-good lab configuration is complete and store it privately.

### Stage 3 — Controlled Production Cutover

- Schedule a maintenance window.
- Keep the ISP router in its original configuration until OPNsense has been proven directly on the ISP handoff.
- Disconnect the ISP Ethernet handoff from the existing router.
- Connect the ISP handoff to the OPNsense WAN interface.
- Connect one wired trusted test client directly to the OPNsense LAN interface.
- Verify that the OPNsense WAN receives working connectivity from the ISP.
- Verify DHCP, DNS resolution, Internet access, and WebGUI access from the wired test client.
- Restore normal public-WAN protections after confirming the WAN is no longer using a private upstream address.
- Only after the direct OPNsense path is proven should the temporary wireless access point be prepared and connected.

**Current cutover status:** The ISP router DHCP lease was released, the ONT Ethernet handoff was moved directly to the OPNsense WAN interface, and OPNsense successfully obtained a public IPv4 WAN lease. The actual public address is intentionally not recorded in this repository. Direct wired connectivity, DNS resolution, normal web access, and internal WebGUI access were then validated. The WAN private-network protection was re-enabled for the production WAN and connectivity remained functional after the change.

### Stage 3A — Temporary ISP Router Access-Point Role

The original ISP router may be reused temporarily for wireless coverage after OPNsense is confirmed as the working edge router.

- Assign the ISP router a fixed LAN management address outside the OPNsense DHCP pool.
- Disable its DHCP server so OPNsense remains the only DHCP authority on the trusted LAN.
- Keep its wireless radios enabled for temporary Wi-Fi service.
- Leave its WAN interface unused for the temporary access-point role.
- Connect an OPNsense LAN/switch port to an ISP-router **LAN** port, creating a LAN-to-LAN connection.
- Confirm wireless clients receive OPNsense-issued addresses, use OPNsense as their gateway/DNS path, and can reach the Internet.
- Confirm the ISP router remains reachable at its reserved management address.

### Stage 4 — Validation

Validate the following before considering the cutover successful:

- WAN connectivity is stable.
- A trusted wired client receives an address through DHCP.
- DNS resolution works.
- Internet browsing works.
- Wired throughput is reasonably close to the pre-change baseline.
- Latency has not increased materially.
- Temporary Wi-Fi clients receive addressing from OPNsense rather than the former router.
- OPNsense management is reachable only from intended internal/admin devices.
- No management services are exposed intentionally to the WAN.

### Stage 5 — Rollback Trigger

Rollback will be initiated if a blocking issue cannot be resolved within the planned maintenance window, including:

- OPNsense does not obtain working WAN connectivity.
- DHCP or DNS fails for trusted clients and cannot be restored quickly.
- Sustained throughput is significantly below the pre-change baseline without an identified cause.
- The firewall becomes inaccessible through both the internal network and local console.

## Rollback Procedure

1. Disconnect the ISP handoff from the OPNsense WAN interface.
2. Reconnect the ISP handoff to the original ISP router WAN interface.
3. Restore the original ISP-router LAN path if it had already been changed for access-point testing.
4. If the ISP router had been converted to access-point operation, restore its saved configuration before relying on it as the primary router again.
5. Power/restart the original router if necessary and allow it to reacquire service.
6. Confirm client DHCP, DNS, Wi-Fi, and Internet access.
7. Leave OPNsense offline for troubleshooting without affecting household connectivity.

## Safety Rules

- Do not factory-reset the ISP router before the OPNsense migration is proven stable.
- Do not delete the ISP-router configuration backup.
- Do not convert the ISP router to access-point operation before the direct OPNsense WAN path has been validated.
- Do not connect OPNsense LAN to the ISP router WAN port when the ISP router is being used as a simple access point.
- Do not expose the OPNsense WebGUI or SSH directly to the WAN.
- Keep local console access available during the initial deployment.
- Back up OPNsense before and after major configuration milestones.
- Make one major change at a time and test before continuing.

## Success Criteria

The migration is considered successful when OPNsense can provide stable routing, DHCP/DNS services, firewall enforcement, wired Internet performance reasonably close to the documented pre-change baseline, and temporary Wi-Fi through a downstream access point, with a repeatable recovery path available if needed.
