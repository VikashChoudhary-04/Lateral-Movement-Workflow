# Existing Access

Existing access is one of the most important findings during lateral movement assessment.

A compromised system may already have authenticated relationships with other systems even when you have not discovered a reusable password, hash, Kerberos ticket, token, or SSH private key.

The practical question is:

> **What access does this current position already have, which targets and services does that access relate to, and can it provide the next movement step?**

The workflow is:

```text
Current Position
      ↓
Identify Existing Access
      ↓
Identify Identity / Context
      ↓
Identify Target
      ↓
Identify Service
      ↓
Determine Authentication State
      ↓
Validate Authorization
      ↓
Determine Movement Value
      ↓
Move or Continue Discovery
      ↓
Re-enumerate New Position
```

---

## 1. Where This Fits

This is the final branch of credential and access discovery:

```text
Credentials
    ↓
Passwords
    ↓
Hashes
    ↓
Kerberos
    ↓
Tokens and Sessions
    ↓
SSH Keys
    ↓
Existing Access
```

The earlier files focused primarily on authentication material.

This file asks a broader question:

```text
What can I already access?
```

That distinction matters because:

```text
Credential Found
    ≠
Access Found
```

But:

```text
Validated Access
    =
Potential Movement Path
```

---

## 2. What Counts as Existing Access?

Existing access can include:

```text
Active Remote Sessions
Existing SMB Connections
Existing SSH Sessions
Mapped Network Resources
Authenticated Service Connections
Existing Administrative Channels
Application-Level Access
Remote Management Relationships
Already-Authorized Accounts
```

The exact possibilities depend on the operating system and environment.

---

## 3. Existing Access vs Credentials

Consider:

```text
Password Found
```

This tells you:

```text
Authentication Material Exists
```

But:

```text
SMB Share Access Validated
```

tells you:

```text
Authentication
      +
Authorization
      +
Target
      +
Service
```

Therefore existing access can be more immediately useful than an unvalidated credential.

---

## 4. The Access Model

Every access finding should be represented as:

```text
Identity
   ↓
Authentication Context
   ↓
Service
   ↓
Target
   ↓
Authorization
   ↓
Action
```

Example:

```text
CORP\alice
     ↓
Existing Authentication
     ↓
SMB
     ↓
FILE01
     ↓
Read Access
     ↓
Investigate Available Resources
```

---

## 5. First Establish Your Current Position

Before investigating additional access, record your current context.

### Linux

```bash id="w2m8q4"
whoami
id
hostname
ip addr
ip route
```

### Windows

```cmd id="x7n3p6"
whoami
whoami /all
hostname
ipconfig /all
route print
```

The objective is to understand:

```text
Current Host
Current User
Current Privileges
Current Network
Current Domain Context
```

This prevents confusion between:

```text
Current Access
```

and:

```text
Access Available to Another Identity
```

---

## 6. Existing SMB Access

On Windows environments, SMB is a common source of existing network access.

Check current connections:

```cmd id="q4k9m2"
net use
```

This can reveal existing network connections.

You may find something conceptually like:

```text
Remote:
\\FILE01\Shared

Status:
OK
```

This is stronger evidence than merely discovering that:

```text
FILE01
```

exists.

You have evidence of an existing SMB relationship.

---

## 7. Interpret SMB Access

Suppose:

```text id="v6p1r8"
net use
```

shows:

```text
\\FILE01\Shared
```

Ask:

```text
Which identity created this connection?
Is the connection still active?
What permissions exist?
What resources are available?
Is the target relevant to the movement objective?
```

Do not immediately assume:

```text
SMB Access
   =
Administrative Access
```

---

## 8. Windows: Network Shares

You can inspect accessible network resources where authorized.

For example:

```cmd id="m3x8q5"
net use
```

and, where appropriate:

```cmd id="c7p2n9"
net view
```

The exact results depend on permissions and network configuration.

The goal is:

```text
Existing Connection
      ↓
Accessible Resource
      ↓
Target
      ↓
Potential Movement Value
```

---

## 9. Windows: Remote Desktop Context

Existing RDP sessions can indicate that the system participates in remote administration.

Check sessions:

```cmd id="r8m4v1"
query user
```

or:

```cmd id="n6q2x7"
qwinsta
```

Record:

```text
Username
Session ID
State
Logon Time
```

A current RDP session is useful environmental evidence.

