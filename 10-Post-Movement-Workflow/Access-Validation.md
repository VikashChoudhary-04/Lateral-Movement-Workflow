# Access Validation

Access validation is the first step after successful lateral movement.

The purpose is to prove exactly what access was obtained rather than assuming that a successful connection means full control of the target.

The workflow is:

```text id="m8q4v2"
Movement Attempt
      ↓
Connection Established
      ↓
Identify Authentication Result
      ↓
Identify Authorization Result
      ↓
Identify User
      ↓
Identify Host
      ↓
Determine Actual Access Level
      ↓
Record Evidence
      ↓
Continue With New-Host Enumeration
```

The central rule is:

> Validate the access you actually obtained, not the access you expected to obtain.

## Objective

Determine:

* whether authentication succeeded
* which identity was authenticated
* which host was reached
* which service provided the access
* what actions are authorized
* whether administrative privileges exist
* what network position was obtained
* whether the access is interactive, resource-based, or service-specific

## Authentication vs Authorization

These must always be evaluated separately.

### Authentication

```text id="p6x2r8"
Credential
   ↓
Service
   ↓
Identity Accepted
```

This answers:

> Who are you?

### Authorization

```text id="w3m7q1"
Authenticated Identity
   ↓
Requested Action
   ↓
Allowed / Denied
```

This answers:

> What are you allowed to do?

A successful authentication does not automatically establish administrative access.

## 1. Confirm the Target Host

First verify that the expected host was actually reached.

### Windows

```powershell id="v4k8m2"
hostname
ipconfig
```

### Linux

```bash id="r7p3x9"
hostname
ip addr
```

Compare the result with the intended target.

Record:

```text id="q2n6w8"
Expected Host:
Actual Host:
Expected IP:
Observed IP:
```

This is particularly important when aliases, DNS, load balancers, or multiple interfaces are involved.

## 2. Confirm the Current Identity

### Windows

```powershell id="m5v9c3"
whoami
whoami /user
whoami /groups
```

For privileges:

```powershell id="x8k2p6"
whoami /priv
```

### Linux

```bash id="j4q7n1"
whoami
id
groups
```

Record:

```text id="c6m3r8"
Username:
UID / SID:
Domain / Local Context:
Groups:
Relevant Privileges:
```

Do not infer identity solely from the credentials that were used to establish the session.

## 3. Determine the Access Type

Identify what kind of access was actually obtained.

Common categories include:

```text id="f9p4w2"
SMB Resource Access
WinRM Session
RDP Session
SSH Session
WMI / Remote Management Access
Application Access
Database Access
```

For example:

```text id="t3m8q5"
SMB Authentication
      ≠
Interactive Shell
```

Likewise:

```text id="n7v2c4"
Application Authentication
      ≠
Operating-System Access
```

Record the exact access type.

## 4. Validate SMB Access

For SMB, determine:

```text id="k8q3m5"
Authentication
      ↓
Visible Shares
      ↓
Readable Resources
      ↓
Writable Resources
      ↓
Administrative Access
```

From Windows, an authorized assessment may use:

```cmd id="p5x9v2"
net view \\TARGET
```

A specific share can be checked through an appropriate authorized connection.

From Linux:

```bash id="r3m7k1"
smbclient -L //TARGET -U 'DOMAIN\username'
```

The important distinction is:

```text id="d6v2q8"
SMB Authentication ✓
      ↓
Share Access?
```

Do not report administrative access unless the evidence confirms it.

## 5. Validate WinRM Access

If WinRM was the movement mechanism:

```powershell id="y4n8p2"
Test-WSMan TARGET
```

After establishing an authorized session:

```powershell id="m7q3v9"
whoami
hostname
```

The session proves remote PowerShell access for the authenticated identity.

Determine whether the identity has the privileges required for the assessment objective.

## 6. Validate RDP Access

For RDP:

```text id="c5v8r2"
RDP Connection
      ↓
Authentication
      ↓
Interactive Session
      ↓
Identity
      ↓
Privileges
```

Once logged in:

```cmd id="n3k7p5"
whoami
hostname
ipconfig
```

If appropriate:

```cmd id="x6m2q8"
whoami /groups
whoami /priv
```

