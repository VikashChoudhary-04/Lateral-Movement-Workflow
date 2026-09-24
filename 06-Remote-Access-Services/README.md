# Remote Access Services

Remote access services are the mechanisms that turn authentication and network reachability into actual interaction with another system.

At this stage of the workflow, the question changes from:

> **"What credentials or access do I have?"**

to:

> **"Which remote services can I use to establish access to another host?"**

The core workflow is:

```text
Current Position
      ↓
Known Identity / Credential
      ↓
Candidate Target
      ↓
Remote Service
      ↓
Authentication Compatibility
      ↓
Reachability
      ↓
Authentication
      ↓
Authorization
      ↓
Remote Access
      ↓
Validate New Position
      ↓
Continue Movement
```

This section covers the main remote-access services that can act as lateral-movement paths across Linux, Windows, and Active Directory environments.

---

## 1. Where This Fits

The overall lateral-movement workflow is:

```text
Initial Foothold
      ↓
Position Assessment
      ↓
Target Discovery
      ↓
Credential / Access Discovery
      ↓
Remote Access Services
      ↓
Movement Technique
      ↓
Post-Movement Enumeration
      ↓
Continue the Chain
```

The previous section answered:

```text
What identities, credentials, sessions, and existing access do I have?
```

This section answers:

```text
Which services can accept that authentication
and provide meaningful remote interaction?
```

---

## 2. Remote Access Service vs Port

A port is not the same thing as a remote-access service.

For example:

```text
445
```

is a port.

```text
SMB
```

is the service/protocol associated with that port.

Likewise:

```text
22
 ↓
SSH
```

and:

```text
3389
 ↓
RDP
```

Therefore:

```text
Open Port
    ≠
Confirmed Service
    ≠
Authenticated Access
    ≠
Authorized Remote Access
```

The workflow must preserve these distinctions.

---

## 3. Services Covered

This section contains dedicated workflows for:

| File       | Service                            | Typical Context        |
| ---------- | ---------------------------------- | ---------------------- |
| `SMB.md`   | SMB                                | Windows / Linux        |
| `WinRM.md` | Windows Remote Management          | Windows / AD           |
| `RDP.md`   | Remote Desktop Protocol            | Windows / AD           |
| `WMI.md`   | Windows Management Instrumentation | Windows / AD           |
| `RPC.md`   | Windows RPC                        | Windows / AD           |
| `SSH.md`   | Secure Shell                       | Linux / Unix / Windows |

These services overlap in capability but differ in:

```text
Authentication
Authorization
Transport
Remote Execution
File Access
Administrative Requirements
Logging
```

---

## 4. The Four Requirements for Remote Movement

A useful remote-access path generally requires four things:

```text
Target
   +
Reachability
   +
Service
   +
Authentication / Authorization
```

Represented as:

```text
Target
  ↓
Reachable?
  ↓
Service Available?
  ↓
Authentication Compatible?
  ↓
Authorized?
  ↓
Remote Access
```

If any major component is missing, the path may fail.

---

## 5. Target

Start with a specific target.

Example:

```text
FILE01
```

or:

```text
10.10.20.15
```

or:

```text
server01.corp.local
```

Do not begin by blindly testing every system.

The target should come from evidence such as:

```text
Host Discovery
Network Enumeration
Service Discovery
DNS
Existing Sessions
SSH Configuration
Kerberos
Application Relationships
Previous Movement
```

---

## 6. Reachability

A target may exist but not be reachable from the current host.

Check:

```text
Routing
Firewalling
Segmentation
Listening Service
Network Path
```

Remember:

```text
Route
  ≠
Reachability
```

and:

```text
Reachability
  ≠
Authentication
```

A target can respond to network traffic while still rejecting remote access.

---

## 7. Service Availability

Once the target is identified, determine which remote services are available.

Common examples:

```text
SSH      → 22
SMB      → 445
RPC      → 135
RDP      → 3389
WinRM    → 5985 / 5986
```

These are common defaults, not guarantees.

Services may:

```text
Use non-standard ports
Be disabled
Be filtered
Require specific protocols
Be bound only to certain interfaces
```

Therefore service discovery must be evidence-driven.

---

## 8. Authentication Compatibility

A credential is only useful if the target service can authenticate it.

Examples:

```text
SSH Key
   ↓
SSH
```

