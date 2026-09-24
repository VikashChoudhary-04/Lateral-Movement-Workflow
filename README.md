# Lateral Movement Workflow

A practical, action-oriented workflow for understanding, enumerating, executing, and validating lateral movement across Linux, Windows, and Active Directory environments.

## Overview

Lateral movement is the process of using an existing foothold, account, credential, authentication material, or trusted access path to move from one system or security context to another within an environment.

This repository is designed as a **workflow guide**, not a command cheatsheet.

The goal is to answer:

> **Where am I, what access do I have, what can I reach, how can I move, what did I gain, and what should I do next?**

The emphasis is on **actions, observations, decisions, and next steps**.

---

## Core Philosophy

Every stage follows the same operational pattern:

```text
Understand
    ↓
Enumerate
    ↓
Observe
    ↓
Interpret
    ↓
Decide
    ↓
Act
    ↓
Validate
    ↓
Re-enumerate
```

Commands are included where useful, but commands are never the primary objective.

For every important action, the workflow should explain:

* What are we trying to discover?
* Why does it matter?
* What should we check?
* What does the result mean?
* What should we do if the result is useful?
* What should we do if the result is not useful?
* What should we investigate next?

---

## Lateral Movement vs Privilege Escalation

These two concepts are closely related but solve different problems.

### Privilege Escalation

Privilege escalation attempts to obtain a higher level of privilege on the current system or security context.

```text
HOST-A
  │
  ├── Low-privileged user
  │
  ▼
  Higher-privileged user
```

The host remains the same.

### Lateral Movement

Lateral movement attempts to use existing access to reach another system or security context.

```text
HOST-A
  │
  │ Existing access
  ▼
HOST-B
```

The destination changes.

In a real assessment, the two commonly form a repeating cycle:

```text
Compromise
    ↓
Enumerate
    ↓
Privilege Escalation
    ↓
Credential / Access Discovery
    ↓
Lateral Movement
    ↓
New Host
    ↓
Enumerate Again
    ↓
Privilege Escalation
    ↓
Lateral Movement
    ↓
...
```

---

## The End-to-End Workflow

The repository follows this lifecycle:

```text
                    Initial Foothold
                          │
                          ▼
                 Position Assessment
                          │
                          ▼
                   Target Discovery
                          │
                          ▼
             Credential / Access Discovery
                          │
                          ▼
                 Remote Service Discovery
                          │
                          ▼
                Movement Path Selection
                          │
                          ▼
                 Establish Remote Access
                          │
                          ▼
                  Validate New Position
                          │
                          ▼
                 Post-Movement Enumeration
                          │
                          ▼
              Credential / Access Discovery
                          │
                          ▼
                 Privilege Escalation
                          │
                          ▼
                    Next Movement
                          │
                          └───────────────► Repeat
```

Not every engagement follows every branch.

The purpose of the workflow is to provide a **decision framework** rather than force every situation into a fixed sequence.

---

## Repository Structure

```text
Lateral-Movement-Workflow/
│
├── README.md
│
├── 01-Foundations/
│   └── README.md
│
├── 02-First-5-Minutes/
│   └── README.md
│
├── 03-Position-Assessment/
│   ├── README.md
│   ├── Current-Host.md
│   ├── Current-User.md
│   ├── Network-Context.md
│   └── Trust-and-Domain-Context.md
│
├── 04-Target-Discovery/
│   ├── README.md
│   ├── Host-Discovery.md
│   ├── Network-Enumeration.md
│   ├── Service-Discovery.md
│   └── Target-Prioritization.md
│
├── 05-Credential-and-Access-Discovery/
│   ├── README.md
│   ├── Credentials.md
│   ├── Passwords.md
│   ├── Hashes.md
│   ├── Kerberos.md
│   ├── Tokens-and-Sessions.md
│   ├── SSH-Keys.md
│   └── Existing-Access.md
│
├── 06-Remote-Access-Services/
│   ├── README.md
│   ├── SMB.md
│   ├── WinRM.md
│   ├── RDP.md
│   ├── WMI.md
│   ├── RPC.md
│   └── SSH.md
│
├── 07-Lateral-Movement-Techniques/
│   ├── README.md
│   ├── Windows-to-Windows.md
│   ├── Linux-to-Linux.md
│   ├── Windows-to-Linux.md
│   ├── Linux-to-Windows.md
│   └── AD-Lateral-Movement.md
│
├── 08-Credential-Based-Movement/
│   ├── README.md
│   ├── Valid-Accounts.md
│   ├── Pass-the-Hash.md
│   ├── Pass-the-Ticket.md
│   └── Overpass-the-Hash.md
│
├── 09-Pivoting-and-Internal-Network-Access/
│   ├── README.md
│   ├── Pivoting-Concepts.md
│   ├── Port-Forwarding.md
│   ├── SOCKS-Proxies.md
│   └── Internal-Network-Access.md
│
├── 10-Post-Movement-Workflow/
│   ├── README.md
│   ├── Access-Validation.md
│   ├── New-Host-Enumeration.md
│   ├── Credential-Reassessment.md
│   └── Continue-the-Chain.md
│
├── 11-End-to-End-Scenarios/
│   ├── README.md
│   ├── Windows-Workstation-to-Server.md
│   ├── AD-Lateral-Movement.md
│   ├── Linux-SSH-Movement.md
│   └── Multi-Host-Chain.md
│
├── 12-Decision-Framework/
│   ├── README.md
│   ├── Movement-Decision-Tree.md
│   ├── Credential-to-Target-Matrix.md
│   └── Service-to-Technique-Matrix.md
│
├── 13-Detection-and-Defense/
│   ├── README.md
│   ├── Windows-Logging.md
│   ├── Authentication-Events.md
│   ├── Network-Detection.md
│   └── Defensive-Considerations.md
│
└── References/
    └── README.md
```

