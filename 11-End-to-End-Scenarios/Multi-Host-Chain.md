# Multi-Host Chain

This scenario demonstrates a longer lateral-movement chain involving multiple hosts, changing network visibility, different access methods, and repeated post-movement reassessment.

The purpose is not to maximize the number of systems accessed.

The purpose is to demonstrate how a disciplined assessment maintains awareness of:

* the current position
* previously validated positions
* available identities
* authentication material
* reachable networks
* remote services
* authorization boundaries
* pivot opportunities
* meaningful stopping conditions

The scenario is intended for authorized penetration tests and controlled labs.

## Scenario Objective

Demonstrate a multi-stage movement chain:

```text id="m8q3v7"
Initial Host
    ↓
Position Assessment
    ↓
Target Discovery
    ↓
Movement
    ↓
Access Validation
    ↓
New-Host Enumeration
    ↓
Credential Reassessment
    ↓
New Target
    ↓
Movement
    ↓
Pivot / Internal Network Access
    ↓
New Position
    ↓
Reassessment
    ↓
Continue or Stop
```

The central loop is:

```text id="p4n8m2"
MOVE
 ↓
VALIDATE
 ↓
ENUMERATE
 ↓
REASSESS
 ↓
DECIDE
 ↓
MOVE AGAIN — OR STOP
```

## Scenario Topology

Use the following illustrative environment:

```text id="q7m3r8"
                    NETWORK A
                  10.10.10.0/24
                         |
                         |
                  +------+------+
                  |             |
               WS-01         SERVER-01
             10.10.10.25    10.10.10.50
                  |             |
                  |             |
                  |       +-----+------+
                  |       |            |
                  |    NETWORK B     NETWORK A
                  |  10.10.20.0/24
                  |       |
                  |       |
                  |    FILE-01
                  |  10.10.20.15
                  |       |
                  |       |
                  |    LINUX-01
                  |  10.10.20.25
                  |
              Initial Foothold
```

The exact topology is illustrative.

The assessment scope determines which systems may actually be tested.

## Example Movement Path

The scenario demonstrates:

```text id="v5m8q2"
P1
WS-01
      |
      | WinRM
      ↓
P2
SERVER-01
      |
      | Pivot / Internal Network Access
      ↓
NETWORK B
      |
      +------→ FILE-01
      |
      +------→ LINUX-01
```

The important concept is that SERVER-01 creates access to a network that was not directly reachable from WS-01.

## Starting Position

Assume:

```text id="n3q7m8"
Host:
WS-01

IP:
10.10.10.25

Identity:
CORP\analyst

Network:
10.10.10.0/24
```

The assessment objective is to determine whether the current position provides an authorized path into the internal server network.

## Phase 1 — Establish the Initial Position

Confirm:

```powershell id="m7p3x9"
hostname
```

```powershell id="q4n8v2"
whoami
```

```powershell id="x6m3r7"
whoami /groups
```

Network:

```powershell id="p8q2m5"
ipconfig /all
```

Routes:

```powershell id="v5m9x3"
route print
```

Record:

```text id="c7m3q8"
Position:
P1

Host:
WS-01

Identity:
CORP\analyst

Network:
10.10.10.0/24
```

## Phase 2 — Identify SERVER-01

Assume SERVER-01 is an in-scope target:

```text id="r8m3p5"
SERVER-01
10.10.10.50
```

Check relevant services:

```powershell id="m4q8v2"
Test-NetConnection SERVER-01 -Port 445
```

```powershell id="x7p3n8"
Test-NetConnection SERVER-01 -Port 5985
```

```powershell id="q5m8r2"
Test-NetConnection SERVER-01 -Port 3389
```

The purpose is to determine which remote services are available from the current position.

## Phase 3 — Map Existing Access

Suppose the assessment establishes:

```text id="v3m8q7"
Current Identity:
CORP\analyst

Target:
SERVER-01

Available Services:
SMB
WinRM
RDP
```

The next question is:

> Which service provides the authorized access required by the assessment?

Assume WinRM is the selected path.

## Phase 4 — Validate Authentication and Authorization

Check WinRM:

```powershell id="m8q3v5"
Test-WSMan SERVER-01
```

Where authorized:

```powershell id="p7n4x2"
Enter-PSSession -ComputerName SERVER-01
```

Validate:

```powershell id="c5m8q3"
whoami
hostname
```

Record:

```text id="q8m3v7"
P1 → P2

Service:
WinRM

Authentication:
Successful

Authorization:
Remote PowerShell Session

New Host:
SERVER-01
```

