# Security Hardening

> **Security note:** This public document intentionally omits the actual firewall hostname, management address, administrator usernames, passwords, OTP secrets, certificates, internal addressing, and other operational details.

## Goal

Apply production-ready security settings incrementally, validating connectivity after each change so that any configuration issue can be isolated and rolled back quickly.

## Step 1 — System Identity, Time, DNS Behavior, and Administration Defaults

The initial hardening pass completed successfully.

Configured or verified:

- A non-public internal firewall hostname was selected.
- The internal domain uses the reserved `home.arpa` namespace rather than `.local`.
- The correct local time zone was configured.
- WAN-provided DNS override was disabled so the firewall does not automatically inherit upstream DNS settings.
- The firewall continues to use its local resolver path.
- Default gateway switching remains disabled because the design currently uses a single WAN.
- Web management remains HTTPS-only.
- SSH remains disabled.
- Management session timeout was reduced.
- DNS rebind protection and HTTP referrer enforcement remain enabled.

After saving these settings, the following validation checks passed:

- OPNsense WebGUI remained reachable.
- Public IP connectivity remained functional.
- DNS resolution remained functional.
- Normal HTTPS web browsing remained functional.

## Step 2 — Dedicated Administrator and TOTP

A separate administrator account was created and granted administrative access through the appropriate administrator group. A TOTP seed was generated and enrolled in a compatible authenticator application.

A dedicated `Local + Timebased One Time Password` authentication server was created and validated with OPNsense's authentication tester before the WebGUI authentication method was changed.

The WebGUI was then configured to use the TOTP-backed authentication server. A fresh private-browser login using the dedicated administrator account and TOTP succeeded, confirming that two-factor authentication is enforced for the normal WebGUI administration path.

The original privileged system account is retained only as a controlled recovery path rather than the routine WebGUI administrator.

## Step 3 — Unbound DNSSEC Validation

The built-in Unbound DNS resolver was retained as the network DNS service and DNSSEC validation was enabled. Client DNS resolution and normal web browsing were tested after the change and remained functional.

This adds validation of signed DNS responses before the project moves on to encrypted upstream DNS transport.

## Step 4 — DNS-over-TLS

Unbound was configured to forward external DNS requests to a certificate-validated DNS-over-TLS upstream on TCP port 853. Normal client DNS resolution and HTTPS browsing remained functional after the change.

A WAN-side packet capture was then performed while generating fresh DNS queries from a test client. Outbound TCP/853 traffic to the configured resolver was observed, confirming that the encrypted DNS transport path is active rather than relying on plaintext DNS fallback.

## Step 5 — DNS Blocklist Validation

A conservative predefined Unbound DNS blocklist was enabled using NXDOMAIN responses for blocked domains. Unbound reporting was enabled to verify activity and blocklist loading.

At the validation checkpoint, the resolver reported a blocklist containing approximately 160,000 entries. Normal browsing and commonly used services continued to function correctly, so the blocklist was retained without adding additional overlapping lists.

The high initial blocked-query percentage was treated as a short-window observation rather than a final effectiveness metric because reporting had only recently been enabled.

## Step 6 — Automated Blocklist Refresh

The built-in Unbound DNS blocklist update task was scheduled to run once per day during an off-hours maintenance window. This keeps the selected blocklist current without requiring manual updates.

## Step 7 — IPv4 DNS Enforcement

LAN firewall rules were added above the broad LAN allow rule so clients can use the firewall's local resolver while direct IPv4 DNS queries to external resolvers on TCP/UDP port 53 are blocked.

Validation confirmed that normal DNS resolution through OPNsense continued to work while direct test queries to public resolvers failed as expected.

A separate LAN rule was then added to block direct client DNS-over-TLS connections on TCP port 853. A controlled client connectivity test to an external resolver on TCP/853 returned a failed connection result, confirming that client DoT bypass is blocked while the firewall itself can continue using its configured encrypted upstream resolver.

## Next Step

Evaluate the current IPv6 client path and apply equivalent DNS enforcement where necessary so that filtering cannot be bypassed over IPv6. After IPv4 and IPv6 DNS policy are validated, export a fresh configuration backup and prepare for production cutover.
