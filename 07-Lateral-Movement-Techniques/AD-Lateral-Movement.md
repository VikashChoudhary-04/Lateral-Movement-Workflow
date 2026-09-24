# Active Directory Lateral Movement

Active Directory (AD) lateral movement occurs when access associated with a domain identity, computer, service, authentication mechanism, or trust relationship is used to establish authorized access to another Windows system in the domain environment.

AD does not create a single lateral-movement technique.

Instead, it creates an identity and trust environment in which movement can occur through services such as:

* SMB
* WinRM
* RDP
* WMI
* RPC
* Kerberos-based authentication
* existing domain sessions

The objective is:

> **Determine which domain identity and relationships are relevant to a target, identify a compatible remote service, validate authorization, establish access, and reassess the new domain position.**

---

## AD Movement Model

A practical AD movement workflow is:

```text id="6r5f8x"
Current Windows Position
        ↓
Current Domain Identity
        ↓
Domain / Trust Context
        ↓
Target Computer Discovery
        ↓
Authentication Context
        ↓
Remote Service Discovery
        ↓
Authorization
        ↓
Movement
        ↓
Validate New Host
        ↓
Re-enumerate Domain Position
        ↓
Continue the Chain
```

The important difference from simple Windows-to-Windows movement is that AD provides additional relationships between:

```text id="4d7k2m"
Users
Computers
Groups
Services
Domains
Trusts
Authentication
Policies
```

---

## Prerequisites

Before attempting AD lateral movement, establish:

| Requirement    | Question                                          |
| -------------- | ------------------------------------------------- |
| Domain         | Which AD domain am I operating in?                |
| Identity       | Which domain or local account am I using?         |
| Groups         | Which groups is the identity a member of?         |
| Target         | Which domain computer is relevant?                |
| Reachability   | Can I reach the target?                           |
| Service        | Which remote service is available?                |
| Authentication | Which domain authentication context is available? |
| Authorization  | Is the identity permitted to use the service?     |
| Objective      | Why is the target relevant?                       |

If these are unknown, return to Position Assessment and Target Discovery.

---

## Step 1 — Establish the Current Domain Position

Identify the current identity:

```cmd id="d8m5j3"
whoami
```

```cmd id="v7x4k2"
whoami /user
```

```cmd id="p9q3n6"
whoami /groups
```

Inspect privileges:

```cmd id="k4r8c1"
whoami /priv
```

Domain context:

```cmd id="a7m2w5"
echo %USERDOMAIN%
```

```cmd id="y6c9f3"
echo %USERDNSDOMAIN%
```

Identify the host:

```cmd id="b5n8q4"
hostname
```

Domain membership:

```powershell id="m3r7v9"
Get-CimInstance Win32_ComputerSystem | Select-Object Name,Domain,PartOfDomain
```

Record:

```text id="h4j7p2"
Host:
User:
Domain:
Groups:
Privileges:
Computer Role:
```

---

## Step 2 — Identify the Authentication Context

Determine how the current identity is authenticated.

Potential contexts include:

* Kerberos
* NTLM
* local authentication
* service authentication
* existing domain session

Inspect Kerberos tickets:

```cmd id="x8c5v1"
klist
```

The presence of tickets provides evidence about the current Kerberos context.

Important distinctions:

```text id="n7w3d5"
Domain Account
     ≠
Kerberos Ticket
     ≠
Administrative Access
```

A domain identity may exist without having authorization to access a particular target.

---

## Step 3 — Understand the Domain Environment

Identify the domain and relevant infrastructure.

Useful commands include:

```cmd id="q2m8j6"
systeminfo
```

```cmd id="f5c9r3"
nltest /dsgetdc:%USERDOMAIN%
```

Where authorized and appropriate:

```cmd id="u4v7p2"
nltest /domain_trusts
```

These can help establish:

* domain controller
* domain name
* domain relationships
* trust information

Trust information does not automatically provide access to trusted systems.

---

