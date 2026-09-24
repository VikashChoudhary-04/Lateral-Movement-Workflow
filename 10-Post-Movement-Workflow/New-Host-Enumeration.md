# New-Host Enumeration

A successful lateral movement creates a new assessment position.

The new host must be treated as a fresh starting point rather than simply as another shell.

The objective is to determine:

* where you are
* who you are
* what the host can reach
* what services are available
* what trust relationships exist
* what credentials or access material are present
* what new targets are visible
* whether the host provides a better position for continuing the assessment

The workflow is:

```text
New Host Access
      ↓
Confirm Host + Identity
      ↓
Enumerate System
      ↓
Enumerate Network
      ↓
Enumerate Services
      ↓
Enumerate Domain / Trust Context
      ↓
Reassess Credentials / Existing Access
      ↓
Identify New Targets
      ↓
Compare With Previous Position
      ↓
Select Next Action
```

## Objective

Rebuild the situational picture from the newly accessed host.

The key question is:

> What can I discover or reach from this host that I could not discover or reach before?

Do not immediately repeat the same movement technique against every discovered system.

First understand the new position.

## 1. Confirm the New Host

### Windows

```powershell
hostname
ipconfig
```

Additional context:

```powershell
Get-ComputerInfo | Select-Object CsName, WindowsProductName, WindowsVersion, OsArchitecture
```

### Linux

```bash
hostname
uname -a
ip addr
```

Confirm:

```text
Hostname:
IP Address:
Operating System:
Architecture:
Interfaces:
```

This prevents confusion when multiple hosts have similar names or when movement occurred through an alias.

## 2. Confirm the Current Identity

### Windows

```powershell
whoami
whoami /user
whoami /groups
```

Where relevant:

```powershell
whoami /priv
```

### Linux

```bash
whoami
id
groups
```

Record:

```text
Identity:
UID / SID:
Local / Domain Context:
Groups:
Relevant Privileges:
```

The identity may be different from the identity used on the previous host.

## 3. Compare the Old and New Positions

Create a simple comparison:

| Property           | Previous Host | New Host |
| ------------------ | ------------- | -------- |
| Hostname           |               |          |
| IP                 |               |          |
| User               |               |          |
| Groups             |               |          |
| OS                 |               |          |
| Interfaces         |               |          |
| Routes             |               |          |
| Domain             |               |          |
| Reachable Networks |               |          |
| Known Services     |               |          |
| Known Credentials  |               |          |

The comparison reveals what changed after movement.

## 4. Enumerate Network Interfaces

Network interfaces determine which networks the new host participates in.

### Windows

```powershell
Get-NetIPConfiguration
```

or:

```powershell
ipconfig /all
```

### Linux

```bash
ip addr
```

Look for:

* multiple interfaces
* internal addresses
* management networks
* virtual interfaces
* VPN interfaces
* container networks
* unexpected subnets

Record each relevant network:

```text
Interface:
IP:
Prefix:
Gateway:
Network:
Purpose:
```

Do not assume every interface represents an independently reachable network.

## 5. Enumerate Routing

### Windows

```cmd
route print
```

PowerShell:

```powershell
Get-NetRoute
```

### Linux

```bash
ip route
```

Look for:

```text
Destination
Gateway
Interface
Metric
```

The routing table can reveal networks that were not visible from the original position.

Example:

```text
Previous Host
    ↓
10.10.10.0/24

New Host
    ↓
10.10.10.0/24
10.10.20.0/24
10.10.30.0/24
```

The newly visible networks may change the target-discovery strategy.

## 6. Enumerate DNS Context

DNS can provide useful information about the environment and help resolve internal systems.

### Windows

```powershell
Get-DnsClientServerAddress
```

```cmd
ipconfig /all
```

### Linux

Depending on the system:

```bash
cat /etc/resolv.conf
```

and:

```bash
resolvectl status
```

Determine:

