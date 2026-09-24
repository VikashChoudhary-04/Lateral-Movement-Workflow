# Current Host

The first step in understanding your position is identifying the system you currently control.

The objective is not to collect every possible system detail.

The objective is to determine:

```text
What system am I on?
What operating system is it running?
What version and architecture does it use?
What role might it serve?
What services and applications may make it useful?
What information can help identify lateral-movement opportunities?
```

---

## 1. Host Identification

Start by identifying the hostname.

### Linux

```bash
hostname
```

```bash
cat /etc/hostname
```

For additional system information:

```bash
hostnamectl
```

### Windows

```cmd
hostname
```

```cmd
echo %COMPUTERNAME%
```

PowerShell:

```powershell
$env:COMPUTERNAME
```

### Record

```text
Hostname:
```

---

## 2. Why the Hostname Matters

A hostname can provide an initial clue about the system's role.

Examples:

```text
WEB01
FILE01
APP01
DB01
DC01
WORKSTATION01
```

Possible interpretations:

| Hostname pattern | Possible indication |
| ---------------- | ------------------- |
| WEB01            | Web server          |
| FILE01           | File server         |
| APP01            | Application server  |
| DB01             | Database server     |
| DC01             | Domain Controller   |
| WS01             | Workstation         |

These are **clues, not proof**.

A hostname may be misleading, outdated, or simply follow an organization's naming convention.

### Action

Use the hostname as a starting hypothesis.

Then validate the system's actual role using:

* Operating system information
* Running services
* Installed applications
* Network connections
* Domain context

---

# 3. Identify the Operating System

Knowing the OS determines which tools, services, authentication mechanisms, and movement paths are relevant.

---

## Linux

Start with:

```bash
uname -a
```

Then identify the distribution:

```bash
cat /etc/os-release
```

Kernel version:

```bash
uname -r
```

Architecture:

```bash
uname -m
```

### Record

```text
OS:
Distribution:
Version:
Kernel:
Architecture:
```

---

## Windows

Start with:

```cmd
systeminfo
```

For a quick version check:

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
Build:
Architecture:
```

---

# 4. Why the OS Matters

The operating system determines the movement mechanisms that are likely to be relevant.

For example:

```text
Linux
  │
  └── SSH and other Linux-compatible remote services

Windows
  │
  ├── SMB
  ├── WinRM
  ├── RDP
  ├── WMI
  └── RPC
```

This does **not** mean that every service exists on every host.

You still need to enumerate what is actually available.

### Decision

```text
Linux
   ↓
Prioritize Linux-compatible services
   ↓
Investigate SSH and other discovered services
```

```text
Windows
   ↓
Investigate discovered Windows remote services
   ↓
SMB / WinRM / RDP / WMI / RPC
```

---

# 5. Identify Architecture

Determine whether the system is running a 32-bit or 64-bit architecture where relevant.

### Linux

```bash
uname -m
```

### Windows

```cmd
systeminfo
```

PowerShell:

```powershell
[Environment]::Is64BitOperatingSystem
```

### Why?

Architecture can affect:

* Available binaries
* Compatibility of tools
* Exploitability of software
* Installed applications
* Remote administration tooling

For lateral movement specifically, architecture is usually **supporting information**, not the primary decision factor.

Record it, but don't spend excessive time on it unless a later action depends on it.

---

# 6. Determine the Likely Host Role

Now move from **identity** to **purpose**.

Ask:

> **What does this machine appear to do?**

Useful evidence includes:

```text
Hostname
OS
Running services
Installed applications
Listening ports
Network connections
Domain membership
Configuration
```

### Linux

Start by examining services/processes:

```bash
ps aux
```

```bash
ss -lntup
```

Depending on the distribution:

```bash
systemctl --type=service --state=running
```

### Windows

Processes:

```cmd
tasklist
```

Services:

```cmd
sc query
```

PowerShell:

```powershell
Get-Service
```

Listening ports:

```cmd
netstat -ano
```

PowerShell:

```powershell
Get-NetTCPConnection -State Listen
```

---

# 7. Interpret the Evidence

Do not simply record every service.

Look for combinations of evidence.

For example:

```text
Hostname: APP01
        +
Application services
        +
Database connection
        ↓
Likely application server
```

Or:

```text
Hostname: FILE01
        +
SMB-related services
        +
Large number of network shares
        ↓
Potential file server
```

Or:

```text
Hostname: DC01
        +
Domain-related services
        +
Directory services
        ↓
