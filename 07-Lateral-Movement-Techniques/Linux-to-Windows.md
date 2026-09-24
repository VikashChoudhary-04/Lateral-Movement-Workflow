# Linux-to-Windows Lateral Movement

Linux-to-Windows lateral movement occurs when an existing foothold on a Linux system is used to establish authorized access to a Windows system.

A Linux foothold can reach Windows infrastructure through several remote services, including:

* SMB
* WinRM
* RDP
* RPC/WMI

The objective is not to test every Windows service.

The objective is:

> **Identify a reachable Windows target, determine which remote service matches the available authentication material, validate authorization, establish access, and reassess the resulting Windows position.**

---

## Movement Model

The workflow is:

```text id="7q1q6n"
Linux Host A
      ↓
Current Identity
      ↓
Network Context
      ↓
Windows Target Discovery
      ↓
Remote Service Discovery
      ↓
Authentication Material
      ↓
Service Compatibility
      ↓
Authentication
      ↓
Authorization
      ↓
Windows Access
      ↓
Validate Position
      ↓
Re-enumerate
      ↓
Continue the Chain
```

---

## Prerequisites

Before attempting movement, determine:

| Requirement    | Question                                                    |
| -------------- | ----------------------------------------------------------- |
| Source         | Which Linux host am I on?                                   |
| Identity       | Which Linux account/security context do I have?             |
| Network        | Which networks can the Linux host reach?                    |
| Target         | Which Windows host am I targeting?                          |
| Services       | Which Windows remote services are available?                |
| Authentication | What authorized authentication material exists?             |
| Compatibility  | Which service can use that authentication?                  |
| Authorization  | Is the account permitted to perform the required operation? |
| Objective      | Why does the Windows target matter?                         |

If important information is missing, return to discovery.

---

## Step 1 — Establish the Linux Starting Position

Identify the current host:

```bash id="0d6gq4"
hostname
```

```bash id="u7o3f9"
hostname -f
```

Current identity:

```bash id="7v4f4k"
whoami
```

```bash id="z7y3u5"
id
```

Network interfaces:

```bash id="r4n7c1"
ip addr
```

Routes:

```bash id="d4w2x6"
ip route
```

DNS:

```bash id="p3v5s7"
cat /etc/resolv.conf
```

Current connections:

```bash id="a8r2k9"
ss -tunap
```

Record:

```text id="k6c3e8"
Linux Source:
User:
Groups:
Privileges:
IP:
Subnet:
Gateway:
Routes:
DNS:
Known Targets:
```

---

## Step 2 — Discover Windows Targets

Potential Windows targets may be identified through:

* DNS
* network enumeration
* SMB evidence
* AD information
* application configuration
* existing connections
* infrastructure documentation
* previous movement

Example:

```text id="8p1v2m"
Target: WIN-SRV01
IP: 10.10.20.40
Role: Windows Server
Reason: Windows host discovered on reachable subnet
```

A target should have an identifiable reason for investigation.

---

## Step 3 — Identify Windows Services

Determine which remote-access services are available.

Common Windows services include:

| Service     | Common Port | Potential Use              |
| ----------- | ----------: | -------------------------- |
| SMB         |         445 | File/resource access       |
| WinRM       |        5985 | Windows remote management  |
| WinRM HTTPS |        5986 | Encrypted WinRM            |
| RDP         |        3389 | Interactive Windows access |
| RPC         |         135 | Windows RPC infrastructure |

Do not assume that an open port guarantees access.

---

## Step 4 — Test Reachability

From Linux, test the relevant ports.

### SMB

```bash id="x4r8t0"
nc -vz WIN-SRV01 445
```

### WinRM

```bash id="w5g3d8"
nc -vz WIN-SRV01 5985
```

or:

```bash id="n6m1v9"
nc -vz WIN-SRV01 5986
```

### RDP

```bash id="r7c2k4"
nc -vz WIN-SRV01 3389
```

### RPC

```bash id="m8p4j2"
nc -vz WIN-SRV01 135
```

Interpret the results as:

```text id="w7x6c3"
Port Reachable
      ≠
Service Validated
      ≠
Authentication
      ≠
Authorization
      ≠
Remote Access
```

---

## Step 5 — Identify Available Authentication

Map the authentication material available from the Linux host.

Potential sources include:

* domain credentials
* local Windows credentials
* authorized passwords
* NTLM-compatible authentication material
* Kerberos authentication context
* existing sessions
* documented administrative credentials

