# Tokens and Sessions

Active sessions and security tokens can reveal authentication context that already exists on a compromised system.

For lateral movement, the key question is not simply:

> "What sessions exist?"

The practical question is:

> **Which identities are currently authenticated, what security context do those sessions represent, what services or resources are associated with them, and can that existing context provide a legitimate path to another system?**

The workflow is:

```text
Current Host
      ↓
Identify Active Sessions
      ↓
Identify Session Owners
      ↓
Identify Security Context
      ↓
Identify Network Connections
      ↓
Identify Authentication Context
      ↓
Map Identity → Service → Target
      ↓
Validate Authorized Access
      ↓
Record Result
      ↓
Re-enumerate
```

---

## 1. Where This Fits

Tokens and sessions are one branch of credential and access discovery:

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

They are especially useful when the current host is:

```text
Shared
Multi-User
Domain Joined
Server
Jump Host
Administrative Workstation
```

A session can reveal another identity or authentication context without requiring you to discover a password.

---

## 2. Session vs Token

These concepts are related but not identical.

### Session

A session represents an active user or service interaction with the system.

Examples:

```text
Interactive Login
Remote Desktop Session
SSH Session
Terminal Session
Service Session
Network Authentication Session
```

### Security Token

A security token represents the security context under which a process or operation executes.

Conceptually:

```text
Identity
   +
Groups
   +
Privileges
   +
Security Context
   ↓
Token
```

The practical relationship is:

```text
Session
   ↓
Processes
   ↓
Security Context / Token
   ↓
Potential Access
```

---

## 3. Why Sessions Matter

Suppose a compromised workstation has:

```text
Current User:
CORP\student
```

but another user is also logged in:

```text
CORP\admin
```

That second session is important information.

It may indicate:

```text
Administrative Activity
Management Workflow
Remote Administration
Privileged User Presence
Shared Workstation
```

However:

```text
Privileged User Session Found
        ≠
Automatic Administrative Access
```

The session must be understood in context.

---

## 4. First Identify the Current User

Before investigating other sessions, establish your own context.

### Linux

```bash
whoami
id
```

### Windows

```cmd
whoami
whoami /user
whoami /groups
```

Record:

```text
Current Identity
Account Authority
Groups
Privileges
```

This gives you the baseline for interpreting other sessions.

---

## 5. Windows: Identify Logged-In Users

A useful command is:

```cmd
query user
```

You can also use:

```cmd
qwinsta
```

These can reveal information such as:

```text
Username
Session ID
Session State
Logon Time
```

The objective is:

```text
Who is currently logged in?
```

---

## 6. Windows: Interpret Session State

Possible session states can indicate whether a session is:

```text
Active
Disconnected
Idle
```

Do not assume every listed session represents an immediately usable authentication context.

For example:

```text
Disconnected Session
        ≠
Current Interactive Access
```

Treat it as evidence that requires further investigation.

---

## 7. Windows: Identify Processes and Owners

Processes can provide context about active sessions.

Useful commands include:

```cmd
tasklist
```

For more detailed information:

```cmd
tasklist /v
```

PowerShell can provide process information:

```powershell
Get-Process
```

The goal is to associate:

```text
Process
   ↓
User / Context
   ↓
Activity
   ↓
Potential Network Relationship
```

---

## 8. Windows: Inspect Security Context

The current identity can be examined using:

```cmd
whoami /all
```

This helps identify:

```text
User SID
Groups
Privileges
Authentication-related context
```

The important distinction is:

```text
Identity
   +
Groups
   +
Privileges
   =
Security Context
```

Do not interpret one group or privilege in isolation.

---

## 9. Windows: Identify Network Connections

Existing network connections can reveal where active sessions are communicating.

Useful commands include:

```cmd
netstat -ano
```

or PowerShell:

```powershell
Get-NetTCPConnection
```

The objective is to identify:

```text
Local Endpoint
Remote Endpoint
Remote Port
Connection State
Process ID
```

This creates a useful relationship:

```text
Process
   ↓
Connection
   ↓
Remote Host
   ↓
Service
```

---

## 10. Connect Sessions to Network Activity

Suppose you identify:

```text
User:
CORP\admin
```

and separately observe:

```text
Remote Host:
FILE01
Port:
445
```

This does not prove the admin user owns that connection.

You need additional evidence connecting:

```text
User
Process
PID
Connection
Remote Host
```

Avoid making assumptions based on timing alone.