The structure is intentionally organized around the **lateral movement lifecycle**, rather than around individual tools.

---

## Workflow Sections

### `01-Foundations`

Establishes only the concepts required to understand and operate the workflow.

Topics include:

* Lateral movement fundamentals
* Lateral movement vs privilege escalation
* Lateral movement vs pivoting
* Hosts, users, credentials, and services
* Authentication and authorization
* Windows, Linux, and Active Directory context
* The lateral movement lifecycle

Theory is kept concise and tied directly to practical decisions.

---

### `02-First-5-Minutes`

The immediate workflow after obtaining a foothold.

Questions:

* Where am I?
* Who am I?
* What privileges do I have?
* What operating system am I on?
* What network am I connected to?
* What domain or workgroup am I part of?
* What networks can I reach?
* What useful connections or sessions already exist?

The objective is to establish the current position before attempting movement.

---

### `03-Position-Assessment`

Builds a detailed understanding of the current position.

```text
Current Host
    ↓
Current User
    ↓
Privileges
    ↓
Network Context
    ↓
Domain / Trust Context
    ↓
Existing Access
```

Every enumeration step should lead to an interpretation or next action.

---

### `04-Target-Discovery`

Answers:

> **Where can I potentially go?**

Covers:

* Host discovery
* Network discovery
* Service discovery
* Reachability
* Candidate targets
* Target prioritization

The goal is not to enumerate everything blindly, but to identify **actionable movement opportunities**.

---

### `05-Credential-and-Access-Discovery`

Answers:

> **What authentication or access material do I currently possess?**

Covers:

* Passwords
* Hashes
* Kerberos material
* Tokens and sessions
* SSH keys
* Service credentials
* Existing authenticated access

The focus is on understanding **what the discovered material can potentially provide**, not merely collecting it.

---

### `06-Remote-Access-Services`

Covers common remote access mechanisms:

* SMB
* WinRM
* RDP
* WMI
* RPC
* SSH

Each service follows a consistent workflow:

```text
Identify
    ↓
Enumerate
    ↓
Determine Requirements
    ↓
Validate Access
    ↓
Establish Access
    ↓
Confirm New Position
    ↓
Continue Post-Movement Workflow
```

---

### `07-Lateral-Movement-Techniques`

Connects remote services and available access to actual movement scenarios.

The section covers:

* Windows-to-Windows movement
* Linux-to-Linux movement
* Windows-to-Linux movement
* Linux-to-Windows movement
* Active Directory lateral movement

The emphasis is on **choosing an appropriate movement path based on observed conditions**.

---

### `08-Credential-Based-Movement`

Covers movement based on authentication material, including:

* Valid accounts
* Pass-the-Hash
* Pass-the-Ticket
* Overpass-the-Hash

Each technique explains:

```text
What material do I have?
        ↓
What authentication mechanism does it represent?
        ↓
Where could it potentially work?
        ↓
What prerequisites exist?
        ↓
How do I validate access?
        ↓
What happens after access?
```

---

### `09-Pivoting-and-Internal-Network-Access`

Separates pivoting from lateral movement while showing how pivoting can enable movement.

