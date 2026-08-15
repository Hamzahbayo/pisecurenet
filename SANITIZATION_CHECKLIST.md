# Sanitization Checklist — run before EVERY push

A public capstone repo must contain **no secrets**. Work through this list before committing.

## Never commit
- [ ] Private SSH keys (`id_*`, `*.pem`) — commit **public** keys only if any
- [ ] Wi-Fi passphrase — replace `wpa_passphrase=...` with `wpa_passphrase=CHANGkME` in `hostapd.conf.sample`
- [ ] Pi-hole / API tokens, web admin password hashes (`setupVars.conf` secrets)
- [ ] Real public IP addresses of your WAN link
- [ ] Real client MAC addresses in `dnsmasq` leases (redact to `aa:bb:cc:00:00:01`)
- [ ] Your home address, full name of household devices, or account emails inside configs
- [ ] `.bash_history`, `known_hosts`, or backups containing the above

## Sanitize, don't delete
- [ ] Keep config **structure** so the build is reproducible — replace only the secret values with obvious placeholders (`<REDACTED>`, `CHANGkME`, `10.0.0.0/24`)
- [ ] Rename real configs to `*.sample` (e.g., `hostapd.conf` → `hostapd.conf.sample`)
- [ ] In screenshots, blur/crop serial numbers, MACs, and any passphrase fields

## Add a `.gitignore`
```
# secrets & local state
*.pem
id_*
!*.pub
setupVars.conf
*.key
*.env
.DS_Store
# pihole / dnsmasq runtime
/etc/pihole/*
dnsmasq.leases
```

## Final pass
- [ ] `git grep -iE "password|passphrase|secret|token|BEGIN .*PRIVATE KEY"` returns nothing real
- [ ] Skim every screenshot at full size before adding
- [ ] Confirm the repo README links to the paper and the EVIDENCE table
- [ ] Make the repo **public** only after the two checks above pass
