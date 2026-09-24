# Movement Decision Tree

The purpose of this decision tree is to determine the next justified lateral-movement action from the evidence already collected.

It is not a list of commands to try randomly.

The workflow is:

```text
Current Position
      ↓
Identity
      ↓
Network Visibility
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
Movement
      ↓
Validation
      ↓
Re-enumeration
      ↓
Next Decision
```

Use the tree after gaining an initial foothold and again after every successful movement.

## 1. Confirm the Current Position

Before choosing a target, establish where you currently are.

Record:

```text
Host:
IP Address:
Hostname:
User:
Privilege:
OS:
Network:
Routes:
Domain / Workgroup:
Available Credentials:
Existing Access:
```

### Decision

```text
Do I know the current host and identity?
        │
   ┌────┴────┐
  No         Yes
  ↓           ↓
Enumerate   Continue
position
```

Do not begin target selection while the current position is unclear.

## 2. Identify the Current Identity

Determine whether the current identity is:

* local user
* domain user
* service account
* privileged account
* standard account
* another application or service identity

On Windows:

```powershell
whoami
whoami /user
whoami /groups
whoami /priv
```

Useful domain context:

```powershell
$env:USERDOMAIN
$env:USERDNSDOMAIN
$env:LOGONSERVER
```

On Linux:

```bash
whoami
id
groups
```

### Decision

```text
Identity known?
   │
 ┌─┴─┐
No  Yes
│    │
↓    ↓
Enum  Continue
```

The identity determines which authentication and authorization paths are relevant.

## 3. Determine Network Visibility

Identify interfaces, routes, DNS context, and reachable networks.

Windows:

```powershell
ipconfig /all
route print
Get-NetIPConfiguration
Get-NetRoute
```

Linux:

```bash
ip addr
ip route
cat /etc/resolv.conf
```

### Decision

```text
Can the target network be reached directly?
              │
        ┌─────┴─────┐
       Yes           No
        ↓             ↓
 Continue        Identify Pivot
```

If the target is not directly reachable, determine whether an authorized pivot path exists.

## 4. Identify Candidate Targets

Build targets from evidence.

Possible sources include:

* routing information
* DNS
* ARP / neighbor information
* domain information
* existing connections
* configuration files
* known hosts
* service discovery
* application configuration
* previously identified systems

Do not treat every discovered host as equally relevant.

Prioritize targets based on:

| Factor         | Question                                         |
| -------------- | ------------------------------------------------ |
| Relevance      | Does the target matter to the assessment?        |
| Reachability   | Can the current position reach it?               |
| Service        | Does it expose a usable service?                 |
| Authentication | Do I have compatible authentication material?    |
| Authorization  | Could the identity have useful access?           |
| New visibility | Could movement reveal another network or system? |

## 5. Identify the Target Service

Once a target is selected, identify the service that could provide the desired access.

Common examples:

| Service |        Typical Port | Possible Access                                |
| ------- | ------------------: | ---------------------------------------------- |
| SMB     |                 445 | File/resource access, administrative workflows |
| WinRM   |           5985/5986 | Remote PowerShell                              |
| RDP     |                3389 | Interactive Windows session                    |
| SSH     |                  22 | Remote shell                                   |
| RPC     | 135 + dynamic ports | Windows management dependencies                |
| WMI     |            RPC/DCOM | Remote Windows management                      |

Ports are indicators, not proof of usable access.

The sequence remains:

```text
Port
 ↓
Service
 ↓
Authentication
 ↓
Authorization
 ↓
Access
```

## 6. Test Reachability

Before troubleshooting credentials, determine whether the service is reachable.

Windows:

```powershell
Test-NetConnection TARGET -Port PORT
```

Linux:

```bash
nc -vz TARGET PORT
```

### Decision

```text
Service reachable?
      │
 ┌────┴────┐
 No        Yes
 ↓          ↓
Network /   Test
service     authentication
problem
```

If reachability fails, investigate:

* routing
* firewall
* segmentation
* incorrect target
* incorrect port
* service availability
* pivot requirement

Do not classify a network failure as a credential failure.

## 7. Map Authentication Material

Identify what authentication material is available.

Examples:

```text
Username + Password
NTLM Hash
Kerberos Ticket
SSH Private Key
SSH Agent Identity
Existing Session
Application Credential
Service Credential
```

Then determine:

```text
Who does it belong to?
Where is it valid?
What authentication protocol does it support?
Which services can use it?
```

## 8. Match Credential to Service

Use the following reasoning:

```text
Credential
    ↓
Authentication Protocol
    ↓
Compatible Service
    ↓
Target
```

Examples:

```text
SSH Key
   ↓
SSH
   ↓
Linux Target
```

```text
Domain Kerberos Context
   ↓
Kerberos-Aware Service
   ↓
Domain Target
```

```text
NTLM Authentication Material
   ↓
NTLM-Compatible Service
   ↓
Windows Target
```

The presence of authentication material does not prove that it is valid for the selected target.

## 9. Validate Authentication

Attempt authentication only when:

* the target is in scope
* the service is identified
* the authentication material is understood
* the attempt is appropriate for the assessment

### Decision

```text
Authentication successful?
          │
     ┌────┴────┐
    No         Yes
    ↓           ↓
Classify      Check
failure       authorization
```

### Authentication Failure

Check:

```text
Username
Credential validity
Authentication protocol
Account context
Target service
Domain / realm
Kerberos time
DNS
Account state
```

Avoid immediately switching to unrelated credentials or broad guessing.

## 10. Determine Authorization

Successful authentication is not equivalent to useful access.

Ask:

```text
What can this identity actually do?
```

Examples:

```text
Authenticated to SMB
      ↓
Can list shares?
      ↓
Can read?
      ↓
Can write?
      ↓
Administrative access?
```

```text
Authenticated to WinRM
      ↓
Is remote logon authorized?
      ↓
What session privileges exist?
```

```text
Authenticated to SSH
      ↓
Shell obtained?
      ↓
Which user?
      ↓
Which privileges?
```

### Decision

```text
Useful authorization?
       │
  ┌────┴────┐
 No         Yes
 ↓           ↓
Reassess    Establish
access      movement
path
```

## 11. Establish Movement

Select the access path that matches the validated authorization.

Examples:

```text
SMB
 ↓
Resource / administrative access
```

```text
WinRM
 ↓
Remote PowerShell
```

```text
RDP
 ↓
Interactive Windows session
```

```text
SSH
 ↓
Remote shell
```

```text
WMI / RPC
 ↓
Remote Windows management
```

The exact technique depends on the authorized assessment objective and available access.

## 12. Validate the New Position

After movement, prove what changed.

### Windows

```powershell
whoami
hostname
ipconfig
whoami /groups
whoami /priv
```

### Linux

```bash
whoami
id
hostname
ip addr
ip route
```

Record:

```text
New Host:
New User:
Privilege:
Network:
Domain:
Access Method:
```

### Decision

```text
Did I actually gain a new position?
          │
     ┌────┴────┐
    No         Yes
    ↓           ↓
Troubleshoot  Re-enumerate
```

A successful connection is not enough.

## 13. Re-enumerate the New Host

Treat the new host as a new assessment position.

Check:

```text
Host
 ↓
User
 ↓
Network
 ↓
Routes
 ↓
Services
 ↓
Connections
 ↓
Domain
 ↓
Credentials / Access
 ↓
New Targets
```

The new host may reveal:

* another network interface
* additional routes
* internal-only services
* different domain context
* new credentials or keys
* different remote services
* additional target systems

## 14. Compare the Old and New Positions

Use:

| Property          | Previous Position | New Position |
| ----------------- | ----------------- | ------------ |
| Host              |                   |              |
| User              |                   |              |
| Privilege         |                   |              |
| Network           |                   |              |
| Routes            |                   |              |
| Domain            |                   |              |
| Services          |                   |              |
| Credentials       |                   |              |
| Reachable Targets |                   |              |

Ask:

> What can I see or access now that I could not before?

That difference is often the most important result of lateral movement.

## 15. Decide Whether to Pivot

If the new host can reach a network that the original host could not:

```text
New Host
   ↓
New Network Visibility
   ↓
Internal Target
   ↓
Need Pivot?
```

Possible mechanisms include:

* port forwarding
* SOCKS proxying
* routing through an authorized pivot
* application-specific forwarding

Choose the smallest mechanism that provides the required access.

## 16. Reassess Credentials

After movement, repeat credential discovery from the new position.

Look for:

```text
Passwords
Hashes
Kerberos Material
SSH Keys
SSH Agent Identities
Application Credentials
Service Credentials
Existing Sessions
```

Then map:

```text
Credential
 ↓
Identity
 ↓
Service
 ↓
Target
 ↓
Authorization
```

Do not assume credentials found on one host have the same scope on another host.

## 17. Evaluate the Next Target

A new target should normally have at least one concrete reason for selection.

Examples:

```text
New Network
+
Known Service
```

```text
Known Credential
+
Compatible Service
```

```text
New Domain Context
+
Relevant Host
```

```text
Existing Access
+
Previously Unreachable Target
```

If no meaningful evidence exists, continue enumeration rather than forcing another movement attempt.

## 18. Avoid Circular Movement

Track previously accessed systems.

Example:

```text
WS-01 → SERVER-01 → FILE-01
```

Before attempting:

```text
FILE-01 → WS-01
```

ask:

> Does this provide new access, new visibility, or a new assessment objective?

If not, the movement is probably not useful.

## 19. Failure Decision Tree

Use this classification:

```text
Movement Attempt Failed
        ↓
Could I Reach the Service?
        │
   ┌────┴────┐
  No        Yes
  ↓          ↓
Network     Did Authentication
Problem     Succeed?
               │
          ┌────┴────┐
         No        Yes
         ↓          ↓
    Authentication  Is Authorization
       Problem      Sufficient?
                       │
                  ┌────┴────┐
                 No        Yes
                 ↓          ↓
            Authorization  Investigate
               Problem     Session / Access
```

This keeps troubleshooting structured.

## 20. Stop Conditions

Stop pursuing a movement path when:

* the target is outside scope
* the assessment objective has been met
* no useful access exists
* the required authorization is absent
* further attempts would require unnecessary credential guessing
* the path provides no new visibility or access
* the movement would create unnecessary risk
* the evidence does not justify the next action

Stopping is part of the workflow.

## 21. Complete Movement Loop

The complete decision tree can be reduced to:

```text
┌──────────────────────┐
│   Current Position   │
└──────────┬───────────┘
           ↓
┌──────────────────────┐
│   Identify Identity  │
└──────────┬───────────┘
           ↓
┌──────────────────────┐
│ Determine Networks   │
└──────────┬───────────┘
           ↓
┌──────────────────────┐
│   Select Target      │
└──────────┬───────────┘
           ↓
┌──────────────────────┐
│   Identify Service   │
└──────────┬───────────┘
           ↓
┌──────────────────────┐
│ Test Reachability    │
└──────────┬───────────┘
           ↓
┌──────────────────────┐
│ Map Authentication   │
│      Material        │
└──────────┬───────────┘
           ↓
┌──────────────────────┐
│ Authenticate         │
└──────────┬───────────┘
           ↓
┌──────────────────────┐
│ Check Authorization  │
└──────────┬───────────┘
           ↓
┌──────────────────────┐
│ Establish Movement   │
└──────────┬───────────┘
           ↓
┌──────────────────────┐
│ Validate New Host    │
└──────────┬───────────┘
           ↓
┌──────────────────────┐
│ Re-enumerate         │
└──────────┬───────────┘
           ↓
┌──────────────────────┐
│ Identify New Targets │
└──────────┬───────────┘
           ↓
      Continue?
       /      \
     Yes       No
      ↓         ↓
   Repeat      Stop
```

## 22. Evidence Record

For every meaningful movement, record:

```text
Source Host:
Source User:
Target Host:
Target IP:
Target Service:
Authentication Context:
Authentication Result:
Authorization Result:
Access Obtained:
New Network Visibility:
New Credentials / Access:
Next Decision:
Reason:
```

This makes the movement chain reproducible during reporting.

## What Next?

Continue to [`Credential-to-Target-Matrix.md`](Credential-to-Target-Matrix.md).

That file maps common authentication material and account contexts to the services and target types where they may become relevant.
