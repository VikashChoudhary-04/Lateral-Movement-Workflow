# Continue the Chain

Lateral movement is not simply a sequence of compromised hosts.

Each successful movement creates a new position from which the assessment must be reassessed.

The decision to continue should therefore be evidence-driven:

```text id="k5m8q2"
New Position
     ↓
Validate Access
     ↓
Enumerate Host
     ↓
Reassess Credentials / Access
     ↓
Identify New Opportunities
     ↓
Evaluate Scope + Value + Risk
     ↓
Continue?
   ┌─┴─┐
  Yes  No
   ↓    ↓
Next   Stop / Report
Position
```

The objective is to determine whether the current position provides a legitimate and useful path to another in-scope target.

## Objective

After movement and reassessment, determine:

* what new targets are available
* what access can reach them
* whether the current identity can authenticate
* what authorization would be obtained
* whether the next host provides meaningful new visibility
* whether pivoting is required
* whether another movement step is justified
* when the chain should stop

The goal is not to maximize the number of hosts accessed.

The goal is to demonstrate the authorized movement path necessary to answer the assessment objective.

## 1. Start From the Current Position

Before choosing another target, summarize the current position.

```text id="a9r4m7"
Current Host:
Current User:
Privilege Level:
Domain:
Networks:
Routes:
Reachable Services:
Known Credentials:
Existing Access:
Potential Targets:
Pivot Capability:
Assessment Objective:
```

This becomes the starting state for the next decision.

## 2. Identify What Changed

Compare the current position with the previous one.

Ask:

```text id="q7m3v8"
What can I reach now?
What can I authenticate to now?
What networks are visible now?
What credentials are available now?
What services are available now?
What privileges are different now?
```

If the new position produces no meaningful change, continuing automatically may not provide additional value.

## 3. Build the Movement Chain

Maintain a simple chain:

```text id="x8p4n2"
Position 1
WORKSTATION-01
DOMAIN\user
10.10.10.25
      |
      | WinRM
      ↓
Position 2
SERVER-01
DOMAIN\user
10.10.10.50
      |
      | SMB
      ↓
Position 3
FILE-01
DOMAIN\user
10.10.20.10
```

For every transition, record:

```text id="m6q9r3"
Source
Target
Service
Authentication
Authorization
Access Type
Evidence
```

This prevents the movement path from becoming ambiguous.

## 4. Identify Candidate Targets

Candidate targets can come from:

* newly visible networks
* routing tables
* DNS
* active connections
* service discovery
* domain information
* existing authenticated access
* previously identified systems
* application dependencies

Prioritize targets based on evidence rather than scanning indiscriminately.

Use:

```text id="v2k7p5"
Current Position
      ↓
Known Network
      ↓
Known Host
      ↓
Known Service
      ↓
Known Authentication Path
```

The more evidence exists, the less unnecessary discovery is required.

## 5. Check Assessment Scope

Before attempting another movement step, verify that the target is in scope.

Use:

```text id="r5m8x2"
Target Identified
      ↓
In Scope?
   ┌──┴──┐
  Yes    No
   ↓      ↓
Continue Stop
```

Do not expand the movement chain into systems that were not authorized for the assessment.

## 6. Determine the Required Access

Ask what the next target actually requires.

Examples:

```text id="p8q3m6"
Target
  ↓
SMB
  ↓
Domain Authentication
  ↓
Share Authorization
```

or:

```text id="n4v7c2"
Target
  ↓
SSH
  ↓
User Authentication
  ↓
Shell Authorization
```

or:

```text id="x6m9r4"
Target
  ↓
WinRM
  ↓
Authentication
  ↓
Remote Management Authorization
```

Do not select a technique before understanding the target's access requirements.

## 7. Map Available Credentials

Use the credential-to-target relationship established during reassessment.

```text id="q3m8v5"
Credential
   ↓
Owner
   ↓
Protocol
   ↓
Target Service
   ↓
Target
```

For example:

```text id="z7p4n2"
DOMAIN\analyst
      ↓
Kerberos
      ↓
SMB
      ↓
FILE-02
```

This is stronger evidence than attempting unrelated credentials against the target.

## 8. Determine Reachability

Before authentication, confirm that the current position can communicate with the target.

Conceptually:

```text id="c8r2m6"
Current Host
     ↓
Route
     ↓
Target IP
     ↓
Target Port
```

If the route does not exist:

```text id="n5q7v3"
Current Position
      ↓
Cannot Reach Target
      ↓
Pivot Required?
```

If the target is reachable directly, avoid unnecessary pivoting.

## 9. Determine Whether Pivoting Is Required

A pivot becomes relevant when:

```text id="j4m8p2"
Target Network
      ↓
Not Directly Reachable
```

but:

```text id="v6q3r9"
Current Host
      ↓
Pivot Host
      ↓
Target Network
```

is possible.

Use the existing pivoting workflow:

[`09-Pivoting-and-Internal-Network-Access/`](../09-Pivoting-and-Internal-Network-Access/)

The pivot should provide a specific network-access purpose.

Do not introduce a pivot simply because one is technically possible.

## 10. Select the Appropriate Service

Once the target is reachable, determine which service provides the intended access.

Typical examples:

| Target Service  | Possible Access Type             |
| --------------- | -------------------------------- |
| SMB             | Resource / administrative access |
| WinRM           | Remote PowerShell                |
| RDP             | Interactive desktop              |
| WMI             | Remote management                |
| SSH             | Remote shell                     |
| Web Application | Application access               |
| Database        | Database access                  |

Service selection should follow the access objective.

## 11. Validate Authentication

Use the least intrusive appropriate validation.

```text id="q8m4v1"
Credential
    ↓
Target Service
    ↓
Authentication
```

Record:

```text id="t3n7p9"
Authentication:
Successful / Failed
```

If authentication fails, determine whether the failure is caused by:

* incorrect credential
* incorrect identity context
* unsupported authentication method
* target configuration
* network problem
* expired or disabled access

Do not immediately switch to unrelated credentials.

## 12. Validate Authorization

Authentication success is only the beginning.

Determine what the identity is permitted to do.

```text id="m7x2c5"
Authentication ✓
       ↓
Authorization?
    ┌──┴──┐
   Yes    No
    ↓      ↓
Continue  Record Boundary
```

Examples:

```text id="y4q8n3"
SMB → Share Access
WinRM → Remote Session
RDP → Interactive Logon
SSH → Shell Access
Database → Query Permission
Application → Authorized Function
```

## 13. Establish the Next Position

If the target provides a new host-level position:

```text id="r6m3v8"
Target Host
    ↓
Session
    ↓
Identity
    ↓
Host Validation
    ↓
Network Validation
```

Immediately perform the access-validation workflow.

See:

[`Access-Validation.md`](Access-Validation.md)

Do not treat the target as fully understood simply because a session was established.

## 14. Re-Enumerate Again

Every meaningful host-level movement should trigger:

```text id="p8n4m7"
Access Validation
      ↓
New-Host Enumeration
      ↓
Credential Reassessment
      ↓
Continue-the-Chain Decision
```

This creates a repeatable cycle:

```text id="w2q6v9"
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
MOVE
```

This is the core loop of the repository.

## 15. Determine Whether the New Position Is Better

A new position may be useful because it provides:

```text id="f5m8q2"
New Network
New Credentials
New Services
New Domain Visibility
New Authorization
New Application Access
New Pivot Capability
```

But a new host is not automatically a better position.

For example:

```text id="x7r3m9"
Position A:
10.10.10.0/24
```

```text id="q4p8v2"
Position B:
10.10.10.0/24
Same Identity
Same Services
No New Access
```

Position B may not provide a meaningful reason to continue.

## 16. Evaluate the Next Action

Use four questions:

### 1. Is there a new target?

```text
Yes → Continue analysis.
No  → Reassess current position.
```

### 2. Is there a valid access path?

```text
Yes → Validate it.
No  → Identify missing access requirements.
```

### 3. Is the target in scope?

```text
Yes → Continue.
No  → Stop that path.
```

### 4. Does the movement answer an assessment objective?

```text
Yes → Continue if necessary.
No  → Consider stopping.
```

## 17. Avoid Circular Movement

A common problem is repeatedly moving between the same systems.

Example:

```text id="m8q2v6"
HOST-A
  ↓
HOST-B
  ↓
HOST-A
  ↓
HOST-B
```

Maintain a movement ledger:

| Source | Target | Service | Identity | Result            |
| ------ | ------ | ------- | -------- | ----------------- |
| A      | B      | WinRM   | user     | Success           |
| B      | C      | SMB     | user     | Success           |
| C      | B      | SMB     | user     | Already validated |

If a path has already been demonstrated and provides no new information, do not repeat it unnecessarily.

## 18. Avoid Credential Chasing Without a Goal

Credential discovery can become an endless loop:

```text id="v3m7p9"
Host A
 ↓
Credential
 ↓
Host B
 ↓
Credential
 ↓
Host C
 ↓
Credential
```

Instead, maintain an assessment objective.

Ask:

> What specific new information or access does the next movement provide?

If there is no meaningful answer, reassess whether the movement is necessary.

## 19. Maintain Position Awareness

At all times, know:

```text id="n6q2r8"
Current Host
Current User
Current Network
Current Access
Previous Position
Available Pivot
Known Targets
```

A simple position ledger:

| Position | Host           | Identity    | Network                       | Access | Pivot |
| -------- | -------------- | ----------- | ----------------------------- | ------ | ----- |
| P1       | WORKSTATION-01 | DOMAIN\user | 10.10.10.0/24                 | Local  | No    |
| P2       | SERVER-01      | DOMAIN\user | 10.10.10.0/24 + 10.10.20.0/24 | WinRM  | Yes   |
| P3       | FILE-01        | DOMAIN\user | 10.10.20.0/24                 | SMB    | No    |

This provides a map of the assessment.

## 20. Maintain Evidence for Every Transition

Each transition should have a concise record:

```text id="c4m8x1"
Source Host:
Source Identity:
Target Host:
Target Service:
Authentication:
Authorization:
Access Obtained:
Evidence:
New Network:
New Credential / Access:
Next Decision:
```

Example:

```text id="k7p3m9"
Source Host: SERVER-01
Source Identity: DOMAIN\analyst
Target Host: FILE-01
Target Service: SMB
Authentication: Kerberos
Authorization: Read access to approved share
Access Obtained: SMB resource access
New Network: None
New Credential / Access: None
Next Decision: Continue share assessment; no new host position
```

## 21. Recognize Different Types of Movement

Not every successful movement produces a new shell.

### Host-Level Movement

```text id="a5q8m2"
Host A
  ↓
Host B
  ↓
New Shell
```

### Resource-Level Movement

```text id="v7n3p9"
Host A
  ↓
Host B
  ↓
SMB Share
```

### Application-Level Movement

```text id="x4m6q2"
Host A
  ↓
Internal Application
  ↓
Application Account
```

### Network-Level Movement

```text id="r8c3v5"
Host A
  ↓
Pivot
  ↓
New Internal Network
```

The next action depends on the type of access obtained.

## 22. When to Continue

Continuing the chain may be justified when the current position reveals:

```text id="n2m7q4"
An in-scope target
+
A plausible authentication path
+
A validated or testable authorization path
+
A meaningful assessment objective
```

For example:

```text id="j6p3v8"
New Host
  ↓
New Network
  ↓
Known Server
  ↓
Known Service
  ↓
Existing Valid Credential
  ↓
Authorized Access
```

This is a clear movement path.

## 23. When to Stop

Stop the movement chain when:

* the assessment objective has been met
* no meaningful in-scope targets remain
* available credentials provide no useful access
* the next action would exceed scope
* further movement creates unnecessary risk
* the current position already demonstrates the relevant security issue
* additional hosts would add no meaningful evidence

Stopping is part of a disciplined workflow.

## 24. Failure Handling

### Target Reachability Fails

Check:

```text id="z8m4q1"
Interface
Route
Firewall
Port
Segmentation
Pivot Requirement
```

### Authentication Fails

Check:

```text id="p5x7n3"
Identity
Credential Type
Protocol
Target Service
Credential Validity
```

### Authorization Fails

Record:

```text id="m4q8v2"
Authentication: Successful
Authorization: Denied
```

Then determine whether another authorized access path exists.

### New Host Provides No Useful Information

Compare:

```text id="c7n3p8"
Previous Position
vs.
New Position
```

If there is no meaningful difference, avoid continuing simply for the sake of adding another host.

### Movement Path Becomes Ambiguous

Return to the position ledger and record:

```text id="v2m6q9"
Last Known Valid Position
Last Successful Transition
Current Access
Current Identity
```

Then rebuild from the last confirmed state.

## 25. Complete Chain Decision Tree

```text id="h4p8m2"
Current Position
      ↓
Identify Candidate Target
      ↓
In Scope?
 ┌────┴────┐
No         Yes
↓           ↓
Stop      Reachable?
             │
          ┌──┴──┐
         No     Yes
         ↓       ↓
      Pivot?   Service?
                │
             ┌──┴──┐
            No     Yes
            ↓       ↓
         Reassess Authentication
                    │
                ┌───┴───┐
               Fail    Success
                ↓        ↓
             Troubleshoot Authorization
                           │
                       ┌───┴───┐
                      Denied  Allowed
                        ↓        ↓
                     Record   Establish Access
                                ↓
                         Validate New Position
                                ↓
                         Enumerate New Host
                                ↓
                       Reassess Credentials
                                ↓
                       Identify New Opportunities
                                ↓
                           Continue?
```

## 26. The Lateral-Movement Loop

The complete operational loop is:

```text id="p9m4x7"
┌─────────────────────────────┐
│      Current Position       │
└──────────────┬──────────────┘
               ↓
       Target Discovery
               ↓
       Service Discovery
               ↓
     Credential / Access Map
               ↓
       Reachability Check
               ↓
     Authentication Check
               ↓
      Authorization Check
               ↓
        Access Established
               ↓
       Access Validation
               ↓
      New-Host Enumeration
               ↓
    Credential Reassessment
               ↓
       New Target Analysis
               ↓
       ┌───────┴───────┐
       ↓               ↓
   Continue           Stop
       │               │
       └───────┐       │
               ↓       ↓
        New Position  Report
               │
               └──────────────→
```

## 27. Chain Quality Rules

A good lateral-movement chain should be:

### Evidence-Based

Every transition has a demonstrated reason.

### Scope-Aware

Every target is explicitly or implicitly covered by the assessment scope.

### Minimal

Use the smallest number of movement steps necessary to demonstrate the relevant path.

### Reproducible

Another tester should be able to understand how the chain was established.

### Traceable

Every transition identifies:

```text id="m7q2v8"
Source
Target
Service
Identity
Authentication
Authorization
Result
```

### Reversible Where Possible

Avoid unnecessary changes to systems merely to prove access.

## 28. Final Chain Record

At the end of the movement sequence, maintain a summary:

```text id="s5m8x2"
Movement Chain:

P1
Host:
Identity:
Access:

   ↓

P2
Host:
Identity:
Access:

   ↓

P3
Host:
Identity:
Access:

   ↓

Final Position
Host:
Identity:
Network:
Access:
Assessment Objective:
```

For each transition:

```text id="r3n7p9"
Service:
Authentication:
Authorization:
Evidence:
```

## 29. Stop-and-Report Checklist

Before ending the chain:

```text id="f8m2q5"
[ ] Current position recorded
[ ] Previous positions recorded
[ ] Every movement transition documented
[ ] Authentication results documented
[ ] Authorization results documented
[ ] New hosts validated
[ ] New networks documented
[ ] Relevant credentials mapped
[ ] Pivot paths documented
[ ] Scope boundaries respected
[ ] No unnecessary persistence introduced
[ ] No unnecessary destructive actions performed
[ ] Assessment objective addressed
[ ] Final position documented
```

## Golden Rule

> Move only when the current position provides a justified, in-scope path to a meaningful new position.

The complete lateral-movement cycle is:

```text id="q6v3m8"
DISCOVER
   ↓
MAP ACCESS
   ↓
VALIDATE
   ↓
MOVE
   ↓
VALIDATE AGAIN
   ↓
ENUMERATE
   ↓
REASSESS
   ↓
DECIDE
   ↓
MOVE AGAIN — OR STOP
```

This prevents lateral movement from becoming a collection of disconnected techniques.

Instead, every movement step becomes part of one continuous, evidence-driven workflow.

## What Next?

Section 10 is now complete.

Continue to [`11-End-to-End-Scenarios/README.md`](../11-End-to-End-Scenarios/README.md).

That section will turn the individual workflows into complete attack-path scenarios, showing how the discovery, credential, remote-access, movement, pivoting, and post-movement processes connect from beginning to end.
