# SMB

Server Message Block (SMB) is a network protocol commonly used for file and resource sharing in Windows environments.

For lateral movement, SMB is important because it can provide access to:

```text
File Shares
Administrative Shares
Remote Resources
Windows Authentication
Windows Management Workflows
```

The practical question is not simply:

> "Is port 445 open?"

The useful question is:

> **Can an identity with an available authentication context establish authorized SMB access to this target, and does that access create a useful movement path?**

The workflow is:

```text
Candidate Target
      ↓
Identify SMB
      ↓
Check Reachability
      ↓
Identify Authentication
      ↓
Identify Account
      ↓
Enumerate Available Shares
      ↓
Validate Authorization
      ↓
Determine Access Level
      ↓
Determine Movement Value
      ↓
Move or Continue Discovery
      ↓
Re-enumerate
```

---

## 1. Where SMB Fits

SMB is the first service-specific workflow in Section 06:

```text
Remote Access Services
      ↓
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

SMB is particularly relevant to:

```text
Windows
Active Directory
Windows File Servers
Administrative Workstations
Linux Systems with SMB Clients
```

---

## 2. SMB Ports

The most important modern SMB port is:

```text
445/TCP
```

Older environments may also involve:

```text
139/TCP
```

which is associated with SMB over NetBIOS.

Do not assume:

```text
445 Open
    =
SMB Confirmed
```

Verify the service.

---

## 3. The SMB Movement Model

The basic movement relationship is:

```text
Identity
   ↓
Authentication
   ↓
SMB
   ↓
Target
   ↓
Share / Resource
   ↓
Authorization
   ↓
Access
```

Potential outcomes include:

```text
Share Read Access
Share Write Access
Administrative Share Access
Resource Discovery
Remote Administration
```

The exact outcome depends on the account and target configuration.

---

## 4. SMB Is Not Automatically Remote Shell Access

One of the most important distinctions is:

```text
SMB Access
    ≠
Interactive Shell
```

For example:

```text
\\FILE01\Public
```

may provide:

```text
Read Access
```

without providing:

```text
Remote Command Execution
```

Therefore always record the actual access level.

---

## 5. Identify the Target

Before investigating SMB, establish:

```text
Target Host
IP Address
Hostname
Domain
Network Location
```

For example:

```text
Target:
FILE01.CORP.LOCAL

IP:
10.10.20.15
```

The target should come from previous evidence such as:

```text
Target Discovery
DNS
Kerberos
Existing Connections
Service Discovery
Previous Movement
Application Relationships
```

---

## 6. Check Reachability

First determine whether SMB is reachable.

From Windows:

```powershell
Test-NetConnection FILE01 -Port 445
```

From Linux:

```bash
nc -vz FILE01 445
```

or use an authorized service-discovery workflow.

The result tells you whether network communication to the SMB port is possible.

It does not establish:

```text
SMB Authentication
```

or:

```text
Share Authorization
```

---

## 7. Confirm SMB

A service-discovery scan can help identify SMB.

For example:

```bash
nmap -p 445 -sV FILE01
```

The objective is:

```text
Target
  ↓
445/TCP
  ↓
SMB Service
```

Do not move directly from an open port to credential testing without establishing the service context.

---

## 8. Identify the Current Identity

Before authentication testing, establish your current identity.

### Windows

```cmd
whoami
whoami /user
whoami /groups
```

### Linux

```bash
whoami
id
```

For a domain environment, also determine:

```text
Local Account
or
Domain Account
```

This matters because SMB authentication behavior depends on the account authority.

---

## 9. SMB Authentication Context

Common Windows authentication mechanisms can include:

```text
Kerberos
NTLM
Local Windows Authentication
Domain Authentication
```

The exact mechanism depends on:

```text
Target
Domain Context
Hostname
Credential Type
Security Policy
Protocol Configuration
```

Therefore:

```text
Credential Found
    ≠
SMB Access
```

You must establish compatibility.

---

## 10. Kerberos and SMB

In an Active Directory environment, SMB can use Kerberos.

Conceptually:

```text
Domain Identity
      ↓
Kerberos
      ↓
