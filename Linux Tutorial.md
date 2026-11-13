# Linux Tutorials for CCDC

> A fast, practical Linux reference aimed at defending boxes under time pressure.
> If you can navigate the filesystem, read logs, manage users/services, and set
> firewall rules confidently, you'll harden Linux hosts far faster than the Red
> Team can abuse them. Practice these until they're muscle memory.

---

## 1. Survival Basics

```bash
pwd                 # where am I
ls -la              # list all files incl. hidden, with perms/owners
cd /path            # change directory
cat file            # print a file
less file           # scroll a file (q to quit)
grep "text" file    # search inside a file
grep -r "text" /dir # recursive search
find / -name "x"    # find files by name
history             # your command history
man <command>       # manual page for a command
clear               # clear screen
```

**Who am I / where am I:**
```bash
whoami              # current user
id                  # user + groups
hostname -I         # my IPs
uname -a            # kernel / OS info
cat /etc/os-release # distro + version
```

---

## 2. Users & Permissions

```bash
# View accounts
cat /etc/passwd                      # all accounts (user:x:UID:GID:...:shell)
cat /etc/group                       # groups
awk -F: '$3==0{print $1}' /etc/passwd  # all UID-0 (root-equivalent) accounts ⚠️

# Manage accounts
passwd <user>                        # change a user's password
useradd -m <user>; passwd <user>     # add user
userdel -r <user>                    # delete user + home
usermod -L <user>                    # lock account (can't log in)
usermod -s /sbin/nologin <user>      # disable shell login
usermod -aG sudo <user>              # add to sudo group

# Sudo / privilege (audit these for attacker changes)
cat /etc/sudoers
ls -l /etc/sudoers.d/
visudo                               # safely edit sudoers
```

**Permissions model:**
```
-rwxr-xr--   owner=rwx  group=r-x  others=r--
chmod 640 file          # rw- r-- ---
chmod +x script.sh      # make executable
chown user:group file   # change owner
find / -perm -4000 2>/dev/null   # SUID binaries ⚠️ (privilege-escalation risk)
```

---

## 3. Processes & Services

```bash
ps aux                       # all running processes
ps aux | grep <name>         # find a process
top    /  htop               # live process view
kill <PID>;  kill -9 <PID>   # stop a process
pstree -p                    # process tree

# systemd services
systemctl status <svc>
systemctl start|stop|restart <svc>
systemctl enable|disable <svc>            # on/off at boot
systemctl disable --now <svc>             # stop AND disable
systemctl list-unit-files --state=enabled # what starts at boot ⚠️
systemctl list-timers                     # systemd timers (persistence ⚠️)
```

---

## 4. Networking & Listening Ports

```bash
ip addr        /  ip -brief addr      # interfaces & IPs
ip route                              # routing table
ss -tulpn                             # listening TCP/UDP + owning process ⚠️
netstat -tulpn                        # (older equivalent)
ss -tnp                               # active connections
ping <host>                           # reachability
curl -I http://localhost              # test a web service quickly
dig <name>  /  nslookup <name>        # DNS queries
```

> `ss -tulpn` is one of your most important commands — it shows **exactly what is
> listening and which program owns it.** Unexpected listeners = investigate.

---

## 5. Logs — Your Detection Lifeline

```bash
# Auth / login activity (attacker logins show here)
tail -f /var/log/auth.log     # Debian/Ubuntu
tail -f /var/log/secure       # RHEL/CentOS/Fedora
last                          # successful logins history
lastb                         # FAILED login attempts ⚠️
journalctl -xe                # systemd journal (recent, errors)
journalctl -u <svc>           # logs for one service
journalctl --since "10 min ago"

# System & service logs
ls /var/log/
tail -f /var/log/syslog       # general system log
tail -f /var/log/apache2/access.log   # web server (path varies)
```

**Spot these in logs:** repeated failed logins then a success (brute force),
logins from odd IPs/times, `sudo` by unexpected users, service restarts you
didn't do.

---

## 6. Finding Attacker Persistence (hunt checklist)

