# Post-Movement Workflow

Successful lateral movement is not the end of the workflow.

It creates a **new assessment position** with a different identity, host, network view, service set, credential set, and potentially new paths to other systems.

The correct workflow is:

```text
Movement Succeeds
      ↓
Validate Access
      ↓
Identify New Host / User
      ↓
Reassess Network Position
      ↓
Reassess Credentials / Access
      ↓
Identify New Targets
      ↓
Determine Whether Another Movement Path Exists
      ↓
Continue the Chain
```

The central principle is:

> Every newly accessed host should be treated as a new starting point.

## Objective

After obtaining access to another host, determine:

* whether the movement actually succeeded
* which identity was obtained
* which host was reached
* what privileges are available
* what networks are now visible
* what services are exposed
* what new credentials or authentication material may be available
* whether another pivot is possible
* whether another lateral-movement path exists
* whether the assessment objective has been reached

## Movement Is a Position Change

A successful movement can be represented as:

```text
Host A
  ↓
Authentication
  ↓
Remote Service
  ↓
Host B
```

After reaching Host B, the workflow should **restart from Host B**:

```text
Host B
  ↓
Current User
  ↓
Network Context
  ↓
Domain / Trust Context
  ↓
Services
  ↓
Credentials / Access
  ↓
Target Discovery
```

Do not continue using Host A's assumptions.

## 1. Validate Access

First confirm that the movement produced the expected access.

Examples:

```text
SMB Access
WinRM Session
RDP Session
SSH Session
WMI Access
Application Access
```

Validation should answer:

```text
Did authentication succeed?
Did authorization succeed?
What resource or session was actually obtained?
```

Do not report movement based solely on a successful connection to a port.

## 2. Identify the New Identity

Determine who you are on the new system.

### Windows

```powershell id="2k7v4m"
whoami
whoami /user
whoami /groups
whoami /priv
```

### Linux

```bash id="n5q8p3"
whoami
id
groups
```

Record:

```text
New User:
Account Type:
Local / Domain:
Groups:
Relevant Privileges:
```

The new identity determines what can be done next.

## 3. Identify the New Host

Determine exactly which system was reached.

### Windows

```powershell id="v3m8x6"
hostname
ipconfig
```

### Linux

```bash id="q7k2p9"
hostname
ip addr
```

Record:

```text
Hostname:
IP Address:
Operating System:
Domain / Workgroup:
```

Do not rely only on the hostname provided by the connection target.

## 4. Reassess Network Context

The new host may have network visibility that the previous host did not.

### Windows

```powershell id="c4m9x2"
Get-NetIPConfiguration
Get-NetRoute
```

### Linux

```bash id="f8p3w6"
ip addr
ip route
ip neigh
```

Determine:

```text
Current Network:
Additional Interfaces:
Routes:
Gateways:
Reachable Networks:
Potential Pivot Paths:
```

The question is:

```text
What can I reach from this host
that I could not reach before?
```

## 5. Reassess Active Connections

Existing connections can reveal relationships to other systems.

### Windows

```powershell id="r6k2v9"
Get-NetTCPConnection
```

### Linux

```bash id="m3q8x5"
ss -tunap
```

Look for evidence of:

* internal servers
* domain controllers
* databases
* management systems
* application servers
* remote administration
* existing sessions

Treat connection information as evidence for further investigation, not automatic authorization to access every destination.

## 6. Reassess Domain Context

If the new host is Windows or domain-connected, determine its domain position.

Useful commands include:

```cmd id="w4n7p2"
whoami
echo %USERDOMAIN%
echo %USERDNSDOMAIN%
```

For domain-controller discovery:

```cmd id="x8m3q6"
nltest /dsgetdc:DOMAIN
```

For Kerberos state:

```cmd id="j5v9c1"
klist
```

Determine:

```text
Domain:
Domain Controller:
Current Principal:
Kerberos Context:
Groups:
Trust Context:
```

## 7. Reassess Credentials and Access

A newly accessed host may contain different authentication material.

The workflow is:

```text
New Host
   ↓
Credential / Access Discovery
   ↓
Identify Material
   ↓
Determine Account
   ↓
Determine Authentication Method
   ↓
Map to Targets
```

Potential evidence includes:

* authorized configuration material
* service credentials
* SSH keys
* Kerberos tickets
* existing sessions
* application authentication material
* documented service accounts
* other approved credential sources

