# RPC

Remote Procedure Call (RPC) is a communication mechanism that allows Windows systems and applications to request operations from services running on another system.

In a lateral movement workflow, RPC is important because several Windows management and administration technologies depend on it. Understanding RPC helps determine whether a remote management path is actually reachable and why a service may fail even when its primary port appears open.

The goal is not simply to find TCP 135 open. The goal is to determine:

> **Can the current host communicate with the RPC infrastructure required by a target service, and does the current identity have the authorization needed to use that service?**

---

## Where RPC Fits

RPC commonly appears underneath other Windows services:

```text
Current Host
    ↓
Target
    ↓
RPC Endpoint Mapper
    ↓
RPC Endpoint / Dynamic RPC
    ↓
Windows Service
    ↓
Authentication
    ↓
Authorization
    ↓
Management Operation
```

RPC can therefore affect:

* WMI
* DCOM
* Windows management operations
* certain administrative interfaces
* service-specific remote procedures
* Active Directory-related operations
* other Windows components

RPC is often a **dependency**, rather than the final lateral-movement technique itself.

---

## RPC vs TCP 135

A common misconception is:

```text
TCP 135 open = RPC access
```

A better model is:

```text
TCP 135
   ↓
RPC Endpoint Mapper
   ↓
Discover/locate required RPC endpoint
   ↓
Connect to required endpoint
   ↓
Perform RPC operation
```

TCP 135 commonly hosts the RPC Endpoint Mapper.

After an application contacts the endpoint mapper, RPC may direct the client toward another endpoint, often using a dynamically assigned RPC port.

Therefore:

> **A reachable TCP 135 does not guarantee that the complete RPC communication path is available.**

---

## RPC and Dynamic Ports

Modern Windows RPC commonly uses a range of dynamic ports in addition to TCP 135.

Conceptually:

```text
Client
  │
  │ TCP 135
  ▼
RPC Endpoint Mapper
  │
  │ endpoint information
  ▼
Dynamic RPC Port
  │
  ▼
Requested Service
```

This creates an important troubleshooting situation:

```text
135/tcp = reachable
dynamic RPC = blocked
```

The result may be:

```text
RPC operation fails
```

even though the first connectivity test succeeded.

The exact dynamic-port behavior depends on Windows configuration and the service involved.

---

## Requirements

Before investigating RPC, identify:

| Requirement     | Question                                                     |
| --------------- | ------------------------------------------------------------ |
| Target          | Which Windows system am I testing?                           |
| Reachability    | Can I reach the target?                                      |
| Endpoint Mapper | Is TCP 135 reachable?                                        |
| Dynamic RPC     | Can the required RPC endpoint be reached?                    |
| Authentication  | Which identity is being used?                                |
| Authorization   | Is that identity allowed to perform the requested operation? |
| Service         | Which higher-level service depends on RPC?                   |

---

## Step 1 — Identify the Target

Start with a known target.

Example:

```text
Target: SERVER01
IP: 10.10.10.20
Role: Windows Server
Reason: Previously discovered management target
```

Do not begin by scanning every possible system without a reason.

Use information already obtained from:

* target discovery
* DNS
* SMB
* AD enumeration
* network routes
* existing connections
* previous movement

---

## Step 2 — Confirm Basic Reachability

From Windows:

```powershell
Test-Connection SERVER01 -Count 2
```

Then test TCP 135:

```powershell
Test-NetConnection SERVER01 -Port 135
```

From Linux:

```bash
nc -vz SERVER01 135
```

Interpretation:

### TCP 135 fails

Possible causes:

* target unavailable
* network segmentation
* routing problem
* firewall
* RPC filtering
* incorrect target

Do not immediately conclude that the Windows service you are investigating is unavailable.

### TCP 135 succeeds

You have evidence that the RPC Endpoint Mapper is reachable.

Continue investigating the actual RPC-dependent service.

---

## Step 3 — Identify Your Current Identity

RPC operations are performed within a security context.

Check:

```cmd
whoami
```

```cmd
whoami /user
```

```cmd
whoami /groups
```

Domain context:

```cmd
echo %USERDOMAIN%
echo %USERDNSDOMAIN%
```

PowerShell:

```powershell
[System.Security.Principal.WindowsIdentity]::GetCurrent().Name
```

Record the identity before testing access.

---

## Step 4 — Identify the Higher-Level Service

RPC by itself does not tell you what you can do.

Ask:

> **Which Windows service or management interface am I trying to reach through RPC?**

Examples:

| Higher-Level Function | RPC Relevance                 |
| --------------------- | ----------------------------- |
| WMI                   | Commonly uses DCOM/RPC        |
| DCOM                  | Built on RPC                  |
| Windows management    | Frequently RPC-dependent      |
| Service management    | May involve RPC               |
| Remote administration | Can involve RPC               |
| AD-related operations | Multiple RPC-based interfaces |

This distinction is important because:

```text
RPC reachable
```

does not mean:

```text
Every RPC-dependent operation is authorized
```