---

## 11. Windows: Remote Desktop Sessions

RDP sessions can be especially useful for understanding the current host.

Use:

```cmd
query user
```

to identify sessions.

You may discover:

```text
Session:
rdp-tcp#5

User:
CORP\administrator

State:
Active
```

This is evidence of an active interactive session.

It does not automatically mean that the session can be reused.

---

## 12. Linux: Identify Logged-In Users

Start with:

```bash
who
```

Then:

```bash
w
```

and:

```bash
users
```

These commands can reveal:

```text
Username
Terminal
Login Time
Source
Current Activity
```

The objective is to understand:

```text
Who else is using this system?
```

---

## 13. Linux: Identify User Processes

Use:

```bash
ps aux
```

or:

```bash
ps -ef
```

The output can help identify:

```text
Process Owner
PID
Command
Parent Process
Activity
```

A useful relationship is:

```text
User
 ↓
Process
 ↓
Network Activity
 ↓
Remote System
```

---

## 14. Linux: Identify SSH Sessions

SSH sessions can be particularly relevant.

Use:

```bash
who
```

or:

```bash
w
```

You can also inspect network connections:

```bash
ss -tnp
```

Look for:

```text
SSH Client
SSH Server
Remote Address
Established Connections
```

For example:

```text
alice
   ↓
SSH session
   ↓
10.10.20.15
```

This tells you that the host has an active SSH relationship with another system.

It does not automatically give you access to that remote system.

---

## 15. Linux: Identify Terminal and Login Context

Useful files and commands include:

```bash
last
```

and:

```bash
lastlog
```

These can provide historical login information.

Distinguish:

```text
Current Session
```

from:

```text
Historical Session
```

A historical login is useful as an environmental clue, but it should not be treated as proof of current access.

---

## 16. Session Recency

When investigating sessions, classify them:

```text
Current
Recent
Historical
Unknown
```

For example:

```text
Current:
Active SSH session

Recent:
User logged in earlier today

Historical:
User last logged in several weeks ago
```

Current and recent activity generally provide stronger evidence about the present environment.

---

## 17. Service Accounts

Not every process owner is a human user.

You may encounter:

```text
SYSTEM
LOCAL SERVICE
NETWORK SERVICE
root
www-data
mysql
postgres
```

or domain service identities.

A service account may have access to:

```text
Application
Database
File Share
Remote Service
Domain Resource
```

But:

```text
Service Account
   ≠
Administrative Account
```

Determine what the service actually needs and where it communicates.

---

## 18. Identify Service-to-Host Relationships

Suppose you find:

```text
Service:
Application

Account:
CORP\websvc

Remote Connection:
DB01:1433
```

You now have:

```text
WEB01
  ↓
websvc
  ↓
DB01
  ↓
MSSQL
```

This may reveal an internal application relationship.

The next question becomes:

> What authorized access does the current position have to that relationship?

---

## 19. Windows Token Context

A Windows process executes under a security context.

That context can include:

```text
User
Groups
Privileges
Integrity Level
Logon Session
```

The current context can be inspected with:

```cmd
whoami /all
```

For process-level investigation, the goal is to understand:

```text
Which identity is operating the process?
```

rather than simply:

```text
Which process exists?
```

---

## 20. Integrity Level Matters

Windows security contexts can operate at different integrity levels.

Conceptually:

```text
Low
 ↓
Medium
 ↓
High
 ↓
System
```

Integrity level helps explain what a process can interact with locally.

It should not be interpreted as a direct lateral-movement privilege ranking.

For lateral movement, the more important questions are:

```text
Which Identity?
Which Authentication Context?
Which Remote Services?
Which Targets?
```

---

## 21. Existing Authentication Contexts

A system may already contain authentication relationships such as:

```text
Kerberos Tickets
Mapped Network Resources
SMB Connections
SSH Sessions
Remote Desktop Sessions
Application Sessions
Service Connections
```

These should be considered together.

Example:

```text
User
 ↓
Kerberos Ticket
 ↓
CIFS
 ↓
FILE01
```

or:

```text
Service
 ↓
Database Connection
 ↓
DB01
```

The objective is to discover existing relationships before creating unnecessary new ones.

---

## 22. Windows Credential Context

Windows may maintain information about previously authenticated network resources.

A useful command for examining stored Windows credential entries is:

```cmd
cmdkey /list
```

This can reveal stored credential targets.