The important question is:

> **Which Windows service can use the authentication material I already have?**

Create a mapping:

```text id="e2w4q8"
Authentication Material
        ↓
Identity
        ↓
Windows Target
        ↓
Remote Service
```

---

## Step 6 — SMB Movement Path

SMB is one of the primary Windows remote-access paths.

First validate connectivity:

```bash id="k8y3z6"
nc -vz WIN-SRV01 445
```

Where authorized, enumerate SMB shares:

```bash id="d2s5x9"
smbclient -L //WIN-SRV01 -U 'CORP/analyst'
```

A successful SMB connection can provide:

* share access
* file access
* information about the target
* evidence of authentication
* potentially administrative access if the identity is authorized

Do not confuse:

```text id="f7m2a9"
SMB share access
```

with:

```text id="j8q4k3"
Remote interactive Windows session
```

The access level must be determined separately.

---

## Step 7 — Map SMB Authorization

If SMB authentication succeeds, determine what the account can actually access.

Possible results include:

```text id="y6v1b3"
No shares
Read-only share
Read/write share
Administrative share
```

Examples of common administrative shares include:

```text id="n4x5k7"
ADMIN$
C$
IPC$
```

Their visibility or availability does not by itself prove that the current account has administrative rights.

Record the actual authorization result.

---

## Step 8 — WinRM Movement Path

If WinRM is available:

```bash id="g5s8n2"
nc -vz WIN-SRV01 5985
```

or:

```bash id="q9k4m6"
nc -vz WIN-SRV01 5986
```

From a Linux assessment environment, use an authorized WinRM-compatible client where appropriate.

The movement workflow remains:

```text id="f6c1d8"
WinRM Reachable
      ↓
Authentication
      ↓
Remote Management Authorization
      ↓
Remote Session
      ↓
Windows Position
```

The important distinction is that WinRM authorization is separate from simply possessing a valid Windows credential.

---

## Step 9 — RDP Movement Path

RDP can provide an interactive Windows desktop session.

First validate:

```bash id="p2q8m4"
nc -vz WIN-SRV01 3389
```

Then determine:

```text id="6x8k1r"
Is RDP enabled?
       ↓
Can the identity authenticate?
       ↓
Is the identity allowed to log in through RDP?
       ↓
What session will be created?
```

RDP access may depend on:

* local groups
* domain groups
* Remote Desktop Users membership
* security policy
* domain policy
* target configuration

A valid Windows account does not automatically have RDP authorization.

---

## Step 10 — RPC/WMI Movement Path

If RPC is reachable:

```bash id="q4x9v6"
nc -vz WIN-SRV01 135
```

RPC is commonly a dependency for Windows management technologies such as WMI/DCOM.

The important workflow is:

```text id="a4n7k8"
Linux Host
    ↓
Windows Target
    ↓
TCP 135
    ↓
RPC
    ↓
WMI/DCOM
    ↓
Authentication
    ↓
Authorization
```

Do not interpret TCP 135 alone as proof that WMI management is available.

---

## Step 11 — Select the Movement Path

If several services are available, build a service matrix.

| Target  | SMB    | WinRM  | RDP    | RPC    | Known Authentication | Investigation |
| ------- | ------ | ------ | ------ | ------ | -------------------- | ------------- |
| `WIN01` | Open   | Open   | Open   | Open   | Domain credential    | WinRM         |
| `WIN02` | Open   | Closed | Open   | Open   | SMB-compatible       | SMB           |
| `WIN03` | Closed | Open   | Closed | Open   | Domain credential    | WinRM         |
| `WIN04` | Open   | Closed | Open   | Closed | User credential      | RDP           |

This is not a ranking of techniques.

It records which paths have supporting evidence.

---

## Step 12 — Validate Authentication

For the selected service, validate the specific identity.

The test should answer:

```text id="e3q9w7"
Can this identity authenticate
to this service on this target?
```

If authentication succeeds, continue to authorization.

If authentication fails, investigate:

* username
* credential
* domain
* authentication mechanism
* account status
* target configuration

Do not immediately conclude that the credential is useless for every Windows service.

---

## Step 13 — Validate Authorization

Authentication answers:

```text id="2m6v8k"
"Who are you?"
```

Authorization answers:

```text id="9r4j6p"
"What are you allowed to do?"
```

Examples:

```text id="w5f2k8"
SMB authentication
     ↓
Share authorization

WinRM authentication
     ↓
Remote management authorization

RDP authentication
     ↓
Interactive logon authorization

WMI authentication
     ↓
WMI/DCOM operation authorization
```

