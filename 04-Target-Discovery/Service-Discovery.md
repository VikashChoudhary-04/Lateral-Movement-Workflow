# Service Discovery

Service discovery is the process of identifying the network services exposed by a candidate target and determining whether those services create a plausible path to remote access.

The objective is not simply to collect open ports.

The objective is to answer:

> **What can I communicate with on this target, how does it authenticate, and could that service provide a movement path?**

The workflow is:

```text
Candidate Host
      ↓
Confirm Reachability
      ↓
Discover Exposed Services
      ↓
Identify Service / Protocol
      ↓
Identify Version / Role Clues
      ↓
Determine Authentication Model
      ↓
Map Available Access
      ↓
Identify Potential Movement Paths
      ↓
Record Evidence
      ↓
Prioritize Target
```

---

# 1. Where This Fits

The target-discovery workflow is:

```text
Position Assessment
       ↓
Host Discovery
       ↓
Network Enumeration
       ↓
Service Discovery
       ↓
Target Prioritization
       ↓
Credential / Access Discovery
```

Previous files answered:

```text
Host Discovery
→ What systems appear to exist?

Network Enumeration
→ How do those systems relate to my current position?
```

This file answers:

```text
Service Discovery
→ What can I interact with on those systems?
```

---

# 2. Port vs Service vs Access

Keep these concepts separate.

```text
Port
  ↓
Potential Service
  ↓
Confirmed Service
  ↓
Authentication Method
  ↓
Authorization
  ↓
Validated Access
```

For example:

```text
445/tcp open
```

does not automatically mean:

```text
SMB access confirmed
```

It means that something is responding on TCP port 445.

The service must be identified and access must be validated separately.

---

# 3. Start With a Specific Target

Do not begin service discovery without context.

For each candidate target, already know as much as possible about:

```text
IP Address
Hostname
Network
Reachability
Potential Role
Current Assessment Objective
```

Example:

```text
Target:
10.10.10.20

Hostname:
FILE01

Network:
10.10.10.0/24

Potential Role:
File Server

Reachability:
Confirmed
```

Now service discovery has a specific purpose:

> **Determine how FILE01 can be interacted with remotely.**

---

# 4. Basic Service Discovery With Nmap

A common starting point is:

```bash
nmap <target>
```

For example:

```bash
nmap 10.10.10.20
```

This can identify common open TCP ports.

Example:

```text
22/tcp   open  ssh
80/tcp   open  http
445/tcp  open  microsoft-ds
```

The result provides candidates for deeper investigation.

It does not establish access.

---

# 5. Service and Version Detection

When a service is relevant, identify it more precisely.

For example:

```bash
nmap -sV 10.10.10.20
```

Example:

```text
22/tcp   open  ssh   OpenSSH
80/tcp   open  http  Apache
445/tcp  open  microsoft-ds
```

Now you have more context:

```text
Port
  ↓
Protocol
  ↓
Service
  ↓
Version / Implementation
```

Version information can help determine which remote-access workflow applies.

---

# 6. Default Scripts for Additional Context

When appropriate and authorized, Nmap's default scripts can provide additional service information:

```bash
nmap -sC -sV 10.10.10.20
```

This may provide information such as:

```text
Service banners
Protocol details
Certificate information
Basic authentication-related information
Host naming information
```

Do not interpret every script result as proof of a vulnerability or access.

Treat the output as evidence that guides the next step.

---

# 7. Scan Only What You Need

Service discovery should be incremental.

A useful progression is:

```text
Initial Discovery
      ↓
Identify Open Ports
      ↓
Identify Relevant Services
      ↓
Enumerate Specific Service
```

For example:

```text
Initial:
nmap 10.10.10.20

Finding:
445/tcp open

Next:
Investigate SMB
```

Instead of immediately performing every possible scan against every system.

---

# 8. Identify Remote Access Services

Some services are particularly relevant to lateral movement.

Common examples include:

