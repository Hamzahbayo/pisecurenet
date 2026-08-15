# PiSecureNet

**A hardened Raspberry Pi 5 router and gateway for small-office / home-office (SOHO) network security.**

CYBR 4350 Senior Capstone · Collin College · Hamzah Bayo

---

## Overview

SOHO networks face enterprise-grade threats without enterprise budgets or staff. **PiSecureNet** layers four independent security controls on a single ~$80 Raspberry Pi 5 to raise a small network's defensive posture, and validates every control with reproducible evidence.

| Layer | Control | Protects |
|---|---|---|
| Firewall | `iptables` default-drop policy + NAT, persisted with `netfilter-persistent` | Confidentiality, Integrity |
| DNS filtering | Pi-hole v6 blocks malicious/ad domains at resolution | Confidentiality, Integrity |
| DHCP + logging | `dnsmasq` DHCP/DNS caching with lease logging | Availability of forensic evidence |
| Wireless | `hostapd` WPA2 access point (CCMP/AES) | Confidentiality |
| Remote admin | OpenSSH, key-only (password auth disabled) | Confidentiality |

**OS:** Raspberry Pi OS Lite 64-bit (Debian 13 "Trixie")

## Network topology

```
Internet ── ISP gateway ──(Cat6)── [ Raspberry Pi 5: PiSecureNet ] ──(Wi-Fi AP)── SOHO clients
                                     eth0 (WAN)   wlan0 10.0.0.1/24
                                     router · firewall · DHCP · DNS · AP
```

## Repository structure

```
pisecurenet/
├── README.md                      <- you are here
├── docs/
│   ├── PiSecureNet_Capstone_Paper.pdf   <- full report (paper)
│   ├── architecture.png                 <- topology diagram
│   └── evidence/                        <- screenshots (see below)
│       ├── iptables-counters.png
│       ├── pihole-dashboard.png
│       └── sysctl-postreboot.png
├── config/                        <- SANITIZED configs (no secrets)
│   ├── iptables.rules
│   ├── sshd_config.snippet
│   ├── hostapd.conf.sample
│   ├── dnsmasq.conf.sample
│   └── 99-pisecurenet.conf        <- /etc/sysctl.d drop-in
├── scripts/
│   └── verify.sh                  <- re-runs the evidence checks
├── EVIDENCE.md                    <- falsifiable-claims table
├── .gitignore
└── LICENSE
```

## Reproduce the build

> All commands run on Raspberry Pi OS Lite (Debian 13). Keep a **physical console** available before changing SSH or networking.

1. **Default-drop firewall + NAT**
   ```bash
   sudo iptables -P INPUT DROP
   sudo iptables -P FORWARD DROP
   sudo iptables -A INPUT -i lo -j ACCEPT
   sudo iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT
   sudo iptables -A INPUT -p tcp --dport 22 -j ACCEPT      # SSH
   sudo iptables -A INPUT -p udp --dport 53 -j ACCEPT      # DNS
   sudo iptables -A INPUT -p tcp --dport 53 -j ACCEPT
   sudo iptables -A INPUT -p udp --dport 67:68 -j ACCEPT   # DHCP
   sudo iptables -A INPUT -p tcp --dport 80 -j ACCEPT      # Pi-hole admin
   sudo iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
   sudo netfilter-persistent save
   ```
2. **IP forwarding persistence (Trixie / systemd-networkd)**
   ```bash
   echo 'net.ipv4.ip_forward = 1' | sudo tee /etc/sysctl.d/99-pisecurenet.conf
   sudo sysctl --system
   ```
3. **SSH hardening** — set in `/etc/ssh/sshd_config`: `PasswordAuthentication no`, `PubkeyAuthentication yes`, `PermitRootLogin no`, then `sudo systemctl restart ssh`.
4. **Wireless AP** — `hostapd` with `wpa_pairwise=CCMP` (see `config/hostapd.conf.sample`).
5. **DNS filtering** — install Pi-hole v6; point `dnsmasq` DHCP clients at the Pi.

## Verify (evidence checks)

```bash
sudo iptables -L -v -n        # non-zero DROP counters on INPUT/FORWARD after reboot
sudo sshd -T | grep -i passwordauthentication   # -> passwordauthentication no
sysctl net.ipv4.ip_forward    # -> net.ipv4.ip_forward = 1  (after reboot)
# Pi-hole: query a known-blocked domain, confirm block + incremented count
```

See **[EVIDENCE.md](EVIDENCE.md)** for the full falsifiable-claims table (claim / what proves it / **what would disprove it** / how collected).

## Scope

**In scope:** one Pi 5, one SOHO subnet, the five controls above.
**Out of scope (stated for credibility):** multi-subnet segmentation, line-rate IPS, enterprise identity federation.
**Deferred (future work):** anomaly detection over Pi-hole DNS logs (stretch goal; lease + query logs already positioned as its input).

## ⚠️ Security note

This repo contains **sanitized** configuration only. Do **not** commit private keys, Wi-Fi passphrases, real MAC addresses, public IPs, or `pihole` API tokens. See `SANITIZATION_CHECKLIST.md` before every push.

## Author

Hamzah Bayo — Dalla/ Fortworth, TX
