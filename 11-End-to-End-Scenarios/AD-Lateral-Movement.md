# Active Directory Lateral Movement

This scenario demonstrates an end-to-end lateral-movement workflow inside an Active Directory environment.

The focus is not on one specific technique. It is on understanding how domain identity, Kerberos, target discovery, remote services, authorization, and post-movement enumeration fit together.

The scenario is intended for authorized penetration tests and controlled labs.

## Scenario Objective

Demonstrate a domain-based movement path:

```text id="p7m3q8"
Domain-Joined Host
      ↓
Establish Domain Context
      ↓
Discover Domain Targets
      ↓
Identify Remote Services
      ↓
Map Identity / Authentication Material
      ↓
Validate Authentication
      ↓
Validate Authorization
      ↓
Establish Remote Access
      ↓
Validate New Position
      ↓
Enumerate New Host
      ↓
Reassess Credentials / Tickets
      ↓
Identify Next Target
      ↓
Continue or Stop
```

The scenario emphasizes:

```text id="m4x8q2"
Identity
   ↓
Authentication
   ↓
Service
   ↓
Authorization
   ↓
Target
```

## Starting Position

Assume the assessment begins from an authorized domain-joined Windows workstation:

```text id="v8n3p6"
Host: WS-01
IP: 10.10.10.25
Domain: CORP.LOCAL
User: CORP\analyst
```

The exact values are illustrative.

Use the values from the authorized lab or assessment environment.

## Scope

Assume the authorized environment includes:

```text id="q5m7r2"
Domain:
CORP.LOCAL

Workstation:
WS-01

Candidate Server:
SRV-01

Domain Controller:
DC-01

Network:
10.10.10.0/24
```

Only explicitly authorized systems should be tested.

## Phase 1 — Establish Domain Identity

Start by confirming the current identity.

```cmd id="x7p3m9"
whoami
```

```cmd id="c4n8q2"
whoami /user
```

```cmd id="v6m2r7"
whoami /groups
```

Determine:

```text id="m8q3p5"
Account:
CORP\analyst

SID:
<validated SID>

Domain:
CORP.LOCAL
```

Do not assume that a domain account has administrative access to other systems.

## Phase 2 — Confirm Domain Membership

Use:

```cmd id="r7m3x8"
whoami /fqdn
```

And:

```cmd id="q5n8p2"
echo %USERDOMAIN%
echo %USERDNSDOMAIN%
```

System-level information:

```powershell id="m4v9c3"
Get-CimInstance Win32_ComputerSystem |
    Select-Object Name, Domain, PartOfDomain
```

Confirm:

```text id="p8m3q7"
Current Host:
Domain:
Domain Membership:
Current Identity:
```

## Phase 3 — Confirm Kerberos Context

Inspect the current ticket context:

```cmd id="x3q8m5"
klist
```

Determine:

```text id="n7m2v4"
Principal:
Realm:
Ticket Types:
Expiration:
Relevant Service Tickets:
```

Conceptually:

```text id="c6p9r3"
CORP\analyst
      ↓
Kerberos Authentication
      ↓
Domain Controller
      ↓
Service Ticket
      ↓
Target Service
```

A Kerberos ticket does not automatically grant administrative access.

## Phase 4 — Identify the Domain Controller

Where appropriate:

```cmd id="v8m3q5"
nltest /dsgetdc:CORP.LOCAL
```

The result can help establish:

```text id="q4p7n2"
Domain:
Domain Controller:
Site:
```

The domain controller provides important authentication and directory context.

Do not treat discovery of the domain controller as permission to perform arbitrary actions against it.

## Phase 5 — Identify Domain Targets

Candidate systems may be identified through:

* known assessment scope
* DNS
* existing domain information
* authorized host discovery
* existing connections
* documented infrastructure
* service discovery

The objective is to identify systems relevant to the assessment.

Example:

```text id="m9x3q7"
CORP.LOCAL
    |
    +-- WS-01
    |
    +-- SRV-01
    |
    +-- FILE-01
    |
    +-- DC-01
```

Do not assume that every domain-joined computer is an appropriate target.

## Phase 6 — Identify Target Services

For a candidate server, identify relevant remote services.

Common Windows services include:

```text id="r6m2p8"
SMB   → 445
WinRM → 5985 / 5986
RDP   → 3389
RPC   → 135
```

The workflow is:

```text id="x8q3m7"
Target
  ↓
Service
  ↓
Authentication
  ↓
Authorization
```

## Phase 7 — Check Target Reachability

For example:

```powershell id="c5m8r2"
Test-NetConnection SRV-01 -Port 445
```

```powershell id="p7q3n9"
Test-NetConnection SRV-01 -Port 5985
```

```powershell id="m4v8x2"
Test-NetConnection SRV-01 -Port 3389
```

RPC:

```powershell id="r9n3q6"
Test-NetConnection SRV-01 -Port 135
```

Record:

```text id="x5m8p3"
Service:
Port:
Reachable:
```

A reachable port only establishes network-level connectivity.

## Phase 8 — Map Domain Identity to the Target

Start with the current domain identity:

```text id="q7m3v8"
CORP\analyst
```

Then determine which target services are relevant.

For example:

```text id="n5p8r2"
CORP\analyst
     ↓
Kerberos / NTLM
     ↓
SMB
     ↓
SRV-01
```

or:

```text id="v3m7q9"
CORP\analyst
     ↓
Kerberos
     ↓
WinRM
     ↓
SRV-01
```

The goal is to establish a specific authentication hypothesis.

## Phase 9 — Validate SMB Access

If SMB is relevant, enumerate authorized shares.

```cmd id="k8m3p6"
net view \\SRV-01
```

The result should be interpreted as:

```text id="r4q7n2"
Domain Authentication
       ↓
SMB
       ↓
Share Enumeration
       ↓
Share Authorization
```

Suppose:

```text id="x6m8p3"
ADMIN$
C$
Public
```

are visible.

The presence of administrative shares does not itself prove that the current identity can access them.

Validate authorization separately.

## Phase 10 — Validate WinRM Access

First confirm the service:

```powershell id="m7q3v9"
Test-WSMan SRV-01
```

If the current identity is authorized for remote management:

```powershell id="p5n8r2"
Enter-PSSession -ComputerName SRV-01
```

Once connected:

```powershell id="c4m7x2"
whoami
hostname
```

The result establishes the new host and identity.

## Phase 11 — Validate RDP Access

For an RDP path:

```text id="v8q3m5"
SRV-01
  ↓
RDP Reachability
  ↓
Domain Authentication
  ↓
Interactive Logon Authorization
```

Once authorized access is established:

```cmd id="n3m8p7"
whoami
hostname
```

Then:

```cmd id="x7q2r4"
whoami /groups
whoami /priv
```

This confirms the actual privilege context.

## Phase 12 — Consider WMI / RPC

For WMI-based management, understand the service relationship:

```text id="q6m3v8"
RPC Endpoint Mapper
       ↓
Dynamic RPC
       ↓
WMI / DCOM
       ↓
Authorization
```

Check RPC reachability:

```powershell id="m8p4q2"
Test-NetConnection SRV-01 -Port 135
```

Do not equate an open RPC endpoint with WMI authorization.

## Phase 13 — Validate Authentication vs Authorization

Maintain separate results.

Example:

```text id="r5m8x2"
Authentication:
Successful

Authorization:
Remote PowerShell session permitted

Effective Access:
WinRM session as CORP\analyst
```

Another result might be:

```text id="p3q7n9"
Authentication:
Successful

Authorization:
SMB share access denied
```

The second result still provides useful information about the permission boundary.

## Phase 14 — Establish the New Position

Suppose WinRM access succeeds.

The movement becomes:

```text id="c8m4v7"
WS-01
CORP\analyst
10.10.10.25
      |
      | Kerberos / WinRM
      ↓
SRV-01
CORP\analyst
10.10.10.50
```

The server is now a validated assessment position.

## Phase 15 — Validate the New Position

Inside SRV-01:

```powershell id="x5m8q3"
hostname
```

```powershell id="r7n3p9"
whoami
```

```powershell id="m4q8v2"
whoami /user
```

```powershell id="v6p3m9"
whoami /groups
```

Network:

```powershell id="q8m2r5"
ipconfig /all
```

Routes:

```powershell id="n3x7p8"
route print
```

The objective is to confirm:

```text id="k5m8q2"
Host:
SRV-01

Identity:
CORP\analyst

Network:
10.10.10.0/24

Privilege:
Validated from group / privilege evidence
```

## Phase 16 — Enumerate the New Server

Check network configuration:

```powershell id="p8m3x7"
Get-NetIPConfiguration
```

Routes:

```powershell id="c4q9m2"
Get-NetRoute
```

Listening services:

```powershell id="v7n3p8"
Get-NetTCPConnection -State Listen
```

Running services:

```powershell id="m5x8q3"
Get-Service |
    Where-Object {$_.Status -eq "Running"}
```

Active connections:

```powershell id="q2r7m4"
Get-NetTCPConnection
```

The goal is to identify the server's role and network visibility.

## Phase 17 — Identify New Network Visibility

Suppose SRV-01 has:

```text id="x8m3q6"
Interface 1:
10.10.10.50

Interface 2:
10.10.20.50
```

and routes:

```text id="n4p7m2"
10.10.10.0/24
10.10.20.0/24
```

The new host may therefore provide access to a network that was not directly available from WS-01.

The decision becomes:

```text id="v6q3m8"
New Network
     ↓
In Scope?
     ↓
Relevant Targets?
     ↓
Directly Reachable?
     ↓
Pivot Required?
```

## Phase 18 — Reassess Kerberos Context

From SRV-01:

```cmd id="m8q3p7"
klist
```

Determine whether the current logon context has relevant tickets.

Record:

```text id="c5n8r2"
Principal:
Realm:
Ticket:
Service:
Expiration:
```

Do not assume that possession of a ticket means unrestricted access.

Map it to an actual service and authorization boundary.

## Phase 19 — Reassess Domain Context

Confirm:

```cmd id="q7m3x9"
whoami /domain
```

Determine domain controller availability:

```cmd id="v4p8m2"
nltest /dsgetdc:CORP.LOCAL
```

If relevant:

```cmd id="n6m3q8"
nltest /domain_trusts
```

The purpose is to understand whether the new host provides additional domain visibility.

## Phase 20 — Reassess Existing Access

Inspect current network relationships:

```powershell id="m7x2p5"
Get-NetTCPConnection
```

Look for systems such as:

```text id="r8q3n6"
Database Servers
Application Servers
File Servers
Management Systems
Domain Infrastructure
```

Do not infer access solely from a connection.

A connection demonstrates communication, not authorization.

## Phase 21 — Identify New Candidate Targets

Suppose SRV-01 reveals:

```text id="p4m8x2"
10.10.20.15 → Database
10.10.20.25 → Application
10.10.20.35 → File Server
```

Build a target table:

| Target  | Network     | Service  | Evidence            | Access Path     |
| ------- | ----------- | -------- | ------------------- | --------------- |
| DB-01   | 10.10.20.15 | Database | Connection observed | To be validated |
| APP-01  | 10.10.20.25 | Web App  | Service observed    | To be validated |
| FILE-01 | 10.10.20.35 | SMB      | Service observed    | To be validated |

Only authorized targets should proceed to validation.

## Phase 22 — Credential Reassessment

Now inspect the server for authentication relationships.

Relevant sources include:

```text id="x6m3q8"
Application Configuration
Service Identities
Environment Variables
Kerberos Context
Existing Sessions
Existing Network Access
```

