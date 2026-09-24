# Decision Framework

This section provides compact decision frameworks for choosing the next lateral-movement action.

The earlier sections explain the workflow in detail.

This section compresses that reasoning into practical decision aids that can be used during an assessment without replacing the underlying workflow.

The objective is not to memorize techniques.

The objective is to answer:

> Given my current position, what evidence do I have, what access is available, what target is relevant, and what should I validate next?

## Purpose

The decision framework connects:

```text id="p7m3q9"
Current Position
      ↓
Target
      ↓
Service
      ↓
Authentication Material
      ↓
Reachability
      ↓
Authentication
      ↓
Authorization
      ↓
Access
      ↓
New Position
      ↓
Next Decision
```

Use the framework when the assessment reaches a decision point.

## Core Decision Model

Every movement decision should answer these questions:

```text id="m4x8q2"
1. Where am I?
2. Who am I?
3. What can I reach?
4. What targets are relevant?
5. What services do those targets expose?
6. What authentication material do I have?
7. Does the material match the service?
8. Is the target in scope?
9. Can I authenticate?
10. What am I authorized to do?
11. What access does that create?
12. What does the new position reveal?
13. Should I continue?
```

If one of these questions cannot be answered, gather the missing evidence before moving.

## Position Model

Maintain a current-position record:

```text id="v8n3p6"
Host:
User:
Privilege:
Network:
Routes:
Domain:
Services:
Credentials / Access:
Reachable Targets:
Pivot Capability:
Assessment Objective:
```

The current position changes after every meaningful movement.

## Target Model

Every candidate target should be described using:

```text id="q5m7r2"
Target:
IP:
Network:
Host Role:
Service:
Port:
Reachability:
Authentication Requirement:
Authorization Requirement:
Known Credential / Access:
Assessment Relevance:
```

This prevents target selection from becoming guesswork.

## Access Model

Use:

```text id="x3p8m4"
Credential / Access
       ↓
Authentication Type
       ↓
Compatible Service
       ↓
Target
       ↓
Authentication
       ↓
Authorization
       ↓
Effective Access
```

The important distinction is:

```text id="r7m3v8"
Credential
   ≠
Authentication
   ≠
Authorization
   ≠
Access
```

## Authentication Decision

When authentication fails, classify the failure.

```text id="m8q3p6"
Connection?
   ↓
Service?
   ↓
Authentication?
   ↓
Authorization?
```

### Network Failure

```text
Cannot reach target
```

Investigate:

* route
* interface
* firewall
* segmentation
* pivot requirement

### Service Failure

```text
Target reachable
Service unavailable
```

Investigate:

* service state
* port
* protocol
* target configuration

### Authentication Failure

```text
Service reachable
Authentication rejected
```

Investigate:

* username
* credential
* authentication protocol
* account context
* credential validity

### Authorization Failure

```text
Authentication successful
Requested action denied
```

Investigate:

* group membership
* service permissions
* resource permissions
* logon rights
* application role

Do not treat these failure types as interchangeable.

## Continue vs Stop

Use this model:

```text id="p4m8q2"
New Evidence?
   ┌──┴──┐
  No     Yes
  ↓       ↓
Stop   New Target?
          │
       ┌──┴──┐
      No     Yes
      ↓       ↓
   Reassess  In Scope?
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
                       Access Path?
                           │
                       Authenticate
                           │
                       Authorize
                           │
                       Move / Validate
```

## Evidence-Based Movement

Prefer:

```text id="n7q3m8"
Known Target
+
Known Service
+
Known Authentication Context
+
Known Reachability
+
In Scope
```

over:

```text id="x5m8r2"
Open Port
+
Guess
+
Try Random Credential
```

The first approach produces a traceable assessment path.

## Credential-to-Target Reasoning

For every credential:

```text id="c8m3q7"
Credential
      ↓
Owner
      ↓
Scope
      ↓
Authentication Type
      ↓
Compatible Services
      ↓
Known Targets
      ↓
Reachability
      ↓
Authorization
```