The same identity may have different permissions across different services.

---

## Step 14 — Validate the Windows Position

After successful movement, immediately establish the new position.

On the Windows host:

```cmd id="y5k7r3"
hostname
```

```cmd id="p4d8n6"
whoami
```

```cmd id="s7q3x1"
whoami /user
```

```cmd id="a6c9v2"
whoami /groups
```

Network:

```cmd id="e3h5w8"
ipconfig /all
```

Routes:

```cmd id="v8m2j6"
route print
```

Current connections:

```cmd id="u4c7n9"
netstat -ano
```

The new position is now:

```text id="c8q1v5"
Windows Host B
    ↓
New User
    ↓
New Privileges
    ↓
New Network
    ↓
New Services
    ↓
New Targets
```

---

## Step 15 — Compare Source and Target Positions

Compare:

| Property    | Linux Source    | Windows Target   |
| ----------- | --------------- | ---------------- |
| Host        | `LINUX-SRV01`   | `WIN-SRV01`      |
| User        | `analyst`       | `CORP\analyst`   |
| Network     | `10.10.10.0/24` | `10.10.20.0/24`  |
| Routes      | Documented      | Documented       |
| Services    | Linux services  | Windows services |
| New Targets | Documented      | Documented       |

The goal is to understand what changed as a result of movement.

---

## Step 16 — Reassess Windows Network Position

Inspect:

```cmd id="q9r2m6"
ipconfig /all
```

```cmd id="h5j8n3"
route print
```

PowerShell:

```powershell id="r4c7v1"
Get-NetIPConfiguration
```

```powershell id="f6m9x2"
Get-NetRoute
```

```powershell id="p7k3d8"
Get-NetNeighbor
```

Look for:

* additional interfaces
* different subnets
* new routes
* domain networks
* internal infrastructure

Then return to target discovery.

---

## Step 17 — Reassess Windows Services

Identify listening services:

```powershell id="g8c5m2"
Get-NetTCPConnection -State Listen
```

Or:

```cmd id="n7v3q1"
netstat -ano
```

If authorized, identify relevant services:

```powershell id="s6d2k8"
Get-Service
```

The goal is to understand the role of the new Windows system.

It may be:

* workstation
* application server
* file server
* database server
* management server
* domain infrastructure

Host role influences the next investigation.

---

## Step 18 — Reassess Windows Sessions

Check active interactive sessions:

```cmd id="t5q8m1"
query user
```

```cmd id="b4n7v2"
qwinsta
```

PowerShell:

```powershell id="c3k6x9"
Get-NetTCPConnection
```

Use this to understand the current host's operational context.

Do not automatically treat another logged-in user as reusable authentication material.

---

## Step 19 — Reassess Domain Context

If the target is domain joined:

```cmd id="m8q2v4"
whoami
```

```cmd id="z5r7k1"
echo %USERDOMAIN%
```

```cmd id="h4c9n6"
echo %USERDNSDOMAIN%
```

PowerShell:

```powershell id="j7x3p8"
Get-CimInstance Win32_ComputerSystem | Select-Object Name,Domain,PartOfDomain
```

Determine whether the Windows target provides:

* additional domain visibility
* different group memberships
* new authentication relationships
* access to additional systems
* different network reachability

---

## Step 20 — Update the Movement Record

Record the cross-platform movement.

| Field                 | Result                  |
| --------------------- | ----------------------- |
| Source Host           | `LINUX-SRV01`           |
| Source User           | `analyst`               |
| Target Host           | `WIN-SRV01`             |
| Target OS             | Windows                 |
| Service               | SMB / WinRM / RDP / WMI |
| Authentication        | Documented              |
| Authentication Result | Success                 |
| Authorization         | Documented              |
| Access                | Documented              |
| New User              | Documented              |
| New Privileges        | Documented              |
| New Network           | Documented              |
| New Services          | Documented              |
| New Targets           | Documented              |
| Next Action           | Re-enumerate            |

---

## Failure Classification

### Target unreachable

Investigate:

```text id="g4k8p2"
Linux routing
Network segmentation
Firewall
Target availability
```

### Service unavailable

Investigate:

```text id="d6m3q7"
Port
Firewall
Service configuration
Target state
```

### Authentication failure

Investigate:

```text id="x2v5n8"
Username
Credential
Domain
Authentication protocol
Account status
```

### Authorization failure