CIFS Service
      ↓
SMB
      ↓
Target
```

A Kerberos service principal may look conceptually like:

```text
cifs/file01.corp.local
```

This can provide useful target information.

However:

```text
Kerberos Context
    ≠
Administrative Access
```

Authorization still needs to be validated.

---

## 11. NTLM and SMB

SMB can also use NTLM authentication in applicable configurations.

The conceptual path is:

```text
Windows Identity
      ↓
NTLM Authentication
      ↓
SMB
      ↓
Target
```

Whether NTLM is accepted depends on:

```text
Target Configuration
Authentication Policy
Account Restrictions
Protocol Configuration
```

Do not assume that NTLM-related authentication material works against every SMB target.

---

## 12. Enumerate Shares

Once SMB is confirmed, determine what resources are exposed.

From Windows, where authorized:

```cmd
net view \\FILE01
```

Existing mapped connections can be reviewed with:

```cmd
net use
```

From Linux, an SMB client can be used to enumerate available resources where authorized.

For example:

```bash
smbclient -L //FILE01 -U 'CORP/alice'
```

The purpose is to identify:

```text
Share Name
Share Type
Potential Resource
Access Status
```

---

## 13. Share Discovery Is Not Authorization

Suppose you identify:

```text
Public
Finance
IT
Backup
```

This does not prove that your identity can access all four.

The correct sequence is:

```text
Share Discovered
      ↓
Access Attempt
      ↓
Authorization Result
```

Record:

```text
Accessible
Inaccessible
Unknown
```

rather than assuming access.

---

## 14. Common SMB Share Types

You may encounter:

```text
ADMIN$
C$
IPC$
NETLOGON
SYSVOL
User Shares
Department Shares
Application Shares
Backup Shares
Public Shares
```

The significance depends on the environment.

For example:

```text
SYSVOL
```

and:

```text
NETLOGON
```

can be particularly relevant in Active Directory environments.

Administrative shares such as:

```text
C$
ADMIN$
```

generally require stronger authorization than ordinary file shares.

---

## 15. Administrative Shares

Administrative shares provide access to administrative resources on Windows systems.

Common examples include:

```text
C$
ADMIN$
```

Conceptually:

```text
Authorized Administrator
      ↓
SMB
      ↓
Administrative Share
      ↓
Remote System Resources
```

Do not assume that discovering an administrative share means you can access it.

The important question is:

```text
Does the current identity have authorization?
```

---

## 16. IPC$

`IPC$` is a special SMB share used for inter-process communication and certain remote Windows operations.

Finding:

```text
IPC$
```

does not mean:

```text
File Access
```

or:

```text
Administrative Access
```

Treat it as evidence that SMB-related communication is available.

---

## 17. SYSVOL and NETLOGON

In an AD environment, shares such as:

```text
SYSVOL
NETLOGON
```

can reveal domain-related configuration and scripts.

They can therefore provide:

```text
Domain Context
Group Policy Information
Login / Logon Configuration
Administrative Workflow Clues
```

Access to these shares should still be evaluated according to the current account's authorization.

---

## 18. SMB and File Access

If a share is accessible, determine the actual permissions.

Possible levels include:

```text
Read
Write
Modify
Create
Delete
Execute
```

Record the result precisely.

For example:

```text
Share:
Finance

Access:
Read

Identity:
CORP\alice
```

is much more useful than:

```text
Finance:
Accessible
```

---

## 19. SMB and Write Access

Write access can be more significant than read-only access because it may allow modification of resources used by another system or process.

However, do not assume:

```text
Write Access
    =
Code Execution
```

Determine:

```text
What can be written?
Who consumes it?
When is it consumed?
Under which identity?
```

This keeps the assessment evidence-driven.

---

## 20. SMB and Administrative Access

If the identity has authorized administrative access through SMB, the movement path may become significantly more useful.

The conceptual chain is:

```text
Credential
      ↓
SMB Authentication
      ↓
Administrative Authorization
      ↓
Remote Administrative Capability
      ↓
