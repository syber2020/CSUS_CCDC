# Blue Team Playbook (Defense Operations)

> Your field manual for the competition. Organized by **phase** (first 15 min →
> first hour → steady state) and by **platform**. Print it, split it by role, and
> work the checklists.

---

## 0. Team Roles (assign before the game)

| Role | Responsibility |
|---|---|
| **Captain** | Big picture, talks to White Team, approves risky changes, tracks score |
| **Inject Coordinator** | Owns every inject: logs, assigns, tracks deadlines, formats submissions |
| **Linux Lead(s)** | Harden & defend all Linux/Unix hosts |
| **Windows Lead(s)** | Harden & defend Windows / Active Directory |
| **Network/Firewall** | Firewall rules, segmentation, monitoring |
| **Scribe / IR** | Incident log, timeline, documentation for reports |

Everyone hardens their own boxes first; specialists float to help.

---

## 1. THE FIRST 15 MINUTES ⏱️ (highest leverage)

Do these **before** deep-diving anything. This is where games are won or lost.

1. **Inventory.** List every host, IP, OS, and scored service. (See the
   Network/Credentials README template.)
2. **Identify scored services and their ports** — do NOT break these.
3. **Find the scoring-engine accounts** your services need. Protect, don't delete.
4. **Change the passwords you can, fast**, starting with:
   - Local admin/root on every box
   - Service/app admin accounts (web app admin, DB root, etc.)
   - **Leave scoring/service accounts working** — coordinate with the person
     testing the service.
5. **Snapshot/back up** critical configs and scored data *before* you change things.
6. **Baseline:** quick list of running services, listening ports, and user
   accounts per host (so you can spot changes later).

> ⚠️ **Change one password at a time on scored services and verify the service
> still works / still passes the check.** A broken service costs uptime points.

---

## 2. THE FIRST HOUR 🔧 (lock it down)

### Credentials
- [ ] Unique, strong password on **every** account on **every** host (no reuse).
- [ ] Disable/remove unused and default accounts.
- [ ] Audit admin/root group membership — remove anyone who shouldn't be there.
- [ ] Rotate SSH keys / remove unknown `authorized_keys`.

### Attack surface
- [ ] Disable services you don't need (telnet, FTP, unused daemons).
- [ ] Close unused ports; firewall everything down to what's required.
- [ ] Restrict management access (SSH/RDP/WinRM) to a jump host or your subnet.

### Patch & mitigate
- [ ] Patch the **critical, known-exploited** vulns first (don't try to patch
      everything — prioritize what Red Team will use).
- [ ] If you can't patch, mitigate (disable the feature, firewall the port).

### Hunt for pre-planted access (assume it exists)
- [ ] Extra accounts, UID-0 users, admin group additions
- [ ] Cron jobs / scheduled tasks
- [ ] Unknown services/listeners
- [ ] Web shells in web roots
- [ ] Odd SSH keys, startup scripts

### Monitoring on
- [ ] Centralize/watch auth logs
- [ ] Watch listening ports and new processes
- [ ] File integrity on scored web content/data

---

## 3. STEADY STATE (rest of the game) 🔁

Run a continuous loop:

```
MONITOR → DETECT → CONTAIN → EVICT → RESTORE → DOCUMENT → repeat
```

- Keep watching logs, processes, ports, accounts.
- When you find Red Team activity: **contain** (isolate/kill), **evict** (remove
  persistence, rotate creds), **restore** (bring scored service back), **document**.
- Periodically **re-verify** scored services are up and passing checks.
- Keep knocking out injects.

---

## 4. Linux Hardening Checklist 🐧

```bash
# --- Accounts & auth ---
awk -F: '($3==0){print $1}' /etc/passwd      # find all UID-0 (root-equiv) accounts
cat /etc/passwd                               # review every account
passwd <user>                                 # change passwords
usermod -L <user>  /  usermod -s /sbin/nologin <user>   # lock/disable

# --- Find attacker persistence ---
crontab -l; ls -la /etc/cron* /var/spool/cron/   # cron jobs
for u in $(cut -f1 -d: /etc/passwd); do echo "== $u =="; cat /home/$u/.ssh/authorized_keys 2>/dev/null; done
cat /root/.ssh/authorized_keys 2>/dev/null
find / -perm -4000 -type f 2>/dev/null        # SUID binaries
systemctl list-unit-files --type=service --state=enabled
systemctl list-timers

# --- Services & ports ---
ss -tulpn                                     # what's listening + which process
systemctl disable --now <service>             # turn off what you don't need

# --- Sudo & integrity ---
cat /etc/sudoers; ls /etc/sudoers.d/          # audit privilege
find / -newermt "today" -type f 2>/dev/null   # recently changed files

# --- SSH hardening (/etc/ssh/sshd_config) ---
# PermitRootLogin no
# PasswordAuthentication yes|no (careful if scored)
# then: systemctl restart sshd

# --- Logs to watch ---
tail -f /var/log/auth.log       # Debian/Ubuntu
tail -f /var/log/secure         # RHEL/CentOS
last; lastb                     # login history / failed logins
```

