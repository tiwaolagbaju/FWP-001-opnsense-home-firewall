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

### Stage 2 — Build and Validate OPNsense in the Lab

- Install OPNsense on the dedicated x86-64 mini PC.
- Assign separate WAN and LAN physical interfaces.
- Confirm console and web-management access from a directly connected test client.
- Update OPNsense before production use.
- Confirm DHCP, DNS, routing, and firewall functionality in an isolated test setup.
- Export an encrypted OPNsense configuration backup after the known-good lab configuration is complete.

### Stage 3 — Controlled Production Cutover

- Schedule a maintenance window.
- Disconnect the ISP Ethernet handoff from the existing router.
- Connect the ISP handoff to the OPNsense WAN interface.
- Connect the OPNsense LAN interface to the internal switching path.
- Verify that the WAN receives connectivity from the ISP.
- Verify DNS resolution and Internet access from one wired trusted client before reconnecting the rest of the network.
- Reconnect temporary access points and remaining trusted devices only after basic routing is confirmed.

### Stage 4 — Validation

Validate the following before considering the cutover successful:

- WAN connectivity is stable.
- A trusted wired client receives an address through DHCP.
- DNS resolution works.
- Internet browsing works.
- Wired throughput is reasonably close to the pre-change baseline.
- Latency has not increased materially.
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
3. Reconnect the original LAN path if it was changed.
4. Power/restart the original router if necessary and allow it to reacquire service.
5. Confirm client DHCP, DNS, and Internet access.
6. Use the saved router configuration backup only if the router configuration itself was changed and needs restoration.
7. Leave OPNsense offline for troubleshooting without affecting household connectivity.

## Safety Rules

- Do not factory-reset the ISP router before the OPNsense migration is proven stable.
- Do not delete the ISP-router configuration backup.
- Do not expose the OPNsense WebGUI or SSH directly to the WAN.
- Keep local console access available during the initial deployment.
- Back up OPNsense before and after major configuration milestones.
- Make one major change at a time and test before continuing.

## Success Criteria

The migration is considered successful when OPNsense can provide stable routing, DHCP/DNS services, firewall enforcement, and wired Internet performance reasonably close to the documented pre-change baseline, with a repeatable recovery path available if needed.
