# End-to-End Scenarios

This section connects the individual lateral-movement workflows into complete, realistic assessment paths.

The earlier sections explain individual decisions and techniques.

This section answers a different question:

> What does the entire lateral-movement process look like when all of those pieces are connected?

The scenarios are designed for authorized penetration tests, security assessments, and controlled labs.

They are intentionally workflow-oriented rather than command-dump walkthroughs.

## Purpose

An end-to-end scenario should demonstrate:

```text id="p7m3q9"
Initial Position
      ↓
Position Assessment
      ↓
Target Discovery
      ↓
Credential / Access Discovery
      ↓
Remote Service Identification
      ↓
Authentication
      ↓
Authorization
      ↓
Lateral Movement
      ↓
Access Validation
      ↓
New-Host Enumeration
      ↓
Credential Reassessment
      ↓
Next Movement Decision
      ↓
Final Objective / Stop
```

The individual techniques are useful only when they fit into this larger process.

## Scenario Philosophy

Each scenario follows the same principles:

### Start With a Position

The tester begins with a known or assumed foothold.

### Build Context

The tester determines:

* current host
* current user
* network position
* domain context
* available services
* existing access

### Discover Before Moving

Targets and authentication paths are identified before movement is attempted.

### Validate Every Transition

Authentication and authorization are treated separately.

### Re-enumerate After Movement

Every meaningful new host becomes a fresh assessment position.

### Stop With Evidence

The scenario ends when the objective is met or no meaningful authorized path remains.

## Standard Scenario Model

All scenarios can be represented as:

```text id="w8q4m2"
CURRENT POSITION
      ↓
WHAT DO I KNOW?
      ↓
WHAT CAN I REACH?
      ↓
WHAT ACCESS DO I HAVE?
      ↓
WHAT TARGET IS RELEVANT?
      ↓
WHAT SERVICE PROVIDES ACCESS?
      ↓
WHAT AUTHENTICATION MATERIAL FITS?
      ↓
IS AUTHENTICATION SUCCESSFUL?
      ↓
IS AUTHORIZATION SUCCESSFUL?
      ↓
WHAT ACCESS DID I ACTUALLY GET?
      ↓
WHAT DID THE NEW POSITION REVEAL?
      ↓
IS ANOTHER MOVEMENT STEP JUSTIFIED?
      ↓
CONTINUE OR STOP
```

## Scenario 1 — Windows Workstation to Server

File:

[`Windows-Workstation-to-Server.md`](Windows-Workstation-to-Server.md)

This scenario demonstrates movement between Windows systems.

Core path:

```text id="r3m7v9"
Windows Workstation
      ↓
Position Assessment
      ↓
Internal Target Discovery
      ↓
Remote Service Discovery
      ↓
Credential / Access Mapping
      ↓
Windows Server
      ↓
Access Validation
      ↓
New-Host Enumeration
      ↓
Continue / Stop
```

Topics demonstrated:

* Windows host enumeration
* identity context
* internal network discovery
* SMB
* WinRM
* RDP
* authentication vs authorization
* new-host enumeration
* credential reassessment

The scenario should not assume that successful authentication automatically means administrative access.

## Scenario 2 — Active Directory Lateral Movement

File:

[`AD-Lateral-Movement.md`](AD-Lateral-Movement.md)

This scenario demonstrates movement within an Active Directory environment.

Core path:

```text id="k4p8m2"
Domain-Joined Host
      ↓
Domain Identity
      ↓
Kerberos / Domain Context
      ↓
Target Discovery
      ↓
Service Identification
      ↓
Credential / Ticket Context
      ↓
Target Authentication
      ↓
Authorization
      ↓
New Domain Position
      ↓
Re-enumeration
```

Topics demonstrated:

* domain identity
* Kerberos
* domain controllers
* SMB
* WinRM
* RDP
* WMI / RPC
* domain target selection
* credential-to-target mapping
* post-movement enumeration

The scenario focuses on understanding the relationship between:

```text id="n6q3r8"
Identity
   ↓
Domain
   ↓
Authentication
   ↓
Service
   ↓
Authorization
   ↓
Target
```

## Scenario 3 — Linux SSH Movement

File:

[`Linux-SSH-Movement.md`](Linux-SSH-Movement.md)

This scenario demonstrates Linux-to-Linux movement using SSH.