## Phase 5 — Validate SERVER-01

Confirm:

```powershell id="x4m8p2"
hostname
```

```powershell id="r7q3n8"
whoami
```

```powershell id="v5m2c9"
whoami /groups
```

Network:

```powershell id="m8p3q7"
Get-NetIPConfiguration
```

Routes:

```powershell id="q4n7m2"
Get-NetRoute
```

The new position is now:

```text id="c6m8r3"
P2
SERVER-01
CORP\analyst
10.10.10.50
```

## Phase 6 — Enumerate SERVER-01

Check listening services:

```powershell id="x8m3q5"
Get-NetTCPConnection -State Listen
```

Running services:

```powershell id="p4m7n8"
Get-Service |
    Where-Object {$_.Status -eq "Running"}
```

Active connections:

```powershell id="r6m3x2"
Get-NetTCPConnection
```

Look for evidence of:

* internal applications
* databases
* file servers
* management systems
* domain infrastructure
* additional network segments

## Phase 7 — Discover the Second Network

Suppose SERVER-01 has:

```text id="n8m3q7"
Interface 1:
10.10.10.50

Interface 2:
10.10.20.50
```

Routes show:

```text id="v4p8m2"
10.10.10.0/24
10.10.20.0/24
```

The assessment has now identified:

```text id="m7q3x8"
NETWORK A
10.10.10.0/24

NETWORK B
10.10.20.0/24
```

The important difference is:

```text id="c5n8r2"
WS-01 → Network B
Not directly reachable

SERVER-01 → Network B
Reachable
```

SERVER-01 may therefore function as a pivot point.

## Phase 8 — Confirm Pivot Relevance

Before using SERVER-01 as a pivot, determine:

```text id="q8m3p7"
Is Network B in scope?
        ↓
Are there relevant targets?
        ↓
Can SERVER-01 reach those targets?
        ↓
Is pivoting necessary?
```

If the answer is yes, continue with the authorized pivot workflow.

See:

[`../09-Pivoting-and-Internal-Network-Access/`](../09-Pivoting-and-Internal-Network-Access/)

## Phase 9 — Identify Internal Targets

Suppose authorized discovery identifies:

```text id="x7m4q3"
FILE-01
10.10.20.15

LINUX-01
10.10.20.25
```

Build the candidate map:

| Target   | IP          | Service | Evidence           | Candidate Access           |
| -------- | ----------- | ------- | ------------------ | -------------------------- |
| FILE-01  | 10.10.20.15 | SMB     | Service discovered | Existing domain identity   |
| LINUX-01 | 10.10.20.25 | SSH     | Service discovered | SSH access to be validated |

Do not treat discovery as authorization.

## Phase 10 — Select FILE-01

Assume SMB is available:

```text id="m5q8p2"
FILE-01
10.10.20.15
TCP 445
```

From the pivoted position, validate the service path.

Conceptually:

```text id="v7m3x9"
WS-01
  ↓
SERVER-01
  ↓
NETWORK B
  ↓
FILE-01:445
```

## Phase 11 — Validate FILE-01 Access

Map the identity:

```text id="q4m8p7"
CORP\analyst
      ↓
Domain Authentication
      ↓
SMB
      ↓
FILE-01
```

Validate the authorized SMB access.

For example:

```cmd id="x8n3m5"
net view \\FILE-01
```

Determine the actual authorization:

```text id="p7m4q8"
Authentication:
Successful

Share Access:
Read / Write / Denied
```

Do not report administrative access unless it has been demonstrated.

## Phase 12 — Decide Whether FILE-01 Creates a New Position

There are two different outcomes.

### Resource-Level Access

```text id="m8q3v5"
FILE-01
   ↓
SMB Share
```

This may provide useful information without creating an interactive host session.

### Host-Level Access

If an authorized management service also provides a host-level session:

```text id="c4m7x2"
FILE-01
   ↓
Remote Session
   ↓
New Position
```

These outcomes must be reported separately.

## Phase 13 — Select LINUX-01

Assume the authorized environment contains:

```text id="r6m3q8"
LINUX-01
10.10.20.25
SSH
```

The current evidence indicates:

```text id="v8p3m5"
SERVER-01
      ↓
Network B
      ↓
LINUX-01:22
```

Before attempting access, determine whether valid SSH authentication material exists.

## Phase 14 — Reassess SSH Access

From the relevant authorized position, inspect known SSH relationships.

Potential sources include:

```text id="m4q8x2"
~/.ssh/config
~/.ssh/known_hosts
SSH Agent
Authorized Key Material
Known Account Relationships
```