```text
DNS Servers:
Search Domains:
DNS Domain:
Internal Resolution Available:
```

In Active Directory environments, DNS configuration can be particularly important because internal host and service discovery often depends on it.

## 7. Enumerate Domain Context

### Windows

```cmd
whoami /fqdn
```

```cmd
echo %USERDOMAIN%
echo %USERDNSDOMAIN%
```

Additional system information:

```powershell
Get-CimInstance Win32_ComputerSystem |
    Select-Object Name, Domain, PartOfDomain
```

Useful questions:

* Is the host domain joined?
* Which domain is it associated with?
* Is the current identity local or domain-based?
* Is a domain controller reachable?
* Are additional trusted domains known?

Do not confuse domain membership with authorization.

## 8. Enumerate Active Connections

Existing connections can reveal the host's role and relationships.

### Windows

```powershell
Get-NetTCPConnection
```

For a compact view:

```powershell
Get-NetTCPConnection |
    Select-Object LocalAddress,LocalPort,RemoteAddress,RemotePort,State
```

### Linux

```bash
ss -tunap
```

Look for connections involving:

* application servers
* databases
* file servers
* domain controllers
* management servers
* internal APIs
* other workstations

A connection is evidence of communication, not automatically evidence of reusable credentials or authorization.

## 9. Enumerate Listening Services

Determine what services the new host exposes.

### Windows

```powershell
Get-NetTCPConnection -State Listen
```

### Linux

```bash
ss -lntup
```

Record:

```text
Port:
Protocol:
Service:
Bind Address:
Likely Purpose:
```

Pay particular attention to services that were not reachable from the original position.

## 10. Enumerate Host Services

### Windows

```powershell
Get-Service
```

For running services:

```powershell
Get-Service |
    Where-Object {$_.Status -eq "Running"}
```

### Linux

On systemd systems:

```bash
systemctl list-units --type=service --state=running
```

Questions:

* What applications run here?
* Is this a server?
* Does it host a database?
* Does it provide file services?
* Is it a management system?
* Does it appear to be a jump host or infrastructure host?

Service enumeration should help determine the host's role.

## 11. Identify the Host's Role

Combine the evidence collected so far.

Possible roles include:

```text
Workstation
File Server
Application Server
Database Server
Domain Controller
Management Server
Jump Host
Development Server
Monitoring Server
Web Server
```

Do not determine the role from hostname alone.

Use multiple signals:

```text
Hostname
+
Interfaces
+
Routes
+
Listening Ports
+
Running Services
+
Domain Context
+
Active Connections
```

## 12. Enumerate Users and Groups

### Windows

```cmd
whoami /groups
```

Where appropriate:

```cmd
net user
```

```cmd
net localgroup
```

### Linux

```bash
getent passwd
```

```bash
getent group
```

The goal is to understand the local account and group structure.

Avoid unnecessary account modification or authentication attempts.

## 13. Reassess Remote Access Services

Determine what remote services the new host provides.

For Windows, consider:

```text
SMB
WinRM
RDP
RPC / WMI
```

For Linux, consider:

```text
SSH
SMB / Samba
Other authorized management services
```

The presence of a service does not prove that your current identity can use it.

Use the workflow:

```text
Service Exists
      ↓
Reachable
      ↓
Authentication Possible
      ↓
Authorization Confirmed
      ↓
Useful Access?
```

## 14. Identify Internal Targets

Now compare the new host's network visibility with the previous host.

For example:

```text
Previous Position:
10.10.10.0/24

New Position:
10.10.10.0/24
10.10.20.0/24
```

The second network becomes a candidate for further discovery.

Identify targets using:

* routing information
* DNS
* existing connections
* known infrastructure
* authorized host discovery
* service discovery

Do not blindly scan every possible address.

Start with networks and systems supported by the assessment scope.

## 15. Prioritize New Targets

A newly discovered system should be evaluated based on evidence.

Useful factors include:

