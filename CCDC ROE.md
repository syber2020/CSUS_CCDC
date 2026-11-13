# CCDC Rules of Engagement (ROE)

> **Purpose:** A working reference for how your team operates during a Collegiate
> Cyber Defense Competition (CCDC). This summarizes the *spirit* of the official
> rules so everyone on the team shares the same expectations. **Always defer to
> the official rulebook and the White Team's briefing for your specific region
> and season** — rules change year to year and region to region.

---

## 1. What CCDC Is

CCDC is a **defensive** competition. Your team (the **Blue Team**) inherits a
small, pre-built business network full of misconfigurations, outdated software,
and weak credentials. Your job is to:

1. **Keep services running** (scored availability).
2. **Keep attackers out** (and evict them when they get in).
3. **Respond to business tasks** ("injects") like a real IT team.
4. **Report** what happened clearly and professionally.

You do **not** attack anyone. Offensive action is the Red Team's role, and they
are hired professionals, not competitors you can strike back at.

---

## 2. The Teams

| Team | Color | Role |
|------|-------|------|
| **Blue Team** | 🔵 | You — the competitors defending the network |
| **Red Team** | 🔴 | Professional attackers trying to compromise your systems |
| **White Team** | ⚪ | Officials: run the event, score, deliver injects, enforce rules |
| **Gold/Orange Team** | 🟡 | Infrastructure/scoring engine operators (region-dependent) |
| **Green Team** | 🟢 | Simulated "customers/users" generating normal traffic (region-dependent) |

---

## 3. Hard Rules — Do Not Break These

Violating these can mean **point penalties or disqualification**.

- **No offensive activity.** Never scan, attack, exploit, or attempt to access
  Red Team, White Team, scoring infrastructure, other competing teams, or
  anything outside your assigned scope.
- **No "hack back."** Even if you identify the Red Team's source IP, you may not
  target it. Document it and move on.
- **No destructive denial of your own services to deny the Red Team.** Taking a
  scored service permanently offline to "protect" it loses availability points
  and may be ruled against you. Harden it, don't kill it.
- **Keep scored services available and reachable** on their required ports from
  the scoring engine. Blocking the scoring engine = lost points.
- **No pre-staged content or outside help** unless explicitly allowed. Typically:
  no bringing in scripts/configs during the competition beyond what the rules
  permit, no remote teammates, no internet assistance during play (varies — read
  the rules).
- **No modifying the scoring engine or injects.**
- **Stay in your scope.** Only touch systems assigned to your team.
- **Physical/social boundaries:** no tampering with other teams' areas, no
  impersonating White Team, follow all venue conduct rules.

> ⚠️ When in doubt, **ask the White Team** before acting. "I wasn't sure if I
> was allowed" is not a defense after the fact.

---

## 4. Scoring Model (Typical)

Your final score is usually a combination of:

| Category | What it measures | How to win it |
|----------|------------------|---------------|
| **Service uptime** | Scored services responding correctly | Keep services up, functional, on the right ports |
| **Injects** | Business tasks completed on time and well | Read carefully, deliver on time, document |
| **Red Team** | Points *lost* when compromised | Fewer compromises, faster eviction |
| **Penalties** | Rule violations, ROE breaches | Don't incur them |

Key insight: **availability is usually worth more than you think.** A perfectly
hardened box that's offline scores zero. Balance security with uptime.

### Service checks
The scoring engine connects to your services on a schedule (e.g., every few
minutes) and verifies a real transaction — not just that a port is open. Example:
it may log into your web app, load a page, and check content. If you change
credentials the engine uses, you break the check. **Find out which accounts the
scoring engine uses and protect (don't delete) them.**

---

## 5. Injects

Injects are tasks delivered during the competition, simulating requests from
management, clients, or compliance. Examples:

- "Create 15 user accounts for new hires with these requirements."
- "Write an incident report for the breach that occurred at 10:42."
- "Produce a network diagram and asset inventory."
- "Implement a password policy and document it."
- "Migrate service X to a new configuration."

**Inject discipline wins competitions:**

- Assign an **inject coordinator** (often the captain) who reads every inject,
  logs it, assigns an owner, and tracks the deadline.
- **Deliver in the exact format requested.** If they want a PDF, send a PDF.
- **Partial credit is real** — submit something before the deadline even if
  imperfect.
- **Professionalism counts.** Spelling, formatting, and clarity are scored.

---

## 6. Red Team Interaction

- The Red Team **will** get in. Assume breach. Your score depends on **limiting
  impact and evicting fast**, not on being impenetrable.
- Red Team may hold persistence you haven't found. Rotate credentials, hunt for
  backdoors, and monitor continuously.
- You may be required to allow certain access (e.g., you usually **cannot block
  the Red Team's entire subnet** if that also blocks the scoring engine or is
  disallowed). Read the ROE on blocking.
- Some competitions **penalize blocking by IP** wholesale or require services to
  stay internet-reachable. Harden the service, don't just firewall the attacker.

---

## 7. Communication Rules

- **Internal team comms** are free game (talk, shared docs, whiteboard).
- **With White Team:** be polite, concise, and use the official channel
  (in-person runner, chat, ticket system — varies).
- **Report compromises** per the rules — some regions require you to notify
  White Team of incidents.
- **No contacting Red Team directly** unless the format explicitly allows a
  debrief.

---

## 8. Team Conduct & Professionalism

CCDC simulates a workplace. You're being evaluated as if you're the IT staff of a
real company.

- Dress and behave per event guidance (many regionals expect business-casual).
- Be respectful to White/Red Team volunteers — many are industry pros and future
  employers. **CCDC is a recruiting event.**
- No cheating, no harassment, no unsafe behavior.

---

## 9. Pre-Competition Checklist (ROE-focused)

- [ ] Every team member has **read the current official rulebook**.
- [ ] Everyone knows what is **in scope** vs. **out of scope**.
- [ ] Everyone knows **what "no offense" means** in practice.
- [ ] You know which **services are scored** and on which **ports**.
- [ ] You know the **scoring engine's source** (so you never block it).
- [ ] You know **how injects are delivered and submitted**.
- [ ] You know **how to contact White Team** and when you're required to.
- [ ] You know the **penalty structure**.
- [ ] Roles assigned: captain, inject coordinator, per-OS defenders, scribe.

---

## 10. Quick "Can I Do This?" Decision Guide

| Situation | Answer |
|-----------|--------|
| Red Team got in — can I patch/reconfigure? | ✅ Yes, that's your job |
| Can I change passwords on all accounts? | ✅ Yes — but keep scoring/service accounts working |
| Can I block the Red Team's IP? | ⚠️ Maybe — check ROE; never block the scoring engine |
| Can I take a scored service offline to stop an attack? | ❌ Usually no — you lose uptime points |
| Can I scan the Red Team / another team? | ❌ Never |
| Can I use a script I wrote before the competition? | ⚠️ Depends — read the rules on pre-staged content |
| Red Team IP is X — can I attack it? | ❌ Never. Document and report |
| Not sure if something's allowed? | 🟡 **Ask the White Team first** |

---

*Remember: CCDC rewards calm, organized, professional defense. Read the official
rules, know your scope, protect uptime, respond to injects, and evict the Red
Team methodically. Good luck. 🛡️*
