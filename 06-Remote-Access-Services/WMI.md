# WMI

Windows Management Instrumentation (WMI) is a Windows management framework that exposes information and management operations for operating system components, services, processes, users, hardware, and applications.

In a lateral movement workflow, WMI matters because an authorized identity may be able to query or manage a remote Windows system through WMI. Remote WMI commonly relies on **DCOM/RPC**, so WMI movement is closely connected to RPC reachability, authentication, authorization, and Windows firewall policy.

The goal is not simply to determine whether WMI exists. The goal is to determine:

> **Can my current position use WMI to obtain useful information or authorized remote management access to another Windows host?**

---

## Where WMI Fits

The normal movement chain is:

```text
Current Host
    ↓
Current Identity
    ↓
Target Discovery
    ↓
Target Reachability
    ↓
RPC/WMI Availability
    ↓
Authentication
    ↓
Authorization
    ↓
Controlled WMI Query
    ↓
Validate Access
    ↓
New Position
    ↓
Re-enumerate
```

WMI should therefore be investigated after you already know:

* your current host
* your current identity
* the target host
* the target's network location
* whether RPC-related connectivity is possible
* what authentication material or existing access you have

---

## WMI vs WinRM

WMI and WinRM are related to Windows remote management but are not the same thing.

| Technology | Primary Role                     | Common Network Dependency |
| ---------- | -------------------------------- | ------------------------- |
| WMI        | Windows management framework/API | DCOM/RPC                  |
| WinRM      | WS-Management remote management  | TCP 5985/5986             |
| RDP        | Interactive graphical session    | TCP 3389                  |
| SMB        | File/resource sharing            | TCP 445                   |
| RPC        | Remote procedure communication   | TCP 135 + dynamic RPC     |

A common mistake is assuming:

```text
WinRM unavailable → WMI unavailable
```

That does not necessarily follow.

A host may expose one management path while another is unavailable because of firewall rules, service configuration, authentication policy, or authorization.

---

## WMI and RPC Relationship

Remote WMI commonly uses Windows DCOM, which depends on RPC.

A simplified relationship is:

```text
Your Host
    ↓
Target TCP 135
    ↓
RPC Endpoint Mapper
    ↓
Dynamic RPC Communication
    ↓
DCOM
    ↓
WMI
    ↓
Requested Management Operation
```

Therefore:

> **WMI failure can actually be an RPC, firewall, authentication, or authorization problem rather than a WMI problem.**

This distinction is important when troubleshooting.

---

## Requirements

Before attempting remote WMI access, establish four things:

| Requirement    | Question                                                         |
| -------------- | ---------------------------------------------------------------- |
| Target         | Which Windows host are you testing?                              |
| Reachability   | Can you reach the required RPC infrastructure?                   |
| Authentication | Which identity will authenticate?                                |
| Authorization  | Is that identity allowed to perform the requested WMI operation? |

Do not assume that possessing valid credentials automatically provides WMI access.

---

## Step 1 — Identify the Target

Start with a specific target rather than attempting indiscriminate remote management.

Useful sources include:

* previously discovered hosts
* DNS records
* AD computer objects
* SMB enumeration
* existing connections
* network routes
* known infrastructure roles
* previously identified Windows servers

Example:

```text
TARGET = SERVER01
```

Record why the target is relevant.

For example:

```text
Target: SERVER01
Reason: Windows server discovered on reachable subnet
Potential path: WMI
```

---

## Step 2 — Confirm the Target Is Reachable

First determine whether the target is reachable at the network layer.

From Windows:

```powershell
Test-Connection SERVER01 -Count 2
```

Check the RPC endpoint mapper:

```powershell
Test-NetConnection SERVER01 -Port 135
```

From Linux:

```bash
nc -vz SERVER01 135
```

A successful connection to TCP 135 is useful evidence, but it does **not** prove that WMI access will succeed.

The correct interpretation is:

```text
135 reachable
    ↓
RPC endpoint mapper is reachable
    ↓
Investigate RPC/DCOM/WMI access
```

Not:

```text
135 reachable
    ↓
WMI access guaranteed
```

---

## Step 3 — Check Your Current Identity

Before testing authentication, understand who you are.

```cmd
whoami
```

```cmd
whoami /user
```

```cmd
whoami /groups
```

For domain context:

```cmd
echo %USERDOMAIN%
echo %USERDNSDOMAIN%
```

PowerShell:

```powershell
[System.Security.Principal.WindowsIdentity]::GetCurrent().Name
```

Determine whether the identity is:

* local
* domain-based
* service-related
* administrative
* delegated
* unknown

This matters because WMI authorization depends heavily on the security context being used.

---

## Step 4 — Understand the Target's WMI Context

If you have authorized access to the target through another channel, first establish whether WMI is functioning locally.

Check the WMI service:

```powershell
Get-Service Winmgmt
```

A normal result may look conceptually like:

```text
Status   Name      DisplayName
------   ----      -----------
Running  Winmgmt   Windows Management Instrumentation
```

The service being present and running is useful evidence.

However:

```text
Winmgmt running
```

does not automatically mean:

```text
Remote WMI access allowed
```

Remote access can still fail because of:

* firewall rules
* DCOM configuration
* RPC restrictions
* authentication policy
* authorization
* network segmentation
* security policy

---

## Step 5 — Validate WMI Locally

On a Windows system where you are authorized to perform local enumeration:

```powershell
Get-CimInstance Win32_ComputerSystem
```

Operating system information:

```powershell
Get-CimInstance Win32_OperatingSystem
```

Running processes:

```powershell
Get-CimInstance Win32_Process
```

These queries establish that the local WMI/CIM subsystem is functioning and provide examples of the type of information that WMI can expose.

For lateral movement, the important question becomes:

> Can the same type of controlled query be performed against the target?

---

## Step 6 — Test Remote WMI Access

PowerShell CIM can be used for controlled remote queries where authorization exists.

For example:

```powershell
Get-CimInstance -ClassName Win32_ComputerSystem -ComputerName SERVER01
```

This is useful because it provides a relatively low-impact validation of:

* target reachability
* remote management communication
* authentication
* authorization
* WMI functionality

If successful, examine the returned information.

For example:

```text
Name
Domain
Manufacturer
Model
```

The important result is not merely the information itself.

The important result is:

```text
Current Identity
        ↓
Successfully authenticated
        ↓
Remote WMI query accepted
        ↓
Target management access confirmed
```

---

## Step 7 — Interpret the Result

### Case A — Query succeeds

You have evidence that your current security context can perform the tested WMI operation.

Record:

```text
Target: SERVER01
Protocol: WMI
Authentication: Successful
Authorization: Query accepted
Access: Remote WMI management
```

Then determine what authorized management operations are available.

Do not stop at:

```text
WMI works
```

The next question is:

> **What useful access or information does this provide?**

---

### Case B — Authentication fails

Possible causes include:

* incorrect credentials
* wrong account context
* domain authentication problem
* authentication protocol mismatch
* expired or invalid authentication material
* target policy

Check the identity and authentication assumptions before changing techniques.

---

### Case C — RPC connection fails

If TCP 135 is unavailable, investigate:

```text
Network path
    ↓
Firewall
    ↓
RPC endpoint mapper
    ↓
Dynamic RPC
```

Possible causes include:

* network segmentation
* host firewall
* intermediary firewall
* RPC filtering
* incorrect target
* unavailable service

Do not immediately conclude that WMI itself is disabled.

---

### Case D — TCP 135 works but WMI query fails

This is an important distinction.

Possible causes include:

* dynamic RPC connectivity
* DCOM restrictions
* Windows firewall rules
* authentication failure
* authorization failure
* WMI configuration
* security policy

The correct next step is to identify which layer failed.

---

### Case E — Authentication succeeds but operation is denied

This indicates:

```text
Authentication ≠ Authorization
```

The identity is recognized, but the requested management operation is not permitted.

Investigate:

* local group membership
* domain group membership
* delegated permissions
* WMI namespace permissions
* DCOM permissions
* local security policy
* domain policy