Do not assume it is automatically reusable.

---

## 10. Windows: Existing WinRM Relationships

If WinRM is in use, existing connections or management configuration may reveal remote administration relationships.

Check relevant service configuration:

```powershell id="p4m9c6"
Get-Service WinRM
```

You can also inspect current network connections:

```powershell id="k7x3n8"
Get-NetTCPConnection
```

The goal is to determine:

```text
Is WinRM active?
Are remote-management relationships visible?
Which systems communicate with this host?
```

---

## 11. Windows: Existing Network Connections

Use:

```cmd id="q9m2v5"
netstat -ano
```

or:

```powershell id="w3c8r1"
Get-NetTCPConnection
```

Look for:

```text
Established
RemoteAddress
RemotePort
OwningProcess
```

Then correlate:

```text
Connection
   ↓
PID
   ↓
Process
   ↓
Identity / Service
   ↓
Target
```

This can reveal existing access relationships that are not obvious from credential files.

---

## 12. Linux: Existing SSH Access

Start with:

```bash id="y6p3m8"
who
```

and:

```bash id="v1k7q4"
w
```

Then:

```bash id="f8n2x5"
ss -tnp
```

Look for established SSH connections:

```text id="z3m6r9"
Local Port:
22 / Ephemeral

Remote:
10.10.20.15:22

State:
ESTABLISHED
```

This indicates an existing SSH relationship.

Determine:

```text
Who owns it?
Is it current?
What remote host is involved?
```

---

## 13. Linux: Existing Network Mounts

Linux systems may have remote filesystems mounted.

Check:

```bash id="j5q8n2"
mount
```

and:

```bash id="r3m7x9"
findmnt
```

You may encounter:

```text id="p6c1v4"
NFS
CIFS
Other Network Filesystems
```

These can reveal existing relationships with remote systems.

The important relationship is:

```text
Local Mount
    ↓
Remote Resource
    ↓
Remote Host
```

---

## 14. Existing NFS Access

An NFS mount may look conceptually like:

```text id="k8v4m2"
SERVER02:/exports/data
```

This tells you:

```text
Remote Host:
SERVER02

Remote Resource:
/exports/data

Local Access:
Mounted
```

The mount itself is evidence of an existing relationship.

It does not automatically mean:

```text
Administrative Access to SERVER02
```

---

## 15. Existing CIFS / SMB Mounts on Linux

A Linux system may have an SMB/CIFS resource mounted.

For example:

```bash id="n7p3x5"
findmnt
```

may reveal a remote filesystem.

Interpret it as:

```text id="q2m8c6"
Current Host
   ↓
CIFS Mount
   ↓
Remote Server
   ↓
Authenticated Resource
```

This can provide useful target and access information.

---

## 16. Existing Database Connections

Applications may already maintain connections to:

```text id="x4r9m1"
MySQL
PostgreSQL
MSSQL
Oracle
Redis
Other Internal Services
```

A running process can reveal that:

```text id="w6p2k8"
WEB01
   ↓
Application
   ↓
DB01
```

This is valuable because it exposes internal infrastructure.

But:

```text id="z1m7q4"
Application → Database
```

does not automatically mean:

```text id="c8n3x5"
Current User → Database
```

Determine the actual security context.

---

## 17. Existing Application Access

The current system may already have access to internal applications.

Examples:

```text id="r5q9m2"
Internal Web Application
Monitoring Dashboard
Admin Panel
Management Interface
Internal API
Git Server
Ticketing System
```

Existing application access may reveal:

```text id="n7x3k6"
Additional Hosts
User Accounts
Services
Infrastructure
Credentials
Movement Paths
```

The access level must be documented accurately.

---

## 18. Existing Cloud Access

A host may already have access to cloud resources through:

```text id="v4m8p2"
Instance Roles
Managed Identities
Cloud CLI Sessions
Environment Configuration
Application Credentials
```

For example, an application may communicate with an internal cloud service.

The lateral-movement question becomes:

```text id="q6r1x9"
What identity is being used?
What service is being accessed?
What resources are authorized?
Does this provide another infrastructure path?
```

Cloud-specific credential handling should remain within the authorized assessment scope.

---

## 19. Existing Domain Access

In Active Directory environments, the current account may already have access to:

```text id="m9c4v7"
File Servers
Management Servers
Workstations
Applications
Databases
Domain Services
```

Do not equate:

```text
Domain Account
```

with:

```text
Access to Every Domain System
```

Instead:

```text id="p2x8n5"
Domain Identity
      ↓
Existing Access
      ↓
Target
      ↓
Service
      ↓
Authorization
```

---

## 20. Existing Access From Service Accounts

A service account may have access that is invisible from the perspective of the interactive user.

For example:

```text id="w5m1q8"
WEB01
  ↓
CORP\websvc
  ↓
DB01
  ↓
MSSQL
```

This is a valuable environmental relationship.

The correct question is:

> What access does the service actually possess?

not:

> Is the service account an administrator?

---

## 21. Access Does Not Mean Movement

Suppose:

```text id="g3v8k1"
FILE01
```

is accessible.

Ask:

```text id="n6p2r5"
Can I authenticate?
What authorization exists?
What service is available?
Does the service permit remote interaction?
Does the access help reach another target?
```

A resource that can only be read may still provide useful information.

But it is not automatically a new interactive foothold.

---

## 22. Access Levels

Record access precisely.

Useful categories include:

```text id="x7m4c9"
No Access
Authentication Only
Read
Write
Execute
Remote Login
Service Access
Administrative Access
Unknown
```

Avoid vague statements such as:

```text id="a2q8n6"
"Full access"
```

unless the evidence supports that conclusion.

---

## 23. Existing Access as a Target-Discovery Source

Access itself can reveal targets.

For example:

```text id="f5p9m3"
Current Host
   ↓
Existing SMB Connection
   ↓
FILE01
```

or:

```text id="c8n2x7"
Current Host
   ↓
Mounted NFS Share
   ↓
SERVER02
```

or:

```text id="r4m6q1"
Current Host
   ↓
Application Connection
   ↓
DB01
```

This creates an important feedback loop:

```text id="w3p8v5"
Access Discovery
      ↓
Target Discovery
      ↓
Service Discovery
      ↓
Movement
```

---

## 24. Existing Access and Credential Discovery

Existing access can also tell you where to look next.

Suppose:

```text id="k7x2m9"
WEB01
   ↓
Application
   ↓
DB01
```

You can investigate the application relationship to determine:

```text id="p4n8c6"
Authentication Method
Service Account
Target Service
Connection Context
```

This can lead to:

```text id="z5m1r7"
Credentials
Hashes
Kerberos
Tokens
Configuration
```

The process becomes iterative.

---

## 25. Existing Access Matrix

Maintain a central matrix:

| Current Identity | Target   | Service | Access Type    | Authentication   | Authorization | Movement Value |
| ---------------- | -------- | ------- | -------------- | ---------------- | ------------- | -------------- |
| CORP\alice       | FILE01   | SMB     | Read           | Existing         | Share Read    | Candidate      |
| deploy           | WEB01    | SSH     | Remote Login   | SSH Key          | User Shell    | Candidate      |
| websvc           | DB01     | MSSQL   | Service Access | Application      | Database User | Candidate      |
| root             | SERVER02 | SSH     | Remote Login   | Existing Session | Root          | Candidate      |

The matrix should be updated whenever new evidence appears.

---

## 26. Access Validation

Every existing-access finding should pass through:

```text id="m8q3x6"
Access Observed
      ↓
Identify Identity
      ↓
Identify Target
      ↓
Identify Service
      ↓
Identify Authentication
      ↓
Confirm Current State
      ↓
Validate Authorization
      ↓
Determine Movement Value
```

This prevents assumptions from becoming facts.

---

## 27. Existing Access Failure

Suppose you discover:

```text id="v9p2m5"
FILE01
```

but cannot currently access it.

Possible explanations include:

```text id="c7x4n8"
Session Expired
Credential Changed
Connection Closed
Service Unavailable
Network Changed
Account Disabled
Authorization Changed
Configuration Changed
```

Do not erase the finding.

Classify it as:

```text id="r3m6q1"
Historical
Potential
Unavailable
Confirmed
Unknown
```

---

## 28. Historical Access

Historical evidence can still be valuable.

Examples:

```text id="k5n8x2"
Old SSH Connection
Previous RDP Session
Old Network Mount
Historical Login
Old Application Configuration
```

It may reveal:

```text id="w4p7m1"
Remote Hosts
Administrative Workflows
Service Accounts
Infrastructure Relationships
```

But:

```text id="f9c2v6"
Historical Access
    ≠
Current Access
```

Always distinguish the two.

---

## 29. Existing Access and Post-Movement Enumeration

After gaining access to a new host, repeat this process.

For example:

```text id="x6m3r8"
Host A
  ↓
Existing SSH Access
  ↓
Host B
  ↓
Position Assessment
  ↓
Existing Access
  ↓
Host C
```

This is how lateral movement becomes a chain.

The workflow is:

```text id="q2v7n4"
Compromised Host
      ↓
Existing Access
      ↓
New Host
      ↓
Re-enumerate
      ↓
Existing Access
      ↓
Next Host
      ↓
Repeat
```

---

## 30. The Lateral Movement Chain

A realistic movement chain may look like:

```text id="n8p4x6"
Initial Host
    ↓
Domain Identity
    ↓
Existing SMB Access
    ↓
File Server
    ↓
Service Account Evidence
    ↓
Database Server
    ↓
Existing Administrative Relationship
    ↓
Management Server
```

The important idea is that each host changes the information available for the next decision.

---

## 31. Existing Access Decision Tree

```text id="r5m8q2"
Start
  │
  ▼
Identify Existing Access
  │
  ▼
Access Found?
  │
 ┌┴──────────┐
No           Yes
 │            │
 ▼            ▼
Continue    Identify
Credential  Identity
Discovery      │
               ▼
           Identify Target
               │
               ▼
           Identify Service
               │
               ▼
       Determine Authentication
               │
               ▼
       Validate Authorization
               │
          ┌────┴────┐
          ▼         ▼
       Useful    Not Useful
          │         │
          ▼         ▼
      Validate   Record /
      Movement   Reassess
          │
          ▼
      New Host?
       │     │
      Yes    No
       │      │
       ▼      ▼
 Re-enumerate Continue
       │
       ▼
 Existing Access
       │
       └──────→ Repeat
```

---

## 32. Practical Windows Workflow

From a Windows foothold:

```text id="y3k8m1"
1. Identify current user
2. Identify current privileges
3. Check existing SMB connections
4. Check logged-in sessions
5. Check existing network connections
6. Identify relevant processes
7. Check remote-management context
8. Identify accessible network resources
9. Map identity → service → target
10. Validate access
11. Validate authorization
12. Determine movement value
13. If movement succeeds, re-enumerate
14. Update the movement map
```

Useful starting commands include:

```cmd id="p7m2x9"
whoami
whoami /all
net use
query user
netstat -ano
```

The commands are evidence-gathering tools, not the workflow itself.

---

## 33. Practical Linux Workflow

From a Linux foothold:

```text id="k4n8q2"
1. Identify current user
2. Identify current groups
3. Check active users
4. Check active SSH sessions
5. Check network connections
6. Check mounted remote filesystems
7. Identify application/service relationships
8. Identify remote hosts
9. Map identity → service → target
10. Validate access
11. Validate authorization
12. Determine movement value
13. If movement succeeds, re-enumerate
14. Update the movement map
```

Useful starting commands include:

```bash id="x8m3r5"
whoami
id
who
w
ss -tnp
findmnt
```

---

## 34. Practical AD Workflow

In an AD environment:

```text id="m6q2v9"
1. Identify current domain identity
2. Identify existing domain authentication context
3. Identify accessible domain resources
4. Identify SMB / WinRM / RDP relationships
5. Identify service accounts
6. Identify reachable systems
7. Validate existing access
8. Determine authorization
9. Select movement path
10. Move to target
11. Re-enumerate domain context
12. Continue the chain
```

The important distinction remains:

```text id="v4n7x1"
Domain Membership
    ≠
Universal Access
```

---

## 35. Choosing Between Existing Access and New Credentials

When both are available, ask:

```text id="z8m3p6"
What is already authenticated?
What access is already validated?
What new credential would add?
Which path provides useful information?
Which path has fewer assumptions?
```

The objective is not to maximize credential collection.

It is to identify the next useful movement path.

---

## 36. Movement Value

An access path can have different values.

### Direct Movement

```text id="j4r8x2"
Existing SSH
    ↓
Remote Shell
```

### Resource Access

```text id="m7p3c9"
Existing SMB
    ↓
Sensitive Internal Resource
```

### Infrastructure Discovery

```text id="q5n1v8"
Existing Application Connection
    ↓
Internal Server
```

### Credential Discovery

