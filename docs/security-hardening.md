# Security Hardening

> **Security note:** This public document intentionally omits the actual firewall hostname, management address, administrator usernames, passwords, OTP secrets, certificates, internal addressing, delegated IPv6 prefixes, global IPv6 addresses, and other operational details.

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

Validation confirmed that normal DNS resolution through OPNsense continued to work while direct test queries to public resolvers failed as expected. Firewall Live View also showed the controlled bypass attempt matching the DNS-block rule, providing log-level evidence that the policy was enforcing the intended traffic path.

A separate LAN rule was then added to block direct client DNS-over-TLS connections on TCP port 853. A controlled client connectivity test to an external resolver on TCP/853 returned a failed connection result, confirming that client DoT bypass is blocked while the firewall itself can continue using its configured encrypted upstream resolver.

## Step 8 — IPv6 Lab Validation

The test workstation received only a link-local IPv6 address (`fe80::/10`) and had no working end-to-end IPv6 connectivity in the double-NAT lab. A direct IPv6 connectivity test to a public IPv6 endpoint returned no replies.

Because no routed IPv6 client path was active in the lab, equivalent IPv6 DNS enforcement rules were deferred until the production WAN was connected and IPv6 behavior could be tested against the ISP handoff.

## Step 9 — Post-Hardening Recovery Point

After all lab hardening and DNS-enforcement checks passed, a fresh full OPNsense configuration backup was exported and stored privately. The backup is intentionally excluded from the public repository because configuration exports can contain sensitive operational data.

This creates a known-good recovery point immediately before the production WAN cutover.

## Step 10 — Production IPv6 Validation

After the direct ISP cutover, native IPv6 became available to LAN clients. A wireless client connected through the temporary downstream access point received globally routable IPv6 addressing and an IPv6 default route through OPNsense.

Production validation included:

- successful IPv6 ICMP reachability to a public IPv6 endpoint
- a completed IPv6 traceroute to a public Internet destination
- OPNsense appearing as the first IPv6 routing hop
- successful AAAA-record resolution through the local DNS path

Exact global IPv6 addresses and delegated prefix information are intentionally omitted from this public document.

This confirms that IPv6 is now an active production path rather than merely a link-local client capability.

## Step 11 — IPv6 DNS Enforcement

Equivalent IPv6 DNS-enforcement rules were added to the LAN ruleset above the broad IPv6 allow rule.

The policy order is:

1. Allow LAN clients to use the firewall's local DNS service over IPv6 on TCP/UDP port 53.
2. Block direct external IPv6 DNS queries from LAN clients on TCP/UDP port 53.
3. Block direct client IPv6 DNS-over-TLS connections on TCP port 853.
4. Allow other intended IPv6 LAN traffic according to the general LAN policy.

Validation confirmed that:

- normal DNS resolution through OPNsense continued to work
- direct DNS queries to an external IPv6 resolver failed as expected
- direct IPv6 TCP/853 connectivity to an external resolver failed as expected
- normal Internet browsing remained functional
- native IPv6 routing remained functional after the enforcement rules were applied

This closes the IPv6 equivalent of the previously validated IPv4 DNS-bypass path. The DNS policy now covers both active IP families for direct port 53 and DNS-over-TLS bypass attempts.

> **Scope note:** These firewall rules do not by themselves prevent DNS-over-HTTPS over TCP/443. DoH control is a separate policy problem and is not claimed as complete in this project checkpoint.

## Step 12 — Post-Cutover Recovery Point

After the production WAN cutover, temporary access-point validation, native IPv6 validation, and IPv4/IPv6 DNS-enforcement checks were completed successfully, a fresh OPNsense configuration backup was exported and stored privately.

The former ISP router configuration backup is also retained privately as part of the rollback plan. Neither backup is stored in the public repository because they may contain sensitive addressing, interface, certificate, firewall, or device information.

This creates a current recovery point representing the known-good production state.

## Next Step

Complete final production validation, including external exposure checks, temporary wireless stability, and documentation cleanup before considering the initial migration phase complete.