Target
```

The exact remote-management capability depends on the Windows environment and other enabled services.

---

## 21. SMB Enumeration From Windows

Useful native commands include:

```cmd
net view \\FILE01
```

and:

```cmd
net use
```

For a specific share:

```cmd
dir \\FILE01\Public
```

This can establish whether the current identity can access a specific resource.

The key question is:

```text
What does the current identity actually have access to?
```

---

## 22. SMB Enumeration From Linux

An SMB client can provide useful information from a Linux assessment host.

For example:

```bash
smbclient -L //FILE01 -U 'CORP/alice'
```

After identifying an authorized share, a client session can be used to validate access.

The workflow remains:

```text
Enumerate
   ↓
Identify Share
   ↓
Authenticate
   ↓
Validate Authorization
   ↓
Determine Access
```

Avoid treating share enumeration as proof of permission to access every resource.

---

## 23. Anonymous or Guest Access

Some environments may expose SMB resources through:

```text
Anonymous
Guest
Unauthenticated
```

If discovered during an authorized assessment, record:

```text
Authentication Required:
No

Resource:
____________________

Access:
____________________
```

Do not assume that anonymous access provides broad access.

Often only specific resources are exposed.

---

## 24. SMB Access and Credentials

When SMB access succeeds, determine what authentication context was used.

Possible examples:

```text
Current Windows Session
Domain Credentials
Local Credentials
Kerberos
NTLM
Existing Network Authentication
```

This matters because successful access may reveal:

```text
Which credential works
Which account is authorized
Which authentication mechanism is accepted
```

Record the evidence.

---

## 25. Existing SMB Connections

Before creating a new SMB connection, inspect existing relationships.

On Windows:

```cmd
net use
```

You may discover:

```text
\\FILE01\Public
\\SERVER02\Shared
```

These can reveal already-authenticated targets.

The workflow becomes:

```text
Existing SMB Access
      ↓
Identify Target
      ↓
Identify Resource
      ↓
Determine Authorization
      ↓
Use Existing Access
```

rather than unnecessarily creating a new authentication path.

---

## 26. SMB and Target Discovery

SMB itself can reveal additional infrastructure.

For example:

```text
FILE01
 ├── Public
 ├── Finance
 ├── IT
 └── Backup
```

The resources may contain information about:

```text
Users
Servers
Applications
Departments
Backups
Deployment Systems
Scripts
Configuration
```

Use this information to expand the target map.

---

## 27. SMB and Domain Environment

In AD environments, SMB often connects:

```text
Workstations
File Servers
Domain Controllers
Application Servers
Administrative Systems
```

Therefore, when SMB is available, ask:

```text
What kind of host is this?
What shares are exposed?
Which identity can access them?
What does the access reveal?
Does the resource identify another target?
```

---

## 28. Domain Controller SMB Access

A domain controller commonly exposes SMB-related resources.

Potential shares include:

```text
SYSVOL
NETLOGON
ADMIN$
C$
IPC$
```

The presence of these shares is expected in many AD environments.

The assessment question is:

```text
What does my current identity have access to?
```

not:

```text
Can I see that the share exists?
```

---

## 29. SMB Authentication Failure

If authentication fails, classify the failure.

Possible causes include:

```text
Wrong Username
Wrong Credential
Wrong Account Authority
Authentication Protocol Restriction
Expired Credential
Disabled Account
Target Policy
```

For Kerberos-related failures, also investigate:

```text
DNS
Hostname
Domain Connectivity
Clock Synchronization
```

Do not immediately conclude that the credential itself is invalid.

---

## 30. SMB Authorization Failure

You may successfully authenticate but receive:

```text
Access Denied
```

This indicates a different problem.

Conceptually:

```text
Authentication
      ↓
SUCCESS
      ↓
Authorization
      ↓
FAILURE
```

Record:

```text
Authentication:
Successful

Authorization:
Denied
```

This distinction helps determine the next action.

---

## 31. SMB Network Failure

If:

```text
Test-NetConnection FILE01 -Port 445
```

fails, investigate:

```text
Routing
Firewall
Network Segmentation
Host Availability
Incorrect Address
```

Do not start changing credentials.

The failure occurred before authentication.

---

## 32. SMB Service Failure

If the network path works but SMB does not respond as expected, investigate:

```text
SMB Service
Windows Firewall
SMB Configuration
Protocol Version
Target Role
```

The correct workflow is:

```text
Network
 ↓