For example:

```bash id="p7m3q8"
ls -la ~/.ssh/
```

Known hosts:

```bash id="x5m8r3"
cat ~/.ssh/known_hosts
```

Agent context:

```bash id="v3q8m2"
ssh-add -L
```

Only use authentication material that is within the authorized assessment scope.

## Phase 15 — Map SSH Access

Suppose an authorized SSH relationship is identified:

```text id="n8m3p7"
User:
dev

Target:
LINUX-01

Authentication:
SSH Key
```

The movement hypothesis becomes:

```text id="q4m8x2"
SSH Key
   ↓
dev
   ↓
LINUX-01
   ↓
SSH
```

Validate the specific target rather than attempting broad key reuse.

## Phase 16 — Validate LINUX-01

After authorized SSH authentication:

```bash id="m7p3q8"
hostname
```

```bash id="x4n8m2"
whoami
```

```bash id="v6m3r7"
id
```

Record:

```text id="c8q3m5"
P3
LINUX-01
User:
dev
```

This is now a new host-level position.

## Phase 17 — Re-enumerate LINUX-01

Interfaces:

```bash id="p5m8q2"
ip addr
```

Routes:

```bash id="r7q3n8"
ip route
```

Listening services:

```bash id="x8m4p2"
ss -lntup
```

Active connections:

```bash id="m3q8v7"
ss -tunap
```

Running services:

```bash id="c5n2r8"
systemctl list-units --type=service --state=running
```

The purpose is to determine whether P3 creates another useful position.

## Phase 18 — Compare All Positions

At this point:

```text id="q7m3x8"
P1
WS-01
10.10.10.25
        |
        | WinRM
        ↓
P2
SERVER-01
10.10.10.50
        |
        | Pivot
        ↓
NETWORK B
10.10.20.0/24
        |
        +------→ FILE-01
        |
        +------→ LINUX-01
```

The position ledger should show:

| Position | Host      | Identity     | Network       | Access           | New Visibility |
| -------- | --------- | ------------ | ------------- | ---------------- | -------------- |
| P1       | WS-01     | CORP\analyst | 10.10.10.0/24 | Initial foothold | Network A      |
| P2       | SERVER-01 | CORP\analyst | A + B         | WinRM            | Network B      |
| P3       | LINUX-01  | dev          | 10.10.20.0/24 | SSH              | To be assessed |

## Phase 19 — Reassess Credentials at P3

Now treat LINUX-01 as a fresh position.

Review relevant sources:

```text id="m8q3p7"
SSH Configuration
SSH Keys
SSH Agent
Application Configuration
Environment Variables
Service Accounts
Existing Sessions
```

For SSH:

```bash id="x5m8q3"
ls -la ~/.ssh/
```

For agent state:

```bash id="r4p7n2"
ssh-add -L
```

For active connections:

```bash id="v8m3q5"
ss -tunap
```

Do not assume that P3 has access to everything P2 had.

## Phase 20 — Identify Whether P3 Adds New Visibility

Suppose P3 reveals:

```text id="q6m3x8"
10.10.20.0/24
10.10.30.0/24
```

while P2 could only directly reach:

```text id="p4n8m2"
10.10.10.0/24
10.10.20.0/24
```

Now P3 may provide another pivot opportunity.

The chain becomes:

```text id="v7m3q8"
Network A
    ↓
SERVER-01
    ↓
Network B
    ↓
LINUX-01
    ↓
Network C
```

This is a meaningful change in network position.

## Phase 21 — Evaluate Network C

Before continuing:

```text id="m5q8r3"
Network C
10.10.30.0/24
```

Determine:

```text id="x4n7p2"
Is Network C in scope?
Are there relevant targets?
Can LINUX-01 reach it?
Is further movement necessary?
```

If there is no relevant authorized objective, stop.

If a relevant target exists, identify the target and service before moving.

## Phase 22 — Maintain the Chain

The complete position chain is now:

```text id="q8m3v7"
P1
WS-01
CORP\analyst
Network A
      |
      | WinRM
      ↓
P2
SERVER-01
CORP\analyst
Network A + B
      |
      | Pivot
      ↓
P3
LINUX-01
dev
Network B + C
```

Each transition has a distinct reason:

```text id="c5m8q2"
P1 → P2
Remote management access

P2 → Network B
New network visibility

Network B → P3
Authorized SSH access

P3 → Network C
New network visibility
```

This is the level of reasoning the movement chain should preserve.

## Phase 23 — Avoid Assuming the Longest Path Is Best

A multi-host chain should not continue merely because another system exists.

For example:

```text id="x7m3p8"
P3
 ↓
P4
 ↓
P5
 ↓
P6
```

may add no meaningful information.

Instead ask:

```text id="r4q8m3"
What does P4 provide?
```

If the answer is:

```text id="m7p3n8"
Nothing new
```

then there is no evidence-based reason to continue.

## Phase 24 — Avoid Circular Movement

The chain must track already validated paths.

Example:

```text id="v8m2q5"
P1 → P2 → P3
```

If P3 provides access back to P2:

```text id="c6m3r8"
P3 → P2
```

that does not automatically create a new position.

Record:

```text id="q7p4m2"
Path Already Validated
```

and continue only if the reverse path provides genuinely new access or information.

## Phase 25 — Credential Reuse Across Positions

Suppose the same domain identity appears at multiple positions:

```text id="x3m8p7"
CORP\analyst
```

Do not assume that every system will authorize that identity equally.

Instead:

```text id="m5q8r3"
Identity
   ↓
Target
   ↓
Service
   ↓
Authentication
   ↓
Authorization
```

The same credential may produce:

```text id="v7p3n8"
Host A → WinRM
Host B → SMB Read
Host C → Denied
```

This is valuable evidence about authorization boundaries.

## Phase 26 — Example Failure: Authentication Works, Authorization Fails

Suppose:

```text id="q4m8x3"
LINUX-01
   ↓
SSH
   ↓
Authentication ✓
   ↓
Shell ✓
```

but the account has no access to a required resource.

Record:

```text id="p8m3r7"
Authentication:
Successful

Authorization:
Insufficient for requested action
```

Do not convert this into an administrative-access claim.

## Phase 27 — Example Failure: Network Reachability Fails

Suppose P3 cannot directly reach Network C.

The decision becomes:

```text id="m7q4v8"
P3
 ↓
Network C
 ↓
Unreachable
```

Check:

```text id="x5n8p2"
Interface
Route
Gateway
Firewall
Segmentation
```

If another authorized pivot is available, evaluate it.

If not, stop that path.

## Phase 28 — Example Failure: Target Has No Useful Service

Suppose an in-scope target is reachable but provides no relevant service.

Record:

```text id="r3m8q5"
Target:
HOST-X

Reachability:
Confirmed

Relevant Service:
Not identified

Decision:
Do not continue this path
```

A reachable host is not automatically a useful movement target.

## Phase 29 — Final Objective

Assume the assessment objective is reached when:

```text id="v6m3p8"
Network C
   ↓
Target Application
   ↓
Authorized Access
```

At that point, further lateral movement may be unnecessary.

The correct endpoint is:

```text id="q8m4r2"
Objective Reached
      ↓
Validate Final Access
      ↓
Document Chain
      ↓
Stop
```

## Phase 30 — Final Chain Record

Record the complete sequence:

```text id="m5p8q3"
P1
WS-01
CORP\analyst
10.10.10.25
Initial Foothold

      ↓ WinRM

P2
SERVER-01
CORP\analyst
10.10.10.50
Network A + B

      ↓ Pivot

Network B
10.10.20.0/24

      ↓ SSH

P3
LINUX-01
dev
10.10.20.25
Network B + C

      ↓

Final Objective / Stop
```

For each transition, record:

```text id="x7m3q8"
Service:
Authentication:
Authorization:
Evidence:
New Position:
Reason for Continuing:
```

## Phase 31 — Full Position Ledger

| Position | Host      | Identity     | Network   | Access Method    | New Visibility | Reason for Next Step    |
| -------- | --------- | ------------ | --------- | ---------------- | -------------- | ----------------------- |
| P1       | WS-01     | CORP\analyst | Network A | Initial foothold | Network A      | Server identified       |
| P2       | SERVER-01 | CORP\analyst | A + B     | WinRM            | Network B      | Internal targets        |
| P3       | LINUX-01  | dev          | B + C     | SSH              | Network C      | New internal visibility |
| Final    |           |              |           |                  |                | Objective reached       |

## Phase 32 — Full Credential / Access Ledger

| Identity / Access | Context | Service | Target    | Authentication | Authorization | Result |
| ----------------- | ------- | ------- | --------- | -------------- | ------------- | ------ |
| CORP\analyst      | Domain  | WinRM   | SERVER-01 |                |               |        |
| CORP\analyst      | Domain  | SMB     | FILE-01   |                |               |        |
| dev               | Linux   | SSH     | LINUX-01  |                |               |        |

Only record validated results.

## Phase 33 — Full Network Map