| Factor       | Question                                                        |
| ------------ | --------------------------------------------------------------- |
| Reachability | Can the new host communicate with it?                           |
| Service      | What service is exposed?                                        |
| Identity     | Do existing credentials map to it?                              |
| Role         | Is it infrastructure, application, file, or management related? |
| Access       | Is authentication possible?                                     |
| Value        | Does it provide useful additional visibility?                   |
| Scope        | Is it explicitly authorized?                                    |

The goal is not simply to maximize the number of compromised hosts.

The goal is to understand and demonstrate the authorized movement path.

## 16. Reassess Existing Credentials

A new host may contain different access material.

Relevant sources can include:

```text
Configuration Files
Environment Variables
Service Configuration
SSH Configuration
Credential Stores
Application Configuration
Existing Sessions
Authentication Tickets
Authorized Key Material
```

Credential reassessment is handled in detail in:

[`Credential-Reassessment.md`](Credential-Reassessment.md).

At this stage, identify **where to investigate**, not every possible credential extraction technique.

## 17. Reassess Existing Access

Determine whether the new identity already has access to additional systems.

Examples:

```text
SMB Shares
SSH Targets
WinRM Targets
RDP Targets
Internal Applications
Databases
Management Interfaces
```

Map:

```text
Identity
   ↓
Access Type
   ↓
Target
   ↓
Authorization
```

A valid credential is only useful when it maps to an authorized target and service.

## 18. Check for Domain and Trust Context

For Windows/AD environments, determine whether the new host provides additional domain visibility.

Useful commands include:

```cmd
whoami /domain
```

```cmd
nltest /dsgetdc:<domain>
```

Where appropriate:

```cmd
nltest /domain_trusts
```

Also examine:

```powershell
klist
```

The goal is to understand:

```text
Current Domain
     ↓
Domain Controller
     ↓
Kerberos Context
     ↓
Known Trust Relationships
     ↓
Potentially Relevant Targets
```

A discovered trust relationship does not automatically grant access to the trusted environment.

## 19. Enumerate the Local Security Context

### Windows

```cmd
whoami /priv
```

```cmd
whoami /groups
```

### Linux

```bash
id
```

Where applicable:

```bash
sudo -l
```

The objective is to determine whether the new host provides a materially different privilege context.

For example:

```text
Previous:
Standard Domain User

New:
Local Administrator
```

would represent a significant change in access.

But this must be demonstrated through actual group and privilege evidence.

## 20. Identify Potential Pivot Capability

A host becomes particularly useful as a pivot when it can reach networks unavailable from the original position.

Evaluate:

```text
Multiple Interfaces
+
Additional Routes
+
Internal Reachability
+
Useful Remote Services
```

Conceptually:

```text
Original Position
       |
       | cannot reach
       v
Internal Network
       ^
       |
       | reachable from
       |
New Host
```

If this condition exists, the host may provide a new path into the internal environment.

See:

[`09-Pivoting-and-Internal-Network-Access/`](../09-Pivoting-and-Internal-Network-Access/)

for the pivoting workflow.

## 21. Build a New Position Record

Create a record for every newly accessed host.

```text
Host:
IP:
OS:
Hostname:
Current User:
Groups:
Privilege Level:
Domain:
Interfaces:
Routes:
DNS:
Listening Services:
Remote Services:
Active Connections:
Reachable Networks:
Known Credentials / Access:
Potential Targets:
Potential Pivot:
Next Action:
```

This prevents the assessment from becoming a sequence of disconnected sessions.

## 22. Compare Positions

Use a simple position map:

```text
Position A
Host: WORKSTATION-01
User: DOMAIN\user
Networks:
- 10.10.10.0/24

        |
        | WinRM
        v

Position B
Host: SERVER-01
User: DOMAIN\user
Networks:
- 10.10.10.0/24
- 10.10.20.0/24

        |
        | New visibility
        v

Candidate Internal Targets
- 10.10.20.x
- 10.10.20.y
```