Service identities can be reviewed without attempting to expose passwords:

```powershell id="r5n8m2"
Get-CimInstance Win32_Service |
    Select-Object Name, StartName, State
```

The important relationship is:

```text id="q8m3v6"
Service
   ↓
Identity
   ↓
Dependency
   ↓
Target
```

## Phase 23 — Evaluate Credential Reuse

Suppose an authorized assessment identifies:

```text id="m3q7p9"
CORP\svc-app
```

associated with an application on SRV-01.

Do not automatically test that identity against every domain system.

Instead:

```text id="v8m4n2"
Identity
   ↓
Known Purpose
   ↓
Known Dependency
   ↓
Known Target
   ↓
Authorized Validation
```

This keeps credential testing targeted and evidence-based.

## Phase 24 — Evaluate Kerberos-Based Access

For a known service, the conceptual relationship is:

```text id="p7m3x8"
Domain Identity
      ↓
Kerberos
      ↓
Service Principal
      ↓
Target Service
      ↓
Authorization
```

For SMB, for example:

```text id="c4n8q2"
Identity
   ↓
Kerberos
   ↓
CIFS Service
   ↓
FILE-01
   ↓
Share Authorization
```

The important point is not merely obtaining a ticket.

The important point is whether the ticket authenticates the identity to the required service and whether that identity is authorized there.

## Phase 25 — Evaluate the New Position

Compare the positions:

| Property         | WS-01                | SRV-01                        |
| ---------------- | -------------------- | ----------------------------- |
| Identity         | CORP\analyst         | CORP\analyst                  |
| Network          | 10.10.10.0/24        | 10.10.10.0/24 + 10.10.20.0/24 |
| Domain           | CORP.LOCAL           | CORP.LOCAL                    |
| Services         | Workstation Services | Server Services               |
| New Visibility   | Baseline             | 10.10.20.0/24                 |
| Pivot Capability | Limited              | Potential                     |
| New Targets      | Initial              | Additional internal systems   |

The new position is useful if it provides meaningful additional visibility or access.

## Phase 26 — Decide Whether to Continue

Use:

```text id="m8q3v7"
New Position
      ↓
New Target?
   ┌──┴──┐
  No     Yes
  ↓       ↓
Stop   In Scope?
           │
        ┌──┴──┐
       No     Yes
       ↓       ↓
      Stop   Reachable?
                  │
               ┌──┴──┐
              No     Yes
              ↓       ↓
           Pivot?   Service?
                       │
                    Authentication
                       ↓
                    Authorization
                       ↓
                    New Position
```

The process repeats only when a meaningful authorized path exists.

## Phase 27 — Example Movement Chain

A complete chain may look like:

```text id="q4m8p2"
P1
WS-01
CORP\analyst
10.10.10.25
      |
      | Kerberos / WinRM
      ↓
P2
SRV-01
CORP\analyst
10.10.10.50
      |
      | New Network Visibility
      ↓
10.10.20.0/24
      |
      | SMB / Application / Database
      ↓
Candidate Targets
```

At each stage:

```text id="n7m3x8"
Validate
   ↓
Enumerate
   ↓
Reassess
   ↓
Decide
```

## Phase 28 — Failure Handling

### Domain Authentication Fails

Check:

```text id="r8p3m6"
Domain Context
DNS
Time Synchronization
Credential Validity
Target Service
```

### Kerberos Authentication Fails

Check:

```text id="m5q8v2"
Domain Controller Reachability
DNS
Time
Realm
Service Principal
Target Service
```

### SMB Authentication Succeeds but Share Access Fails

Record:

```text id="x7n4p8"
Authentication: Successful
Authorization: Denied
```

Do not report the target as fully accessible.

### WinRM Is Reachable but Session Fails

Separate:

```text id="q3m8v5"
Network
 ↓
WinRM Service
 ↓
Authentication
 ↓
Authorization
```