Investigate:

```text id="f3r7k9"
Group membership
Remote-access policy
Service permissions
Local policy
Domain policy
```

### RPC/WMI failure

Investigate:

```text id="p5c8m1"
TCP 135
Dynamic RPC
Firewall
DCOM
Authentication
Authorization
```

### RDP failure

Investigate:

```text id="q7h2v6"
3389
RDP service
NLA
Authentication
Remote Desktop Users
Security policy
```

### SMB failure

Investigate:

```text id="w8d4k3"
445
SMB service
Authentication
Share permissions
NTFS permissions
Account authorization
```

Classify the failure before selecting another path.

---

## Linux-to-Windows Target Matrix

Maintain a record when multiple Windows systems are available.

| Target  | SMB    | WinRM  | RDP    | RPC    | Auth Material     | Result            |
| ------- | ------ | ------ | ------ | ------ | ----------------- | ----------------- |
| `WIN01` | Open   | Open   | Open   | Open   | Domain credential | Investigate WinRM |
| `WIN02` | Open   | Closed | Open   | Open   | SMB-compatible    | Investigate SMB   |
| `WIN03` | Closed | Open   | Closed | Open   | Domain credential | Investigate WinRM |
| `WIN04` | Open   | Closed | Open   | Closed | User credential   | Investigate RDP   |

This records evidence without assuming that an open service is automatically usable.

---

## Example Movement Chain

A Linux-to-Windows movement chain may look like:

```text id="j9r3c7"
LINUX-APP01
      │
      │ SMB
      ▼
WIN-SRV01
      │
      ├── New domain context
      ├── New internal network
      ├── Additional Windows services
      │
      │ WinRM
      ▼
WIN-SRV02
```

The correct workflow is:

```text id="z5m7p2"
Linux foothold
      ↓
Windows target discovery
      ↓
Service discovery
      ↓
Authentication mapping
      ↓
Authorized access
      ↓
Windows position validation
      ↓
Re-enumeration
      ↓
Next movement
```

---

## Decision Tree

```text id="k5v8r2"
Start on Linux Host
        │
        ↓
Identify user and network
        │
        ↓
Discover Windows target
        │
        ↓
Which Windows service is reachable?
        │
   ┌────┼─────┬─────┐
  SMB  WinRM  RDP  RPC
   │     │     │     │
   └─────┴─────┴─────┘
        │
        ↓
Do I have compatible authentication?
        │
   ┌────┴────┐
   No       Yes
   │          │
Credential    ↓
Discovery   Does authentication
            succeed?
                │
           ┌────┴────┐
          No        Yes
          │           │
     Investigate      ↓
     identity/    Is the requested
     auth policy  operation authorized?
                       │
                  ┌────┴────┐
                 No        Yes
                 │           │
            Investigate      ↓
              authz      Establish access
                              │
                              ↓
                        Validate Windows host
                              │
                              ↓
                        Re-enumerate
                              │
                              ↓
                        Continue chain
```

---

## Common Mistakes

### Assuming Linux credentials work on Windows

Different operating systems may use completely different identity systems.

### Treating an open Windows port as access

The port only establishes network connectivity.

### Testing every service blindly

Select the service based on target evidence and available authentication.

### Confusing SMB with remote shell access

SMB share access and remote command execution are different capabilities.

### Assuming WinRM authentication means administration

The account must also be authorized for remote management.

### Assuming a valid Windows account can use RDP

RDP logon requires appropriate authorization.

### Treating TCP 135 as WMI access

RPC reachability does not prove WMI authorization.

### Ignoring domain context

A Windows domain account can have different access relationships from a local account.

### Stopping after Windows access

The Windows host becomes the new position.

### Forgetting the network transition

The Windows system may expose networks unavailable from the original Linux host.

---

## Completion Criteria

Linux-to-Windows movement is complete when you can document:

* Linux source host
* source identity
* source privileges
* Windows target
* target role
* target reachability
* available services
* selected movement path
* authentication material
* authentication result
* authorization result
* resulting Windows access
* new Windows identity
* new privileges
* new network position
* new services
* newly discovered targets
* next movement path

If movement fails, classify the failure and update the next hypothesis.

---

## What Next?

Continue with:

**[`AD-Lateral-Movement.md`](AD-Lateral-Movement.md)**

The final workflow in this section covers lateral movement in Active Directory environments, where domain identities, Kerberos, computer accounts, groups, trusts, and Windows remote services create additional movement relationships.
