# Network Design, IP Plan & Credential Inventory

> **This is a documentation TEMPLATE for your own practice lab / competition
> network.** Fill in the real values during setup. The credentials shown are
> **placeholders and well-known defaults** — their only purpose is to remind you
> **what to change first**. Never store real competition secrets in a shared repo;
> keep the live credential sheet in a controlled place (password manager or a
> printed sheet the captain holds).

---

## 1. Example Network Design

A typical CCDC-style small-business topology. Use this as a reference to design
your **practice lab** (e.g., in VMware/VirtualBox/Proxmox) so you train on
something close to the real thing.

```
                        ┌─────────────────────────┐
                        │   Scoring Engine / WAN   │   (out of your control)
                        └───────────┬─────────────┘
                                    │
                        ┌───────────┴─────────────┐
                        │   Edge Firewall/Router   │  10.0.0.1
                        │   (pfSense / Cisco)      │
                        └───────────┬─────────────┘
          ┌─────────────────────────┼──────────────────────────┐
          │                         │                           │
   ┌──────┴──────┐          ┌───────┴────────┐          ┌───────┴────────┐
   │ DMZ Segment │          │  Internal LAN  │          │  Mgmt Segment  │
   │ 172.16.1.0/24│         │ 192.168.10.0/24│          │ 192.168.99.0/24│
   └──────┬──────┘          └───────┬────────┘          └───────┬────────┘
          │                         │                           │
  Public-facing services    Internal servers/clients      Jump host, monitoring
```

**Segmentation principle:** public services in the DMZ, internal servers on the
LAN, and a separate management segment. Management ports (SSH/RDP/WinRM) reachable
**only** from the jump host. This limits lateral movement after a breach.

---

## 2. IP Address Plan (template)

| Segment | CIDR | Gateway | Purpose |
|---|---|---|---|
| DMZ | `172.16.1.0/24` | `172.16.1.1` | Internet-facing scored services |
| Internal LAN | `192.168.10.0/24` | `192.168.10.1` | App/DB/internal servers, clients |
| Management | `192.168.99.0/24` | `192.168.99.1` | Jump host, monitoring, admin |

### Host Inventory (fill in during the first 15 minutes)

| # | Hostname | IP | OS / Version | Role | Scored Service(s) | Scored Port(s) |
|---|----------|-----|--------------|------|-------------------|----------------|
| 1 | fw01 | 10.0.0.1 | pfSense x.x | Firewall/Router | — | — |
| 2 | web01 | 172.16.1.10 | Ubuntu xx.xx | Web server | HTTP/HTTPS | 80, 443 |
| 3 | dns01 | 172.16.1.11 | CentOS x | DNS | DNS | 53 |
| 4 | mail01 | 172.16.1.12 | Debian xx | Mail | SMTP/IMAP | 25, 143 |
| 5 | db01 | 192.168.10.20 | Ubuntu xx.xx | Database | MySQL/Postgres | 3306/5432 |
| 6 | ad01 | 192.168.10.5 | Win Server 20xx | Domain Controller | AD/DNS/LDAP | 389, 88, 53 |
| 7 | app01 | 192.168.10.21 | Win Server 20xx | App server | HTTP / custom | varies |
| 8 | ws01 | 192.168.10.100 | Windows 10/11 | Workstation | — | — |
| 9 | jump01 | 192.168.99.10 | Linux | Jump/admin host | — | — |

> ✏️ **Add columns you need:** "Hardened? ✅/❌", "Passwords rotated? ✅/❌",
> "Persistence checked? ✅/❌", "Owner".

---

## 3. Credential Inventory & Rotation Tracker

> **Priority #1 action in the game: rotate every one of these.** Track it here.
> The "Default" column lists *common well-known defaults* so you know what you're
> replacing — **assume the Red Team already knows all of them.**