Identify the failed layer.

### RDP Authentication Works but Logon Is Denied

This indicates:

```text id="v8m2r7"
Authentication
      ✓
Interactive Logon Authorization
      ✗
```

Record the exact boundary.

### Domain Information Is Incomplete

Check:

```text id="c5m9q3"
Current Identity
Domain Membership
DNS
Domain Controller Reachability
```

## Phase 29 — Evidence Record

For each movement:

```text id="p7m3x9"
Source Host:
Source Identity:
Target Host:
Target IP:
Domain:
Service:
Authentication Method:
Authentication Result:
Authorization Result:
Access Type:
Privilege:
New Network:
New Credential / Access:
Next Target:
Next Action:
```

Example:

```text id="n4q8m2"
Source Host: WS-01
Source Identity: CORP\analyst
Target Host: SRV-01
Target Service: WinRM
Authentication: Kerberos
Authentication Result: Successful
Authorization Result: Remote Session Allowed
Access Type: PowerShell Session
Privilege: Standard Domain User
New Network: 10.10.20.0/24
Next Action: Assess authorized targets on the new network
```

## Phase 30 — Position Ledger

Maintain:

| Position | Host   | Identity     | Network                       | Access           | New Visibility   |
| -------- | ------ | ------------ | ----------------------------- | ---------------- | ---------------- |
| P1       | WS-01  | CORP\analyst | 10.10.10.0/24                 | Initial foothold | Baseline         |
| P2       | SRV-01 | CORP\analyst | 10.10.10.0/24 + 10.10.20.0/24 | WinRM            | Internal network |

If another host is accessed:

| Position | Host | Identity | Network | Access | New Visibility |
| -------- | ---- | -------- | ------- | ------ | -------------- |
| P3       |      |          |         |        |                |

## Phase 31 — Domain Movement Model

The complete AD workflow can be summarized as:

```text id="m8q3p7"
Current Domain Identity
        ↓
Domain Context
        ↓
Target Discovery
        ↓
Service Discovery
        ↓
Authentication Material
        ↓
Target Service
        ↓
Authentication
        ↓
Authorization
        ↓
Movement
        ↓
New Domain Position
        ↓
Kerberos / Network / Service Reassessment
        ↓
New Target
        ↓
Continue or Stop
```

This model should be preferred over blindly trying multiple movement techniques.

## What This Scenario Demonstrates

The scenario connects:

```text id="q5m8r3"
Active Directory Identity
        ↓
Kerberos
        ↓
Target Discovery
        ↓
SMB / WinRM / RDP / WMI
        ↓
Authentication
        ↓
Authorization
        ↓
New Host
        ↓
New Network Visibility
        ↓
Credential Reassessment
        ↓
Next Movement Decision
```

The key lesson is that domain membership and Kerberos provide context, not automatic authorization.

## Safety and Scope

Perform this workflow only against explicitly authorized systems.

Avoid:

* broad credential spraying
* unnecessary authentication attempts
* unauthorized domain enumeration
* destructive changes
* unnecessary persistence
* unnecessary credential exposure
* access outside the assessment scope

Use targeted, evidence-driven validation.

## Golden Rule

> In an Active Directory environment, do not ask only whether an identity can authenticate. Determine which identity is being used, which authentication protocol applies, which service is being accessed, what authorization exists, and what the resulting position makes possible.

The movement chain is:

```text id="v7m3q8"
IDENTITY
   ↓
AUTHENTICATION
   ↓
SERVICE
   ↓
AUTHORIZATION
   ↓
ACCESS
   ↓
NEW POSITION
   ↓
RE-ENUMERATION
   ↓
NEXT DECISION
```

## What Next?

Continue to [`Linux-SSH-Movement.md`](Linux-SSH-Movement.md).

That scenario will apply the same end-to-end methodology to Linux-to-Linux movement using SSH, including SSH keys, agent context, account mapping, network reassessment, and post-movement enumeration.
