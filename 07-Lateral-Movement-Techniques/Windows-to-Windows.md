# Windows-to-Windows Lateral Movement

Windows-to-Windows lateral movement occurs when an existing foothold, identity, credential, session, or authorized access on one Windows system is used to establish access to another Windows system.

The objective is not to try every Windows remote-access protocol.

The objective is to determine:

> **Which Windows target can I reach, which remote service can I use, what authentication material is available, and what level of access does that identity have?**

This workflow combines the Windows remote-access services covered in Section 06:

* SMB
* WinRM
* RDP
* WMI
* RPC

The same evidence-driven process should be used throughout.

---

## Movement Model

The basic workflow is:

```text
Windows Host A
      ↓
Current Identity
      ↓
Network Context
      ↓
Discover Windows Targets
      ↓
Identify Remote Services
      ↓
Map Authentication Material
      ↓
Determine Authorization
      ↓
Select Movement Path
      ↓
Establish Access
      ↓
Validate Windows Host B
      ↓
Re-enumerate
      ↓
Continue the Chain
```

The critical point is that **Host B becomes the new position** after successful movement.

---

## Prerequisites

Before attempting movement, establish:

| Requirement    | Question                                     |
| -------------- | -------------------------------------------- |
| Source         | Which Windows host am I currently on?        |
| Identity       | Which account/security context do I have?    |
| Target         | Which Windows host am I targeting?           |
| Reachability   | Can Source reach Target?                     |
| Service        | Which remote service is available?           |
| Authentication | What authentication material can I use?      |
| Authorization  | Is the identity allowed to use that service? |
| Objective      | Why does Target matter?                      |

If several answers are unknown, return to the earlier discovery sections before attempting movement.

---

## Step 1 — Establish the Starting Position

From the current Windows host:

```cmd
hostname
```

```cmd
whoami
```

```cmd
whoami /user
```

```cmd
whoami /groups
```

Network configuration:

```cmd
ipconfig /all
```

Routes:

```cmd
route print
```

Current connections:

```cmd
netstat -ano
```

PowerShell can provide additional context:

```powershell
Get-NetIPConfiguration
```

```powershell
Get-NetRoute
```

```powershell
Get-NetTCPConnection
```

Record:

```text
Source Host:
Current User:
Domain:
Privileges:
IP Address:
Subnet:
Gateway:
Routes:
Known Targets:
```

---

## Step 2 — Identify Available Authentication

Determine what authorized authentication material is already available.

Potential sources include:

* current domain identity
* local credentials
* authorized passwords
* SSH/OpenSSH credentials where applicable
* Kerberos authentication context
* existing sessions
* documented administrative access
* authorized service credentials

Do not immediately test every credential against every host.

Instead create a mapping:

```text
Identity
   ↓
Authentication Material
   ↓
Compatible Service
   ↓
Candidate Target
```

For example:

```text
CORP\analyst
     ↓
Domain authentication
     ↓
WinRM
     ↓
SERVER01
```

---

## Step 3 — Discover Candidate Windows Targets

Use evidence already collected from:

* network routes
* DNS
* SMB
* AD enumeration
* existing connections
* application configuration
* known infrastructure
* previous hosts

A candidate might look like:

```text
Target: SERVER01
IP: 10.10.20.15
OS: Windows Server
Role: Application Server
Reason: Reachable from current subnet
```

Do not assume every discovered Windows host is a valid movement target.

The target should have a reason for investigation.

---

## Step 4 — Confirm Reachability

Test the target at the network layer first.

```powershell
Test-Connection SERVER01 -Count 2
```

Then test the relevant service.

### SMB

```powershell
Test-NetConnection SERVER01 -Port 445
```

### WinRM

```powershell
Test-NetConnection SERVER01 -Port 5985
```

or:

```powershell
Test-NetConnection SERVER01 -Port 5986
```

### RDP

```powershell
Test-NetConnection SERVER01 -Port 3389
```

### RPC/WMI

```powershell
Test-NetConnection SERVER01 -Port 135
```

Interpret these results as service reachability, not proof of access.

---

## Reachability vs Access

Always distinguish:

```text
Target reachable
      ↓
Service reachable
      ↓
Authentication possible
      ↓
Authentication succeeds
      ↓
Authorization granted
      ↓
Remote access established
```

For example:

```text
445 open
```

means SMB is reachable.

It does not mean:

```text
SMB authentication succeeds
```

and it certainly does not mean:

```text
Remote administrative access exists
```

---

## Step 5 — Identify the Best-Supported Service

Use evidence to select the service.

| Evidence                                          | Possible Path                              |
| ------------------------------------------------- | ------------------------------------------ |
| SMB reachable + compatible account                | SMB                                        |
| WinRM reachable + remote-management authorization | WinRM                                      |
| RDP reachable + interactive-login authorization   | RDP                                        |
| RPC reachable + authorized WMI/DCOM operation     | WMI/RPC                                    |
| Multiple services available                       | Select based on objective and known access |