## Step 4 — Discover Candidate Computers

Potential AD targets may come from:

* existing domain knowledge
* DNS
* SMB
* previously discovered hosts
* authorized AD enumeration
* application infrastructure
* management systems
* domain computer information

A target record might look like:

```text id="e6n4r8"
Computer: SERVER01
Domain: CORP.LOCAL
IP: 10.10.20.15
Role: Application Server
Reason: Domain computer reachable from current host
```

The target should have a reason for investigation.

---

## Step 5 — Understand Computer Roles

Different AD computers have different movement relevance.

### Workstations

May provide:

* user sessions
* administrative relationships
* application access
* additional network visibility

### Member Servers

May provide:

* application infrastructure
* databases
* file services
* management interfaces
* additional network paths

### Domain Controllers

Provide critical domain infrastructure and should be treated as high-impact systems.

The fact that a system is a domain controller does not by itself establish that the current identity is authorized to administer it.

---

## Step 6 — Identify Remote Services

For each target, identify available Windows remote services.

Common services include:

| Service  |   Common Port | Movement Context                        |
| -------- | ------------: | --------------------------------------- |
| SMB      |           445 | File/resource and administrative access |
| WinRM    |     5985/5986 | Remote management                       |
| RDP      |          3389 | Interactive access                      |
| RPC      |           135 | Windows management infrastructure       |
| WMI/DCOM | RPC-dependent | Remote management                       |

Test specific services based on evidence.

For example:

```powershell id="v2n5q7"
Test-NetConnection SERVER01 -Port 445
```

```powershell id="m6j8r4"
Test-NetConnection SERVER01 -Port 5985
```

```powershell id="y5p3c9"
Test-NetConnection SERVER01 -Port 3389
```

```powershell id="k7d4w2"
Test-NetConnection SERVER01 -Port 135
```

---

## Step 7 — Map Domain Identity to Target

The central question is:

> **What relationship exists between my current domain identity and this target computer?**

Possible evidence includes:

* group membership
* delegated administration
* documented administrative role
* service permissions
* existing sessions
* target-specific access
* domain policy
* management-group membership

Do not assume:

```text id="b7q4m1"
Domain User
   ↓
Every Domain Computer
```

Instead:

```text id="x9k3p5"
Domain Identity
      ↓
Authorization Relationship
      ↓
Specific Target
      ↓
Specific Service
      ↓
Specific Operation
```

---

## Step 8 — Kerberos Context

Kerberos is central to many AD authentication workflows.

A simplified model is:

```text id="m8x5r2"
Domain Identity
      ↓
Kerberos Authentication
      ↓
Service Ticket
      ↓
Target Service
      ↓
Authorization
```

Common service classes include:

```text id="c5v7n3"
CIFS
HTTP
MSSQLSvc
HOST
```

The service must still authorize the authenticated identity.

A valid Kerberos ticket does not automatically provide administrative access to the target.

---

## Step 9 — SMB in AD

SMB is one of the most common Windows movement paths in domain environments.

Validate:

```powershell id="r4y7p9"
Test-NetConnection SERVER01 -Port 445
```

Where authorized:

```cmd id="n5x8c2"
net view \\SERVER01
```

Review existing SMB connections:

```cmd id="z6m3v1"
net use
```

A domain identity may authenticate to SMB using the environment's supported authentication mechanisms.

Record:

```text id="u8q5r4"
Target:
Domain:
Identity:
SMB:
Authentication:
Share Access:
Authorization:
```

Remember:

```text id="p6w2n8"
SMB Access
    ≠
Interactive Session
```

---

## Step 10 — WinRM in AD

WinRM can provide PowerShell-based remote management where the identity is authorized.

Validate:

```powershell id="j3v6q8"
Test-NetConnection SERVER01 -Port 5985
```

Then:

```powershell id="w5k8r2"
Test-WSMan SERVER01
```

Where authorized, validate remote management:

```powershell id="a4n7m3"
Enter-PSSession -ComputerName SERVER01
```

