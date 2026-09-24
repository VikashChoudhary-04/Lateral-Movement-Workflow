# Windows Workstation to Server

This scenario demonstrates an end-to-end lateral-movement workflow from a Windows workstation to an in-scope Windows server.

The purpose is not to assume that one particular technique will work.

Instead, the workflow starts from the compromised workstation, builds the available context, identifies a suitable server and remote service, maps existing authentication material to that service, validates authentication and authorization, and then treats the server as a new assessment position.

The scenario is intended for authorized penetration tests and controlled labs.

## Scenario Objective

Demonstrate a Windows-to-Windows movement path:

```text id="r4m8p2"
Compromised Workstation
        ↓
Position Assessment
        ↓
Internal Target Discovery
        ↓
Remote Service Discovery
        ↓
Credential / Access Mapping
        ↓
Authentication
        ↓
Authorization
        ↓
Server Access
        ↓
Access Validation
        ↓
New-Host Enumeration
        ↓
Credential Reassessment
        ↓
Continue or Stop
```

The objective is to establish **validated server access**, not to assume full administrative control.

## Starting Position

Assume the assessment begins with an authorized foothold on:

```text id="p7q3m9"
Host: WORKSTATION-01
IP: 10.10.10.25
OS: Windows
Identity: DOMAIN\analyst
Network: 10.10.10.0/24
```

The exact values are illustrative.

Replace them with values from the authorized lab or assessment environment.

## Scope

The scenario assumes:

```text id="x5m8r2"
Source:
WORKSTATION-01

Authorized Target:
SERVER-01

Authorized Network:
10.10.10.0/24

Potential Services:
SMB
WinRM
RDP
WMI / RPC
```

Only systems explicitly included in the assessment scope should be tested.

## Phase 1 — Establish the Current Position

Start by confirming the compromised workstation.

### Host

```powershell id="m8q2v6"
hostname
```

### Identity

```powershell id="c4n7p9"
whoami
whoami /user
```

### Groups

```powershell id="v6m3r8"
whoami /groups
```

### Privileges

```powershell id="q9p4m2"
whoami /priv
```

### Network

```powershell id="t7x3m8"
ipconfig /all
```

### Routes

```powershell id="n5q8r3"
route print
```

The initial position record should look like:

```text id="w2m7p4"
Host:
WORKSTATION-01

Identity:
DOMAIN\analyst

Network:
10.10.10.0/24

Routes:
10.10.10.0/24

Objective:
Identify and validate authorized access to SERVER-01.
```

## Phase 2 — Identify the Target

Assume the assessment scope identifies:

```text id="f8m3q7"
SERVER-01
10.10.10.50
```

Before attempting movement, determine what services are exposed.

The relevant questions are:

```text id="p4v8m2"
Can WORKSTATION-01 reach SERVER-01?
        ↓
Which remote services are available?
        ↓
Which authentication methods are supported?
        ↓
Does DOMAIN\analyst have authorization?
```

## Phase 3 — Check Reachability

For SMB:

```powershell id="k7n2m5"
Test-NetConnection 10.10.10.50 -Port 445
```

For WinRM:

```powershell id="x3q8v6"
Test-NetConnection 10.10.10.50 -Port 5985
```

For HTTPS WinRM:

```powershell id="m9p4r2"
Test-NetConnection 10.10.10.50 -Port 5986
```

For RDP:

```powershell id="v5n8q3"
Test-NetConnection 10.10.10.50 -Port 3389
```

For RPC:

```powershell id="c6m2x9"
Test-NetConnection 10.10.10.50 -Port 135
```

Record only the services relevant to the authorized assessment.

## Phase 4 — Interpret Reachability

Use the following model:

```text id="q3m8v5"
Port Reachable
      ↓
Service Likely Available
      ↓
Authentication Test
      ↓
Authorization Test
```

Do not conclude:

```text id="j7p2n9"
445 open
  =
SMB administrative access
```

Instead:

```text id="a4m8q3"
445 reachable
  ↓
SMB available
  ↓
Authentication
  ↓
Share / Administrative Authorization
```

## Phase 5 — Identify Remote Access Options

Suppose the assessment identifies:

```text id="y8r3m6"
445  → SMB
5985 → WinRM
3389 → RDP
135  → RPC
```

