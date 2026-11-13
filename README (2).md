# Red Team Playbook (Know Your Adversary)

> **Important framing:** In CCDC, **you are Blue Team** — you do not attack
> anyone. This "playbook" exists so you can *think like the Red Team* and
> anticipate their moves. Every attacker technique below is paired with **what to
> watch for** and **how to defend**. Use this to harden faster, not to attack.
>
> Everything here is standard, publicly documented security knowledge
> (NIST, MITRE ATT&CK, vendor docs). Keep it that way — practice only on your own
> lab or assigned competition systems.

---

## 1. The Attacker's Mindset & Kill Chain

CCDC Red Teams are professionals working fast against a known-vulnerable network.
They follow a predictable lifecycle (MITRE ATT&CK stages):

```
Recon → Initial Access → Execution → Persistence → Privilege Escalation
   → Defense Evasion → Credential Access → Discovery → Lateral Movement
   → Collection → Exfiltration → Impact
```

Your defense mirrors it: **detect early, deny persistence, rotate credentials,
contain lateral movement.**

---

## 2. Reconnaissance

**What they do:** Map your network before and during the game — live hosts, open
ports, service versions, and default/obvious credentials.

| Attacker activity | Example tooling (so you recognize it) | Blue Team response |
|---|---|---|
| Host discovery / port scanning | `nmap`, `masscan` | Expect it immediately. Scanning isn't the threat — what they find is. Close unused ports |
| Service/version fingerprinting | `nmap -sV`, banner grabbing | Minimize banners, patch known-vuln versions first |
| Web app mapping | `gobuster`, `nikto`, `dirb` | Remove default/admin pages, test apps yourself first |

> **Key takeaway:** You can't stop recon. You *can* make sure it turns up a
> hardened, patched, minimal-attack-surface network. **Do your own nmap scan
> first** and fix what you find before they do.

---

## 3. Initial Access — The Low-Hanging Fruit

Red Team almost always gets in through things the organizers left broken on
purpose. These are your **first-hour priorities**.

| Entry vector | Why it works in CCDC | Fix it FIRST |
|---|---|---|
| **Default / weak credentials** | Boxes ship with `admin/admin`, `root/toor`, etc. | **Rotate every password** (see Blue Team playbook) |
| **Reused passwords** | Same password across many hosts | Unique strong passwords per host/account |
| **Exposed services** | Telnet, FTP, old SMB, open databases | Disable unneeded services, firewall the rest |
| **Unpatched known-vuln software** | Old Apache/Struts/Samba/OS versions | Patch or mitigate the critical ones early |
| **Web app flaws** | SQLi, file upload, auth bypass in scored web apps | Review app config, restrict uploads, patch |
| **Already-planted backdoors** | Organizers/Red Team pre-seed access | Hunt for extra accounts, cron jobs, services |

> 🔑 **The single highest-value action in the first 15 minutes is rotating
> credentials** — but do it without breaking scored/scoring-engine accounts.

---

## 4. Execution & Persistence — How They Stay In

Assume they get a foothold. Their goal becomes *surviving your cleanup*. Persistence
is where most teams lose points for the rest of the day because they never fully
evicted the attacker.

**Common persistence mechanisms — hunt all of these:**

### Linux
- Extra or UID-0 accounts in `/etc/passwd`; check `awk -F: '$3==0' /etc/passwd`
- Added SSH keys in `~/.ssh/authorized_keys` (every user, including root)
- Malicious cron jobs: `crontab -l`, `/etc/cron*`, `/var/spool/cron/`
- `systemd` services/timers, `rc.local`, profile scripts (`~/.bashrc`, `/etc/profile.d/`)
- Modified SUID binaries; `find / -perm -4000 2>/dev/null`
- Web shells in web roots (`/var/www/`) — unexpected `.php`/`.jsp` files

### Windows
- New local/domain accounts; members added to Administrators
- Scheduled Tasks (`schtasks /query`), Run keys in the registry
- New or modified Services
- WMI event subscriptions
- Added RDP access, sticky-keys/accessibility backdoors