After successful access:

```powershell id="c8q2v5"
whoami
hostname
```

The resulting session must be treated as a new position.

---

## Step 11 — RDP in AD

RDP can provide interactive access to a domain-joined Windows system.

First validate:

```powershell id="y7m4q8"
Test-NetConnection SERVER01 -Port 3389
```

Then determine:

```text id="n2x6c5"
Domain authentication
       ↓
RDP authorization
       ↓
Interactive session
```

Authorization may depend on:

* Remote Desktop Users
* local administrators
* domain groups
* security policy
* Group Policy
* target configuration

Domain membership alone does not guarantee RDP access.

---

## Step 12 — WMI/RPC in AD

WMI and other Windows management interfaces may use RPC/DCOM.

Validate:

```powershell id="x4p7n2"
Test-NetConnection SERVER01 -Port 135
```

Then, where authorized, validate a controlled WMI query:

```powershell id="r8c3m6"
Get-CimInstance -ClassName Win32_ComputerSystem -ComputerName SERVER01
```

Interpret the result as evidence of:

```text id="j5w9q2"
RPC Reachability
      ↓
WMI Communication
      ↓
Authentication
      ↓
Authorization
```

Not:

```text id="k3m7x1"
TCP 135 open
      ↓
Administrative access
```

---

## Step 13 — Identify Existing Sessions

Existing domain sessions can explain why a target is relevant.

On the current Windows host:

```cmd id="s7n2v8"
query user
```

```cmd id="h5m3q9"
qwinsta
```

Network connections:

```powershell id="c6r8w4"
Get-NetTCPConnection
```

These findings may reveal:

* interactive users
* remote connections
* service connections
* management relationships

Do not automatically equate a session with possession of reusable credentials.

---

## Step 14 — Validate the New Domain Position

After successful movement, validate:

```cmd id="m4x7p1"
hostname
```

```cmd id="z8c5n3"
whoami
```

```cmd id="q6v2r9"
whoami /groups
```

```cmd id="f3k8m5"
whoami /priv
```

Domain:

```cmd id="n7j4w2"
echo %USERDOMAIN%
```

Network:

```cmd id="p5r9x3"
ipconfig /all
```

Routes:

```cmd id="v8m2c6"
route print
```

Kerberos context:

```cmd id="d4q7n1"
klist
```

Record the new position.

---

## Step 15 — Compare Domain Positions

Compare the original and new hosts.

| Property         | Source          | Target          |
| ---------------- | --------------- | --------------- |
| Host             | `WORKSTATION01` | `SERVER01`      |
| User             | `CORP\analyst`  | `CORP\analyst`  |
| Groups           | Documented      | Documented      |
| Privileges       | Documented      | Documented      |
| Network          | `10.10.10.0/24` | `10.10.20.0/24` |
| Domain           | `CORP.LOCAL`    | `CORP.LOCAL`    |
| Kerberos Context | Documented      | Documented      |
| New Targets      | Documented      | Documented      |

The identity may remain the same while the **position** changes significantly.

---

## Step 16 — Reassess Domain Visibility

From the new Windows host, reassess:

* domain context
* DNS
* reachable domain computers
* authentication infrastructure
* network routes
* available management services
* existing sessions
* authorized credentials/access

The movement may expose systems that were not reachable from the original host.

---

## Step 17 — Reassess Authentication Context

Check:

```cmd id="t6m8q2"
klist
```

Determine:

* whether Kerberos tickets are present
* which identity they correspond to
* whether they are current
* which service relationships are visible

Do not treat the mere existence of a ticket as proof that a target will accept the requested operation.

Authentication and authorization remain separate.

---

## Step 18 — Reassess Groups and Authorization

Check:

```cmd id="x2v5m7"
whoami /groups
```

This can show the security groups associated with the current token.

Interpret the result in context.

A group membership may indicate a possible authorization relationship, but the actual target service and policy still determine whether movement succeeds.

