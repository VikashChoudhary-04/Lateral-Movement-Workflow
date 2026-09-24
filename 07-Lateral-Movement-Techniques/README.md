# Lateral Movement Techniques

This section turns the remote services and access paths identified earlier into actual **cross-host movement workflows**.

The previous sections answered:

```text
Where am I?
What do I have?
What can I reach?
Which services are available?
Which authentication methods can I use?
```

This section answers:

> **Given my current position and available access, how do I move to another host?**

The focus remains on authorized security assessments and controlled lab environments.

---

## Purpose

Lateral movement is not a single technique.

The appropriate movement path depends on:

* current operating system
* target operating system
* network reachability
* available credentials
* authentication material
* remote services
* account authorization
* target role
* domain/trust relationships
* network segmentation
* objective of the assessment

The workflow should therefore be:

```text
Current Position
      ↓
Available Access
      ↓
Target
      ↓
Remote Service
      ↓
Compatible Authentication
      ↓
Authorization
      ↓
Movement
      ↓
Validate New Position
      ↓
Re-enumerate
      ↓
Continue the Chain
```

---

## Movement Is a Decision

Do not start with:

```text
"What lateral movement technique should I use?"
```

Start with:

```text
"What access do I already have?"
```

Then determine:

```text
What targets can I reach?
        ↓
What services are available?
        ↓
What authentication material do I have?
        ↓
Which service accepts that authentication?
        ↓
What is my authorization level?
        ↓
Which movement path is supported by evidence?
```

This prevents technique-first behavior.

---

## Cross-Platform Coverage

This section covers five major movement directions:

| Workflow            | Primary Question                                         |
| ------------------- | -------------------------------------------------------- |
| Windows → Windows   | Can Windows access another Windows host?                 |
| Linux → Linux       | Can Linux access another Linux host?                     |
| Windows → Linux     | Can a Windows foothold reach a Linux target?             |
| Linux → Windows     | Can a Linux foothold reach a Windows target?             |
| AD Lateral Movement | Can domain identity/trust relationships enable movement? |

These are movement directions, not isolated attack categories.

---

## Windows-to-Windows

Typical movement paths may involve:

* SMB
* WinRM
* RDP
* WMI
* RPC
* existing Windows management access
* domain authentication

Example decision flow:

```text
Windows Host A
     ↓
Current Identity
     ↓
Discover Windows Host B
     ↓
Identify SMB / WinRM / RDP / WMI
     ↓
Map available authentication
     ↓
Validate authorization
     ↓
Establish authorized access
     ↓
Windows Host B
```

After movement:

```text
Host B
 ↓
whoami
hostname
network
routes
domain
sessions
services
 ↓
Re-enumerate
```

---

## Linux-to-Linux

SSH is usually the primary remote-access path.

Potential movement sources include:

* valid passwords
* SSH keys
* SSH agents
* existing SSH configuration
* authorized access to another Linux account

Example:

```text
Linux Host A
     ↓
SSH configuration / credentials / keys
     ↓
Discover Linux Host B
     ↓
TCP 22 or identified SSH port
     ↓
Authenticate
     ↓
Authorization
     ↓
SSH session
     ↓
Linux Host B
```

After movement, reassess:

```bash
whoami
id
hostname
ip addr
ip route
ss -tunap
```

---

## Windows-to-Linux

A Windows foothold does not limit movement to Windows systems.

First identify Linux targets that are reachable from the current host.

Then determine whether an appropriate Linux remote-access service is available.

Typical path:

```text
Windows Host
     ↓
Network Discovery
     ↓
Linux Target
     ↓
SSH Service
     ↓
Linux Account
     ↓
Password / Key / Other Authorized Authentication
     ↓
SSH Access
     ↓
Linux Host
```

The key decision is not:

```text
"Linux target found."
```

It is:

```text
"Linux target + reachable SSH + compatible authentication + authorization."
```

---

## Linux-to-Windows

A Linux foothold can also move into Windows infrastructure.

Potential services include:

* SMB
* WinRM
* RDP
* RPC/WMI
* other authorized Windows management interfaces

Example:

```text
Linux Host
     ↓
Windows Target Discovery
     ↓
SMB / WinRM / RDP / RPC
     ↓
Available Authentication
     ↓
Authorization
     ↓
Windows Access
     ↓
Validate
     ↓
Re-enumerate
```

Linux-based assessment tools can help validate connectivity and service availability, but the movement decision should still be based on the same evidence-driven workflow.

---

## Active Directory Lateral Movement

Active Directory introduces additional relationships between:

* users
* computers
* groups
* domains
* services
* authentication systems
* trusts

A domain identity may therefore provide access to multiple systems without requiring a separate local account on every target.

The workflow becomes:

```text
Current Domain Identity
       ↓
Domain Context
       ↓
Target Computer
       ↓
Authentication Relationship
       ↓
Remote Service
       ↓
Authorization
       ↓
Remote Access
       ↓
New Domain Position
```

The existence of a domain relationship does not automatically mean that every domain account can access every domain computer.

Always validate the specific target and operation.

---

## Credential-to-Technique Mapping

Do not choose a movement technique before understanding the available authentication material.

| Available Access                      | Possible Movement Path                                    |
| ------------------------------------- | --------------------------------------------------------- |
| SSH password                          | SSH                                                       |
| SSH private key                       | SSH                                                       |
| SSH agent identity                    | SSH                                                       |
| SMB-compatible credentials            | SMB                                                       |
| Authorized Windows management account | WinRM/WMI/RPC depending on permissions                    |
| RDP-authorized account                | RDP                                                       |
| Kerberos authentication context       | Appropriate domain services                               |
| Existing authenticated session        | Potentially useful depending on service and authorization |

