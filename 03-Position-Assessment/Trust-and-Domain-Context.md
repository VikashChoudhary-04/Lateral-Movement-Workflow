# Trust and Domain Context

In Windows and Active Directory environments, the identity and network position of a compromised host exist within a larger security environment.

Understanding that environment helps answer:

> **Which systems, accounts, domains, and authentication relationships are relevant to my current position?**

The objective is **not** to fully enumerate Active Directory here.

That belongs to dedicated AD enumeration and Privilege Escalation workflows.

The objective here is to understand enough of the domain and trust context to make informed lateral-movement decisions.

---

# 1. The Domain Context Workflow

```text id="d2d0gc"
Current Host
     │
     ▼
Identify Domain / Workgroup
     │
     ▼
Identify Current Account Context
     │
     ▼
Determine Domain Membership
     │
     ▼
Identify Relevant Domain Infrastructure
     │
     ▼
Understand Trust Relationships
     │
     ▼
Identify Potentially Relevant Systems
     │
     ▼
Determine Authentication Context
     │
     ▼
Choose Next Investigation
```

---

# 2. Domain vs Workgroup

A Windows system may be:

```text id="b6jz3x"
Domain-Joined
```

or:

```text id="4o3x4w"
Workgroup / Standalone
```

This distinction changes the available identity and authentication relationships.

### Domain Environment

```text id="2m7w2j"
                 DOMAIN
                    │
        ┌───────────┼───────────┐
        ▼           ▼           ▼
      HOST-A      HOST-B      HOST-C
        │
        └── Domain Identity
```

### Workgroup / Standalone

```text id="bqf8gd"
HOST-A
  │
  └── Local Accounts
```

A domain environment generally provides more interconnected authentication and authorization relationships.

---

# 3. Identify the Domain

On Windows, start with:

```cmd id="o5a6f4"
whoami
```

You can also inspect the computer's domain information:

```cmd id="r7kn0p"
systeminfo
```

PowerShell:

```powershell id="2l8fr4"
(Get-CimInstance Win32_ComputerSystem).Domain
```

You can also inspect:

```cmd id="3w6m9z"
echo %USERDOMAIN%
```

### Record

```text id="z2o4r0"
Computer:
Domain:
Current Account:
```

---

# 4. Determine Whether the Current Account Is Domain-Based

Compare the identity context.

For example:

```text id="p7t2fh"
CORP\alice
```

suggests a domain account.

Whereas:

```text id="6l7j5b"
WORKSTATION01\alice
```

indicates a local account context.

PowerShell can provide additional identity information:

```powershell id="a5u4k5"
[System.Security.Principal.WindowsIdentity]::GetCurrent().Name
```

### Why?

The identity context affects which authentication relationships may be relevant.

A domain account may potentially authenticate to multiple domain-connected systems, subject to authorization and account restrictions.

A local account is primarily associated with the local system, although local accounts may also be used for remote administration where policy and configuration permit.

---

# 5. Identify Domain Infrastructure

Once a host is confirmed to be domain-connected, identify the relevant infrastructure.

Typical components include:

```text id="m7flk7"
Domain Controller
DNS
File Servers
Application Servers
Database Servers
Workstations
Management Systems
```

A simplified environment might look like:

```text id="r7d0if"
                 DOMAIN
                   │
              ┌────┴────┐
              ▼         ▼
            DC01       DNS
              │
       ┌──────┼──────┐
       ▼      ▼      ▼
     WS01   FILE01  APP01
```

The goal is not to map everything immediately.

The goal is to understand:

> **What infrastructure is relevant to the system I currently control?**

---

# 6. Domain Controller Awareness

A Domain Controller is a particularly important part of an AD environment because it provides core domain services.

A host that appears to be a Domain Controller may expose several relevant services.

Do not identify a DC solely because the hostname contains:

```text id="4z5j6w"
DC
```

Validate the role using multiple pieces of evidence.

Useful information may include:

* Domain membership
* Directory-related services
* DNS
* Kerberos
* LDAP
* SMB
* RPC

### Important

This repository does not treat:

```text id="8q7b2v"
"Domain Controller found"
```

as:

```text id="a6z8w0"
"Attack the Domain Controller immediately."
```

Instead:

```text id="u8b6x4"
DC identified
    ↓
Understand its role
    ↓
Determine current access
    ↓
Determine relevant authentication paths
    ↓
Continue the authorized assessment workflow
```

---

# 7. Identify Relevant Domain Accounts

The current identity can provide useful context.

For example:

```text id="m5s7x4"
Current User:
CORP\alice
```

You may want to understand:

```text id="8p4f2x"
Is this a normal domain user?
Is this a service account?
Is this an administrative account?
What groups is it associated with?
What systems or services might use it?
```