```bash
# Cron jobs (scheduled attacker tasks)
crontab -l                           # current user
for u in $(cut -f1 -d: /etc/passwd); do crontab -l -u "$u" 2>/dev/null; done
ls -la /etc/cron* /var/spool/cron/   # system cron dirs

# SSH keys (unauthorized access)
cat /root/.ssh/authorized_keys 2>/dev/null
for h in /home/*; do cat "$h/.ssh/authorized_keys" 2>/dev/null; done

# Startup / profile scripts
cat /etc/rc.local 2>/dev/null
ls -la /etc/profile.d/
cat ~/.bashrc /etc/bash.bashrc

# Recently modified files (after game start)
find / -newermt "today" -type f 2>/dev/null | grep -v /proc

# Web shells in web roots
ls -laR /var/www/ 2>/dev/null | grep -iE "\.php|\.jsp"
```

---

## 7. Firewall Quick Reference

Identify which your host uses, then lock it to only required ports.

**ufw (Ubuntu):**
```bash
ufw status verbose
ufw default deny incoming
ufw default allow outgoing
ufw allow 443/tcp          # allow a scored port
ufw allow from 192.168.99.10 to any port 22   # SSH only from jump host
ufw enable
```

**firewalld (RHEL/CentOS):**
```bash
firewall-cmd --state
firewall-cmd --list-all
firewall-cmd --permanent --add-port=443/tcp
firewall-cmd --permanent --remove-service=telnet
firewall-cmd --reload
```

**iptables (generic/legacy):**
```bash
iptables -L -n -v                      # view rules
iptables -A INPUT -p tcp --dport 443 -j ACCEPT
iptables -P INPUT DROP                 # default-deny (⚠️ be careful not to lock yourself/scoring out)
```

> ⚠️ Before default-deny, make sure you've ALLOWed: scored service ports, your
> management access, and the scoring engine.

---

## 8. SSH Hardening

Edit `/etc/ssh/sshd_config`:
```
PermitRootLogin no
PermitEmptyPasswords no
MaxAuthTries 3
# AllowUsers <you> <teammate>        # whitelist who can SSH
# PasswordAuthentication no          # keys only (only if scoring doesn't need password SSH)
```
Then:
```bash
sshd -t                 # test config for syntax errors FIRST
systemctl restart sshd
```

---

## 9. Patching / Package Management

```bash
# Debian/Ubuntu
apt update
apt list --upgradable
apt install --only-upgrade <pkg>     # patch one package
apt upgrade                          # ⚠️ big upgrades can break scored services — be selective

# RHEL/CentOS
yum check-update   /  dnf check-update
yum update <pkg>   /  dnf update <pkg>

# What's installed / versions
dpkg -l            # Debian
rpm -qa            # RHEL
```

> Patch the **critical, known-exploited** things first. A full `upgrade` mid-game
> can restart or break a scored service — prefer targeted patches.

---

## 10. Backups & Restore (protect scored data/configs)

```bash
# Back up a config before changing it
cp /etc/ssh/sshd_config /etc/ssh/sshd_config.bak

# Back up a directory
tar -czvf /root/backup-www-$(date +%H%M).tar.gz /var/www/

# Restore
tar -xzvf /root/backup-www-HHMM.tar.gz -C /

# Snapshot DBs (examples)
mysqldump -u root -p --all-databases > /root/db-backup.sql
pg_dumpall -U postgres > /root/pg-backup.sql
```

> Snapshot **before** you harden, and keep periodic backups of scored content so
> you can restore fast after Red Team impact.

---

## 11. Handy One-Liners

```bash
# Everyone who logged in today
last | grep "$(date +%a)"

# Watch for new listening ports every 5s
watch -n5 'ss -tulpn'

# Kill all processes owned by a suspicious user
pkill -u <user>

# Show the 20 largest files (find hidden exfil staging / filled disk)
du -ah / 2>/dev/null | sort -rh | head -20

# Compare current users to a saved baseline
cut -d: -f1 /etc/passwd | sort > /root/users.now
diff /root/users.baseline /root/users.now
```

---

## 12. Practice Plan

1. Spin up 2–3 Linux VMs (Ubuntu + CentOS/Rocky) in VirtualBox/VMware.
2. Intentionally weaken one (default passwords, open services, a cron "backdoor").
3. Time yourself: inventory → rotate creds → close ports → find the backdoor.
4. Repeat until the first-15-minutes checklist (Blue Team playbook) takes <15 min.

> **Speed + accuracy on these fundamentals is what separates winning CCDC teams.**
> Drill them before competition day. 🐧🛡️