**Firewall (pick what the box uses):**
```bash
# ufw (Ubuntu)
ufw default deny incoming; ufw allow <scored-port>; ufw enable
# firewalld (RHEL)
firewall-cmd --permanent --add-port=<port>/tcp; firewall-cmd --reload
# iptables (generic)
iptables -L -n -v
```

---

## 5. Windows Hardening Checklist 🪟

```powershell
# --- Accounts ---
net user                                  # list local users
net localgroup administrators             # who is admin
net user <name> <NewStrongPass!>          # change password
Disable-LocalUser -Name <name>            # disable account

# --- Persistence hunting ---
schtasks /query /fo LIST /v               # scheduled tasks
Get-Service | Where-Object {$_.Status -eq 'Running'}
Get-ItemProperty "HKLM:\Software\Microsoft\Windows\CurrentVersion\Run"
Get-ItemProperty "HKCU:\Software\Microsoft\Windows\CurrentVersion\Run"

# --- Network / listeners ---
netstat -ano                              # connections + PID
Get-NetTCPConnection -State Listen

# --- Firewall ---
netsh advfirewall set allprofiles state on
netsh advfirewall firewall add rule name="Block X" dir=in action=block ...

# --- Logs (Event Viewer) ---
# Security log: 4624 (logon), 4625 (failed logon), 4720 (user created),
#               4728/4732 (added to group), 4722 (account enabled)
```

**Active Directory (if present):**
- Limit **Domain Admins** to the minimum.
- Rotate the **`krbtgt`** account password (twice, correctly) if trained to.
- Reset privileged account passwords.
- Audit GPOs for anything suspicious.
- Disable RDP where not needed; restrict where it is.

---

## 6. Protecting Scored Services (don't lose uptime)

| Service type | Keep working | Harden |
|---|---|---|
| **Web (HTTP/HTTPS)** | Right port, app loads, scoring account logs in | Patch app, restrict uploads, remove admin pages, WAF if allowed |
| **DNS** | Resolves the records the engine checks | Lock down zone transfers, patch |
| **Mail (SMTP/IMAP/POP)** | Sends/receives for the test account | Disable open relay, patch, auth required |
| **Database** | App can connect | Strong root pass, no remote root, bind to localhost if app is local |
| **SSH/RDP** | Admins can get in | Restrict source, strong creds, no root/admin direct login if allowed |
| **File shares** | Engine can read/write test files | Remove anonymous access, tight permissions |

> **Golden rule:** change → **test the service** → confirm the scoring check still
> passes. Never assume.

---

## 7. Incident Response Mini-Procedure

When you spot Red Team:

1. **Confirm** — is it really malicious? (vs. scoring engine / teammate)
2. **Record** — timestamp, host, what you saw (scribe logs it).
3. **Contain** — kill the process/session, block the source if ROE allows and it
   won't break scoring.
4. **Evict** — remove persistence, rotate affected creds.
5. **Restore** — bring scored service back, verify.
6. **Verify eviction** — re-check they're gone (they often have a second foothold).
7. **Document** — finalize the incident entry for later reports/injects.

**Incident log template:**
```
Time | Host/IP | Observation | Indicator | Action Taken | Status | Who
-----|---------|-------------|-----------|--------------|--------|----
```

---

## 8. Common Mistakes That Lose Points

- ❌ Breaking a scored service while hardening (uptime loss).
- ❌ Blocking the scoring engine's IP.
- ❌ Changing a password the scoring engine uses.
- ❌ Deleting a service account because it "looked suspicious."
- ❌ Changing one password and thinking you evicted the attacker.
- ❌ Ignoring injects to chase the Red Team (injects are big points).
- ❌ No documentation — then the report inject is painful.
- ❌ Everyone working on the same box while others sit unhardened.

---

## 9. First-Hour One-Page Battle Plan (print this)

```
[ ] Inventory hosts + scored services + ports
[ ] Snapshot/back up configs & scored data
[ ] Identify scoring-engine accounts (protect)
[ ] Rotate ALL passwords (unique, strong) — verify services after
[ ] Disable default/unused accounts & services
[ ] Firewall down to required ports; restrict mgmt access
[ ] Hunt persistence: users, cron/tasks, keys, services, web shells
[ ] Patch/mitigate critical vulns
[ ] Turn on monitoring (auth logs, ports, processes, file integrity)
[ ] Start incident log + begin injects
```

🛡️ **Calm, organized, documented defense beats frantic clicking. Work the checklist.**