The goal is to determine **what kind of identity you control**.

Detailed account enumeration belongs in the appropriate AD workflow.

---

# 8. Understand Groups as Authorization Signals

Domain group membership can influence access.

For example:

```text id="0g6v0d"
CORP\alice
      │
      ├── Domain Users
      ├── Application Group
      └── Remote Access Group
```

These memberships may indicate possible authorization relationships.

But remember:

```text id="z9q0o8"
Group Membership
      ≠
Guaranteed Access
```

The correct workflow is:

```text id="4kw7bl"
Group
  ↓
Potential Authorization
  ↓
Candidate Target
  ↓
Relevant Service
  ↓
Validate Authentication
  ↓
Validate Actual Access
```

---

# 9. Understand Trust Relationships

A trust relationship allows authentication or authorization relationships between security domains.

Conceptually:

```text id="ax4m0s"
DOMAIN-A
    │
    │ Trust
    ▼
DOMAIN-B
```

This does **not** mean:

```text id="q5q5jm"
Every account in DOMAIN-A
can access everything in DOMAIN-B
```

Instead, the trust is an additional relationship that may affect authentication possibilities.

---

# 10. Why Trusts Matter for Lateral Movement

Imagine:

```text id="4d2k3r"
DOMAIN-A
    │
    └── Trust ──► DOMAIN-B
```

Your current position is:

```text id="x7b2f8"
HOST-A
  │
  └── DOMAIN-A
```

A trust relationship may make systems in `DOMAIN-B` relevant to your assessment.

The workflow becomes:

```text id="z8j6k4"
Current Domain
      ↓
Identify Trust
      ↓
Understand Trust Direction
      ↓
Identify Relevant Accounts
      ↓
Identify Candidate Targets
      ↓
Determine Authentication Possibilities
      ↓
Validate Authorized Access
```

Do not treat the existence of a trust as proof of access.

---

# 11. Trust Direction Matters

A simplified representation:

```text id="6r4y2s"
DOMAIN-A
   │
   │ Trust
   ▼
DOMAIN-B
```

The direction and type of trust matter.

You need to determine:

* Which domain trusts which?
* What type of trust exists?
* What authentication relationship does it provide?
* Which accounts are relevant?
* What authorization still applies?

The important operational principle is:

> **A trust relationship changes the authentication landscape; it does not automatically grant privileges.**

---

# 12. Identify Relevant Authentication Mechanisms

In Active Directory environments, common authentication mechanisms include:

```text id="0g0x3a"
Kerberos
NTLM
```

These mechanisms matter because the authentication material available to you can influence possible movement paths.

For example:

```text id="q1e3n9"
Available Authentication Material
              │
              ▼
       Authentication Type
              │
              ▼
       Compatible Services
              │
              ▼
        Candidate Targets
```

Detailed credential-based movement is covered later in:

```text id="j2l3f7"
08-Credential-Based-Movement/
```

This file only establishes the context needed to understand why authentication relationships matter.

---

# 13. Domain Name Resolution

DNS is particularly important in AD environments.

A domain such as:

```text id="e9j1n6"
corp.local
```

may provide a naming structure such as:

```text id="8f7b0f"
dc01.corp.local
file01.corp.local
app01.corp.local
```

This can make internal systems easier to identify.

Useful information may come from:

```text id="y8q9b2"
DNS configuration
Hostnames
Search domains
Resolved names
Existing connections
```

Treat DNS information as a source of hypotheses.

Validate important findings before using them for movement decisions.

---

# 14. Domain Context → Target Discovery

Once you understand the current domain, begin thinking about potential targets.

For example:

```text id="9u5g3w"
Current Host
     │
     ▼
CORP.LOCAL
     │
     ├── FILE01
     ├── APP01
     ├── SQL01
     ├── WS01
     └── DC01
```

You now have candidate systems.

But you still need to determine:

```text id="k3d8g9"
Which systems are reachable?
Which services are exposed?
What authentication do they require?
What access do I possess?
```

This leads directly into:

```text id="e7a6h3"
04-Target-Discovery/
```

---

# 15. Workgroup Environment

Not every Windows environment uses Active Directory.

If the host belongs to a workgroup:

```text id="1h3y5d"
WORKGROUP
    │
    ├── HOST-A
    ├── HOST-B
    └── HOST-C
```

The domain-specific workflow is reduced.

Focus instead on:

```text id="2b6r3z"
Local Accounts
      ↓
Network Reachability
      ↓
Remote Services
      ↓
Available Credentials
      ↓
Target Validation
```

Do not force AD assumptions onto a workgroup environment.

---

# 16. Linux in an AD Environment