Treat all authentication material as sensitive.

## 8. Reassess Remote Services

Determine what services the new host exposes or can access.

Common Windows services:

```text
SMB
WinRM
RDP
RPC / WMI
```

Common Linux services:

```text
SSH
HTTP / HTTPS
Databases
Application Services
```

The new service set may create additional movement paths.

## 9. Identify New Targets

Return to the target-discovery workflow.

The process becomes:

```text
New Host
   ↓
Network Enumeration
   ↓
Service Discovery
   ↓
Target Identification
   ↓
Target Prioritization
```

Do not automatically repeat every scan.

Use the new position to focus on what changed.

## 10. Compare the Old and New Positions

A useful assessment technique is to compare:

| Category       | Previous Host     | New Host      |
| -------------- | ----------------- | ------------- |
| Identity       | Previous user     | New user      |
| Host           | Host A            | Host B        |
| Networks       | Network A         | Network A + B |
| Routes         | Previous routes   | New routes    |
| Services       | Previous services | New services  |
| Credentials    | Previous material | New material  |
| Domain Context | Previous context  | New context   |
| Targets        | Previous targets  | New targets   |

This makes the value of the movement explicit.

## 11. Determine Whether to Continue

Not every successful movement should automatically lead to another movement.

Ask:

```text
Is the assessment objective reached?
        │
   ┌────┴────┐
  Yes       No
   ↓          ↓
Document   Is there a useful
Result     next path?
             │
        ┌────┴────┐
       No        Yes
       ↓           ↓
    Document    Validate
    Limitation  Next Path
```

A movement chain should continue only when the next step is justified by evidence and assessment scope.

## 12. Identify a New Pivot

The new host may provide access to another network.

Check:

```text
Interfaces
Routes
Gateways
DNS
Reachable Hosts
```

Then determine:

```text
New Network
    ↓
Previously Unreachable?
    ↓
Relevant?
    ↓
Potential Pivot
```

If yes, return to the pivoting workflow.

## 13. Identify New Credentials

A new host can reveal new authentication paths.

For example:

```text
Host B
  ↓
Service Account
  ↓
Authorized Credential Evidence
  ↓
Server C
```

Or:

```text
Host B
  ↓
Kerberos Ticket
  ↓
Domain Service
  ↓
Server C
```

The process remains:

```text
Credential
 ↓
Account
 ↓
Protocol
 ↓
Target
 ↓
Authentication
 ↓
Authorization
```

## 14. Reassess Existing Access

The new identity may already have access to systems that were not previously accessible.

Potential access types include:

```text
SMB
SSH
WinRM
RDP
Database
Internal Web Application
Administrative Interfaces
```

Map access based on evidence.

Do not assume that a new identity automatically inherits all access associated with another identity.

## 15. Build the Movement Chain

Record each successful transition.

Example:

```text id="e7m2v9"
WORKSTATION-01
    │
    │ Domain Account
    ↓
SERVER-01
    │
    │ New Network Visibility
    ↓
APP-01
    │
    │ Service Account
    ↓
SERVER-02
```

For each arrow, record:

```text
Source:
Identity:
Authentication Material:
Target:
Service:
Authentication:
Authorization:
Result:
```

This makes the entire attack path understandable.

## 16. Avoid Losing the Original Position

During complex assessments, maintain awareness of the original foothold.

A useful structure is:

```text id="m4q8x2"
Initial Foothold
      ↓
Movement 1
      ↓
Movement 2
      ↓
Movement 3
```

Record how each position was obtained.

This helps answer:

* Where did access originate?
* Which credential enabled each step?
* Which host provided each pivot?
* Which systems became reachable?
* Where did the chain terminate?

## 17. Access Validation

Validate the exact access level obtained.

For example:

```text
SMB:
Authenticated → Share Access → Administrative Share?

WinRM:
Authenticated → Remote Session → Administrative Rights?

RDP:
Authenticated → Interactive Session → Privileged Session?

SSH:
Authenticated → Shell → Privileged User?

Application:
Authenticated → Application Functionality → Administrative Functionality?
```

Avoid describing an access level more broadly than the evidence supports.

## 18. New-Host Enumeration Workflow

After every successful movement:

```text id="r9v3k6"
1. Who am I?
      ↓
2. Where am I?
      ↓
3. What networks can I reach?
      ↓
4. What services exist?
      ↓
5. What credentials / access exist?
      ↓
6. What targets are now visible?
      ↓
7. Is another pivot possible?
      ↓
8. Is another movement path justified?
```

This should become automatic during a multi-host assessment.

## 19. Stop Conditions

A movement chain should have explicit stop conditions.

Examples:

```text
Assessment Objective Reached
```

```text
No Additional Relevant Access
```

```text
No Authorized Next Step
```

```text
Target Is Out of Scope
```

```text
Further Testing Would Create Unnecessary Risk
```

Stopping is part of a controlled assessment.

## 20. Failure Handling

### Movement Succeeded but New Host Cannot Be Enumerated

Possible causes:

* limited shell/session
* restricted permissions
* endpoint security controls
* command availability
* service-specific access

Record what was actually obtained.

### New Host Has No Additional Network Visibility

This does not mean the movement was unsuccessful.

It may still provide:

* new credentials
* new services
* different authorization
* access to a specific application or resource

### New Credentials Do Not Work

Return to:

```text
Credential Type
Account Context
Target
Service
Authentication
Authorization
```

### New Target Is Unreachable

Determine whether the issue is:

* network routing
* firewall
* segmentation
* service availability
* incorrect address

## 21. Evidence Recording

Maintain a movement ledger.

```text
Movement ID:
Source Host:
Source User:
Authentication Material:
Target Host:
Target Service:
Authentication Result:
Authorization Result:
New User:
New Network:
New Credentials:
New Targets:
Next Action:
```

Example:

```text
Movement ID: LM-02
Source Host: WORKSTATION-01
Source User: DOMAIN\user
Target Host: SERVER-01
Target Service: WinRM
Authentication: Successful
Authorization: Remote session permitted
New User: DOMAIN\user
New Network: 10.10.20.0/24
New Credentials: Service account evidence identified
New Target: APP-01
Next Action: Assess authorized access to APP-01
```

## 22. Decision Framework

Use this after every successful movement:

```text
Movement Successful
        ↓
Validate Identity
        ↓
Validate Host
        ↓
Validate Network Position
        ↓
Re-enumerate Services
        ↓
Reassess Credentials / Access
        ↓
Identify New Targets
        ↓
Assessment Objective Reached?
        │
   ┌────┴────┐
  Yes       No
  ↓           ↓
Document    Useful Next Path?
Result          │
           ┌────┴────┐
          No        Yes
          ↓           ↓
       Document    Validate
       Limitation  Next Movement
                       ↓
                  New Position
                       ↓
                  Repeat Workflow
```

## 23. Operational Rules

### Always Validate the New Position

Never assume movement succeeded merely because a connection was established.

### Re-enumerate Every Host

A new host is a new network and identity perspective.

### Compare Positions

Document what became newly visible or accessible.

### Protect Credentials

New authentication material can expand the assessment scope of possible movement but must remain controlled.

### Keep the Chain Documented

Every movement should have a clear source, target, service, identity, and result.

### Know When to Stop

The goal is to answer the assessment objective, not to move indefinitely.

## What Not to Assume

```text
Movement Successful
   ≠
Administrative Access
```

```text
New Host
   ≠
New Network
```

```text
New Credential
   ≠
Valid Everywhere
```

```text
New Network
   ≠
Every Internal Host Reachable
```

```text
New Access
   ≠
Need to Continue Moving
```

The value of a movement is determined by what the new position actually provides.

## Section Structure

The remaining files provide the detailed post-movement workflow:

| File                                                       | Focus                                                |
| ---------------------------------------------------------- | ---------------------------------------------------- |
| [`Access-Validation.md`](Access-Validation.md)             | Confirm exactly what access was obtained             |
| [`New-Host-Enumeration.md`](New-Host-Enumeration.md)       | Rebuild the host and network picture                 |
| [`Credential-Reassessment.md`](Credential-Reassessment.md) | Reassess authentication material and access          |
| [`Continue-the-Chain.md`](Continue-the-Chain.md)           | Decide whether and how another movement should occur |

## What Next?

Continue to [`Access-Validation.md`](Access-Validation.md).

That file focuses on the first thing to do after movement: prove exactly what access was obtained before making assumptions about the new host.