| Service            |      Typical Port | Movement Relevance                     |
| ------------------ | ----------------: | -------------------------------------- |
| SSH                |                22 | Remote shell / administration          |
| SMB                |               445 | Windows file and administrative access |
| RDP                |              3389 | Interactive Windows access             |
| WinRM              |         5985/5986 | Windows remote management              |
| RPC                |               135 | Windows RPC infrastructure             |
| WMI-related access | 135 + dynamic RPC | Windows remote administration          |
| FTP                |                21 | File transfer; context-dependent       |
| VNC                |             5900+ | Remote desktop; context-dependent      |

The port number is only an initial clue.

Always identify the actual service.

---

# 9. SSH

If you discover:

```text
22/tcp open ssh
```

the next questions are:

```text
Is SSH actually running?
What implementation/version is present?
Does it permit the authentication methods relevant to the assessment?
Do I possess potentially valid credentials or keys?
```

Do not immediately attempt random credentials.

The dedicated movement workflow later covers:

```text
06-Remote-Access-Services/SSH.md
```

---

# 10. SMB

If you discover:

```text
445/tcp open
```

on a Windows target, investigate SMB context.

Questions include:

```text
Is SMB available?
What host/domain information is exposed?
Is authentication required?
What account context is relevant?
Do I have potentially valid credentials?
```

The detailed workflow belongs in:

```text
06-Remote-Access-Services/SMB.md
```

---

# 11. RDP

If:

```text
3389/tcp open
```

appears, the target may expose Remote Desktop Services.

The important questions are:

```text
Is RDP actually available?
Who is authorized to use it?
What authentication context applies?
Do I have a potentially valid account?
Is interactive access relevant to the current objective?
```

Do not treat:

```text
3389 open
```

as:

```text
RDP access confirmed
```

---

# 12. WinRM

If:

```text
5985/tcp open
```

or:

```text
5986/tcp open
```

appears, investigate Windows Remote Management.

The movement decision becomes:

```text
WinRM Service
      ↓
Authentication Model
      ↓
Available Credentials
      ↓
Authorization
      ↓
Remote Access Validation
```

Detailed usage belongs in:

```text
06-Remote-Access-Services/WinRM.md
```

---

# 13. HTTP and HTTPS

Web services may not provide direct lateral movement by themselves.

For example:

```text
80/tcp open http
443/tcp open https
```

may indicate:

```text
Application Server
Internal Portal
Management Interface
API
Web Administration Console
```

The important question is:

> **Does this service provide information or an authorized access path that matters to the movement chain?**

Do not automatically treat every web server as a lateral-movement path.

---

# 14. Service Role Matters

Consider:

```text
Target A
445/SMB
```

versus:

```text
Target B
80/HTTP
443/HTTPS
```

The services suggest different investigation paths.

Now add host roles:

```text
FILE01
  └── SMB

APP01
  └── HTTP/HTTPS

ADMIN01
  └── RDP / WinRM
```

The combination of:

```text
Host Role
+
Service
+
Authentication
```

is much more useful than a port number alone.

---

# 15. Identify Service Ownership and Context

On the target, if you have legitimate administrative visibility, determine which process/service owns an interesting port.

For example, on Windows:

```powershell
Get-NetTCPConnection -LocalPort <PORT>
```

On Linux:

```bash
ss -lntp
```

This can help answer:

```text
What process is listening?
Which application owns it?
Is this an expected service?
Does the service correspond to the hypothesized host role?
```

This is particularly useful when a service appears unexpected.

---

# 16. Banner Information

Some services expose banners.

For example:

```text
SSH-2.0-OpenSSH_...
```

or HTTP headers may reveal:

```text
Server:
Application:
Technology:
```

Banner information can provide:

```text
Implementation
Version
Application
Operating System Clues
```

But banners can be:

```text
Hidden
Modified
Incomplete
Incorrect
```

Treat them as evidence, not absolute truth.

---

# 17. Service Identification Workflow

Use:

```text
Open Port
    ↓
Identify Protocol
    ↓
Identify Service
    ↓
Identify Implementation
    ↓
Identify Version
    ↓
Understand Authentication
    ↓
Determine Relevant Access
```

Example:

```text
445/tcp
    ↓
SMB
    ↓
Windows File Sharing
    ↓
Domain Authentication Relevant
    ↓
Known Domain Credential
    ↓
Validate SMB Access
```

---

# 18. Authentication Is the Key Transition

Service discovery becomes useful for lateral movement when you connect it to authentication.

For every interesting service ask:

```text
How does this service authenticate users?
```

Examples:

```text
SSH
→ Password / SSH key / other configured methods

SMB
→ Windows authentication mechanisms

RDP
→ Windows interactive authentication

WinRM
→ Windows remote management authentication

Web Application
→ Application-specific authentication
```

The exact authentication mechanism depends on configuration.

Do not assume it solely from the port number.

---

# 19. Map Services to Existing Access

Suppose you discover:

```text
FILE01
445/tcp SMB
```

and separately know:

```text
CORP\alice
```

The next question becomes:

```text
Can CORP\alice authenticate to FILE01?
```

If you have evidence of valid credentials:

```text
Credential
   ↓
Compatible Service
   ↓
Candidate Target
   ↓
Validate Access
```

This is the bridge between:

```text
04-Target-Discovery/
```

and:

```text
05-Credential-and-Access-Discovery/
```

---

# 20. Service-to-Movement Mapping

Use this mental model:

```text
Service
   │
   ▼
Remote Access Capability
   │
   ▼
Authentication Requirement
   │
   ▼
Available Credential / Session / Token
   │
   ▼
Authorization
   │
   ▼
Movement Opportunity
```

For example:

```text
SSH
 +
Valid SSH Credential
 +
Network Reachability
 =
Potential Remote Access
```

The final access still needs validation.

---

# 21. Multiple Services on One Target

A single host may expose several movement paths.

Example:

```text
SERVER01
 │
 ├── 22   SSH
 ├── 80   HTTP
 ├── 445  SMB
 ├── 3389 RDP
 └── 5985 WinRM
```

Do not immediately try every service.

Instead, compare them against the evidence you already have.

```text
Available Service
       +
Known Credential
       +
Current Objective
       ↓
Select Most Justified Investigation
```

---

# 22. Service Discovery Record

For every interesting target, record:

```text
Target:
IP:
Hostname:

Port:
Protocol:
Service:
Version:

Host Role:

Authentication Model:

Potential Credential:
Potential Access:

Evidence:

Access Status:

Next Action:
```

Example:

```text
Target:
FILE01

IP:
10.10.10.20

Port:
445/tcp

Protocol:
SMB

Service:
Microsoft-DS

Host Role:
Possible File Server

Authentication Model:
Windows authentication

Potential Credential:
CORP\alice

Evidence:
SMB reachable from current host

Access Status:
Not validated

Next Action:
Review available credentials and validate authorized SMB access
```

---

# 23. Compare Service Findings Across Hosts

Suppose discovery produces:

| Target  | Services   | Initial Interpretation          |
| ------- | ---------- | ------------------------------- |
| WS01    | 445, 3389  | Windows workstation             |
| FILE01  | 445        | Possible file server            |
| APP01   | 80, 443    | Application server              |
| ADMIN01 | 3389, 5985 | Remote administration candidate |

This allows you to move from:

```text
List of IP addresses
```

to:

```text
Environment Model
```

That model is what later enables target prioritization.

---

# 24. Unexpected Services

Sometimes discovery produces a service that does not match the expected host role.

For example:

```text
Expected:
Application Server

Observed:
22/SSH
445/SMB
3389/RDP
```

Do not immediately assume the service is exploitable.

Instead ask:

```text
Why is this service exposed?
Is it expected?
Who uses it?
What authentication does it require?
Does it provide a legitimate movement path?
```

Unexpected services are clues that may justify further investigation.

---

# 25. When a Service Is Filtered

You may encounter:

```text
Filtered
No Response
Timeout
Connection Refused
```

These states provide different information.

### Connection Refused

The host responded but the service may not be listening on that port.

### Timeout / Filtered

Traffic may be blocked or filtered.

### Open

The service appears reachable.