Treat the output as evidence about authentication context.

Do not assume that every listed entry provides a currently usable credential or remote administrative access.

---

## 23. Linux SSH Authentication Context

Linux systems may have active SSH relationships.

Inspect:

```bash
ss -tnp
```

and:

```bash
who
```

You may also inspect SSH configuration and authorized access paths when relevant:

```bash
ls -la ~/.ssh
```

The objective is to understand:

```text
Current SSH Sessions
SSH Client Configuration
Known Remote Hosts
Existing Authentication Relationships
```

SSH keys themselves are covered separately in:

```text
05-Credential-and-Access-Discovery/SSH-Keys.md
```

---

## 24. Sessions and Target Discovery

Sessions can reveal targets that ordinary host discovery might not immediately identify.

For example:

```text
Current Host
     ↓
Active SSH Connection
     ↓
10.10.20.15
     ↓
Candidate Target
```

or:

```text
Current Host
     ↓
SMB Connection
     ↓
FILE01
     ↓
Candidate Target
```

This is valuable because active connections provide environmental evidence.

---

## 25. Session-to-Target Mapping

Create a table:

| Identity   | Session / Context | Remote Host   | Service | Current State | Access Known    |
| ---------- | ----------------- | ------------- | ------- | ------------- | --------------- |
| CORP\alice | RDP               | WORKSTATION02 | RDP     | Active        | Unknown         |
| CORP\admin | SMB connection    | FILE01        | SMB     | Established   | Unknown         |
| root       | SSH               | SERVER02      | SSH     | Established   | Current session |

This turns session enumeration into movement analysis.

---

## 26. Do Not Assume Session Ownership

Suppose:

```text
User A
```

is logged in and:

```text
SERVER01
```

has an active network connection.

You cannot automatically conclude:

```text
User A → SERVER01
```

The connection may belong to:

```text
Service
Another User
System Process
Scheduled Task
Background Application
```

Always connect:

```text
User
 ↓
Process
 ↓
PID
 ↓
Connection
 ↓
Target
```

when the evidence supports it.

---

## 27. Existing Sessions vs Credential Discovery

A useful priority rule is:

```text
Existing Valid Context
        ↓
Investigate First
```

before:

```text
Search for New Credentials
```

Why?

Because an existing authenticated session may already provide the access needed for the next step.

The workflow becomes:

```text
Existing Session
      ↓
Identify Access
      ↓
Validate
      ↓
Move
```

rather than:

```text
Existing Session
      ↓
Ignore
      ↓
Search Everywhere for Passwords
```

---

## 28. Session Expiration and State

Not every session remains valid indefinitely.

Consider:

```text
Active
Disconnected
Expired
Logged Out
Idle
Unknown
```

A disconnected session is evidence of previous activity.

It should not automatically be treated as an active authentication channel.

---

## 29. Session Investigation Failure Handling

If you find no useful sessions:

```text
No Useful Session
       ↓
Check Network Connections
       ↓
Check Service Processes
       ↓
Check Authentication Context
       ↓
Check Credential Stores
       ↓
Continue to Next Credential Branch
```

Do not repeatedly enumerate the same information without a new hypothesis.

---

## 30. Decision Tree

```text
Start
  │
  ▼
Identify Current Identity
  │
  ▼
Identify Active Sessions
  │
  ▼
Other Identity Found?
  │
 ┌┴─────────────┐
No              Yes
 │               │
 ▼               ▼
Continue      Identify Session
Other Branch      │
                  ▼
             Identify Processes
                  │
                  ▼
             Identify Connections
                  │
                  ▼
             Identify Remote Hosts
                  │
                  ▼
             Identify Authentication
                  │
                  ▼
             Is Access Plausible?
               │        │
              No       Yes
               │        │
               ▼        ▼
           Record /   Validate
           Continue    Access
                         │
                         ▼
                  Check Authorization
                         │
                         ▼
                  Update Movement Map
```

---

## 31. Windows Workflow

From a Windows foothold:

```text
1. whoami
2. whoami /all
3. query user
4. qwinsta
5. tasklist /v
6. netstat -ano
7. cmdkey /list
8. Identify relevant processes
9. Correlate PIDs with connections
10. Identify remote hosts
11. Identify authentication context
12. Validate authorized access
13. Record findings
14. Reassess movement paths
```

The goal is not to run every command blindly.

The goal is:

```text
Session
 ↓
Identity
 ↓
Process
 ↓
Connection
 ↓
Remote System
 ↓
Authentication
 ↓
Authorization
```

---

## 32. Linux Workflow

From a Linux foothold:

```text
1. whoami
2. id
3. who
4. w
5. last
6. ps aux
7. ss -tnp
8. Identify SSH sessions
9. Identify service processes
10. Correlate users with connections
11. Identify remote hosts
12. Identify authentication relationships
13. Validate authorized access
14. Record findings
15. Reassess movement paths
```

Again, the important part is the interpretation.

---

## 33. Example: Windows Session

Suppose:

```text
query user
```

shows:

```text
CORP\admin    Active
```

and:

```text
netstat -ano
```

shows an established connection to:

```text
FILE01:445
```

Do not immediately conclude:

```text
Admin access to FILE01
```

Instead:

```text
1. Identify the connection PID.
2. Identify the owning process.
3. Determine the process security context.
4. Determine whether the connection belongs to CORP\admin.
5. Determine the authentication mechanism.
6. Validate authorized access to FILE01.
```

This is the workflow mindset.

---

## 34. Example: Linux Session

Suppose:

```text
who
```

shows:

```text
alice pts/0
```

and:

```text
ss -tnp
```

shows an SSH connection to:

```text
10.10.20.15:22
```

The evidence suggests:

```text
alice
 ↓
SSH
 ↓
10.10.20.15
```

Now determine:

```text
Is the SSH connection associated with alice?
Is the session active?
What authentication context is being used?
Is 10.10.20.15 a relevant target?
What access does the current assessment already have?
```

Only then choose the next action.

---

## 35. Common Mistakes

### Mistake 1: Treating Every Logged-In User as a Credential

A logged-in user is evidence of a session, not a password.

---

### Mistake 2: Assuming a Privileged Session Can Be Reused

A privileged session may be isolated or protected.

Determine the actual security context.

---

### Mistake 3: Ignoring Service Accounts

Important remote relationships may belong to services rather than humans.

---

### Mistake 4: Ignoring Process Ownership

A network connection alone does not identify its owner.

---

### Mistake 5: Confusing Historical Activity With Current Access

`last` can show historical logins.

That does not prove current access.

---

### Mistake 6: Treating a Disconnected Session as Active

Session state matters.

---

### Mistake 7: Ignoring Existing Authentication

Existing sessions and connections may provide better evidence than searching blindly for new credentials.

---

## 36. Session Investigation Record

Use this structure:

```text
Host:
____________________

Session Owner:
____________________

Session Type:
____________________

Session State:
____________________

Process / PID:
____________________

Remote Host:
____________________

Remote Service:
____________________

Authentication Context:
____________________

Current Access:
____________________

Authorization:
____________________

Evidence:
____________________

Movement Relevance:
____________________

Next Action:
____________________
```

---

## 37. Completion Criteria

Session and token investigation is sufficiently complete for the current position when you can answer:

```text
[ ] Who am I?
[ ] Who else is currently logged in?
[ ] Which sessions are active?
[ ] Which sessions are disconnected or historical?
[ ] Which processes belong to relevant identities?
[ ] Which remote connections exist?
[ ] Which processes own those connections?
[ ] Which remote hosts are involved?
[ ] Which authentication mechanisms are relevant?
[ ] Which existing authentication contexts exist?
[ ] Which targets are potentially reachable?
[ ] Has useful access been validated?
[ ] Has authorization been validated?
[ ] Have findings been recorded?
[ ] What is the next movement decision?
```

The objective is not:

```text
Find every process.
```

It is:

```text
Identify authentication and session context that can
change the current movement possibilities.
```

---

## 38. What Next?

The next branch focuses on a particularly important Linux and Unix authentication mechanism:

> **"Are SSH keys or SSH configuration artifacts present that identify existing remote access relationships?"**

Continue with:

```text
05-Credential-and-Access-Discovery/
└── SSH-Keys.md
```

The workflow becomes:

```text
Tokens and Sessions
      ↓
SSH Keys
      ↓
Identify Key / Configuration
      ↓
Identify Associated User
      ↓
Identify Candidate Host
      ↓
Determine Authentication Compatibility
      ↓
Validate Authorized SSH Access
      ↓
Update Movement Map
```

The core principle is:

> **A session or token is an authentication context, not automatically reusable access. Identify its owner, state, process, remote relationship, and authorization before treating it as a lateral-movement opportunity.**
