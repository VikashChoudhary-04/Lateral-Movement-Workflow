# First 5 Minutes

The first few minutes after obtaining a foothold are about **understanding your current position before attempting lateral movement**.

Do not immediately start trying random remote services or credentials.

First determine:

```text id="0cmh4v"
Where am I?
Who am I?
What privileges do I have?
What system am I on?
What network am I connected to?
What domain or workgroup am I part of?
What systems can I reach?
What existing access is available?
```

The objective is to turn an unknown foothold into a **known position**.

---

## 1. The First 5-Minute Workflow

Use this sequence as the default starting point:

```text id="f6d0w7"
        INITIAL FOOTHOLD
              │
              ▼
        1. Identify Host
              │
              ▼
        2. Identify User
              │
              ▼
        3. Identify Privileges
              │
              ▼
        4. Identify OS
              │
              ▼
        5. Identify Network
              │
              ▼
        6. Identify Domain / Workgroup
              │
              ▼
        7. Identify Reachable Networks
              │
              ▼
        8. Identify Existing Sessions / Access
              │
              ▼
        9. Record Findings
              │
              ▼
       10. Choose Next Action
```

This is not a rigid checklist.

If a finding immediately reveals a useful movement opportunity, follow it.

---

# 2. Step 1 — Identify the Current Host

First determine exactly which system you have accessed.

## Linux

```bash
hostname
hostnamectl
```

Also inspect the system identity:

```bash
cat /etc/hostname
```

## Windows

```cmd
hostname
```

or:

```cmd
echo %COMPUTERNAME%
```

PowerShell:

```powershell
$env:COMPUTERNAME
```

## What to Record

```text
Hostname:
OS:
IP:
Domain / Workgroup:
Current User:
```

### Why?

You need to know which system represents your current position.

A hostname may also reveal the system's role.

For example:

```text
WEB01
FILE01
APP01
DC01
SQL01
```

These names are useful clues, but **do not assume the role solely from the hostname**.

### Next Action

Continue to identifying the current user.

---

# 3. Step 2 — Identify the Current User

Determine which account your current session represents.

## Linux

```bash
whoami
```

Additional context:

```bash
id
```

```bash
groups
```

## Windows

```cmd
whoami
```

```cmd
whoami /user
```

```cmd
whoami /groups
```

PowerShell:

```powershell
[System.Security.Principal.WindowsIdentity]::GetCurrent().Name
```

## What to Record

```text
Username:
UID / SID:
Groups:
Domain:
```

### Why?

The account determines what access may already be available.

For example:

```text
LOCAL USER
DOMAIN USER
LOCAL ADMINISTRATOR
DOMAIN ADMINISTRATOR
SERVICE ACCOUNT
SYSTEM
ROOT
```

Do not assume the account's name tells you its effective privileges. Verify them.

### Next Action

Determine the effective privileges.

---

# 4. Step 3 — Identify Current Privileges

Knowing the username is not enough.

Determine what the current session can actually do.

## Linux

```bash
id
```

Check sudo permissions where appropriate:

```bash
sudo -l
```

Check group membership:

```bash
groups
```

## Windows

```cmd
whoami /groups
```

```cmd
whoami /priv
```

For administrative context:

```cmd
net localgroup administrators
```

### What to Look For

Examples include:

```text
Linux:
- root
- sudo access
- privileged groups

Windows:
- Administrators
- SYSTEM
- SeImpersonatePrivilege
- SeDebugPrivilege
- Other relevant privileges
```

### Why?

Privilege level changes what you can discover and what access paths may be available.

For example:

```text
Low privilege
     │
     ├── Limited local access
     │
     ▼
Higher privilege
     │
     ├── More credential visibility
     ├── More system visibility
     └── Potentially more remote access
```

If privilege escalation is required, use the appropriate PrivEsc workflow.

### Next Action

Identify the operating system and environment.

---

# 5. Step 4 — Identify the Operating System

You need to know what environment you're operating in.

## Linux

```bash
uname -a
```

```bash
cat /etc/os-release
```

For kernel information:

```bash
uname -r
```

## Windows

```cmd
systeminfo
```

or:

```cmd
ver
```

PowerShell:

```powershell
Get-ComputerInfo
```

### Record

```text
OS:
Version:
Architecture:
Kernel / Build:
```

### Why?

The operating system determines:

* Available tools
* Available remote services
* Authentication mechanisms
* Network commands
* Credential locations
* Likely movement paths

### Next Action

Map the network context.

---

