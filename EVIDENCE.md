# Evidence — Falsifiable Claims

For every major security claim, PiSecureNet documents what proves it, **what would disprove it**, and how the evidence is collected. A control is only trusted once the disproving condition has been tested for and not found.

| Claim | What Proves It | What Would Disprove It | How It Is Collected |
|---|---|---|---|
| Default-drop firewall enforces policy | INPUT/FORWARD chains set to DROP with live packet counters (e.g., `[132:14488]`) after reboot | Traffic passes on blocked ports, counters stay at 0, or rules absent after reboot | `iptables -L -v -n`; reboot then re-verify; `netfilter-persistent` |
| SSH access is key-only | Key login succeeds; password login rejected | Any successful password login | Attempt password auth; confirm `PasswordAuthentication no` |
| DNS-layer blocking is active | Pi-hole blocks queries; blocked count increments | A known malicious/ad domain resolves, or block count static | Query a blocked domain; check Pi-hole dashboard & logs |
| IP forwarding persists across reboot | `net.ipv4.ip_forward = 1` after reboot via `/etc/sysctl.d/99-pisecurenet.conf` | Value reverts to 0, or ordering conflicts with `systemd-networkd` | Reboot; `sysctl net.ipv4.ip_forward` |
| Wi-Fi AP uses strong crypto | `hostapd wpa_pairwise = CCMP`; clients associate | TKIP or open auth present in `hostapd.conf` | Inspect `hostapd.conf`; client association test |

## Screenshots

Captures live in `docs/evidence/`.

**In hand (DNS-layer filtering — Pi-hole):**
- ✅ `pihole-dashboard.png` — 456 queries, 44 blocked (9.6%), 82,622 blocklist domains
- ✅ `pihole-recent-blocked.png` — malicious/tracking domains blocked in real time with timestamps
- ✅ `pihole-network-overview.png` — client `10.0.0.29` + MAC and gateway `10.0.0.1` (DHCP attribution)
- ✅ `pihole-top-blocked.png`, `pihole-upstream-servers.png`, `pihole-blocklists.png`, `pihole-baseline-zero.png`, `pihole-dns-failure.png` — supporting captures across the config timeline

**In hand (firewall, persistence, access control):**
- ✅ `fw-sysctl-ssh.png` — `sudo iptables -L -v -n` showing **INPUT policy DROP (1,364 packets dropped)** with only the seven explicit allow rules matching; `sysctl net.ipv4.ip_forward = 1` from the sysctl.d drop-in; and `PasswordAuthentication no`. **Strongest single capture.**
- ✅ `iptables-save-lease.png` — full `iptables-save` ruleset (INPUT/FORWARD policy DROP + POSTROUTING MASQUERADE) and a live DHCP lease binding `10.0.0.29` to a client.
- ✅ `nat-crypto-os.png` — NAT MASQUERADE, dnsmasq DHCP scope, WPA2 **CCMP** cipher, and Debian 13 (Trixie).

> Note: the raw `hostapd.conf` photo was **cropped to remove the `wpa_passphrase` line** before inclusion. Never commit the passphrase.

> **Key insight:** a static configuration is not proof of enforcement. An active packet counter observed *after a reboot* demonstrates rules are live and dropping traffic; a rule in a file proves only intent.