This makes the movement chain visible.

## 23. Decision Framework

After enumeration, ask:

### Did the new host reveal a new network?

```text
Yes → Investigate authorized targets in that network.
No  → Continue evaluating credentials and services.
```

### Did the new host reveal new access material?

```text
Yes → Map it to compatible targets/services.
No  → Continue target and service analysis.
```

### Did the new host reveal useful remote services?

```text
Yes → Determine whether the current identity is authorized to use them.
No  → Continue environmental enumeration.
```

### Did the new host provide a stronger network position?

```text
Yes → Consider it as a pivot candidate.
No  → Continue from the current position.
```

### Did enumeration reveal nothing useful?

```text
Reassess:
- Identity
- Credentials
- Network
- DNS
- Services
- Scope
```

Do not repeat the same movement attempt without new evidence.

## 24. Avoid Losing the Original Position

Maintain a record of every successful position.

```text
Position 1
   ↓
Position 2
   ↓
Position 3
```

For each position record:

```text
Host
Identity
Access Method
Network
Useful Credentials
Reachable Targets
Pivot Capability
```

This allows the assessment to distinguish:

```text
Current Position
```

from:

```text
Previously Validated Position
```

and makes recovery easier if a session ends.

## 25. Stop Conditions

New-host enumeration should stop when:

* the relevant environment has been sufficiently mapped
* no new authorized targets are identified
* the assessment objective has been reached
* the next action would exceed scope
* further enumeration would create unnecessary impact
* available access provides no new meaningful information

More enumeration is not automatically better enumeration.

## 26. Failure Handling

### Host Information Is Incomplete

Check:

```text
Permissions
OS Tools
Network Configuration
Remote Session Type
```

### DNS Resolution Fails

Check:

```text
DNS Servers
Search Domain
Routing
Internal Resolver Reachability
```

### Network Appears Unreachable

Check:

```text
Interface
Route
Firewall
Service Port
Segmentation
```

### Domain Information Is Missing

Check:

```text
Domain Membership
Current Identity
DNS
Domain Controller Reachability
```

### Services Are Visible but Inaccessible

Separate:

```text
Reachability
Authentication
Authorization
```

Do not treat service visibility as usable access.

## 27. Evidence Checklist

Record at minimum:

```text
[ ] Hostname confirmed
[ ] IP address confirmed
[ ] Current identity confirmed
[ ] Privilege context identified
[ ] OS identified
[ ] Interfaces identified
[ ] Routes identified
[ ] DNS context identified
[ ] Domain context identified
[ ] Listening services identified
[ ] Active connections reviewed
[ ] Remote access services identified
[ ] New networks identified
[ ] Candidate targets identified
[ ] Existing access reassessed
[ ] Credential sources identified
[ ] Pivot capability considered
[ ] Next action selected
```

## 28. Complete New-Host Workflow

```text
Successful Movement
        ↓
Confirm Host
        ↓
Confirm Identity
        ↓
Enumerate OS
        ↓
Enumerate Interfaces
        ↓
Enumerate Routes
        ↓
Enumerate DNS
        ↓
Enumerate Domain / Trust
        ↓
Enumerate Services
        ↓
Review Connections
        ↓
Identify Host Role
        ↓
Reassess Credentials
        ↓
Reassess Existing Access
        ↓
Identify New Networks
        ↓
Identify New Targets
        ↓
Evaluate Pivot Capability
        ↓
Compare With Previous Position
        ↓
Choose Next Action
```

## Golden Rule

> Every successful movement should produce a new network, identity, credential, service, target, privilege, or other meaningful piece of information—or provide validated evidence that none was obtained.

Lateral movement is a chain of changing positions.

The quality of the assessment depends not only on reaching the next host, but on understanding what the new host makes possible.

## What Next?

Continue to [`Credential-Reassessment.md`](Credential-Reassessment.md).

That file focuses on reassessing the new host for authentication material and existing access that may enable the next movement step.
