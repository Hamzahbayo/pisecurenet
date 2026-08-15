# How to put PiSecureNet on GitHub (step by step)

You have two easy paths. **Path A (web upload)** needs no tools and is fastest for a one-time submission. **Path B (git)** is cleaner if you want to keep updating it.

---

## Before you start — assemble the folder

On your computer, make one folder called `pisecurenet` and put these inside (I've generated all of them for you):

```
pisecurenet/
├── README.md
├── EVIDENCE.md
├── SANITIZATION_CHECKLIST.md
├── LICENSE
├── .gitignore
├── config/
│   ├── rules.v4
│   ├── hostapd.conf.sample      <- passphrase already redacted
│   ├── dnsmasq.conf.sample
│   ├── sshd_config.snippet
│   └── 99-pisecurenet.conf
├── scripts/
│   └── verify.sh
└── docs/
    ├── PiSecureNet_Capstone_Paper.pdf
    └── evidence/                <- put your screenshots here
        ├── pihole-dashboard.png
        ├── pihole-recent-blocked.png
        ├── pihole-network-overview.png
        ├── firewall-counters.png       <- new (crop out nothing sensitive)
        ├── sysctl-postreboot.png       <- new
        ├── ssh-hostapd.png             <- CROP OUT the wpa_passphrase line first
        └── iptables-save.png           <- new
```

> ⚠️ Run through `SANITIZATION_CHECKLIST.md` first. Above all: the `ssh-hostapd`
> screenshot shows your Wi-Fi passphrase — crop that line out before adding it.

---

## Path A — Upload through the website (no install)

1. Go to **github.com** and sign in (create a free account if needed).
2. Click the **+** (top right) → **New repository**.
3. Repository name: `pisecurenet`. Description: *"Hardened Raspberry Pi 5 SOHO router — CYBR 4350 capstone."*
4. Choose **Public** (so your professor can open the link). Leave "Add a README" **unchecked** (you already have one).
5. Click **Create repository**.
6. On the next page click **uploading an existing file**.
7. Drag your whole `pisecurenet` folder contents in (or zip → it keeps folders). Wait for upload.
8. Under "Commit changes" type: `Initial commit — PiSecureNet capstone`. Click **Commit changes**.
9. Your link is the page URL: `https://github.com/<your-username>/pisecurenet`. Submit that.

## Path B — Git from the command line (optional, cleaner)

On your computer (or the Pi) with git installed:

```bash
cd pisecurenet
git init
git add .
git commit -m "Initial commit — PiSecureNet capstone"
git branch -M main
# create the empty repo on github.com first (steps 1-5 above), then:
git remote add origin https://github.com/<your-username>/pisecurenet.git
git push -u origin main
```

---

## Make it presentable

- The `README.md` renders automatically as your front page — it already has the overview, topology, stack table, and reproduce steps.
- Add a short **About** blurb (gear icon, top right of the repo): *"A low-cost, evidence-backed hardened router for SOHO networks. Default-drop firewall, DNS-layer filtering, WPA2/CCMP, key-only SSH."*
- After it's public, paste the link back to me and I'll wire it into the paper and portfolio.