| Host | Account | Common Default (CHANGE!) | New Pass Set? | Scoring uses it? | Notes |
|------|---------|--------------------------|:---:|:---:|-------|
| web01 | root | (os default / `toor`) | ☐ | ☐ | verify web still serves after change |
| web01 | web-admin | `admin` / `password` | ☐ | ✅ maybe | coordinate before changing |
| dns01 | root | — | ☐ | ☐ | |
| mail01 | root | — | ☐ | ☐ | |
| mail01 | test-mailbox | — | ☐ | ✅ | **scoring account — don't break** |
| db01 | mysql root | blank / `root` | ☐ | ☐ | app must still connect |
| db01 | app-db-user | `app`/`app` | ☐ | ✅ | update app config too |
| ad01 | Administrator | `P@ssw0rd` etc. | ☐ | ☐ | |
| ad01 | Domain Admins | — | ☐ | ☐ | reset all privileged |
| app01 | Administrator | — | ☐ | ☐ | |
| ws01 | local admin | — | ☐ | ☐ | |
| jump01 | admin | — | ☐ | ☐ | |

### Well-known default credentials to hunt for and change
These are published defaults for common gear/software — **not secrets**, just a
checklist of what ships weak:

- Network/appliances: `admin/admin`, `admin/password`, `cisco/cisco`, `root/calvin`
- Linux: `root/toor`, `root/root`, `user/user`
- Databases: MySQL root with **blank** password; `postgres/postgres`
- Web apps/CMS: `admin/admin`, `admin/changeme`, default installer accounts
- IPMI/BMC, printers, cameras, routers: vendor defaults (look them up for the
  model you're given)

> If the box came with it, **change it or disable it.** If an account is used by
> the scoring engine, change it **carefully** and update wherever it's referenced.

---

## 4. Firewall / Access Policy (template)

Document intended rules so the whole team agrees:

| # | Source | Destination | Port/Proto | Action | Reason |
|---|--------|-------------|-----------|--------|--------|
| 1 | Scoring engine | DMZ services | scored ports | ALLOW | must never block |
| 2 | Internal LAN | DMZ services | scored ports | ALLOW | normal use |
| 3 | jump01 only | all hosts | SSH/RDP/WinRM | ALLOW | management only |
| 4 | any | all hosts | mgmt ports | DENY | block direct admin |
| 5 | any | any | unused ports | DENY | shrink attack surface |
| 6 | any | any | all else | DENY (default) | deny by default |

> ⚠️ **Rule 1 is sacred.** Verify the scoring engine's source before writing any
> block rule. A default-deny posture that accidentally blocks scoring = disaster.

---

## 5. Secure Handling of the Real Credential Sheet

- Keep the **live** credential list **out of this repo** and off shared drives.
- Use a team password manager, or a single printed master sheet the captain controls.
- Record: host, account, **current** password, who changed it, when.
- Use strong, unique passwords (16+ chars, generated). Don't reuse across hosts —
  reuse is exactly how lateral movement succeeds.
- After the competition, consider all of it burned; never reuse elsewhere.

---

## 6. Quick-Fill Inventory Script (practice lab)

Run on a Linux host to speed up documentation of **your own** systems:

```bash
echo "=== HOSTNAME ==="; hostname
echo "=== IPs ==="; ip -brief addr
echo "=== LISTENING PORTS + PROCESS ==="; ss -tulpn
echo "=== USERS ==="; cut -d: -f1,3 /etc/passwd
echo "=== UID 0 (root-equiv) ==="; awk -F: '$3==0{print $1}' /etc/passwd
echo "=== ENABLED SERVICES ==="; systemctl list-unit-files --state=enabled 2>/dev/null
echo "=== SUDOERS ==="; cat /etc/sudoers; ls /etc/sudoers.d/ 2>/dev/null
```

Paste the output into your host inventory. Do the equivalent with PowerShell on
Windows (`ipconfig`, `net user`, `Get-NetTCPConnection -State Listen`,
`Get-Service`).

---

*Fill this in fast and completely in the opening minutes — a good inventory is the
foundation every other defensive decision rests on.*