or:

```text
Kerberos
   ↓
SMB
```

or:

```text
Windows Account
   ↓
WinRM
```

The relationship must be evaluated rather than assumed.

---

## 9. Authorization

Authentication answers:

> Who are you?

Authorization answers:

> What are you allowed to do?

For example:

```text
Authentication:
CORP\alice

Authorization:
SMB Read Access
```

This is different from:

```text
Authentication:
CORP\alice

Authorization:
Remote Administrative Access
```

Record the actual result.

---

## 10. Remote Access Categories

Remote services can provide different kinds of access.

### Interactive Access

Examples:

```text
SSH Shell
RDP Session
WinRM Session
```

### Remote Administration

Examples:

```text
WMI
RPC
WinRM
Administrative SMB
```

### Resource Access

Examples:

```text
SMB Share
NFS Mount
Remote Application
```

### Management / Service Interaction

Examples:

```text
RPC
WMI
Database Service
Management APIs
```

Not every remote service gives an interactive shell.

---

## 11. Match Credentials to Services

Use a credential-to-service matrix:

| Credential / Context  |      SMB |                   WinRM |                     RDP |                     WMI |                   SSH |
| --------------------- | -------: | ----------------------: | ----------------------: | ----------------------: | --------------------: |
| Windows Account       | Possible |                Possible |                Possible |                Possible |           Usually not |
| Kerberos Context      | Possible |                Possible |                Possible |                Possible | Environment-dependent |
| NTLM-related Material | Possible | Configuration-dependent | Configuration-dependent | Configuration-dependent |                    No |
| SSH Private Key       |       No |                      No |                      No |                      No |                   Yes |
| Linux Password        |       No |                      No |                      No |                      No |                   Yes |

This is a conceptual compatibility matrix, not a guarantee of access.

Actual behavior depends on:

```text
Protocol
Configuration
Account Rights
Security Policy
Network Controls
```

---

## 12. Start With the Smallest Useful Test

If you already know:

```text
Target:
SERVER01

Service:
SMB

Credential:
CORP\alice
```

do not immediately test unrelated services.

Start with:

```text
SERVER01
  ↓
SMB
  ↓
CORP\alice
```

This follows the principle:

> **Use the smallest action that can confirm or reject the current hypothesis.**

---

## 13. Service-Specific Investigation

Each service should be investigated using the same general pattern:

```text
Goal
 ↓
Prerequisites
 ↓
Service Discovery
 ↓
Authentication Compatibility
 ↓
Authorization Requirements
 ↓
Validation
 ↓
Remote Access
 ↓
Post-Movement Enumeration
 ↓
Failure Handling
 ↓
Next Step
```

This keeps the repository consistent.

---

## 14. SMB

SMB is a major Windows file-sharing and remote-management protocol.

Common port:

```text
445/TCP
```

Potential movement relationships include:

```text
Credential
   ↓
SMB Authentication
   ↓
File Share Access
```

or, when authorized administrative access exists:

```text
Credential
   ↓
SMB / Windows Administration
   ↓
Remote Host
```

Start with:

```text
06-Remote-Access-Services/SMB.md
```

Key questions:

```text
Is SMB reachable?
Which authentication mechanism is available?
Which account is being used?
What shares are accessible?
Is administrative access available?
What does the access enable?
```

---

## 15. WinRM

Windows Remote Management provides a remote management interface for Windows systems.

Common ports include:

```text
5985/TCP
5986/TCP
```

The movement model is:

```text
Identity
   ↓
WinRM Authentication
   ↓
Authorized Remote Management
   ↓
Windows Target
```

Key questions:

```text
Is WinRM available?
Which authentication mechanism is supported?
Does the account have the required remote-management rights?
Can a remote management session be established?
```

Continue with:

```text
06-Remote-Access-Services/WinRM.md
```

---

## 16. RDP

Remote Desktop Protocol provides interactive graphical access to Windows systems.

Common port:

```text
3389/TCP
```

The movement model is:

```text
Identity
   ↓
RDP Authentication
   ↓
Remote Desktop Authorization
   ↓
Interactive Session
```

Important considerations include:

```text
Remote Desktop Enabled
Network Level Authentication
Account Authorization
Session Limits
Network Reachability
```

Continue with:

```text
06-Remote-Access-Services/RDP.md
```