Do not treat valid credentials as equivalent to administrative access.

---

## Step 8 — Understand Authorization

Remote WMI access depends on the requested operation and the target's security configuration.

Authorization can be influenced by:

* local administrators
* domain groups
* WMI namespace permissions
* DCOM permissions
* UAC-related behavior
* local security policy
* domain security policy
* firewall policy

Therefore, always distinguish:

```text
Credential
    ↓
Authentication
    ↓
WMI Access
    ↓
Specific Operation Authorization
```

A successful read-only query does not necessarily mean that every management operation is permitted.

---

## WMI Information That Can Aid Movement

Authorized WMI queries can expose useful information about a target.

Examples include:

| Information           | Why It Matters                          |
| --------------------- | --------------------------------------- |
| Hostname              | Confirms target identity                |
| Domain                | Establishes trust context               |
| OS version            | Identifies system type                  |
| Architecture          | Describes target platform               |
| Logged-on context     | May reveal active users                 |
| Processes             | Shows applications and services         |
| Services              | Identifies management/application roles |
| Network configuration | Reveals additional network position     |
| Installed software    | Helps understand target role            |

The objective is to convert the information into a movement decision.

For example:

```text
WMI access
    ↓
Target is a domain server
    ↓
Target has additional network interfaces
    ↓
Target may provide access to another network
    ↓
Reassess network position
    ↓
Discover additional targets
```

---

## WMI as a Movement Path

WMI can provide more than information depending on the authorization context and the management operations permitted by the environment.

The workflow should therefore be:

```text
Discover WMI
    ↓
Confirm RPC reachability
    ↓
Identify authentication context
    ↓
Validate remote WMI query
    ↓
Determine authorized operations
    ↓
Use only the required management capability
    ↓
Validate resulting access
    ↓
Re-enumerate target
```

Avoid jumping directly from:

```text
WMI available
```

to:

```text
remote execution
```

First establish what your identity is actually authorized to do.

---

## WMI vs CIM

Modern PowerShell commonly uses the CIM cmdlets for management operations.

Examples:

```powershell
Get-CimInstance Win32_ComputerSystem
```

```powershell
Get-CimInstance Win32_OperatingSystem
```

```powershell
Get-CimInstance Win32_Process
```

For remote queries:

```powershell
Get-CimInstance -ClassName Win32_ComputerSystem -ComputerName SERVER01
```

Older `wmic.exe` usage is not the preferred approach on modern Windows systems because the WMIC command-line utility has been deprecated.

For this workflow, prefer current PowerShell/CIM tooling where available.

---

## Existing WMI-Related Evidence

If you already have access to a Windows host, inspect information that may explain the management environment.

Current system:

```powershell
Get-CimInstance Win32_ComputerSystem
```

WMI service:

```powershell
Get-Service Winmgmt
```

Network configuration:

```powershell
Get-NetIPConfiguration
```

Listening connections:

```powershell
Get-NetTCPConnection -State Listen
```

The objective is to build a picture of:

```text
Host
 ├── Identity
 ├── Domain
 ├── WMI state
 ├── Network
 ├── Services
 └── Potential targets
```

---

## WMI and Domain Environments

WMI is particularly relevant in Windows domain environments because administrative and delegated management operations may span multiple systems.

Useful context includes:

* current domain
* target domain
* account type
* group membership
* authentication mechanism
* target role
* network segmentation
* management policies

A successful WMI query against one system does not automatically prove equivalent access to every system in the domain.

Always validate access against the specific target.

---

## Failure Classification

When WMI fails, classify the failure before changing techniques.

| Failure                         | Investigate                                  |
| ------------------------------- | -------------------------------------------- |
| Target unreachable              | Routing, segmentation, target availability   |
| TCP 135 blocked                 | Firewall, RPC filtering, network path        |
| RPC unavailable                 | RPC service/configuration, firewall          |
| Authentication failure          | Identity, credentials, authentication policy |
| Access denied                   | Authorization, groups, WMI/DCOM permissions  |
| WMI service unavailable         | `Winmgmt`, target configuration              |
| Local query works, remote fails | RPC/DCOM/firewall/authz                      |
| One target works, another fails | Target-specific configuration/policy         |

