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
Identif
```
