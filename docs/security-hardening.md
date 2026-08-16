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

The encrypted resolver path will be independently verified with a WAN-side packet capture before DNS blocklists or DNS enforcement rules are added.

## Next Step

Verify that outbound DNS traffic from the firewall is actually using DNS-over-TLS on TCP port 853, then add DNS blocklist policy and DNS enforcement controls only after the transport path is confirmed.