A credential should not be considered a movement opportunity until its context is understood.

## Service Selection

Choose the service based on the desired access.

| Desired Access              | Relevant Service          |
| --------------------------- | ------------------------- |
| File / resource access      | SMB                       |
| Remote PowerShell           | WinRM                     |
| Interactive Windows desktop | RDP                       |
| Remote Windows management   | WMI / RPC                 |
| Linux shell                 | SSH                       |
| Application access          | Web / application service |
| Database access             | Database service          |

The presence of a service does not prove that the current identity is authorized to use it.

## New-Host Decision

After movement:

```text id="m4q8p3"
New Host
   ↓
Confirm Host
   ↓
Confirm User
   ↓
Confirm Privilege
   ↓
Enumerate Network
   ↓
Enumerate Services
   ↓
Reassess Credentials
   ↓
Identify New Targets
```

Never skip the validation stage simply because the connection succeeded.

## Pivot Decision

Consider pivoting when:

```text id="v8m3q5"
Target Network
       ↓
Not Reachable From Current Position
       ↓
Reachable From Another Authorized Host
```

Then:

```text id="q7p3n8"
Pivot Host
   ↓
Target Network
   ↓
Target Service
   ↓
Target Access
```

Pivoting should solve a specific network-access problem.

## Movement Chain Model

Maintain:

```text id="x4m8p7"
P1
 ↓
P2
 ↓
P3
 ↓
P4
```

For every transition:

```text id="m7q3v8"
Source:
Target:
Service:
Authentication:
Authorization:
Access:
New Information:
Reason for Next Step:
```

This prevents circular movement and unclear chains.

## Practical Decision Questions

When uncertain, ask these in order:

### Position

> Where am I?

### Identity

> Who am I?

### Network

> What networks can I reach?

### Target

> Which in-scope system is relevant?

### Service

> Which service provides the access I need?

### Authentication

> What authentication material fits that service?

### Authorization

> What will this identity actually be allowed to do?

### Validation

> What access did I actually obtain?

### Reassessment

> What changed because of this movement?

### Continuation

> Is there a justified next step?

## Decision Framework Files

This section contains three detailed decision aids:

### Movement Decision Tree

[`Movement-Decision-Tree.md`](Movement-Decision-Tree.md)

Provides a step-by-step decision tree for moving from the current position to the next validated position.

### Credential-to-Target Matrix

[`Credential-to-Target-Matrix.md`](Credential-to-Target-Matrix.md)

Maps authentication material and identities to compatible services and target types.

### Service-to-Technique Matrix

[`Service-to-Technique-Matrix.md`](Service-to-Technique-Matrix.md)

Maps discovered remote services to relevant access and movement workflows.

## Quick Decision Loop

During an assessment, use:

```text id="p8m3q6"
WHERE AM I?
      ↓
WHO AM I?
      ↓
WHAT CAN I REACH?
      ↓
WHAT TARGET MATTERS?
      ↓
WHAT SERVICE IS AVAILABLE?
      ↓
WHAT AUTHENTICATION MATERIAL FITS?
      ↓
CAN I AUTHENTICATE?
      ↓
WHAT AM I AUTHORIZED TO DO?
      ↓
WHAT ACCESS DID I GET?
      ↓
WHAT DID THE NEW POSITION REVEAL?
      ↓
CONTINUE OR STOP?
```

## Golden Rule

> Choose the next action from evidence already collected, not from a list of techniques you want to try.

The decision framework exists to keep lateral movement:

```text id="r5m8x2"
Evidence-Based
Scope-Aware
Targeted
Traceable
Repeatable
```

rather than becoming uncontrolled credential or service testing.

## What Next?

Continue to [`Movement-Decision-Tree.md`](Movement-Decision-Tree.md).

That file expands this section's reasoning into a detailed step-by-step decision tree that can be followed during an active assessment.