Linux systems can also participate in an Active Directory environment.

A Linux host may be integrated with domain identity services and may use domain accounts.

Therefore:

```text id="9f6w3v"
Linux Host
    │
    ▼
Domain Integration
    │
    ├── Domain Users
    ├── Domain Groups
    └── Domain Authentication
```

The assessment principle remains the same:

```text id="x9f7j6"
Identify Identity
     ↓
Identify Domain Context
     ↓
Understand Network Position
     ↓
Identify Relevant Targets
     ↓
Determine Authentication Options
```

The commands and integration technologies may differ from Windows.

---

# 17. Do Not Turn This Into AD PrivEsc

This repository already has a dedicated AD Privilege Escalation workflow.

Therefore, when you discover something such as:

```text id="k3t8v2"
Interesting group membership
Potential privilege escalation path
Delegation issue
Misconfigured ACL
Privileged account
```

ask:

> **Is this helping me understand lateral movement, or am I now doing privilege escalation?**

If the objective becomes privilege escalation, switch to the appropriate AD PrivEsc workflow.

This separation keeps the repositories focused.

---

# 18. Domain Context Decision Tree

Use this simplified decision process:

```text id="x2o9r5"
Is the host domain-joined?
        │
    ┌───┴───┐
   YES      NO
    │        │
    ▼        ▼
Identify   Workgroup /
Domain     Local Context
    │
    ▼
Identify Current Account
    │
    ▼
Identify Relevant Groups
    │
    ▼
Identify Domain Infrastructure
    │
    ▼
Identify Trust Relationships
    │
    ▼
Identify Candidate Targets
    │
    ▼
Determine Authentication Options
    │
    ▼
Continue to Target / Access Discovery
```

---

# 19. Build the Domain Context Map

Record the environment in a compact form.

```text id="2y0r9a"
DOMAIN / TRUST CONTEXT
======================

Computer:
Current User:
Account Type:

Domain:
Workgroup:

Domain Infrastructure:
Relevant Servers:

Trusts:
Trusted Domains:

Authentication Context:
Relevant Groups:

Potential Targets:

Next Investigation:
```

Example:

```text id="g6z4rx"
Computer: WS01
Current User: CORP\alice
Account Type: Domain User

Domain: CORP.LOCAL

Relevant Infrastructure:
DC01
FILE01
APP01

Trust:
CORP.LOCAL ↔ PARTNER.LOCAL

Authentication Context:
Kerberos / NTLM

Potential Targets:
FILE01
APP01

Next Investigation:
Determine reachability, services, and available authorized access.
```

---

# 20. Trust and Domain Assessment Checklist

```text id="u2j6h8"
[ ] Domain/workgroup status identified
[ ] Domain name identified
[ ] Current account context identified
[ ] Local/domain account distinction confirmed
[ ] Relevant groups identified
[ ] Domain infrastructure identified
[ ] Potential Domain Controllers identified
[ ] DNS/domain naming context understood
[ ] Relevant authentication mechanisms identified
[ ] Trust relationships considered
[ ] Trust direction/type investigated where relevant
[ ] Candidate systems identified
[ ] Authorization assumptions avoided
[ ] AD PrivEsc scope kept separate
[ ] Next workflow selected
```

---

# 21. Final Domain Context Assessment

You should be able to summarize:

```text id="9b6a0k"
This host belongs to:
____________________

My current identity is:
____________________

The relevant domain is:
____________________

Relevant domain infrastructure:
____________________

Relevant trust relationships:
____________________

Relevant authentication mechanisms:
____________________

Potentially relevant systems:
____________________

The next thing I need to determine is:
____________________
```

The objective is:

> **Understand how the current identity and host fit into the surrounding domain environment, then use that context to identify legitimate movement opportunities.**

---

# 22. Position Assessment Complete

At this point, the four components of the position map should be available:

```text id="3g8f9v"
                  CURRENT POSITION
                         │
       ┌─────────────────┼─────────────────┐
       ▼                 ▼                 ▼
     HOST             USER             NETWORK
       │                 │                 │
       └─────────────────┼─────────────────┘
                         ▼
                  DOMAIN / TRUST
                         │
                         ▼
                  ACCESS CONTEXT
                         │
                         ▼
                 MOVEMENT OPTIONS
```

You should now know:

```text id="e8m8my"
Where am I?
Who am I?
What can I do?
What network am I on?
What domain am I part of?
What systems may be relevant?
What authentication relationships matter?
```

That is enough context to begin actively looking for **where you can move**.

---

## Next

The next section is:

```text id="5u2v4k"
04-Target-Discovery/
```

The question now changes from:

> **"Where am I?"**

to:

> **"Where can I potentially go?"**