Do not turn one result into an absolute conclusion.

---

# 26. Service Discovery Failure Handling

Use:

```text
Service Not Identified
       │
       ▼
Confirm Host Reachability
       │
       ▼
Confirm Port State
       │
       ▼
Try Appropriate Service Detection
       │
       ▼
Check DNS / Hostname
       │
       ▼
Correlate With Host Role
       │
       ▼
Record Uncertainty
```

If the service remains unknown:

```text
Unknown Service
      ↓
Record Evidence
      ↓
Do Not Invent Identification
      ↓
Continue With Better-Supported Findings
```

Accuracy is more useful than guessing.

---

# 27. Common Service Discovery Mistakes

## Mistake 1: Treating Port Numbers as Services

```text
445
```

is a port.

It is not, by itself, proof of successful SMB access.

---

## Mistake 2: Treating Open Services as Vulnerabilities

```text
SSH Open
```

does not mean:

```text
SSH Vulnerable
```

Likewise:

```text
RDP Open
```

does not mean:

```text
RDP Compromisable
```

---

## Mistake 3: Ignoring Authentication

A service is useful for movement only if you can establish an appropriate access path.

---

## Mistake 4: Trying Everything

Do not blindly attempt every discovered service.

Use:

```text
Evidence
+
Credentials
+
Host Role
+
Objective
```

to select the next action.

---

## Mistake 5: Ignoring Non-Remote Services

Some services may not directly provide remote access but can reveal:

```text
Host Role
Application
Infrastructure
Naming
```

That information can still improve target prioritization.

---

# 28. Service Discovery Decision Tree

```text
Candidate Host
      │
      ▼
Is It Reachable?
   │       │
  No      Yes
   │       │
   ▼       ▼
Reassess  Discover Services
Network       │
              ▼
        Service Identified?
          │          │
         No         Yes
          │          │
          ▼          ▼
      Record       Identify
      Uncertainty  Authentication
                       │
                       ▼
                 Potential Access?
                   │        │
                  No       Yes
                   │        │
                   ▼        ▼
              Credential   Validate
               Discovery    Access
```

---

# 29. Service-to-Next-Step Mapping

Use the result to decide where to go next.

```text
SMB
 ↓
06-Remote-Access-Services/SMB.md
```

```text
WinRM
 ↓
06-Remote-Access-Services/WinRM.md
```

```text
RDP
 ↓
06-Remote-Access-Services/RDP.md
```

```text
SSH
 ↓
06-Remote-Access-Services/SSH.md
```

```text
RPC
 ↓
06-Remote-Access-Services/RPC.md
```

```text
WMI / Windows Remote Administration
 ↓
06-Remote-Access-Services/WMI.md
```

Credential questions may instead lead to:

```text
05-Credential-and-Access-Discovery/
```

---

# 30. Completion Criteria

Service discovery for the current target is sufficiently complete when you know:

```text
[ ] Which ports appear reachable
[ ] Which protocols are present
[ ] Which services are running
[ ] Relevant implementation/version information
[ ] Likely host role
[ ] Relevant authentication model
[ ] Potential access paths
[ ] Existing credentials that may be relevant
[ ] Uncertainty or filtering
[ ] Appropriate next investigation
```

You do not need to identify every service on every host.

You need enough information to determine:

> **Which service, if any, provides a justified path toward remote access?**

---

# 31. What Next?

You now have:

```text
Host
 +
Network Context
 +
Services
 +
Authentication Clues
```

The next question is:

> **"Which discovered targets should I investigate first, and what evidence supports that choice?"**

Continue with:

```text
04-Target-Discovery/
└── Target-Prioritization.md
```

The workflow becomes:

```text
Host Discovery
      ↓
Network Enumeration
      ↓
Service Discovery
      ↓
Target Prioritization
      ↓
Credential / Access Discovery
      ↓
Remote Access
```

The core principle is:

> **A discovered service is an opportunity to investigate, not proof of access. Identify the service, understand its authentication requirements, correlate it with the target's role and your available access, and use that evidence to determine the next step.**