> **Eviction mindset:** changing one password isn't eviction. You must remove
> *every* persistence foothold or they log right back in. Document each one you
> find (great inject/report material).

---

## 5. Privilege Escalation

**What they do:** Turn a low-priv foothold into root/SYSTEM/Domain Admin.

| Technique | Blue Team hardening |
|---|---|
| Kernel/OS exploits (unpatched) | Patch; this is why patching matters |
| Misconfigured `sudo` | Audit `/etc/sudoers` and `sudoers.d` |
| Writable SUID/service binaries | Fix permissions; audit SUID set |
| Weak service account privileges | Least privilege for service accounts |
| Stored/cached credentials | Clear, rotate, limit who's admin |
| Windows unquoted service paths, token abuse | Fix service configs, limit admin membership |

---

## 6. Credential Access & Lateral Movement

Once they own one box, they harvest credentials and pivot to others.

| Attacker goal | Tooling you'll hear about | Defense |
|---|---|---|
| Dump hashes/passwords | `mimikatz`, `/etc/shadow`, LSASS | Rotate creds, limit admin, enable protections |
| Pass-the-hash / reuse | `crackmapexec`, `psexec` | **Unique passwords per host** kills reuse |
| Remote movement | SSH, RDP, SMB, WinRM | Restrict management ports, segment network |
| Domain takeover | Kerberos attacks (AD environments) | Harden AD, rotate `krbtgt`, limit Domain Admins |

> **Lateral movement is defeated by segmentation + unique credentials.** If every
> box has a different admin password and management ports are firewalled to a
> jump host, one compromise doesn't become ten.

---

## 7. Impact — What They Do To Hurt Your Score

The Red Team's points often come from *demonstrating* control and from *denying
you availability*. Expect:

- **Defacing or breaking scored services** (web page changes, stopping daemons)
- **Deleting/corrupting data** the scoring engine checks
- **Ransomware-style lockouts** (simulated)
- **Dropping "flags"/tokens** to prove compromise (region-dependent)
- **Knocking services offline** right before a scoring check

**Counter:** backups/snapshots of configs and data, service monitoring with
auto-restart, file integrity checks on scored content, and fast detection so you
restore before the next scoring tick.

---

## 8. Priority Map — What Red Team Hits, In Order

Use this to sequence your hardening to match their likely sequence:

```
1. Scan everything                  → You: scan first, shrink attack surface
2. Log in with default creds        → You: rotate ALL creds immediately
3. Exploit obvious unpatched vulns  → You: patch/mitigate criticals
4. Plant persistence everywhere     → You: hunt & remove backdoors
5. Escalate to root/admin           → You: patch, least privilege
6. Harvest creds, move laterally    → You: unique creds, segment
7. Break/deface scored services     → You: monitor, back up, restore fast
8. Maintain quiet long-term access  → You: continuous hunting all day
```

---

## 9. Indicators of Compromise — Your Detection Checklist

Watch continuously for:

- [ ] Logins at odd times / from unexpected IPs (`last`, `/var/log/auth.log`, Event Viewer)
- [ ] New accounts or group membership changes
- [ ] New processes/services/listeners (`ss -tulpn`, `netstat`, Task Manager)
- [ ] Unexpected outbound connections (beaconing/exfil)
- [ ] New cron jobs / scheduled tasks
- [ ] Modified system binaries or web content (file integrity)
- [ ] Spikes in CPU/network
- [ ] Log files truncated/cleared (evasion sign)

---

## 10. Turning This Into Points

Every Red Team action you detect and document is **inject and report gold**:

- Keep a running **incident log**: timestamp, host, what you saw, what you did.
- When an inject asks for an incident report, you'll already have the raw material.
- Screenshots + clear timelines = professional credit.

> **Bottom line:** You win not by being unbreachable, but by being *fast and
> thorough* — rotate creds, kill persistence, monitor relentlessly, restore
> quickly, and document everything.