```text id="p4m8q2"
NETWORK A
10.10.10.0/24
│
├── WS-01
│   10.10.10.25
│
└── SERVER-01
    10.10.10.50
          │
          │ Pivot
          ↓
NETWORK B
10.10.20.0/24
│
├── FILE-01
│   10.10.20.15
│
└── LINUX-01
    10.10.20.25
          │
          │ Potential Pivot
          ↓
NETWORK C
10.10.30.0/24
```

The map shows why each host may matter.

## Phase 34 — Complete Multi-Host Workflow

```text id="q7m3x8"
INITIAL FOOTHOLD
      ↓
POSITION ASSESSMENT
      ↓
TARGET DISCOVERY
      ↓
SERVICE DISCOVERY
      ↓
ACCESS / CREDENTIAL MAPPING
      ↓
AUTHENTICATION
      ↓
AUTHORIZATION
      ↓
MOVEMENT
      ↓
ACCESS VALIDATION
      ↓
NEW-HOST ENUMERATION
      ↓
CREDENTIAL REASSESSMENT
      ↓
NEW NETWORK DISCOVERY
      ↓
PIVOT IF REQUIRED
      ↓
NEW TARGET
      ↓
AUTHENTICATION
      ↓
AUTHORIZATION
      ↓
NEW POSITION
      ↓
REPEAT
      ↓
OBJECTIVE REACHED?
   ┌──┴──┐
  Yes    No
   ↓      ↓
 STOP   CONTINUE
```

## Phase 35 — Chain Quality Checklist

```text id="m8q3p7"
[ ] Initial position confirmed
[ ] Every target was in scope
[ ] Every transition had a documented reason
[ ] Authentication was separated from authorization
[ ] Every new host was validated
[ ] Every new host was re-enumerated
[ ] Credentials were mapped to specific contexts
[ ] Network changes were documented
[ ] Pivot requirements were justified
[ ] Previously validated paths were tracked
[ ] Circular movement was avoided
[ ] Unnecessary credential testing was avoided
[ ] Sensitive material was protected
[ ] Assessment objective remained clear
[ ] Final stopping condition was documented
```

## Phase 36 — Failure Recovery

If the current session is lost:

```text id="v3m8q5"
Lost Session
    ↓
Identify Last Valid Position
    ↓
Review Position Ledger
    ↓
Review Access Method
    ↓
Re-establish Authorized Access
    ↓
Continue From Last Known State
```

Do not restart the entire assessment blindly.

The position ledger should provide enough information to recover the movement chain.

## Phase 37 — When to Stop

Stop the multi-host chain when:

* the assessment objective is satisfied
* no new meaningful targets are identified
* further movement would add no useful evidence
* the next target is outside scope
* access cannot be justified
* additional movement would create unnecessary risk
* the final position already demonstrates the relevant security issue

The longest chain is not necessarily the most informative chain.

## What This Scenario Demonstrates

This scenario integrates the entire repository:

```text id="x5m8q3"
Foundations
    ↓
Position Assessment
    ↓
Target Discovery
    ↓
Credential / Access Discovery
    ↓
Remote Access Services
    ↓
Lateral Movement
    ↓
Credential-Based Movement
    ↓
Pivoting
    ↓
Post-Movement Workflow
    ↓
Repeated Reassessment
    ↓
Decision to Continue or Stop
```

The important skill is not memorizing every movement technique.

It is maintaining an accurate model of:

```text id="q8m3v7"
WHERE AM I?
WHO AM I?
WHAT CAN I REACH?
WHAT CAN I AUTHENTICATE TO?
WHAT AM I AUTHORIZED TO DO?
WHAT DID THE NEW POSITION REVEAL?
WHY SHOULD I MOVE AGAIN?
```

## Golden Rule

> Every movement step should create a justified new position, and every new position should be treated as a fresh source of evidence.

The complete chain is:

```text id="m7p3x8"
POSITION
   ↓
DISCOVER
   ↓
MAP ACCESS
   ↓
MOVE
   ↓
VALIDATE
   ↓
ENUMERATE
   ↓
REASSESS
   ↓
PIVOT IF NEEDED
   ↓
MOVE AGAIN
   ↓
STOP WHEN THE OBJECTIVE IS MET
```

This is the difference between executing isolated lateral-movement techniques and following a complete lateral-movement workflow.

## What Next?

Section 11 is now complete.

Continue to [`../12-Decision-Framework/README.md`](../12-Decision-Framework/README.md).

Section 12 will turn the repository's workflow into practical decision aids: a movement decision tree, a credential-to-target matrix, and a service-to-technique matrix.