The selection should answer:

> **Which available service has the strongest evidence of compatibility with my current access?**

---

## Step 6 — SMB Movement Path

If SMB is the relevant path, first validate connectivity:

```powershell
Test-NetConnection SERVER01 -Port 445
```

Review available shares where authorized:

```cmd
net view \\SERVER01
```

If authenticated SMB access is already available:

```cmd
net use
```

A successful share connection demonstrates SMB access, but share access and remote command execution are different capabilities.

Record:

```text
Target:
SMB:
Authentication:
Share Access:
Read:
Write:
Administrative Access:
```

If SMB provides useful access, continue with the SMB workflow.

---

## Step 7 — WinRM Movement Path

If WinRM is the relevant path:

```powershell
Test-NetConnection SERVER01 -Port 5985
```

Validate the service:

```powershell
Test-WSMan SERVER01
```

Where authorized, a controlled remote session can validate the movement path:

```powershell
Enter-PSSession -ComputerName SERVER01
```

Then immediately establish the new identity and host:

```powershell
whoami
hostname
```

For a controlled command execution model:

```powershell
Invoke-Command -ComputerName SERVER01 -ScriptBlock {
    hostname
    whoami
}
```

A successful connection demonstrates remote-management access under the current authorization context.

---

## Step 8 — RDP Movement Path

If RDP is the relevant service:

```powershell
Test-NetConnection SERVER01 -Port 3389
```

The next questions are:

```text
Is RDP enabled?
       ↓
Can the identity authenticate?
       ↓
Is the identity permitted to log on through RDP?
       ↓
What session will be created?
```

RDP authorization may depend on:

* local groups
* domain groups
* Remote Desktop Users membership
* security policy
* domain policy
* target configuration

A valid account does not automatically mean RDP logon is permitted.

---

## Step 9 — WMI/RPC Movement Path

If WMI is the relevant management path:

```powershell
Test-NetConnection SERVER01 -Port 135
```

Then validate a controlled WMI query where authorized:

```powershell
Get-CimInstance -ClassName Win32_ComputerSystem -ComputerName SERVER01
```

This helps establish:

```text
RPC Reachability
      ↓
Remote WMI Communication
      ↓
Authentication
      ↓
Authorization
```

Do not interpret TCP 135 alone as proof of WMI access.

---

## Step 10 — Compare Movement Paths

When multiple services are available, record them before selecting a path.

| Target   | SMB    | WinRM  | RDP    | RPC/WMI | Known Auth   | Next Investigation |
| -------- | ------ | ------ | ------ | ------- | ------------ | ------------------ |
| SERVER01 | Open   | Open   | Open   | Open    | Domain       | WinRM              |
| SERVER02 | Open   | Closed | Open   | Open    | Domain       | SMB                |
| SERVER03 | Closed | Open   | Closed | Open    | Unknown      | WinRM              |
| SERVER04 | Open   | Closed | Open   | Closed  | User account | RDP                |

This is not a ranking of techniques.

It is a record of **evidence and compatibility**.

---

## Step 11 — Validate Successful Movement

After establishing remote access to Host B, verify the new position.

```powershell
hostname
```

```powershell
whoami
```

```powershell
whoami /user
```

```powershell
whoami /groups
```

Network:

```powershell
ipconfig /all
```

Routes:

```powershell
route print
```

Connections:

```powershell
Get-NetTCPConnection
```

Determine whether Host B differs meaningfully from Host A.

---

## New Position Questions

Immediately ask:

### Identity

```text
Who am I now?
```

### Privileges

```text
What groups and privileges do I have?
```

### Network

```text
What networks can this host reach?
```

### Services

```text
What services are exposed from this host?
```

### Sessions

```text
Which users or remote connections are present?
```

### Credentials and access

```text
What new authorized access is visible?
```

### Targets

```text
What systems can Host B reach that Host A could not?
```

---

## Step 12 — Reassess the Network

A successful movement may expose a new network segment.

For example:

```text
Host A
10.10.10.25
      ↓
Host B
10.10.20.15
      ↓
10.10.30.0/24
```

Host B may therefore become a more useful reconnaissance position.

Inspect:

```powershell
Get-NetIPConfiguration
```

```powershell
Get-NetRoute
```

```powershell
Get-NetNeighbor
```

Then determine which networks are newly visible.

Do not assume that every reachable network should be scanned.

Return to the target-discovery workflow and investigate relevant systems within scope.

---

## Step 13 — Reassess Domain Context

If the target is domain joined:

```cmd
whoami
```

```cmd
echo %USERDOMAIN%
```

```cmd
echo %USERDNSDOMAIN%
```

PowerShell:

```powershell
Get-CimInstance Win32_ComputerSystem | Select-Object Name,Domain,PartOfDomain
```

Determine whether Host B provides:

* a different domain context
* additional domain visibility
* different group memberships
* different authentication relationships
* access to additional management systems

---

## Step 14 — Check Existing Sessions

On Host B:

```cmd
query user
```

```cmd
qwinsta
```

PowerShell:

```powershell
Get-NetTCPConnection
```

The goal is to understand:

```text
Current User
     ↓
Existing Sessions
     ↓
Remote Connections
     ↓
Potential New Access Relationships
```

Do not confuse evidence of another session with possession of reusable credentials.

---

## Step 15 — Update the Movement Record

After successful movement, record:

| Field             | Result                |
| ----------------- | --------------------- |
| Source Host       | `WORKSTATION01`       |
| Source User       | `CORP\analyst`        |
| Target Host       | `SERVER01`            |
| Service           | WinRM                 |
| Authentication    | Domain                |
| Access            | Remote PowerShell     |
| Target User       | `CORP\analyst`        |
| Target Privileges | Documented            |
| New Subnets       | `10.10.20.0/24`       |
| New Services      | Documented            |
| New Targets       | Documented            |
| Next Step         | Position reassessment |

This becomes especially important when the assessment contains several movement stages.

---

## Failure Classification

Classify failures before changing techniques.

### Target unreachable

Investigate:

```text
Routing
Segmentation
Target availability
Firewall
```

### Service unreachable

Investigate:

```text
Port
Firewall
Service state
Target configuration
```

### Authentication failure

Investigate:

```text
Username
Credential
Authentication protocol
Domain context
Account status
```

### Authorization failure

Investigate:

```text
Group membership
Remote-access policy
Service permissions
Local security policy
Domain policy
```

### Access succeeds but expected capability is unavailable

Investigate:

```text
Access level
Service limitations
Account privileges
Target policy
```

The failure itself should update the next hypothesis.

---

## Decision Tree

```text
Start on Windows Host A
        │
        ↓
Identify current user and privileges
        │
        ↓
Identify reachable Windows targets
        │
        ↓
Identify target services
        │
        ↓
Do I have compatible authentication?
        │
   ┌────┴────┐
   No       Yes
   │          │
Credential    ↓
Discovery   Is the service reachable?
              │
         ┌────┴────┐
        No        Yes
        │           │
   Investigate     ↓
   network/      Does authentication
   service       succeed?
                     │
                ┌────┴────┐
               No        Yes
               │           │
          Investigate      ↓
          identity/    Is the required
          authentication operation authorized?
                              │
                         ┌────┴────┐
                        No        Yes
                        │           │
                    Investigate    ↓
                      authz    Establish access
                                   │
                                   ↓
                            Validate Host B
                                   │
                                   ↓
                            Re-enumerate
                                   │
                                   ↓
                            Discover next path
```

---

## Example Movement Chain

A generic assessment chain may look like:

```text
WORKSTATION01
    │
    │ Domain authentication
    │ WinRM
    ▼
SERVER01
    │
    ├── New subnet discovered
    ├── Additional services discovered
    ├── New target discovered
    │
    │ SMB
    ▼
SERVER02
    │
    ├── New network position
    └── Additional authorized access
```

The important pattern is:

```text
Movement
   ↓
Validation
   ↓
Re-enumeration
   ↓
New Target Discovery
   ↓
Next Movement
```

not:

```text
Movement → Movement → Movement
```

without reassessment.

---

## Common Mistakes

### Testing every service blindly

Start with the services supported by evidence.

### Treating an open port as access

A port only establishes network-level connectivity.

### Assuming domain membership grants access

Domain membership and authorization are different concepts.

### Using the wrong username

Local and domain accounts may behave differently.

### Ignoring target role

A server's role can determine why movement to it matters.

### Stopping after remote login

The new host must be reassessed.

### Forgetting the source host's network position

The source may have access that the new host does not, while the new host may have access to networks the source cannot reach.

### Reusing a failed hypothesis

If WinRM fails because of authorization, switching to the same assumption repeatedly does not solve the underlying problem.

### Losing track of movement history

Maintain a movement record throughout the assessment.

---

## Completion Criteria

Windows-to-Windows movement is complete when you can document:

* source host
* source identity
* source privileges
* target host
* target role
* target reachability
* available remote services
* authentication material
* authentication result
* authorization result
* movement method
* resulting access
* new identity
* new privileges
* new network position
* newly discovered targets
* next movement path

If movement fails, the failure should be classified and recorded rather than simply abandoned.

---

## What Next?

Continue with the Linux-to-Linux movement workflow:

**[`Linux-to-Linux.md`](Linux-to-Linux.md)**

This will apply the same decision model to Linux systems, with SSH as the primary remote-access path.