Service
 ↓
Authentication
 ↓
Authorization
```

Troubleshoot the layer that actually failed.

---

## 33. SMB Protocol Version

Modern Windows environments generally use SMB2/SMB3 rather than obsolete SMB1.

Do not assume that an environment supports legacy SMB protocols.

Protocol negotiation and security configuration can affect compatibility.

The movement workflow should therefore focus on:

```text
Current Target Configuration
```

rather than forcing an older protocol.

---

## 34. SMB Signing

SMB signing is a security control that can affect certain SMB attack scenarios.

For lateral movement workflow purposes, record whether signing or other SMB security controls are relevant to the environment.

The important distinction is:

```text
SMB Service Available
```

versus:

```text
SMB Security Configuration
```

Do not treat SMB signing status as proof of authentication or authorization.

---

## 35. SMB and Credential Matching

Use the credential-to-service model:

| Authentication Context   | SMB Compatibility       | Notes                                      |
| ------------------------ | ----------------------- | ------------------------------------------ |
| Domain account           | Possible                | Depends on target and policy               |
| Local Windows account    | Possible                | Depends on remote authorization and policy |
| Kerberos context         | Possible                | Common in AD environments                  |
| NTLM-compatible material | Configuration-dependent | Depends on target policy                   |
| SSH private key          | No                      | Not an SMB credential                      |

The table describes compatibility, not guaranteed access.

---

## 36. SMB Movement Decision Tree

```text
Candidate Target
      │
      ▼
Is TCP/445 Reachable?
      │
 ┌────┴────┐
No        Yes
 │          │
 ▼          ▼
Check     Confirm
Network   SMB
             │
             ▼
       Identify Identity
             │
             ▼
       Authentication Compatible?
          │           │
         No          Yes
          │           │
          ▼           ▼
     Continue       Enumerate
     Other Path     Shares
                       │
                       ▼
                Share Accessible?
                  │          │
                 No         Yes
                  │          │
                  ▼          ▼
              Record /   Determine
              Reassess    Access
                            │
                            ▼
                    Useful Movement Path?
                       │          │
                      No         Yes
                       │          │
                       ▼          ▼
                   Continue     Validate
                   Discovery    Movement
                                    │
                                    ▼
                              New Position
                                    │
                                    ▼
                              Re-enumerate
```

---

## 37. Practical Windows Workflow

From a Windows foothold:

```text
1. Identify current identity
2. Identify domain context
3. Identify candidate target
4. Test TCP/445
5. Confirm SMB
6. Check existing SMB connections
7. Enumerate available shares
8. Identify authentication context
9. Validate share access
10. Determine read/write/administrative authorization
11. Identify useful information or movement paths
12. Validate authorized movement
13. Re-enumerate the new position
```

Useful commands include:

```cmd
whoami
net use
net view \\FILE01
dir \\FILE01\Public
```

and:

```powershell
Test-NetConnection FILE01 -Port 445
```

---

## 38. Practical Linux Workflow

From a Linux assessment host:

```text
1. Identify current identity
2. Identify candidate target
3. Test TCP/445
4. Confirm SMB
5. Enumerate shares
6. Identify authentication context
7. Validate authorized share access
8. Determine permissions
9. Identify useful resources
10. Determine whether SMB creates a movement path
11. Validate authorized movement
12. Re-enumerate after movement
```

Useful commands include:

```bash
whoami
id
nc -vz FILE01 445
smbclient -L //FILE01 -U 'CORP/alice'
```

---

## 39. Example: Read-Only SMB Access

Suppose:

```text
Target:
FILE01

Share:
Public

Identity:
CORP\alice

Access:
Read
```

The correct interpretation is:

```text
Authenticated SMB Access
        ↓
Read Authorization
        ↓
Resource Discovery Opportunity
```

Do not write:

```text
FILE01 compromised
```

because no host-level access has been established.

---

## 40. Example: Administrative SMB Access

Suppose:

```text
Target:
SERVER01