The next decision is not:

> Which technique is most powerful?

It is:

> Which available service matches the access already available and the assessment objective?

## Phase 6 — Reassess Existing Credentials

Check the current access context.

### Current Identity

```powershell id="p6m9x4"
whoami
```

### Kerberos Context

```cmd id="v3q8n2"
klist
```

The assessment may already have a domain authentication context.

Record:

```text id="r7m4c8"
Identity:
DOMAIN\analyst

Authentication Context:
Domain / Kerberos

Potential Services:
SMB
WinRM
RDP
```

Do not assume that the identity is authorized for all three services.

## Phase 7 — Map Identity to Service

Create the access hypothesis:

```text id="n5q2m8"
DOMAIN\analyst
       ↓
Domain Authentication
       ↓
SERVER-01
       ↓
SMB / WinRM / RDP
```

Now test the specific service required by the scenario.

## Phase 8 — Validate SMB

If SMB is the intended access path, identify available shares.

From the authorized workstation:

```cmd id="k8m3p6"
net view \\SERVER-01
```

Where appropriate, an SMB client can also enumerate shares.

The result should be interpreted as:

```text id="q4v7n2"
SMB Authentication
      ↓
Share Enumeration
      ↓
Share Authorization
```

Suppose the result indicates:

```text id="c9m5r8"
Public
Projects
```

The next question is:

> What access does DOMAIN\analyst actually have?

Possible results:

```text id="x2p7m4"
Read
Write
Read/Write
Denied
```

Record the permission boundary.

## Phase 9 — Validate WinRM

If WinRM is available and authorized for testing:

```powershell id="v8q3m6"
Test-WSMan SERVER-01
```

A successful WS-Man response demonstrates service reachability.

It does not by itself prove that the current identity can establish a remote PowerShell session.

If the identity is authorized for a session:

```powershell id="m4n7p2"
Enter-PSSession -ComputerName SERVER-01
```

After session establishment:

```powershell id="r9x3q6"
whoami
hostname
```

The session must be validated from inside the target.

## Phase 10 — Validate RDP

If RDP is in scope:

```text id="p7m3v8"
SERVER-01
    ↓
RDP Reachable
    ↓
Account Authentication
    ↓
Interactive Logon Authorization
```

After successful login:

```cmd id="c5q8n2"
whoami
hostname
ipconfig
```

Then determine the actual privilege context:

```cmd id="v3m9r7"
whoami /groups
whoami /priv
```

Successful RDP login does not automatically mean administrative access.

## Phase 11 — Consider WMI / RPC

If WMI-based management is relevant, first understand the dependency:

```text id="x8m4q2"
RPC Endpoint Mapper
      ↓
Dynamic RPC
      ↓
WMI / DCOM
      ↓
Authorization
```

Check RPC reachability:

```powershell id="m6p2v9"
Test-NetConnection SERVER-01 -Port 135
```

Do not confuse:

```text id="r3q8n5"
RPC Connectivity
```

with:

```text id="j7m4c2"
WMI Authorization
```

Validate only the management capability required by the assessment.

## Phase 12 — Select the Demonstrated Path

Suppose:

```text id="a8m3q7"
SMB:
Authentication ✓
Share Access ✓

WinRM:
Authentication ✓
Remote Session ✗

RDP:
Not Required

WMI:
Not Required
```

The demonstrated path becomes:

```text id="p5v9m2"
WORKSTATION-01
      ↓
SMB
      ↓
SERVER-01
      ↓
Authorized Share Access
```

If the objective requires host-level movement, a resource-level SMB result may not be sufficient.

This distinction must be documented.

## Phase 13 — Validate Server Access

If a host-level session is established, immediately confirm:

```text id="w7q3n8"
Hostname
Identity
Privilege
Network
```

### Hostname

```powershell id="c4m8r2"
hostname
```

### Identity

```powershell id="n6p3v9"
whoami
whoami /user
```

### Network

```powershell id="x8q2m5"
ipconfig
```

### Privilege Context

```powershell id="m7r4c9"
whoami /groups
whoami /priv
```

The access record should now state exactly what was obtained.

Example:

```text id="t3n8p5"
Target:
SERVER-01

Identity:
DOMAIN\analyst

Access:
WinRM PowerShell Session

Privilege:
Standard Domain User

Network:
10.10.10.0/24
```