Use:

```text id="a9k4r6"
Identity
  ↓
Group
  ↓
Target
  ↓
Service
  ↓
Policy
  ↓
Authorization
```

---

## Step 19 — Update the AD Movement Record

Record each movement explicitly.

| Field                 | Result                    |
| --------------------- | ------------------------- |
| Source Host           | `WORKSTATION01`           |
| Source Identity       | `CORP\analyst`            |
| Domain                | `CORP.LOCAL`              |
| Target Host           | `SERVER01`                |
| Target Role           | Application Server        |
| Service               | WinRM                     |
| Authentication        | Kerberos                  |
| Authentication Result | Success                   |
| Authorization         | Remote management allowed |
| Resulting Access      | Remote PowerShell         |
| New Groups            | Documented                |
| New Network           | Documented                |
| New Targets           | Documented                |
| Next Action           | Re-enumerate              |

This becomes the evidence trail for the movement chain.

---

## AD Movement Decision Framework

When considering a target, answer:

```text id="b7m4q9"
1. What domain am I in?
        ↓
2. Who am I?
        ↓
3. What groups am I in?
        ↓
4. What authentication context do I have?
        ↓
5. Which computers can I reach?
        ↓
6. Which remote services are available?
        ↓
7. Which service matches my authentication?
        ↓
8. What authorization exists?
        ↓
9. Can I establish controlled access?
        ↓
10. What changed after movement?
```

This prevents AD movement from becoming a collection of unrelated techniques.

---

## Failure Classification

### Target unreachable

Investigate:

```text id="e3r8w5"
Routing
Network segmentation
DNS
Firewall
Target availability
```

### SMB unavailable

Investigate:

```text id="p6k2m9"
445
SMB service
Firewall
Target configuration
```

### WinRM unavailable

Investigate:

```text id="v7x3c1"
5985/5986
WinRM service
Firewall
Management policy
```

### RDP unavailable

Investigate:

```text id="n4q8j2"
3389
RDP service
Firewall
NLA
Remote Desktop policy
```

### RPC/WMI unavailable

Investigate:

```text id="y5m7r3"
135
Dynamic RPC
Firewall
DCOM
WMI
```

### Authentication failure

Investigate:

```text id="c8p4x6"
Identity
Credential
Kerberos
NTLM
DNS
Time synchronization
Authentication policy
```

### Authorization failure

Investigate:

```text id="f2k7m5"
Groups
Delegation
Remote-access policy
Local policy
Domain policy
Service permissions
```

---

## Kerberos-Specific Troubleshooting

When Kerberos-based movement behaves unexpectedly, investigate:

### DNS

Kerberos depends heavily on correct domain naming and service discovery.

### Time

Significant clock differences can interfere with Kerberos authentication.

### Identity

Confirm:

```cmd id="q3n6v8"
whoami
```

### Tickets

Check:

```cmd id="m7x2c4"
klist
```

### Target Service

Confirm that the target service corresponds to the intended authentication path.

The troubleshooting chain is:

```text id="d5r8p2"
Identity
 ↓
Domain
 ↓
DNS
 ↓
Time
 ↓
Kerberos Ticket
 ↓
Service
 ↓
Authorization
```

---

## Trust Relationships

AD environments can contain trusts between domains.

Conceptually:

```text id="w4n7q2"
Domain A
   │
   │ Trust
   ▼
Domain B
```

A trust can create an authentication relationship.

It does not automatically create:

```text id="c9m3x5"
Universal Administrative Access
```

For any trusted-domain movement path, determine:

* which identity is being used
* which domain owns the identity
* which target belongs to which domain
* which authentication mechanism applies
* what authorization exists
* whether the specific service accepts the identity

---

## Multi-Domain Movement

A multi-domain environment can look like:

```text id="j8q5v3"
Domain A
   │
   │ Authorized Trust
   ▼
Domain B
   │
   ├── Server B1
   ├── Server B2
   └── Workstation B1
```