# 6. Step 5 — Identify Network Interfaces

Determine which networks the current host is connected to.

## Linux

```bash
ip addr
```

or:

```bash
ip a
```

Check routes:

```bash
ip route
```

## Windows

```cmd
ipconfig /all
```

Check routes:

```cmd
route print
```

PowerShell:

```powershell
Get-NetIPConfiguration
```

and:

```powershell
Get-NetRoute
```

## Record

```text
Interface:
IP:
Subnet:
Gateway:
DNS:
Additional networks:
```

### Why?

This is one of the most important steps for lateral movement.

Your current host may have access to multiple networks:

```text id="d2b5n4"
             COMPROMISED HOST
                    │
        ┌───────────┴───────────┐
        ▼                       ▼
    Network A                Network B
    10.10.10.0/24            10.10.20.0/24
```

That can reveal movement opportunities that were not visible from the original attacker position.

### Next Action

Inspect routing and reachability.

---

# 7. Step 6 — Understand Routing

Interfaces tell you where the host is connected.

Routes tell you **where traffic can go**.

## Linux

```bash
ip route
```

## Windows

```cmd
route print
```

or:

```cmd
netstat -rn
```

### Look For

```text
Local subnet
Additional subnets
Default gateway
Internal routes
Unexpected networks
```

### Example

```text id="s6e1pl"
10.10.10.0/24
10.10.20.0/24
10.10.30.0/24
```

This tells you that the current host may have routes toward multiple networks.

It does **not** automatically mean every host on those networks is reachable.

Reachability must be validated.

### Next Action

Move into target discovery.

---

# 8. Step 7 — Identify the Domain or Workgroup

This is particularly important in Windows and Active Directory environments.

## Windows

```cmd
whoami
```

```cmd
echo %USERDOMAIN%
```

```cmd
systeminfo
```

You can also inspect:

```cmd
wmic computersystem get domain
```

PowerShell:

```powershell
(Get-CimInstance Win32_ComputerSystem).Domain
```

## Linux

In an AD-integrated Linux environment, inspect the available identity/domain context using the tools configured on the system.

Useful starting points may include:

```bash
hostname
```

```bash
id
```

```bash
getent passwd
```

If domain integration is present, investigate it using the environment's configured identity-management tooling.

### Record

```text
Domain:
Workgroup:
Computer:
Current account context:
```

### Why?

Domain context can significantly change the available movement paths.

For example:

```text id="6u6s2g"
WORKSTATION01
      │
      ▼
CORP.LOCAL
      │
      ├── FILE01
      ├── APP01
      ├── SQL01
      └── DC01
```

### Next Action

Determine what other systems may be reachable.

---

# 9. Step 8 — Look for Existing Sessions and Connections

Existing sessions can reveal useful information about the current environment.

## Linux

Start with:

```bash
who
```

```bash
w
```

Check active network connections:

```bash
ss -tunap
```

If available:

```bash
netstat -tunap
```

## Windows

Logged-in users:

```cmd
query user
```

Network connections:

```cmd
netstat -ano
```

PowerShell:

```powershell
Get-NetTCPConnection
```

### What to Look For

You may find:

```text
Other logged-in users
Active remote sessions
Connections to servers
Database connections
Administrative connections
Internal services
```

### Why?

Existing connections can reveal:

* Other systems
* Internal services
* User activity
* Potential targets
* Network relationships

Do not automatically treat every connection as an available movement path.

Treat it as **evidence about the environment**.

---

# 10. Step 9 — Identify Useful Local Information

Before leaving the current host, perform a quick assessment for information that can influence movement.

Look for:

```text
Users
Groups
Running services
Network connections
Mounted resources
Shared resources
Configuration files
Application information
Credential locations
SSH configuration
Windows remote-management configuration
```

The objective is not to fully enumerate the host here.

The objective is:

> **Find information that changes your lateral-movement decisions.**

Detailed credential discovery belongs in:

```text
05-Credential-and-Access-Discovery/
```

Detailed target discovery belongs in:

```text
04-Target-Discovery/
```

---

# 11. Step 10 — Record Your Position

Do not rely entirely on memory.

Create a simple mental or written position summary:

```text
Current Host:
Current User:
Privileges:
OS:
IP:
Subnet:
Gateway:
Routes:
Domain:
Reachable Networks:
Interesting Connections:
Potential Targets:
Available Credentials / Access:
```

Example:

```text id="w0ykr6"
Host: WEB01
User: websvc
Privilege: Low
OS: Windows Server
IP: 10.10.10.15
Subnet: 10.10.10.0/24
Domain: CORP.LOCAL
Routes: 10.10.10.0/24, 10.10.20.0/24
Interesting Connection: FILE01
Potential Target: FILE01
Access: Credential discovered for investigation
```

This turns scattered observations into a usable **attack-position map**.

---

# 12. Decide What to Do Next

At this point, do not automatically move.

Use the information you've gathered.

```text id="wbyqk8"
                    Current Position
                          │
          ┌───────────────┼───────────────┐
          ▼               ▼               ▼
     No Target        Target Found    New Network
          │               │               │
          ▼               ▼               ▼
   Target Discovery   Service Check    Pivoting /
                                      Network Access
                          │
                          ▼
                  Credential / Access
                      Discovery
                          │
                          ▼
                  Remote Service
                      Discovery
                          │
                          ▼
                 Movement Decision
```

### If you found a potential target

Continue to:

```text
04-Target-Discovery/
```

### If you found useful credentials or authentication material

Continue to:

```text
05-Credential-and-Access-Discovery/
```

### If you already know the target and remote service

Continue to:

```text
06-Remote-Access-Services/
```

### If the target is on an inaccessible internal network

Investigate:

```text
09-Pivoting-and-Internal-Network-Access/
```

### If you need higher privileges first

Use the relevant Privilege Escalation workflow.

---

# 13. What Not to Do

Avoid these common mistakes.

### Don't immediately attack every host

```text
Host discovered
    ↓
Immediately attempt access
```

Instead:

```text
Host discovered
    ↓
Understand target
    ↓
Identify services
    ↓
Determine available access
    ↓
Choose movement path
```

---

### Don't assume credentials work everywhere

```text
Credential found
    ≠
Access everywhere
```

Validate where and how the credential can be used.

---

### Don't confuse reachability with access

```text
Host reachable
    ≠
Host accessible
```

You may be able to communicate with a service without being authorized to use it.

---

### Don't stop after gaining access

```text
HOST-A
  ↓
HOST-B
```

is not the end.

Immediately treat `HOST-B` as your new position:

```text
HOST-B
  ↓
Who am I?
Where am I?
What can I reach?
What access do I have?
What can I discover?
```

---

# 14. The First-5-Minutes Decision Tree

Use this condensed workflow when operating in a lab or authorized assessment:

```text id="f2c5b7"
                  INITIAL FOOTHOLD
                         │
                         ▼
                   Identify Host
                         │
                         ▼
                   Identify User
                         │
                         ▼
                Identify Privileges
                         │
                         ▼
                    Identify OS
                         │
                         ▼
                 Map Network Interfaces
                         │
                         ▼
                    Check Routes
                         │
                         ▼
                Identify Domain Context
                         │
                         ▼
              Inspect Existing Connections
                         │
                         ▼
               Record Current Position
                         │
                         ▼
                 Is a Target Known?
                    /          \
                  NO            YES
                  │              │
                  ▼              ▼
          Target Discovery   Service Discovery
                                   │
                                   ▼
                           Access Discovery
                                   │
                                   ▼
                          Movement Decision
                                   │
                                   ▼
                              Move
                                   │
                                   ▼
                           Validate Access
                                   │
                                   ▼
                        Start Again on New Host
```

---

# 15. First-5-Minutes Checklist

Use this as a quick operational checklist:

```text
[ ] Identify hostname
[ ] Identify current user
[ ] Identify current privileges
[ ] Identify operating system
[ ] Identify IP addresses
[ ] Identify network interfaces
[ ] Identify routes
[ ] Identify gateway
[ ] Identify DNS/domain context
[ ] Identify existing sessions
[ ] Identify active network connections
[ ] Note interesting systems
[ ] Note interesting services
[ ] Note available access material
[ ] Record the current position
[ ] Select the next workflow branch
```

---

# 16. The Goal

The goal of the first five minutes is **not** to complete lateral movement.

The goal is to transform:

```text id="rj1qef"
"I have a shell."
```

into:

```text id="5byfhs"
"I know where I am,
I know who I am,
I understand my privileges,
I understand my network position,
I know what I can potentially reach,
I know what access I have,
and I know what I should investigate next."
```

That is the foundation for making deliberate lateral-movement decisions.

---

## Next

Continue to:

```text id="r4bq8p"
03-Position-Assessment/
```

The next section goes deeper into understanding the current position and turns the initial five-minute survey into **structured, actionable enumeration**.