## Phase 14 — Compare the New Position

Compare:

| Property        | WORKSTATION-01 | SERVER-01      |
| --------------- | -------------- | -------------- |
| Identity        | DOMAIN\analyst | DOMAIN\analyst |
| Network         | 10.10.10.0/24  | 10.10.10.0/24  |
| Access          | Workstation    | WinRM          |
| Role            | Workstation    | Server         |
| New Routes      |                |                |
| New Services    |                |                |
| New Credentials |                |                |
| New Targets     |                |                |

The purpose is to determine whether SERVER-01 creates a better assessment position.

## Phase 15 — Enumerate the Server

Start with the standard new-host workflow.

### Network Interfaces

```powershell id="q5m8r3"
Get-NetIPConfiguration
```

### Routes

```powershell id="v7n2p6"
Get-NetRoute
```

### Listening Services

```powershell id="m3x8q4"
Get-NetTCPConnection -State Listen
```

### Running Services

```powershell id="r6p2n9"
Get-Service |
    Where-Object {$_.Status -eq "Running"}
```

### Active Connections

```powershell id="c8m4v7"
Get-NetTCPConnection
```

The objective is to understand the server's role and network position.

## Phase 16 — Identify the Server Role

Combine the evidence.

Suppose the server exposes:

```text id="x5q8m3"
SMB
WinRM
SQL Server
Internal Web Application
```

and has:

```text id="n7m2p9"
10.10.10.50
10.10.20.50
```

The second interface may indicate access to another internal network.

Do not assume that the network is reachable from the original workstation.

That difference is precisely why the new position matters.

## Phase 17 — Reassess Credentials

Now inspect the server for relevant authentication material.

Focus on the server's role.

For example:

```text id="p8m3v6"
Application Configuration
Service Accounts
Environment Variables
Kerberos Context
Existing Sessions
Existing Network Access
```

On a Windows host, confirm the current Kerberos context:

```cmd id="m4q8n2"
klist
```

Review service identities where authorized:

```powershell id="x7p3v9"
Get-CimInstance Win32_Service |
    Select-Object Name, StartName, State
```

The goal is to identify relationships rather than unnecessarily expose secrets.

## Phase 18 — Identify New Networks

Suppose the server shows:

```text id="c3m8q5"
Interface 1:
10.10.10.50

Interface 2:
10.10.20.50
```

and:

```powershell id="r7n2m4"
Get-NetRoute
```

shows routes for:

```text id="v5q8p3"
10.10.10.0/24
10.10.20.0/24
```

The new position may now provide access to:

```text id="k4m7x2"
10.10.20.0/24
```

which was not directly reachable from the original workstation.

This creates a potential pivot opportunity.

## Phase 19 — Decide Whether to Pivot

Use:

```text id="n8p3m6"
New Network
      ↓
Is It In Scope?
      ↓
Can SERVER-01 Reach It?
      ↓
Are There Relevant Targets?
      ↓
Is Pivoting Necessary?
```

If the answer to these questions supports further assessment, use the pivoting workflow.

See:

[`../09-Pivoting-and-Internal-Network-Access/`](../09-Pivoting-and-Internal-Network-Access/)

Do not pivot simply because another interface exists.

## Phase 20 — Reassess Existing Access

Suppose SERVER-01 already communicates with:

```text id="q6m3v8"
10.10.20.15
10.10.20.30
10.10.20.40
```

Review the connections:

```powershell id="x2m8p4"
Get-NetTCPConnection
```

Determine whether these systems correspond to:

```text id="p7n4c9"
Database
Application Server
File Server
Management System
```

This creates evidence-based candidate targets.

## Phase 21 — Build the Movement Record

The scenario now has:

```text id="v8m3q2"
P1
WORKSTATION-01
DOMAIN\analyst
10.10.10.25

        |
        | WinRM
        | Authentication: Successful
        | Authorization: Remote Session
        ↓

P2
SERVER-01
DOMAIN\analyst
10.10.10.50
Networks:
10.10.10.0/24
10.10.20.0/24
```

The important point is that P2 is now a validated position.

## Phase 22 — Determine the Next Step

Evaluate:

```text id="m5q8r3"
Did SERVER-01 reveal a new network?
```

If yes:

```text id="x7n2p6"
Identify authorized targets
```

Then:

```text id="c4m9v8"
Identify services
        ↓
Map credentials
        ↓
Validate authentication
        ↓
Validate authorization
```

If no:

```text id="r8p3m5"
Reassess existing access
```

If no meaningful path exists:

```text id="n6m2q9"
Stop and document the chain.
```

## Phase 23 — Failure Branches

### Port Reachable, Authentication Fails

```text id="w4q8m3"
Network ✓
Service ✓
Authentication ✗
```

Check:

* identity context
* authentication protocol
* credential validity
* target configuration

### Authentication Succeeds, Authorization Fails

```text id="p7m3x9"
Network ✓
Service ✓
Authentication ✓
Authorization ✗
```

Record the permission boundary.

### SMB Works, Host Shell Does Not

This is still useful:

```text id="m8q2v5"
SMB Access
```

but should not be reported as:

```text id="c3n7r9"
Interactive Host Access
```

### WinRM Works, Privilege Is Limited

Record:

```text id="q5m8p2"
WinRM Session:
DOMAIN\analyst

Privilege:
Standard User
```

Then determine whether the new position provides useful network or credential visibility.

### Server Is Reachable but No Useful Access Exists

Do not force movement.

Record:

```text id="x7m4n8"
Target:
SERVER-01

Reachability:
Confirmed

Useful Authorization:
Not demonstrated
```

Then return to target discovery or stop.

## Phase 24 — Complete Scenario Flow

The entire scenario becomes:

```text id="v3m8q2"
WORKSTATION-01
      ↓
Confirm Identity
      ↓
Enumerate Network
      ↓
Identify SERVER-01
      ↓
Check SMB / WinRM / RDP / RPC
      ↓
Map DOMAIN\analyst
      ↓
Validate Authentication
      ↓
Validate Authorization
      ↓
Establish Authorized Server Access
      ↓
Validate SERVER-01
      ↓
Enumerate SERVER-01
      ↓
Reassess Credentials
      ↓
Identify New Networks / Targets
      ↓
Continue?
   ┌──┴──┐
  Yes    No
   ↓      ↓
Next    Document
Step    Chain
```

## Scenario Evidence Table

| Step | Source         | Target      | Service | Authentication | Authorization | Result |
| ---- | -------------- | ----------- | ------- | -------------- | ------------- | ------ |
| 1    | WORKSTATION-01 | SERVER-01   | SMB     |                |               |        |
| 2    | WORKSTATION-01 | SERVER-01   | WinRM   |                |               |        |
| 3    | WORKSTATION-01 | SERVER-01   | RDP     |                |               |        |
| 4    | SERVER-01      | Next Target |         |                |               |        |

Populate only the paths actually tested.

## Position Ledger

| Position | Host           | Identity       | Network                                | Access           | New Visibility   |
| -------- | -------------- | -------------- | -------------------------------------- | ---------------- | ---------------- |
| P1       | WORKSTATION-01 | DOMAIN\analyst | 10.10.10.0/24                          | Initial foothold | Baseline         |
| P2       | SERVER-01      | DOMAIN\analyst | 10.10.10.0/24 + possible 10.10.20.0/24 | WinRM / SMB      | Internal network |

## What This Scenario Demonstrates

This scenario demonstrates the complete reasoning chain:

```text id="q8m3v7"
A host is compromised
        ↓
The position is assessed
        ↓
A server is identified
        ↓
Services are identified
        ↓
Authentication material is mapped
        ↓
Authentication is tested
        ↓
Authorization is tested
        ↓
Access is established
        ↓
The new host is validated
        ↓
The new host is enumerated
        ↓
New credentials and networks are reassessed
        ↓
The next movement decision is evidence-based
```

## Golden Rule

> Do not move from a Windows workstation to a server simply because the server is reachable. Move when there is a known, authorized, and validated relationship between the current access, the target service, and the required authorization.

The value of the movement is not merely reaching SERVER-01.

The value is understanding what SERVER-01 makes possible from the new position.

## What Next?

Continue to [`AD-Lateral-Movement.md`](AD-Lateral-Movement.md).

That scenario will apply the same end-to-end workflow to a domain environment, where identity, Kerberos, domain services, authorization, and post-movement enumeration become central to the movement chain.