Identity:
CORP\admin

Authentication:
Successful

Administrative Resource:
Accessible
```

This creates a stronger movement hypothesis.

The next step is to determine:

```text
What remote administrative capabilities are authorized?
What management services are available?
What is the minimum action required?
```

Do not assume that every administrative share automatically means arbitrary remote execution.

---

## 41. Example: SMB Reveals a New Target

Suppose a share contains:

```text
deployment/
scripts/
configs/
```

and a configuration references:

```text
DB01.CORP.LOCAL
```

Now the movement map becomes:

```text
Current Host
      ↓
FILE01
      ↓
Accessible Share
      ↓
Configuration
      ↓
DB01
```

The next action may therefore be:

```text
Target Discovery
```

rather than immediate exploitation.

This demonstrates why existing resource access can be valuable even without shell access.

---

## 42. SMB and Post-Movement Enumeration

After successful movement to another Windows host:

```text
whoami
hostname
ipconfig /all
route print
```

Then return to:

```text
03-Position-Assessment/
```

and continue:

```text
04-Target-Discovery/
05-Credential-and-Access-Discovery/
06-Remote-Access-Services/
```

A new host is a new position.

---

## 43. Common Mistakes

### Mistake 1: Treating Port 445 as Proof of SMB

Always verify the service.

### Mistake 2: Treating Share Enumeration as Access

Seeing a share does not prove authorization.

### Mistake 3: Treating SMB Read Access as Host Compromise

A file share is a resource-level access path unless stronger authorization is established.

### Mistake 4: Ignoring Existing Connections

`net use` may reveal useful authenticated relationships.

### Mistake 5: Assuming Domain Credentials Work Everywhere

Authentication and authorization are target-specific.

### Mistake 6: Ignoring Account Authority

A local account and a domain account with the same username are not necessarily the same identity.

### Mistake 7: Ignoring Kerberos Context

In AD environments, hostname, DNS, domain, and time can affect Kerberos authentication.

### Mistake 8: Assuming Administrative Shares Equal Execution

Administrative resource access is evidence of authorization, but the exact remote-management capability must still be established.

### Mistake 9: Skipping Re-enumeration

Successful movement changes your position and therefore changes your available targets and credentials.

---

## 44. SMB Investigation Record

Use this structure:

```text
Target:
____________________

Hostname:
____________________

IP:
____________________

Port:
____________________

SMB Confirmed:
____________________

Current Identity:
____________________

Account Authority:
____________________

Authentication:
____________________

Shares:
____________________

Accessible Shares:
____________________

Access Level:
____________________

Existing Connection:
____________________

Authorization:
____________________

Movement Value:
____________________

Validation Result:
____________________

Failure Reason:
____________________

New Position:
____________________

Next Action:
____________________
```

---

## 45. SMB Completion Criteria

SMB investigation is sufficiently complete when you can answer:

```text
[ ] Is the target reachable?
[ ] Is SMB actually available?
[ ] Which SMB endpoint is reachable?
[ ] Which identity is being used?
[ ] Which authentication mechanism is relevant?
[ ] Which shares are exposed?
[ ] Which shares are actually accessible?
[ ] What permissions does the identity have?
[ ] Is administrative access authorized?
[ ] Does the access reveal additional targets?
[ ] Does SMB provide a movement path?
[ ] If movement succeeds, what is the new position?
[ ] What caused any failure?
[ ] What is the next action?
```

The objective is not:

```text
Find every SMB share.
```

It is:

```text
Determine whether SMB converts an existing identity or
authentication context into useful, authorized access to another system or resource.
```

---

## 46. What Next?

Now continue to:

```text
06-Remote-Access-Services/
└── WinRM.md
```

The service workflow changes from:

```text
SMB
 ↓
File / Resource / Administrative Access
```

to:

```text
WinRM
 ↓
Remote Windows Management
 ↓
PowerShell Remoting
```

The core principle is:

> **Do not treat SMB as merely an open port or a list of shares. Map the identity, authentication mechanism, target, share, authorization, and actual access level to determine whether SMB creates a useful lateral-movement path.**