Record whether the session is:

* local account
* domain account
* standard user
* administrative identity

Do not infer administrative privileges from successful RDP login alone.

## 7. Validate SSH Access

After SSH access:

```bash id="v8q3m6"
whoami
id
hostname
ip addr
ip route
```

Determine:

```text id="m2r7k9"
User:
UID:
Groups:
Hostname:
Interfaces:
Routes:
```

The SSH shell may provide access without providing root privileges.

For example:

```text id="z4p8c1"
SSH Authentication ✓
Root Access ✗
```

is still successful lateral movement.

## 8. Validate WMI / Remote Management Access

WMI-based access can provide remote management functionality without necessarily providing the same session type as WinRM.

Distinguish:

```text id="q7m3v9"
RPC Connectivity
      ↓
WMI Functionality
      ↓
Authorization
      ↓
Remote Action
```

If WMI is available, determine exactly which operations the authenticated identity can perform.

Do not equate RPC connectivity with WMI authorization.

## 9. Validate Application Access

If movement leads to an internal application:

```text id="f5n8m2"
Network Access
      ↓
Application
      ↓
Authentication
      ↓
Application Authorization
```

Determine:

* authenticated user
* application role
* accessible functionality
* administrative functions
* data access
* session scope

An application account may have significant application privileges while having no operating-system access.

## 10. Validate Database Access

For a database service:

```text id="w2q6p9"
Network
  ↓
Database Port
  ↓
Database Service
  ↓
Authentication
  ↓
Database Authorization
```

Determine:

* database identity
* accessible databases
* permissions
* available schemas
* authorized read/write scope

Do not assume database authentication means operating-system access.

## 11. Determine Privilege Level

Privilege level should be based on evidence.

### Windows

Useful checks:

```cmd id="n8m4r7"
whoami /groups
whoami /priv
```

Determine whether the identity belongs to relevant privileged groups.

### Linux

```bash id="p6x3v9"
id
sudo -l
```

Where authorized, `sudo -l` can identify commands permitted through sudo.

Record the actual privilege rather than describing the account generically as "privileged."

## 12. Determine Network Position

Once access is validated, identify the network context.

### Windows

```powershell id="r4m8q2"
ipconfig
route print
```

### Linux

```bash id="k7v3n5"
ip addr
ip route
```

Determine:

```text id="x2q9m6"
Interfaces:
Subnets:
Gateways:
Routes:
DNS:
Potential Internal Networks:
```

This becomes the starting point for the next stage.

## 13. Validate Existing Connections

Existing connections may reveal additional systems.

### Windows

```powershell id="v5n2p8"
Get-NetTCPConnection
```

### Linux

```bash id="m9q4r7"
ss -tunap
```

Look for evidence of:

* internal application servers
* databases
* domain controllers
* file servers
* management systems

Treat this as discovery evidence rather than permission to access every destination.

## 14. Confirm Persistence of Access

Where relevant to the assessment, determine whether the access remains usable after the initial connection closes.

For example:

```text id="c8m3x6"
Initial Connection
      ↓
Session Established
      ↓
Connection Closed
      ↓
Can Authorized Access Be Re-established?
```

This helps distinguish:

```text id="y7p2n9"
Reusable Account Access
```

from:

```text id="h4v8q1"
Temporary Session-Only Access
```

Do not create persistence merely to test this property unless it is explicitly within scope.

## 15. Validate Resource Access

For resource-based access, determine what is actually available.

Examples:

### SMB

```text id="k3m7v9"
Share
 ↓
Directory
 ↓
Read / Write
```

### Application

```text id="p8q2x5"
Application
 ↓
Function
 ↓
Role
 ↓
Allowed Action
```

### Database

```text id="r5v9c3"
Database
 ↓
Schema
 ↓
Table
 ↓
Read / Write
```

The objective is to identify the **effective authorization boundary**.

## 16. Do Not Overstate Access

Use precise descriptions.

Prefer:

```text id="x6m3q8"
"Authenticated to SMB and accessed the Finance share."
```

over:

```text id="w2p7k4"
"Compromised the server."
```

Prefer:

```text id="n9r4v6"
"Established a WinRM session as DOMAIN\user."
```