```text id="x2k6r4"
Existing Access
    ↓
Configuration
    ↓
Authentication Material
```

Not every useful finding immediately produces a shell.

---

## 37. Common Mistakes

### Mistake 1: Ignoring Existing Access

Immediately searching for passwords can waste time when an authenticated path already exists.

---

### Mistake 2: Confusing Reachability With Access

```text
Host Reachable
    ≠
Authentication
```

---

### Mistake 3: Confusing Authentication With Authorization

```text
Authenticated
    ≠
Administrative
```

---

### Mistake 4: Treating Historical Access as Current

Always record the current state.

---

### Mistake 5: Ignoring Service Accounts

Applications and services can have important remote relationships.

---

### Mistake 6: Failing to Re-enumerate

A new host is a new position.

Always restart position assessment after movement.

---

### Mistake 7: Treating Resource Access as Full Host Access

Reading an SMB share does not automatically provide an interactive session on the server.

---

## 38. Existing Access Investigation Record

Use this structure:

```text id="c6m9r2"
Current Host:
____________________

Identity:
____________________

Target:
____________________

Service:
____________________

Access Type:
____________________

Authentication Context:
____________________

Current / Historical:
____________________

Authorization:
____________________

Evidence:
____________________

Movement Value:
____________________

Validated:
____________________

Next Action:
____________________
```

---

## 39. Final Credential-and-Access Checklist

Before leaving Section 05, verify that you have considered:

```text id="v7p2x5"
[ ] Credentials
[ ] Passwords
[ ] Hashes
[ ] Kerberos
[ ] Tokens and Sessions
[ ] SSH Keys
[ ] Existing Access
```

For the current position, ask:

```text id="n4m8q1"
What identities exist?
What authentication material exists?
What authentication contexts already exist?
What services can use that material?
What targets are compatible?
What access has actually been validated?
What authorization exists?
What movement paths are available?
What information is still missing?
```

---

## 40. Credential-to-Access Decision Loop

The complete Section 05 workflow is:

```text id="r3k7m2"
Current Identity
      ↓
Authentication Material
      ↓
Authentication Context
      ↓
Compatible Service
      ↓
Candidate Target
      ↓
Reachability
      ↓
Authentication
      ↓
Authorization
      ↓
Existing / New Access
      ↓
Movement Decision
      ↓
New Host
      ↓
Re-enumerate
```

The critical distinction is:

```text id="x9p4v6"
Credential
    ↓
Potential Access
    ↓
Validated Access
    ↓
Movement
```

Do not skip the validation step.

---

## 41. Section 05 Completion Criteria

Credential and access discovery is sufficiently complete for the current position when you can answer:

```text id="m6x2q8"
[ ] What identity am I using?
[ ] What other identities are relevant?
[ ] What passwords were identified?
[ ] What hash material was identified?
[ ] What Kerberos context exists?
[ ] What active sessions exist?
[ ] What tokens / authentication contexts are relevant?
[ ] What SSH keys or configuration artifacts exist?
[ ] What existing remote access already exists?
[ ] Which services can use the discovered authentication material?
[ ] Which targets are candidates?
[ ] Which targets are reachable?
[ ] Which authentication attempts succeeded?
[ ] Which authorization levels were confirmed?
[ ] Which findings are historical or uncertain?
[ ] Which movement path should be investigated next?
```

The objective is not:

```text id="q8n3m5"
Collect every credential.
```

It is:

```text id="w4p7x1"
Build an evidence-based map of identities,
authentication contexts, services, targets, and
validated access that can support the next movement decision.
```

---

## 42. What Next?

Section 05 is now complete.

The next section changes the focus from:

```text
"What authentication material or access do I have?"
```

to:

```text
"Which remote services can I use to move?"
```

Continue with:

```text id="c9m2v7"
06-Remote-Access-Services/
└── README.md
```

The workflow becomes:

```text
Credential / Access Discovery
          ↓
Remote Service Discovery
          ↓
SMB
WinRM
RDP
WMI
RPC
SSH
          ↓
Match Authentication to Service
          ↓
Validate Remote Access
          ↓
Move to Target
          ↓
Post-Movement Enumeration
```

The core principle of Section 05 is:

> **Do not stop at finding credentials. Determine what authentication contexts and existing access they create, map them to reachable services and targets, validate authorization, and use the resulting evidence to choose the next movement path.**