---

## Step 5 — Check RPC-Related Services Locally

On an authorized Windows host, inspect relevant services.

RPC Endpoint Mapper:

```powershell
Get-Service RpcEptMapper
```

RPC service:

```powershell
Get-Service RpcSs
```

DCOM Server Process Launcher:

```powershell
Get-Service DcomLaunch
```

These services provide useful local context.

A service being present and running locally does not automatically establish that remote RPC communication is permitted through the network.

---

## Step 6 — Examine Listening Ports

On Windows:

```powershell
Get-NetTCPConnection -State Listen
```

Or:

```cmd
netstat -ano
```

You may identify:

```text
135
```

and other listening ports.

Remember:

> A listening port tells you that something is listening. It does not by itself prove what operations your identity can perform.

If you identify a dynamic RPC port, correlate it with the relevant service and configuration rather than treating every high-numbered port as an independent attack path.

---

## Step 7 — Understand RPC Endpoint Mapping

The Endpoint Mapper helps clients locate RPC services.

Simplified:

```text
Client
   ↓
"I need RPC interface X"
   ↓
TCP 135
   ↓
Endpoint Mapper
   ↓
"Interface X is available at endpoint Y"
   ↓
Client connects to endpoint Y
```

This explains why testing only:

```text
Test-NetConnection SERVER01 -Port 135
```

may not be enough to validate the complete communication path.

---

## RPC and WMI

The previous WMI workflow demonstrated this relationship.

```text
WMI
 ↓
DCOM
 ↓
RPC
 ↓
Endpoint Mapper / RPC Endpoint
```

Therefore, if a remote WMI query fails:

```text
WMI query failed
```

investigate:

```text
Network
 ↓
TCP 135
 ↓
Dynamic RPC
 ↓
DCOM
 ↓
Authentication
 ↓
Authorization
 ↓
WMI
```

This prevents incorrect conclusions such as:

```text
"WMI is disabled."
```

when the actual problem is a firewall blocking RPC communication.

---

## RPC and Authentication

RPC communication can operate within different Windows authentication contexts.

Depending on the service and environment, authentication may involve mechanisms such as:

* Kerberos
* NTLM
* local account authentication
* domain authentication
* certificate-based mechanisms for some management scenarios

The important workflow is:

```text
Identity
   ↓
Authentication mechanism
   ↓
RPC connection
   ↓
Service authorization
```

Do not assume that successful network communication means authentication succeeded.

---

## Authentication vs Authorization

This distinction is critical.

### Authentication

Question:

> Who are you?

### Authorization

Question:

> What are you allowed to do?

A useful model is:

```text
TCP 135 reachable
       ↓
RPC communication possible
       ↓
Authentication succeeds
       ↓
Authorization checked
       ↓
Requested operation
```

Each layer can fail independently.

---

## Step 8 — Validate a Specific RPC-Dependent Operation

Do not attempt random RPC operations.

Choose the higher-level service that you are investigating.

For example, if the movement hypothesis is WMI:

```text
Target
  ↓
TCP 135
  ↓
RPC
  ↓
WMI
  ↓
Controlled read-only query
```

If the query succeeds, RPC is functioning sufficiently for that operation.

If it fails, classify the failure.

This is more useful than simply recording:

```text
135 open
```

---

## Failure Classification

Use the following model:

| Result                                           | Likely Investigation Area             |
| ------------------------------------------------ | ------------------------------------- |
| Target unreachable                               | Routing / segmentation / availability |
| TCP 135 blocked                                  | Firewall / network policy             |
| TCP 135 open but RPC operation fails             | Dynamic RPC / firewall / service      |
| RPC connects but authentication fails            | Identity / credentials / auth policy  |
| Authentication succeeds but access denied        | Authorization / permissions           |
| Local RPC works but remote fails                 | Firewall / network / remote policy    |
| One target works and another fails               | Target-specific configuration         |
| WMI fails but other RPC-dependent function works | WMI/DCOM/configuration-specific issue |

This classification tells you what to investigate next.

---

## Step 9 — Check Network Segmentation

RPC is particularly sensitive to network boundaries because communication may involve more than one port.

Example:

```text
Workstation
    │
    │ TCP 135
    ▼
Firewall
    │
    │ dynamic RPC
    ▼
Server
```

A firewall may permit TCP 135 but restrict the dynamic RPC range.

This can produce:

```text
135 = open
RPC operation = failure
```

Therefore, compare the result with the network architecture you discovered earlier.

---

## RPC in Active Directory

RPC is widely used in Windows domain environments.

It may support communication between:

* domain-joined hosts
* management systems
* domain controllers
* administrative tools
* Windows services

This makes RPC relevant when investigating movement between Windows systems in an AD environment.

However:

```text
RPC available
```

does not mean:

```text
Domain administrative access
```

The identity, service, operation, and authorization context still matter.

---

## RPC and Host Roles

A target's role changes the importance of RPC.

Examples:

### Workstation

Possible relevance:

* WMI
* remote management
* administrative operations
* user/session investigation

### Member Server

Possible relevance:

* service administration
* application management
* WMI
* remote administration

### Domain Controller

Possible relevance:

* domain management operations
* directory-related communication
* authentication infrastructure
* administrative interfaces

A host role should therefore influence your next investigation.

---

## Existing RPC Evidence

If you already have access to a Windows host, collect:

```powershell
hostname
```

```powershell
Get-CimInstance Win32_ComputerSystem
```

```powershell
Get-NetIPConfiguration
```

```powershell
Get-NetTCPConnection -State Listen
```

```powershell
Get-Service RpcEptMapper
```

```powershell
Get-Service RpcSs
```

This creates a baseline:

```text
Host
 ├── Identity
 ├── Domain
 ├── Network
 ├── RPC services
 ├── Listening endpoints
 └── Potential RPC-dependent services
```

---

## RPC Investigation Workflow

A practical sequence is:

```text
1. Identify target
       ↓
2. Confirm basic network reachability
       ↓
3. Test TCP 135
       ↓
4. Identify RPC-dependent service
       ↓
5. Identify current identity
       ↓
6. Determine authentication context
       ↓
7. Validate the specific service
       ↓
8. Determine authorization
       ↓
9. Classify failure or confirm access
       ↓
10. Validate the resulting position
       ↓
11. Re-enumerate the target
```

Do not stop at step 3.

---

## Practical Windows Workflow

Example:

```powershell
# 1. Identify current identity
whoami
whoami /user
whoami /groups

# 2. Test basic connectivity
Test-Connection SERVER01 -Count 2

# 3. Test RPC Endpoint Mapper
Test-NetConnection SERVER01 -Port 135

# 4. Inspect local RPC services where authorized
Get-Service RpcEptMapper
Get-Service RpcSs
Get-Service DcomLaunch

# 5. Review local listening endpoints
Get-NetTCPConnection -State Listen

# 6. Test the specific higher-level service
# Example: controlled WMI query
Get-CimInstance -ClassName Win32_ComputerSystem -ComputerName SERVER01
```

The final command is only an example of validating an RPC-dependent service. Use the appropriate service-specific workflow for the actual movement path.

---

## Decision Tree

```text
Do I have a specific target?
        │
       No
        ↓
Return to Target Discovery
        │
       Yes
        ↓
Can I reach the target?
        │
   ┌────┴────┐
   No       Yes
   │          │
Investigate   ↓
network      Is TCP 135 reachable?
              │
         ┌────┴────┐
        No        Yes
        │           │
   Investigate      ↓
   network/       Which RPC-dependent
   firewall       service matters?
                       │
                       ↓
                 Can that service
                 communicate remotely?
                       │
                  ┌────┴────┐
                 No        Yes
                 │           │
          Investigate        ↓
          RPC/dynamic     Does authentication
          ports/firewall  succeed?
                              │
                         ┌────┴────┐
                        No        Yes
                        │           │
                 Investigate        ↓
                 identity/auth   Is the operation
                                  authorized?
                                      │
                                 ┌────┴────┐
                                No        Yes
                                │           │
                           Investigate      ↓
                             authz      Validate access
                                           │
                                           ↓
                                  Re-enumerate target
                                           │
                                           ↓
                                  Continue the chain
```

---

## Common Mistakes

### Assuming port 135 is the complete RPC path

TCP 135 is commonly the Endpoint Mapper, not necessarily the final RPC endpoint.

### Treating every high port as an attack path

Dynamic RPC ports should be interpreted in the context of the RPC service and environment.

### Confusing RPC with a specific application

RPC is a communication mechanism used by many Windows components.

### Ignoring the higher-level service

Always ask:

> What am I trying to accomplish through RPC?

### Assuming authentication equals administration

A successfully authenticated account may still lack authorization for the requested operation.

### Ignoring network segmentation

RPC may require communication beyond TCP 135.

### Changing techniques before classifying failure

Determine whether the problem is:

```text
Network
→ RPC
→ Service
→ Authentication
→ Authorization
```

before switching movement paths.

### Forgetting to reassess after successful access

A successful remote management operation may reveal:

* new hosts
* new interfaces
* new routes
* new users
* new services
* new trust relationships

Return to the position-assessment workflow.

---

## Completion Criteria

RPC investigation is complete when you can answer:

* What target am I testing?
* Can I reach the target?
* Is TCP 135 reachable?
* Which RPC-dependent service am I investigating?
* Is the required RPC communication working?
* Which identity am I using?
* What authentication mechanism applies?
* Did authentication succeed?
* Is the requested operation authorized?
* What useful access or information did I obtain?
* Did the target reveal a new network position?
* What should I investigate next?

If you cannot answer these questions, continue investigating the relevant layer.

---

## What Next?

After understanding the RPC layer, the final remote-access service in this section is:

**[`SSH.md`](SSH.md)**

SSH provides the primary remote-access path for many Linux systems and is also available on modern Windows systems.