Potential Domain Controller
```

The purpose is to build a **working understanding of the host**.

---

# 8. Identify Listening Services

A listening service can represent a potential communication path into or out of the host.

### Linux

```bash
ss -lntup
```

If available:

```bash
netstat -lntup
```

### Windows

```cmd
netstat -ano
```

PowerShell:

```powershell
Get-NetTCPConnection -State Listen
```

### Record Interesting Findings

```text
Port:
Protocol:
Process / Service:
Purpose:
```

For example:

```text
22/tcp  → SSH
445/tcp → SMB
3389/tcp → RDP
5985/tcp → WinRM
5986/tcp → WinRM over HTTPS
```

These are **potential remote-access paths**, not automatic movement opportunities.

The next step is to determine:

```text
Is it reachable?
Is authentication required?
Do I have suitable credentials?
Is the account authorized?
```

---

# 9. Identify Running Applications

Applications can reveal the role of the host and potential relationships with other systems.

### Linux

Useful starting points include:

```bash
ps aux
```

Package information depends on the distribution.

For Debian-based systems:

```bash
dpkg -l
```

For RPM-based systems:

```bash
rpm -qa
```

### Windows

Installed applications can be reviewed through PowerShell or the system's installed-program information.

A practical starting point:

```powershell
Get-Process
```

For software inventory, use the appropriate system management interfaces available in the environment.

### Why?

Applications can reveal:

* Web applications
* Databases
* Development environments
* Management software
* Security tools
* Enterprise applications
* Connections to other systems

The goal is to identify **relationships**, not simply produce a software inventory.

---

# 10. Inspect Current Network Connections

The current host may already be communicating with other systems.

### Linux

```bash
ss -tunap
```

### Windows

```cmd
netstat -ano
```

PowerShell:

```powershell
Get-NetTCPConnection
```

### Look For

```text
Current Host
     │
     ├──→ Web Server
     ├──→ Database
     ├──→ File Server
     ├──→ Domain Controller
     └──→ Other Internal Host
```

These connections can reveal:

* Internal servers
* Databases
* APIs
* Authentication infrastructure
* Administrative relationships
* Potential movement targets

### Important

A connection does not automatically mean:

> "I can access that system."

It means:

> **"This host communicates with that system, so I should investigate the relationship."**

---

# 11. Identify the Host's Role in the Environment

Combine the evidence you've gathered.

Create a short assessment:

```text
Host:
OS:
Likely Role:
Domain:
IP:
Interesting Services:
Interesting Applications:
Interesting Connections:
Potential Targets:
```

Example:

```text
Host: WEB01
OS: Windows Server
Likely Role: Web/Application Server
Domain: CORP.LOCAL
IP: 10.10.10.15
Interesting Services: HTTP, WinRM
Interesting Application: Internal Web Application
Interesting Connection: DB01
Potential Target: DB01
```

The final line is the important part.

The objective is to convert host information into **potential movement opportunities**.

---

# 12. Role → Movement Thinking

Different host roles can reveal different next steps.

### Web/Application Server

Ask:

```text
What systems does the application communicate with?
What databases does it use?
What internal APIs exist?
What administrative services are available?
What accounts operate the application?
```

### File Server

Ask:

```text
What remote file-sharing services exist?
Who accesses this server?
What systems connect to it?
What administrative services are exposed?
```

### Database Server

Ask:

```text
Which applications connect to it?
Which accounts authenticate to it?
What management services are exposed?
```

### Domain-Connected Workstation

Ask:

```text
What domain is it part of?
Which users access it?
What domain systems can it reach?
What authentication relationships exist?
```

### Potential Domain Controller

Ask:

```text
Is this actually a Domain Controller?
What domain does it serve?
What services confirm its role?
What systems depend on it?
```

Do not assume the role from the hostname alone.

---

# 13. What If the Host Appears Uninteresting?

That's a valid result.

For example:

```text
No unusual services
No interesting connections
No obvious applications
No obvious targets
```

Do not force a movement attempt.

Instead:

```text
Current Host
     ↓
No obvious movement opportunity
     ↓
Review:
- Network context
- Credentials / access
- Other hosts
- Existing sessions
     ↓
Continue with the appropriate workflow
```

The absence of an obvious opportunity is itself useful information.

---

# 14. What If You Discover a Potential Target?

Suppose you identify:

```text
WEB01
   │
   └── communicates with → DB01
```

Do not immediately attempt access.

Move into the target-discovery workflow:

```text
04-Target-Discovery/
```

The next questions are:

```text
Can I reach DB01?
What services does DB01 expose?
Do I have relevant credentials?
What authentication does the service require?
What access would successful authentication provide?
```

---

# 15. Host Assessment Checklist

```text
[ ] Hostname identified
[ ] Operating system identified
[ ] OS version identified
[ ] Architecture identified
[ ] Likely host role assessed
[ ] Running services reviewed
[ ] Listening services identified
[ ] Interesting applications identified
[ ] Current network connections reviewed
[ ] Domain context noted
[ ] Interesting system relationships identified
[ ] Potential targets recorded
[ ] Next workflow selected
```

---

# 16. Final Host Assessment

You should finish this file with a concise statement:

```text
I am currently on:
____________________

The system appears to be:
____________________

The operating system is:
____________________

The most relevant services are:
____________________

The system communicates with:
____________________

The most interesting potential target is:
____________________

The next thing I need to determine is:
____________________
```

If you can answer these questions, you have converted the host from an unknown foothold into a **known operational position**.

---

## Next

Continue to:

```text
03-Position-Assessment/Current-User.md
```

The next file answers:

> **"Who exactly am I on this system, what account context do I have, and what does that identity mean for lateral movement?"**