Core path:

```text id="v7m3q9"
Linux Host
      ↓
Current User
      ↓
Network Enumeration
      ↓
SSH Discovery
      ↓
SSH Key / Credential Context
      ↓
Target Linux Host
      ↓
SSH Authentication
      ↓
Shell Validation
      ↓
Network Re-enumeration
      ↓
Continue / Stop
```

Topics demonstrated:

* Linux host enumeration
* routing
* SSH discovery
* SSH configuration
* SSH keys
* SSH agent context
* account mapping
* shell validation
* post-movement enumeration

The scenario emphasizes that:

```text id="p2x8m4"
SSH Authentication
      ≠
Root Access
```

The resulting privilege must be validated separately.

## Scenario 4 — Multi-Host Chain

File:

[`Multi-Host-Chain.md`](Multi-Host-Chain.md)

This scenario demonstrates a longer movement path involving multiple positions.

Example structure:

```text id="c8q3m7"
Initial Host
    ↓
Host B
    ↓
Host C
    ↓
Internal Network
    ↓
Host D
```

The purpose is not to maximize the number of hosts.

The purpose is to demonstrate how the workflow changes after each movement:

```text id="m5v8r2"
Move
 ↓
Validate
 ↓
Enumerate
 ↓
Reassess
 ↓
Discover
 ↓
Decide
 ↓
Move
```

Topics demonstrated:

* position tracking
* credential reassessment
* pivoting
* internal network access
* multiple authentication contexts
* movement-chain documentation
* avoiding circular movement
* deciding when to stop

## Scenario Structure

Every scenario should use a consistent structure.

### 1. Scenario Objective

State what the assessment is trying to demonstrate.

### 2. Starting Position

Document:

```text id="t4m8p2"
Host:
User:
Network:
Known Access:
Scope:
```

### 3. Initial Assessment

Determine:

```text id="q7n3v9"
Identity
Network
Services
Domain
Existing Access
```

### 4. Target Discovery

Identify:

```text id="x5m8r3"
Candidate Host
Candidate Service
Reason for Interest
Reachability
```

### 5. Access Mapping

Map:

```text id="j3p7q9"
Credential / Access
      ↓
Authentication Method
      ↓
Target Service
      ↓
Target
```

### 6. Authentication

Document the result:

```text id="v8m2c6"
Authentication:
Successful / Failed
```

### 7. Authorization

Determine what the authenticated identity can actually do.

```text id="p4q9n7"
Authorization:
Allowed / Denied
```

### 8. Movement

Establish the next position using the appropriate authorized service.

### 9. Access Validation

Confirm:

```text id="r6m3x8"
Host
Identity
Access Type
Privilege
Network
```

### 10. New-Host Enumeration

Repeat the assessment from the new position.

### 11. Credential Reassessment

Determine whether the new host provides additional authentication material or existing access.

### 12. Next Decision

Decide whether:

```text id="n8q4m2"
Continue
```

or:

```text id="c5v7p9"
Stop
```

based on evidence and assessment scope.

## Scenario Decision Framework

Use this model throughout the scenarios:

```text id="w3m8q2"
New Position
     ↓
New Information?
  ┌──┴──┐
 No     Yes
 ↓       ↓
Stop   New Target?
          │
       ┌──┴──┐
      No     Yes
      ↓       ↓
    Reassess Reachable?
               │
            ┌──┴──┐
           No     Yes
           ↓       ↓
        Pivot?   Access Path?
                    │
                 ┌──┴──┐
                No     Yes
                ↓       ↓
             Reassess Authenticate
                          │
                      ┌───┴───┐
                     No       Yes
                     ↓         ↓
                  Troubleshoot Authorization
                                  │
                              ┌───┴───┐
                             No       Yes
                             ↓         ↓
                           Stop      Move
                                      ↓
                               Validate Again
```

## Scenario Evidence Model

For every movement transition, record:

```text id="k7m3p8"
Source Host:
Source Identity:
Target Host:
Target IP:
Target Service:
Authentication Method:
Authentication Result:
Authorization Result:
Access Obtained:
New Network:
New Credentials:
New Targets:
Next Action:
```

This allows the entire chain to be reconstructed.

## Scenario Position Ledger

Maintain a position ledger:

| Position | Host | Identity | Network | Access | New Information |
| -------- | ---- | -------- | ------- | ------ | --------------- |
| P1       |      |          |         |        |                 |
| P2       |      |          |         |        |                 |
| P3       |      |          |         |        |                 |
| P4       |      |          |         |        |                 |

This is especially useful in multi-host scenarios.

## Scenario Credential Ledger

Track authentication material separately:

| Credential / Access | Owner | Type | Scope | Target | Authentication | Authorization |
| ------------------- | ----- | ---- | ----- | ------ | -------------- | ------------- |
|                     |       |      |       |        |                |               |
|                     |       |      |       |        |                |               |
|                     |       |      |       |        |                |               |

Do not place real secrets into a public repository.

Use placeholders such as:

```text id="e7m3q8"
<USERNAME>
<PASSWORD>
<DOMAIN>
<TOKEN>
<PRIVATE_KEY>
```

## Scenario Network Map

Represent network movement visually:

```text id="q4m8v2"
Network A
10.10.10.0/24
       |
       | Host B
       ↓
Network B
10.10.20.0/24
       |
       | Pivot
       ↓
Network C
10.10.30.0/24
```

Record which host provides access to each network.

## What Scenarios Should Demonstrate

A complete scenario should make the reader understand:

```text id="m7p3x9"
Why the target was selected
        ↓
Why the service was selected
        ↓
Why the credential / access was relevant
        ↓
How authentication was validated
        ↓
How authorization was validated
        ↓
What access was actually obtained
        ↓
What the new position revealed
        ↓
Why the next action was selected
        ↓
Why the chain eventually stopped
```

That reasoning is more important than the individual commands.

## Common Scenario Mistakes

### Moving Without Target Context

Avoid:

```text id="c8m4p7"
"Port 445 is open, so move."
```

Instead:

```text id="n3q7v2"
Target identified
   ↓
SMB identified
   ↓
Credential maps to SMB
   ↓
Authentication validated
   ↓
Authorization validated
   ↓
Movement justified
```

### Treating Credentials as Universal

A credential is not automatically valid everywhere.

Always map:

```text id="x5r8m3"
Credential
+
Context
+
Protocol
+
Target
```

### Treating Authentication as Full Access

Always separate:

```text id="v7p2q9"
Authentication
```

from:

```text id="m4n8c3"
Authorization
```

### Skipping Re-enumeration

A new host is a new position.

Always repeat the relevant enumeration.

### Continuing Without a Purpose

Every movement step should answer:

> What does this give us that we did not have before?

## Safety and Scope

All scenarios should be performed only against systems explicitly authorized for testing.

Keep demonstrations controlled and avoid:

* destructive actions
* unnecessary persistence
* broad credential spraying
* unnecessary credential exposure
* unauthorized target access
* indiscriminate scanning
* actions outside the assessment scope

The scenarios are intended to demonstrate methodology, not uncontrolled access.

## Relationship With Earlier Sections

The scenarios combine the earlier workflow stages:

```text id="j6q2m8"
01 Foundations
      ↓
02 First 5 Minutes
      ↓
03 Position Assessment
      ↓
04 Target Discovery
      ↓
05 Credential / Access Discovery
      ↓
06 Remote Access Services
      ↓
07 Movement Techniques
      ↓
08 Credential-Based Movement
      ↓
09 Pivoting
      ↓
10 Post-Movement Workflow
      ↓
11 End-to-End Scenarios
```

Section 11 therefore acts as the practical integration layer of the repository.

## Scenario Completion Criteria

A scenario is complete when the reader can identify:

```text id="r8m3q5"
[ ] Starting position
[ ] Assessment objective
[ ] Initial identity
[ ] Network context
[ ] Candidate targets
[ ] Target service
[ ] Relevant authentication material
[ ] Authentication result
[ ] Authorization result
[ ] Movement method
[ ] New position
[ ] New-host enumeration
[ ] Credential reassessment
[ ] Next movement decision
[ ] Final stopping condition
[ ] Evidence trail
```

## Golden Rule

> A complete lateral-movement scenario is not a list of commands. It is a traceable chain of decisions showing why each movement step was possible, what access it produced, what the new position revealed, and why the next step was or was not taken.

## What Next?

Continue to [`Windows-Workstation-to-Server.md`](Windows-Workstation-to-Server.md).

That scenario will apply the complete workflow to a Windows workstation-to-server movement path.