---

## 17. WMI

Windows Management Instrumentation provides management functionality across Windows systems.

WMI can be relevant to lateral movement because authorized remote management can interact with another Windows host.

The conceptual workflow is:

```text
Windows Identity
      ↓
RPC / WMI Connectivity
      ↓
Authorization
      ↓
Remote Management
      ↓
Target
```

WMI often depends on supporting Windows networking and RPC functionality.

Continue with:

```text
06-Remote-Access-Services/WMI.md
```

---

## 18. RPC

Remote Procedure Call is a foundational Windows communication mechanism.

Common endpoint:

```text
135/TCP
```

Additional dynamic ports may also be involved.

RPC is important because several Windows management technologies depend on it.

The movement workflow is therefore:

```text
RPC Reachability
      ↓
Identify RPC-Dependent Service
      ↓
Determine Authentication
      ↓
Determine Authorization
      ↓
Validate Management Path
```

Continue with:

```text
06-Remote-Access-Services/RPC.md
```

---

## 19. SSH

SSH is the primary remote-access service covered for Linux and Unix environments.

Common port:

```text
22/TCP
```

The movement model is:

```text
Username
   +
Password / Key / Other Authentication
   ↓
SSH
   ↓
Remote Shell
```

SSH may also be present on Windows.

Continue with:

```text
06-Remote-Access-Services/SSH.md
```

---

## 20. Service Discovery Before Authentication

Do not assume a service exists because you have credentials for it.

Correct order:

```text
Target
 ↓
Service Discovery
 ↓
Service Validation
 ↓
Authentication Compatibility
 ↓
Authentication
 ↓
Authorization
```

For example:

```text
SSH Key Found
```

should not automatically lead to:

```text
SSH Login Attempt
```

First establish:

```text
Target
SSH Service
Port
Reachability
Username
```

---

## 21. Service Fingerprinting

Service discovery may identify:

```text
Service
Version
Protocol
Product
Authentication Mechanisms
```

For example:

```bash id="c4r9m1"
nmap -sV <target>
```

can help identify exposed services.

The purpose is not to collect version numbers for their own sake.

The purpose is:

```text
Service Discovery
      ↓
Movement Path Selection
```

---

## 22. Existing Access Changes the Workflow

Suppose you already have:

```text
SMB Access to FILE01
```

Do not repeat broad service discovery without purpose.

Instead ask:

```text
What does this access provide?
Can it reveal another target?
Can it expose configuration?
Can it identify a service account?
Can it establish the next movement path?
```

Existing access becomes evidence for the next stage.

---

## 23. Remote Service Decision Loop

For every candidate service:

```text
Do I know the target?
       │
       ▼
Is the target reachable?
       │
       ▼
Is the service available?
       │
       ▼
Do I have compatible authentication?
       │
       ▼
Does the account have authorization?
       │
       ▼
Can I establish useful access?
       │
       ▼
What did I gain?
       │
       ▼
What is the next target?
```

This is the core workflow for Section 06.

---

## 24. Failure Classification

When remote access fails, classify the failure.

### Network Failure

```text
Timeout
No Route
Filtered Traffic
Connection Refused
```

### Service Failure

```text
Service Disabled
Wrong Port
Unexpected Protocol
Service Unavailable
```

### Authentication Failure

```text
Invalid Credential
Unsupported Authentication
Expired Authentication Material
Protocol Mismatch
```

### Authorization Failure

```text
Authenticated
but insufficient permissions
```

### Policy Failure

```text
Remote Logon Restricted
Account Restricted
Protocol Restricted
Security Policy
```

This classification determines the next action.

---

## 25. Do Not Brute-Force the Workflow

Lateral movement should be evidence-driven.

Avoid:

```text
Every Host
 ×
Every Credential
 ×
Every Service
```

Instead:

```text
Evidence
   ↓
Candidate Target
   ↓
Candidate Service
   ↓
Compatible Authentication
   ↓
Smallest Useful Validation
```

This reduces noise and prevents unnecessary testing.

---

## 26. Service-to-Technique Mapping

A useful decision map is:

| Service | Typical Movement Direction          | Main Question                                                     |
| ------- | ----------------------------------- | ----------------------------------------------------------------- |
| SMB     | Windows ↔ Windows / Linux → Windows | What authenticated resource or administrative access exists?      |
| WinRM   | Windows → Windows                   | Does the identity have remote-management rights?                  |
| RDP     | Windows → Windows                   | Is interactive remote logon authorized?                           |
| WMI     | Windows → Windows                   | Is remote management authorized through WMI/RPC?                  |
| RPC     | Windows → Windows                   | Which management service depends on RPC?                          |
| SSH     | Linux ↔ Linux / Windows ↔ Linux     | Which account and authentication method can establish SSH access? |

These are starting points, not automatic movement decisions.

---

## 27. Remote Service Record

For every candidate service, record:

```text id="h6n2q8"
Target:
____________________

IP / Hostname:
____________________

Service:
____________________

Port:
____________________

Reachability:
____________________

Authentication:
____________________

Identity:
____________________

Authorization:
____________________

Access Type:
____________________

Validation Result:
____________________

Movement Value:
____________________

Failure Reason:
____________________

Next Action:
____________________
```

This creates an auditable movement map.

---

## 28. Service Priority

Do not prioritize services simply because they are common.

Prioritize based on:

```text
Known Credential Compatibility
+
Reachability
+
Authorization Evidence
+
Target Relevance
+
Expected Information Gain
```

For example:

```text
Known SSH key
+
Reachable SSH server
+
Known username
```

provides a stronger immediate hypothesis than:

```text
Unknown service
+
Unknown credentials
+
Uncertain target
```

---

## 29. After Successful Remote Access

A successful remote connection is not the end.

Immediately ask:

```text
Who am I?
Where am I?
What privileges do I have?
What network am I on?
What domain am I in?
What can this host reach?
What credentials or sessions exist?
```

Then return to:

```text
03-Position-Assessment/
```

and:

```text
04-Target-Discovery/
```

followed by:

```text
05-Credential-and-Access-Discovery/
```

This creates the lateral-movement loop.

---

## 30. The Repetition Loop

The complete process is:

```text
Current Host
      ↓
Position Assessment
      ↓
Target Discovery
      ↓
Credential / Access Discovery
      ↓
Remote Service
      ↓
Authentication
      ↓
Authorization
      ↓
New Host
      ↓
Position Assessment
      ↓
Target Discovery
      ↓
Credential / Access Discovery
      ↓
Remote Service
      ↓
Repeat
```

Lateral movement is therefore not:

```text
Find Credential
   ↓
Run Tool
```

It is:

```text
Assess
 ↓
Discover
 ↓
Map
 ↓
Validate
 ↓
Move
 ↓
Reassess
```

---

## 31. Recommended Reading Order

Work through the service files in this order:

```text
SMB
 ↓
WinRM
 ↓
RDP
 ↓
WMI
 ↓
RPC
 ↓
SSH
```

The order is organizational rather than a universal attack priority.

Choose the actual service based on evidence from the current environment.

---

## 32. Section 06 Completion Criteria

Remote-access-service investigation is sufficiently complete when you can answer:

```text
[ ] Which candidate targets are reachable?
[ ] Which remote services are available?
[ ] Which services provide meaningful remote interaction?
[ ] Which authentication mechanisms do they support?
[ ] Which credentials or authentication contexts match them?
[ ] Which accounts are authorized?
[ ] Which access levels are available?
[ ] Which services failed?
[ ] Why did they fail?
[ ] Which service provides the next realistic movement path?
[ ] What will be re-enumerated after successful movement?
```

The objective is not:

```text
Find every open port.
```

It is:

```text
Identify remote services that can turn an existing
identity or authentication context into useful, authorized access.
```

---

## 33. What Next?

The first service-specific workflow is:

```text
06-Remote-Access-Services/SMB.md
```

SMB is a natural starting point because it connects several lateral-movement concepts:

```text
Windows Identity
      ↓
SMB Authentication
      ↓
File / Administrative Shares
      ↓
Remote Host
      ↓
Authorization
      ↓
Movement
```

The investigation will answer:

```text
Is SMB reachable?

Which shares exist?

Which authentication mechanism is available?

Which identity can authenticate?

What authorization does that identity have?

Does SMB provide a useful movement path?

What should happen after successful access?
```

The core principle of Section 06 is:

> **A remote service is useful only when a reachable target, compatible authentication context, and sufficient authorization come together to create meaningful remote access.**