over:

```text id="c3m8q2"
"Got administrator access."
```

unless administrative privileges have actually been demonstrated.

Accurate access descriptions are essential for reporting.

## 17. Access Validation Matrix

Use this model:

| Access Type | Authentication | Service Access | Authorization                      | Host Access                           |
| ----------- | -------------- | -------------- | ---------------------------------- | ------------------------------------- |
| SMB         | Confirm        | Confirm        | Determine shares                   | Not necessarily                       |
| WinRM       | Confirm        | Confirm        | Determine remote-management rights | Usually interactive management access |
| RDP         | Confirm        | Confirm        | Determine logon rights             | Interactive session                   |
| SSH         | Confirm        | Confirm        | Determine shell privileges         | Shell access                          |
| WMI         | Confirm        | Confirm        | Determine permitted operations     | Remote management capability          |
| Web App     | Confirm        | Confirm        | Determine application role         | Not necessarily                       |
| Database    | Confirm        | Confirm        | Determine DB permissions           | Not necessarily                       |

The exact behavior depends on configuration.

## 18. Evidence Recording

Record the validated access:

```text id="q5m8x2"
Source Host:
Target Host:
Target IP:
Service:
Authentication Method:
Authenticated Identity:
Authentication Result:
Authorization Result:
Access Type:
Privilege Level:
Network Position:
New Targets:
Next Step:
```

Example:

```text id="v7r3m9"
Source Host: WORKSTATION-01
Target Host: SERVER-01
Service: WinRM
Authentication: Kerberos
Identity: DOMAIN\user
Authentication Result: Successful
Authorization Result: Remote session permitted
Access Type: PowerShell Session
Privilege Level: Standard Domain User
New Network: 10.10.20.0/24
Next Step: Re-enumerate SERVER-01
```

## 19. Failure Classification

### Connection Succeeds but Authentication Fails

```text id="m4q8v2"
Network ✓
Service ✓
Authentication ✗
```

Investigate credentials and authentication context.

### Authentication Succeeds but Authorization Fails

```text id="p7x3n9"
Network ✓
Service ✓
Authentication ✓
Authorization ✗
```

The identity is valid but lacks the requested rights.

### Authorization Succeeds but Expected Resource Is Missing

Possible causes:

* resource does not exist
* incorrect path
* application-specific restrictions
* different authorization scope

### Interactive Access Is Not Available

A valid account may still provide useful resource access without providing a shell.

Record the exact access obtained.

## 20. Decision Workflow

Use this workflow immediately after movement:

```text id="z8m4q1"
Movement Succeeded
      ↓
Confirm Target Host
      ↓
Confirm Identity
      ↓
Identify Access Type
      ↓
Authentication Confirmed?
      │
 ┌────┴────┐
 No        Yes
 ↓           ↓
Reassess   Validate Authorization
Auth Path       │
            ┌───┴────┐
           No        Yes
           ↓           ↓
      Record Limit   Determine
                     Effective Access
                          ↓
                  Determine Privilege
                          ↓
                  Determine Network Position
                          ↓
                  Record Evidence
                          ↓
                  New-Host Enumeration
```

## 21. Operational Rules

### Prove the Host

Verify the actual destination.

### Prove the Identity

Verify the authenticated principal.

### Prove the Permission

Determine what that identity can actually do.

### Prove the Position

Determine what networks and services are visible from the new host.

### Use Precise Language

Report exactly what was demonstrated.

### Do Not Create Unnecessary Persistence

Access validation should not become persistence activity unless explicitly authorized.

## What Not to Assume

```text id="f4m8q2"
Port Open
   ≠
Authentication
```

```text id="x7n3v9"
Authentication
   ≠
Authorization
```

```text id="p2q6m8"
Authorization
   ≠
Administrative Privileges
```

```text id="c8r4k1"
Interactive Access
   ≠
Full Host Control
```

```text id="m5v9x3"
Resource Access
   ≠
Operating-System Access
```

The purpose of validation is to replace assumptions with evidence.

## What Next?

Continue to [`New-Host-Enumeration.md`](New-Host-Enumeration.md).

That file takes the validated position and rebuilds the host, network, service, domain, and access picture from the newly compromised system.