This prevents random troubleshooting.

---

## Practical Windows Workflow

Use the following sequence during an authorized assessment:

```powershell
# 1. Identify current identity
whoami
whoami /user
whoami /groups

# 2. Identify domain context
echo $env:USERDOMAIN
echo $env:USERDNSDOMAIN

# 3. Test RPC reachability
Test-NetConnection SERVER01 -Port 135

# 4. Check local WMI service when you have local access
Get-Service Winmgmt

# 5. Validate local WMI/CIM
Get-CimInstance Win32_ComputerSystem

# 6. Validate remote WMI with a controlled query
Get-CimInstance -ClassName Win32_ComputerSystem -ComputerName SERVER01
```

Interpret each result before moving to the next step.

---

## Decision Tree

```text
Do I have a specific Windows target?
        │
       No
        ↓
Return to Target Discovery
        │
       Yes
        ↓
Can I reach TCP 135?
        │
   ┌────┴────┐
   No       Yes
   │          │
Investigate   ↓
network/     Can remote WMI
firewall     communication proceed?
              │
         ┌────┴────┐
        No        Yes
        │           │
   Investigate      ↓
   RPC/DCOM/      Does authentication
   firewall       succeed?
                    │
               ┌────┴────┐
              No        Yes
              │           │
       Investigate        ↓
       identity/       Is the requested
       auth policy     operation authorized?
                          │
                     ┌────┴────┐
                    No        Yes
                    │           │
             Investigate        ↓
               authz       Validate useful
                            management access
                                  │
                                  ↓
                         Re-enumerate target
                                  │
                                  ↓
                         Continue the chain
```

---

## WMI Investigation Record

Record findings instead of relying on memory.

| Field              | Finding              |
| ------------------ | -------------------- |
| Target             | `SERVER01`           |
| IP                 | `10.10.10.20`        |
| TCP 135            | Reachable / Blocked  |
| Current Identity   | `DOMAIN\user`        |
| Domain             | `DOMAIN.LOCAL`       |
| Winmgmt            | Running / Unknown    |
| Authentication     | Success / Failure    |
| WMI Query          | Success / Failure    |
| Authorization      | Allowed / Denied     |
| Target Role        | Unknown / Identified |
| Useful Information | ...                  |
| Next Action        | ...                  |

This creates a reusable movement record.

---

## Common Mistakes

### Treating TCP 135 as WMI access

```text
135 open
```

only establishes RPC endpoint mapper reachability.

It does not establish WMI authorization.

### Assuming WMI equals WinRM

They are different management technologies with different network dependencies.

### Assuming valid credentials are enough

Authentication and authorization are separate.

### Ignoring dynamic RPC

TCP 135 may be reachable while later RPC communication is blocked.

### Testing only one layer

A complete WMI assessment checks:

```text
Network
→ RPC
→ DCOM
→ Authentication
→ Authorization
→ WMI
```

### Jumping directly to execution

First establish what management operations your current identity is authorized to perform.

### Forgetting the new network position

A remote management connection may reveal:

* another subnet
* another interface
* new hosts
* new services
* new trust relationships
* new authentication context

Return to the position-assessment workflow after successful movement.

---

## Completion Criteria

Consider WMI investigation complete when you can answer:

* What target am I testing?
* Can I reach its RPC infrastructure?
* Is WMI available?
* Which identity am I using?
* What authentication context applies?
* Did authentication succeed?
* What WMI operation did I test?
* Was that operation authorized?
* What information or management capability did I gain?
* Did the target reveal a new network or trust position?
* What should I investigate next?

If the answers are incomplete, continue enumeration rather than assuming WMI is unusable.

---

## What Next?

If WMI depends on RPC and you need to understand the underlying communication path, continue with:

**[`RPC.md`](RPC.md)**

The next step is to understand how RPC fits into Windows remote management and lateral movement, and how to distinguish RPC reachability problems from authentication and authorization failures.