This is a **mapping**, not a guarantee.

The actual environment determines whether the path works.

---

## Service-to-Technique Mapping

Remote services provide the transport for movement.

| Service | Typical Movement Context                  |
| ------- | ----------------------------------------- |
| SSH     | Linux / OpenSSH-enabled Windows           |
| SMB     | Windows file/resource access              |
| WinRM   | Windows remote management                 |
| RDP     | Windows interactive access                |
| WMI     | Windows management                        |
| RPC     | Windows management/service infrastructure |

The correct question is:

> **Which available service matches both my authentication material and the access I need?**

---

## Movement Prerequisites

Before attempting movement, verify:

### 1. Target

```text
Do I know which host I am targeting?
```

### 2. Reachability

```text
Can my current host reach the target?
```

### 3. Service

```text
Which remote service provides the movement path?
```

### 4. Authentication

```text
What identity or authentication material will be used?
```

### 5. Authorization

```text
Is that identity allowed to perform the required operation?
```

### 6. Objective

```text
Why am I moving to this host?
```

The objective matters because movement should produce a meaningful new position rather than simply increasing the number of compromised systems.

---

## Movement Validation

A movement attempt is not complete when the connection command returns success.

Validate the new position.

### Windows

```cmd
whoami
hostname
whoami /groups
ipconfig /all
route print
```

### Linux

```bash
whoami
id
hostname
ip addr
ip route
ss -tunap
```

Determine:

* host
* user
* privileges
* network
* routes
* domain context
* reachable targets
* available services

---

## Post-Movement Reassessment

Every successful movement should return to the position-assessment workflow.

```text
New Host
   ↓
Current User
   ↓
Privileges
   ↓
Network Context
   ↓
Trust / Domain Context
   ↓
Sessions
   ↓
Services
   ↓
Credentials / Access
   ↓
Target Discovery
   ↓
Next Movement
```

This creates a repeatable chain:

```text
Host A
  ↓
Movement
  ↓
Host B
  ↓
Re-enumerate
  ↓
Movement
  ↓
Host C
  ↓
Re-enumerate
  ↓
...
```

---

## Failure Does Not Mean Stop

A failed movement attempt provides information.

For example:

```text
SSH connection refused
```

may indicate:

* no SSH service
* different SSH port
* service stopped
* firewall behavior

Whereas:

```text
SSH authentication failed
```

points toward:

* wrong credentials
* wrong username
* authentication policy
* invalid key

Similarly:

```text
WinRM connection blocked
```

is different from:

```text
WinRM authentication succeeded but operation denied
```

Always classify the failure before choosing the next action.

---

## Movement Decision Loop

Use this loop for every candidate movement path:

```text
1. Where am I?
        ↓
2. What identity do I have?
        ↓
3. What authentication material do I have?
        ↓
4. What targets can I reach?
        ↓
5. What services are available?
        ↓
6. Which services match my access?
        ↓
7. What authorization do I have?
        ↓
8. Can I establish controlled access?
        ↓
9. Did I reach the intended host?
        ↓
10. What did I gain?
        ↓
11. What changed in my position?
        ↓
12. Re-enumerate
        ↓
13. What is the next supported path?
```

---

## Movement Evidence Record

Maintain a record for each movement attempt.

| Field                 | Example                   |
| --------------------- | ------------------------- |
| Source Host           | `WORKSTATION01`           |
| Source User           | `CORP\analyst`            |
| Target Host           | `SERVER01`                |
| Target OS             | Windows                   |
| Service               | WinRM                     |
| Port                  | 5985                      |
| Authentication        | Domain credential         |
| Authentication Result | Success                   |
| Authorization         | Remote management allowed |
| Access Result         | Remote session            |
| New User              | `CORP\analyst`            |
| New Network           | `10.10.20.0/24`           |
| New Targets           | `SERVER02`, `DB01`        |
| Next Action           | Re-enumerate network      |

This makes multi-host movement easier to understand and reproduce.

---

## Common Mistakes

### Moving without a target

A successful connection is not useful if you do not know why the target matters.

### Treating every credential as universal

Credentials are often limited by:

* account type
* authentication protocol
* target
* service
* policy
* authorization

### Ignoring service compatibility

A credential that works for one service may not work for another.

### Confusing reachability with access

```text
Host reachable
    ≠
Service available
    ≠
Authentication successful
    ≠
Authorization granted
    ≠
Remote session
```

### Stopping after successful login

A new host is a new reconnaissance position.

### Forgetting network changes

The new host may provide access to networks that were previously unreachable.

### Repeating failed attempts without learning

Every failure should update the movement hypothesis.

---

## Safety and Assessment Boundaries

Lateral movement should be performed only where explicitly authorized.

Before testing:

* confirm scope
* identify permitted hosts
* identify permitted accounts
* understand testing windows
* avoid unnecessary disruption
* avoid indiscriminate credential testing
* document successful and failed paths

The objective of an assessment is to demonstrate the movement path while maintaining control over scope and impact.

---

## Section Structure

The remaining files provide movement workflows by operating-system direction:

```text
Windows-to-Windows.md
Linux-to-Linux.md
Windows-to-Linux.md
Linux-to-Windows.md
AD-Lateral-Movement.md
```

Each file focuses on:

```text
Starting Position
      ↓
Target Discovery
      ↓
Service Selection
      ↓
Authentication
      ↓
Authorization
      ↓
Movement
      ↓
Validation
      ↓
Post-Movement Enumeration
      ↓
Next Path
```

---

## What Next?

Start with the most common Windows cross-host movement scenario:

**[`Windows-to-Windows.md`](Windows-to-Windows.md)**

This workflow will combine the Windows remote-access services covered in Section 06 into a practical movement process from one Windows host to another.