Covers:

* Pivoting concepts
* Port forwarding
* SOCKS proxies
* Internal network access
* Reaching otherwise inaccessible systems

---

### `10-Post-Movement-Workflow`

Reaching a new host is **not the end of the workflow**.

After movement:

```text
New Host
   ↓
Validate Access
   ↓
Identify User
   ↓
Identify Privileges
   ↓
Assess Network
   ↓
Discover Credentials / Access
   ↓
Identify New Targets
   ↓
Privilege Escalation if Required
   ↓
Continue the Chain
```

This section turns lateral movement into a continuous process rather than a one-time action.

---

### `11-End-to-End-Scenarios`

Combines individual techniques into realistic workflows.

Examples:

* Windows workstation → Windows server
* Active Directory lateral movement
* Linux → Linux through SSH
* Multi-host movement chain

These scenarios demonstrate how the individual pieces work together.

---

### `12-Decision-Framework`

Provides practical decision-making aids.

Examples:

```text
Credential / Access
        ↓
Possible Targets
        ↓
Available Services
        ↓
Required Authentication
        ↓
Possible Movement Paths
```

The goal is to answer:

> **"Given what I have discovered, what should I investigate or attempt next?"**

---

### `13-Detection-and-Defense`

Documents the defensive side of lateral movement.

Covers:

* Authentication events
* Windows logging
* Remote service activity
* Network indicators
* Credential misuse
* Detection opportunities
* Defensive considerations

The purpose is to understand the observable footprint of the workflow.

---

## Technique Documentation Standard

Every major technique should follow a consistent structure:

```text
1. Goal
2. Where It Fits
3. Prerequisites
4. What You Need
5. Discovery
6. Decision Point
7. Validation
8. Execution
9. Access Verification
10. Post-Movement Enumeration
11. Common Failure Conditions
12. Detection Considerations
13. Next Steps
```

This prevents individual pages from becoming isolated command references.

---

## Action-Oriented Documentation Rules

This repository follows several rules.

### 1. Start With the Objective

Every practical section should explain what we are trying to accomplish.

### 2. Show the Action

Provide the appropriate command, technique, or procedure.

### 3. Explain the Observation

Explain what output or behavior matters.

### 4. Interpret the Result

Explain what the observation tells us.

### 5. Provide the Decision

Explain what to investigate or attempt next.

### 6. Handle Failure

Explain common failure conditions and the appropriate branch in the workflow.

### 7. Close the Loop

After successful movement, always return to enumeration and reassessment.

---

## The Core Decision Loop

The central loop of this repository is:

```text
Where am I?
    ↓
What do I have?
    ↓
What can I reach?
    ↓
What services are available?
    ↓
What authentication can I use?
    ↓
Which movement paths are possible?
    ↓
Which path should I investigate?
    ↓
Can I establish access?
    ↓
Did I successfully reach the target?
    ↓
What changed?
    ↓
What can I discover from the new position?
    ↓
Repeat
```

This loop is more important than memorizing individual commands.

---

## Scope

This repository focuses on authorized security testing, lab environments, and defensive understanding of lateral movement.

The primary environments are:

* Linux
* Windows
* Active Directory
* Mixed enterprise environments

The repository focuses on:

* Enumeration
* Authentication
* Remote access
* Credential-based movement
* Remote services
* Pivoting
* Target selection
* Access validation
* Post-movement enumeration
* Detection and defense

---

## Learning Objective

By the end of this workflow, the reader should be able to approach an authorized lab or assessment with a structured process:

```text
Compromised Host
      ↓
Understand Position
      ↓
Discover Targets
      ↓
Discover Available Access
      ↓
Identify Remote Services
      ↓
Choose a Movement Path
      ↓
Validate Access
      ↓
Reach New Host
      ↓
Re-enumerate
      ↓
Continue the Chain
```

The objective is not to memorize commands.

The objective is to develop the ability to:

> **Observe → Interpret → Decide → Act → Validate → Repeat.**

---

## Related Workflow Repositories

This repository is part of a broader post-compromise workflow collection:

* Linux Privilege Escalation Workflow
* Windows Privilege Escalation Workflow
* Active Directory Privilege Escalation Workflow
* Lateral Movement Workflow

Together, these repositories provide complementary workflows for understanding what can happen **after an initial foothold has been obtained**.

---

## Disclaimer

This repository is intended for authorized security testing, controlled laboratories, education, and defensive research.

Only perform techniques against systems and accounts for which you have explicit authorization.