The movement workflow remains:

```text id="k6r2m8"
Identity
   ↓
Trust
   ↓
Target
   ↓
Service
   ↓
Authentication
   ↓
Authorization
   ↓
Access
```

Do not skip the target-specific authorization step.

---

## Example AD Movement Chain

A generic chain may look like:

```text id="w5n3c8"
DOMAIN WORKSTATION
        │
        │ Domain identity
        │ WinRM
        ▼
MEMBER SERVER
        │
        ├── New subnet
        ├── Additional domain visibility
        ├── New management services
        │
        │ SMB / WinRM
        ▼
ANOTHER SERVER
```

The important pattern is:

```text id="v4q7m1"
Move
 ↓
Validate
 ↓
Reassess domain context
 ↓
Discover targets
 ↓
Map authentication
 ↓
Validate authorization
 ↓
Move again
```

---

## Decision Tree

```text id="r7m4x2"
Start on Domain-Joined Host
        │
        ↓
Identify domain identity
        │
        ↓
Identify groups and authentication context
        │
        ↓
Discover candidate domain computers
        │
        ↓
Which remote services are available?
        │
   ┌────┼─────┬─────┐
  SMB  WinRM  RDP  RPC/WMI
   │     │     │      │
   └─────┴─────┴──────┘
              │
              ↓
Does current authentication match
the selected service?
              │
         ┌────┴────┐
        No        Yes
        │           │
 Reassess auth      ↓
 context        Does authentication
                succeed?
                    │
               ┌────┴────┐
              No        Yes
              │           │
       Investigate       ↓
       Kerberos/      Is the operation
       NTLM/identity authorized?
                           │
                      ┌────┴────┐
                     No        Yes
                     │           │
                Investigate      ↓
                   authz     Establish access
                                  │
                                  ↓
                            Validate target
                                  │
                                  ↓
                       Reassess domain position
                                  │
                                  ↓
                            Continue chain
```

---

## Common Mistakes

### Assuming domain membership equals broad access

Being a domain user does not mean that every domain computer is accessible.

### Treating Kerberos tickets as administrative privileges

Authentication material and authorization are separate.

### Ignoring service compatibility

A domain identity may be accepted by one service while being restricted from another.

### Assuming a trust grants access

A trust establishes an authentication relationship, not universal authorization.

### Ignoring DNS and time

Kerberos-dependent workflows can fail because of environmental issues rather than invalid credentials.

### Treating TCP 135 as WMI access

RPC reachability is only one layer of the WMI path.

### Ignoring target role

Workstations, member servers, application servers, and domain controllers have different security and operational significance.

### Stopping after the first domain movement

Every successful movement creates a new domain and network position that should be reassessed.

### Testing domain credentials indiscriminately

Use evidence-based, target-specific validation.

### Losing the movement chain

Record:

```text id="x7q4m2"
Source
 ↓
Identity
 ↓
Target
 ↓
Service
 ↓
Authentication
 ↓
Authorization
 ↓
Result
 ↓
New Position
```

---

## Completion Criteria

AD lateral movement investigation is complete when you can document:

* source host
* source domain
* source identity
* source groups
* authentication context
* target computer
* target role
* target domain
* network reachability
* available remote services
* selected movement path
* authentication result
* authorization result
* resulting access
* new host
* new identity
* new privileges
* new network position
* new domain visibility
* newly discovered targets
* next movement path

If movement fails, classify the failure and return to the appropriate discovery layer.

---

## What Next?

The cross-platform lateral-movement technique section is now complete.

Continue with:

**[`08-Credential-Based-Movement/README.md`](../08-Credential-Based-Movement/README.md)**

The next section focuses specifically on movement where the primary enabling factor is an authentication artifact or identity, including:

* valid accounts
* pass-the-hash
* pass-the-ticket
* overpass-the-hash

The emphasis remains on understanding **what authentication material exists, which service can use it, what authorization applies, and how to validate the resulting position**.
