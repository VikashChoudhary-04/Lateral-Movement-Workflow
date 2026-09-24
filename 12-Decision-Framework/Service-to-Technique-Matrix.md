# Service-to-Technique Matrix

This matrix connects a discovered remote service to the appropriate lateral-movement workflow.

The objective is not to treat an open port as an immediate movement opportunity.

Use:

```text
Service Discovery
      ↓
Identify Service
      ↓
Understand Authentication
      ↓
Identify Compatible Access
      ↓
Validate Authorization
      ↓
Establish Access
      ↓
Validate New Position
```

A discovered service is an entry point for investigation, not proof of access.

## Core Matrix

| Service          |                      Typical Port(s) | Primary Platform                    | Authentication Context                                                  | Possible Access                                | First Validation                                      |
| ---------------- | -----------------------------------: | ----------------------------------- | ----------------------------------------------------------------------- | ---------------------------------------------- | ----------------------------------------------------- |
| SMB              |                   445, sometimes 139 | Windows / file servers              | Local/domain credentials, NTLM, Kerberos depending on configuration     | File/resource access, administrative workflows | Confirm SMB reachability and share/access permissions |
| WinRM            |                           5985, 5986 | Windows                             | Windows authentication such as Kerberos/NTLM or configured alternatives | Remote PowerShell                              | Validate WinRM and remote authorization               |
| RDP              |                                 3389 | Windows                             | Windows account authentication, often with NLA                          | Interactive desktop session                    | Validate RDP reachability and logon authorization     |
| WMI              | RPC/DCOM, commonly 135 + dynamic RPC | Windows                             | Windows authentication                                                  | Remote management                              | Validate RPC/WMI availability and authorization       |
| RPC              |                    135 + dynamic RPC | Windows                             | Windows authentication                                                  | Management dependencies / RPC-based operations | Validate endpoint mapper and required RPC path        |
| SSH              |                22 or configured port | Linux/Unix and Windows with OpenSSH | Password or public-key authentication                                   | Remote shell                                   | Validate SSH service and account authorization        |
| HTTP/HTTPS       |          80, 443, or configured port | Any                                 | Application-specific                                                    | Web/application access                         | Identify application and authentication model         |
| Database service |                    Service-dependent | Any                                 | Database-specific                                                       | Database access                                | Identify database and validate account/role           |
| NFS              |                                 2049 | Linux/Unix                          | Configuration-dependent                                                 | File/resource access                           | Identify exports and access controls                  |

Ports shown above are common defaults, not guarantees.

## How to Read the Matrix

For every discovered service, answer:

```text id="9b2j7q"
Is the port reachable?
        ↓
What service is actually running?
        ↓
How does it authenticate?
        ↓
What identity do I have?
        ↓
Does my authentication material fit?
        ↓
What authorization is required?
        ↓
What access would successful use provide?
```

## 1. SMB

### Typical Ports

```text
TCP 445
TCP 139
```

TCP 445 is the common modern SMB transport.

### What SMB Can Provide

Depending on authorization:

* share enumeration
* file access
* resource access
* administrative share access
* Windows administrative workflows

SMB access does not automatically mean interactive command execution.

### Decision Path

```text id="t0q5yv"
SMB
 ↓
Enumerate Shares
 ↓
Identify Accessible Shares
 ↓
Determine Read / Write / Administrative Access
 ↓
Identify Relevant Resources
 ↓
Determine Whether Further Authorized Movement Is Possible
```

### Authentication Material

Potentially relevant:

* local Windows credentials
* domain credentials
* NTLM authentication material
* Kerberos authentication context

### First Validation

Windows:

```powershell id="j2x2sk"
Test-NetConnection TARGET -Port 445
```

Then, where authorized:

```powershell id="fyjq3s"
net view \\TARGET
```

Linux:

```bash id="e8e0w7"
nc -vz TARGET 445
smbclient -L //TARGET -U USER
```

### Failure Classification

| Result                                            | Investigate                         |
| ------------------------------------------------- | ----------------------------------- |
| Port unreachable                                  | Network / firewall / routing        |
| Port open but SMB unavailable                     | Service / protocol                  |
| Authentication rejected                           | Credential / authentication context |
| Authentication succeeds but shares denied         | Authorization                       |
| Share accessible but desired resource unavailable | Share/resource permissions          |

## 2. WinRM

### Typical Ports

```text
TCP 5985
TCP 5986
```

Commonly:

* 5985 = HTTP
* 5986 = HTTPS

### What WinRM Can Provide

When the identity is authorized:

```text id="x4xw2k"
WinRM
 ↓
Remote PowerShell
 ↓
Command Execution in Authorized Session
```

### Decision Path

```text id="h2q9kz"
WinRM
 ↓
Confirm Port
 ↓
Confirm WinRM
 ↓
Identify Account
 ↓
Validate Authentication
 ↓
Validate Remote Logon Authorization
 ↓
Establish Remote Session
 ↓
Validate New Position
```

### First Validation

Windows:

```powershell id="1baxd0"
Test-NetConnection TARGET -Port 5985
Test-WSMan TARGET
```

If authorized and appropriate:

```powershell id="8qv0h5"
Enter-PSSession -ComputerName TARGET
```

### Failure Classification

```text id="f2c5a1"
Cannot reach 5985/5986
        ↓
Network / service issue

Port reachable
        ↓
Test-WSMan fails
        ↓
WinRM / protocol / configuration issue

WinRM works
        ↓
Authentication fails
        ↓
Credential / authentication issue

Authentication works
        ↓
Remote session denied
        ↓
Authorization issue
```

## 3. RDP

### Typical Port

```text
TCP 3389
```

### What RDP Can Provide

RDP can provide an interactive Windows desktop session when:

* the service is available
* the account authenticates
* the account is permitted to log on through RDP
* other applicable controls permit the session

### Decision Path

```text id="4pk2qr"
RDP
 ↓
Reachability
 ↓
Identify Account
 ↓
Authentication
 ↓
RDP Logon Authorization
 ↓
Interactive Session
 ↓
Validate User / Host / Privilege
```

### First Validation

Windows:

```powershell id="x4fgdu"
Test-NetConnection TARGET -Port 3389
```

Linux:

```bash id="tqifg7"
nc -vz TARGET 3389
```

### Important Distinction

```text id="0rj23j"
Valid Windows Credential
        ≠
RDP Access
```

The account must also be permitted to log on through the relevant remote desktop policy and configuration.

## 4. WMI

### Typical Network Dependencies

WMI remote management commonly relies on:

```text
TCP 135
+
Dynamic RPC Ports
```

The exact network path depends on Windows configuration.

### What WMI Can Provide

When authorized:

* remote system information
* remote management operations
* WMI-based administrative workflows

WMI is not the same transport as WinRM.

### Decision Path

```text id="o9ldu2"
WMI
 ↓
RPC Endpoint Mapper
 ↓
Dynamic RPC Connectivity
 ↓
Windows Authentication
 ↓
WMI Authorization
 ↓
Remote Management
```

### First Validation

Check RPC reachability:

```powershell id="6zj7d5"
Test-NetConnection TARGET -Port 135
```

Then investigate the WMI path using authorized Windows management tooling.

### Failure Classification

```text
135 unreachable
    ↓
Network / firewall / RPC problem

135 reachable
    ↓
Dynamic RPC unavailable
    ↓
RPC / firewall problem

RPC available
    ↓
Authentication fails
    ↓
Credential problem

Authentication succeeds
    ↓
WMI operation denied
    ↓
Authorization problem
```

## 5. RPC

RPC is a foundation for several Windows remote-management mechanisms.

### Typical Port

```text
TCP 135
```

Additional dynamic RPC ports may also be required.

### Why It Matters

RPC can support or coordinate remote Windows operations involving:

* WMI
* DCOM
* other Windows management services

Therefore:

```text
RPC
≠
A standalone interactive shell
```

### Decision Path

```text id="1h9clz"
RPC
 ↓
Endpoint Mapper
 ↓
Identify Required RPC Service
 ↓
Validate Dynamic RPC Connectivity
 ↓
Authenticate
 ↓
Validate Authorization
 ↓
Use Relevant Management Workflow
```

### First Validation

```powershell id="6w8npu"
Test-NetConnection TARGET -Port 135
```

Do not conclude that WMI or another RPC-dependent service is usable solely because TCP 135 is open.

## 6. SSH

### Typical Port

```text
TCP 22
```

SSH may also run on a different configured port.

### What SSH Can Provide

When authorized:

```text id="1q4wvn"
SSH
 ↓
Authentication
 ↓
Remote Shell
 ↓
New Host Position
```

### Authentication Material

Potentially:

* username/password
* SSH private key
* SSH agent identity

### Decision Path

```text id="h3z2d6"
SSH
 ↓
Confirm Target / Port
 ↓
Identify Account
 ↓
Inspect Key / Agent Context
 ↓
Authenticate
 ↓
Validate Shell
 ↓
Validate User / Privilege / Network
 ↓
Re-enumerate
```

### First Validation

```bash id="7n0ap2"
nc -vz TARGET 22
```

Then, where authorized:

```bash id="lnw8o4"
ssh USER@TARGET
```

Or with a specific key:

```bash id="fd2s6x"
ssh -i /path/to/key USER@TARGET
```

### Important Distinction

```text id="m3j6d0"
SSH Authentication
       ≠
Root Access
```

Always validate the resulting account and privileges.

## 7. HTTP / HTTPS

Web services are different from traditional remote administration services.

### Decision Path

```text id="t4s9kq"
HTTP / HTTPS
      ↓
Identify Application
      ↓
Identify Authentication
      ↓
Identify Account / Session
      ↓
Determine Authorized Functionality
      ↓
Identify Infrastructure Relationships
      ↓
Assess Whether the Application Provides a Relevant Movement Path
```

Potentially relevant information includes:

* application identity
* authentication mechanism
* roles
* backend services
* APIs
* internal host references
* application configuration
* service accounts

Do not automatically treat a web service as a remote shell.

## 8. Database Services

Database access should be analyzed according to the database and its authorization model.

The workflow is:

```text id="t2w4nz"
Database Port
      ↓
Identify Database
      ↓
Identify Authentication Method
      ↓
Identify Account
      ↓
Authenticate
      ↓
Determine Database Role
      ↓
Identify Accessible Data / Functions
      ↓
Assess Infrastructure Relationships
```

A valid database credential does not automatically imply operating-system access.

## 9. NFS

NFS can expose remote filesystem resources depending on export and access configuration.

### Typical Port

```text
TCP/UDP 2049
```

### Decision Path

```text id="6x7bq8"
NFS
 ↓
Identify Export
 ↓
Identify Client Restrictions
 ↓
Determine Mount / Access Permissions
 ↓
Validate Authorized Access
 ↓
Assess Relevant Files / Resources
```

NFS access should be treated as resource access unless evidence establishes another authorized capability.

## 10. Service-to-Authentication Matrix

| Service    | Password                | NTLM                    | Kerberos                | SSH Key               | Existing Session                 |
| ---------- | ----------------------- | ----------------------- | ----------------------- | --------------------- | -------------------------------- |
| SMB        | Possible                | Possible                | Possible                | —                     | Possible                         |
| WinRM      | Possible                | Possible                | Possible                | —                     | Possible                         |
| RDP        | Possible                | Possible                | Possible                | —                     | Existing Windows session context |
| WMI        | Possible                | Possible                | Possible                | —                     | Existing Windows session context |
| RPC        | Possible                | Possible                | Possible                | —                     | Existing Windows session context |
| SSH        | Possible                | Configuration-dependent | Configuration-dependent | Possible              | Possible                         |
| HTTP/HTTPS | Application-dependent   | Application-dependent   | Application-dependent   | Application-dependent | Session-dependent                |
| Database   | Database-dependent      | Database-dependent      | Database-dependent      | —                     | Session-dependent                |
| NFS        | Configuration-dependent | —                       | Configuration-dependent | —                     | Existing client context          |

`Possible` means the authentication mechanism can be relevant depending on configuration; it does not mean the mechanism is universally supported.

## 11. Service-to-Access Matrix

| Service    |   Resource Access |             Remote Management | Interactive Session |      Potential New Position |
| ---------- | ----------------: | ----------------------------: | ------------------: | --------------------------: |
| SMB        |               Yes |             Context-dependent |        No by itself |             Not necessarily |
| WinRM      | Context-dependent |                           Yes |  PowerShell session |                         Yes |
| RDP        | Context-dependent |                           Yes |                 Yes |                         Yes |
| WMI        | Context-dependent |                           Yes |      Not inherently |                 Potentially |
| RPC        | Context-dependent | Supports management workflows |      Not inherently |         Depends on workflow |
| SSH        | Context-dependent |                           Yes |               Shell |                         Yes |
| HTTP/HTTPS |               Yes |         Application-dependent |        No by itself | Usually application context |
| Database   |               Yes |           Database management |        No by itself |    Usually database context |
| NFS        |               Yes |                  No by itself |                  No |     Usually resource access |

## 12. Service Selection by Objective

Choose the service based on what the assessment needs to validate.

| Objective                            | Relevant Service(s)           |
| ------------------------------------ | ----------------------------- |
| Determine file access                | SMB, NFS, application storage |
| Validate Windows remote shell access | WinRM                         |
| Validate Windows interactive access  | RDP                           |
| Validate Linux remote shell access   | SSH                           |
| Validate remote Windows management   | WMI / RPC                     |
| Validate application-level access    | HTTP/HTTPS                    |
| Validate database access             | Database service              |
| Reach internal-only services         | Pivot / forwarding mechanism  |

The objective should determine the service selection, not the other way around.

## 13. Service Discovery Decision Tree

```text id="8gk2zi"
Discovered Port
      ↓
Identify Service
      ↓
Is It a Remote Access Service?
      │
 ┌────┴────┐
No        Yes
↓          ↓
Assess    Identify
Service   Authentication
             ↓
       Identify Credential
             ↓
        Test Reachability
             ↓
        Authenticate
             ↓
        Check Authorization
             ↓
        Establish Access
             ↓
        Validate Position
```

## 14. Service Failure Classification

Use the following model:

```text id="l8n2k5"
Target
  ↓
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

A failure should be attached to the correct layer.

| Failure                                | Likely Layer                  |
| -------------------------------------- | ----------------------------- |
| No route                               | Network                       |
| Connection refused                     | Service / host                |
| Timeout                                | Network / firewall / service  |
| Wrong protocol                         | Service                       |
| Authentication rejected                | Authentication                |
| Logon denied                           | Authorization                 |
| Resource denied                        | Resource authorization        |
| Shell unavailable after authentication | Service/session configuration |

## 15. Cross-Platform Movement Matrix

| Source → Target        | Primary Services to Investigate                     |
| ---------------------- | --------------------------------------------------- |
| Windows → Windows      | SMB, WinRM, RDP, WMI, RPC                           |
| Linux → Linux          | SSH, NFS, application services                      |
| Windows → Linux        | SSH, application services                           |
| Linux → Windows        | SMB, WinRM, RDP, WMI, RPC                           |
| AD → Windows           | SMB, WinRM, RDP, WMI, RPC, Kerberos-backed services |
| Any → Internal Network | Pivoting, port forwarding, SOCKS, routing           |

This matrix identifies where to start; it does not establish that access exists.

## 16. Service-to-Technique Workflow

For every service:

```text id="c8m5z2"
1. Identify the service
        ↓
2. Confirm reachability
        ↓
3. Identify authentication methods
        ↓
4. Map available authentication material
        ↓
5. Identify target authorization requirements
        ↓
6. Validate authentication
        ↓
7. Validate authorization
        ↓
8. Establish the appropriate access
        ↓
9. Validate the new position
        ↓
10. Re-enumerate
```

## 17. Practical Example

Suppose target discovery identifies:

```text
SERVER-01
TCP 445
TCP 5985
TCP 3389
```

Do not immediately attempt all three.

Instead:

```text id="6l7r9x"
Target = SERVER-01
       ↓
What access is needed?
       ↓
Remote PowerShell?
       ↓
WinRM
       ↓
Do I have compatible authentication?
       ↓
Can I reach 5985?
       ↓
Authenticate
       ↓
Check authorization
       ↓
Validate session
```

If the objective instead requires file access:

```text id="k4d8m2"
Target = SERVER-01
       ↓
Need file/resource access
       ↓
SMB
       ↓
Identify shares
       ↓
Validate authorization
```

The desired access determines the relevant service.

## 18. Decision Record

For each service considered, record:

```text id="w8q3n7"
Target:
Service:
Port:
Reachability:
Authentication Method:
Available Authentication Material:
Authentication Result:
Authorization Result:
Access Obtained:
New Position:
Next Action:
Reason:
```

This creates an auditable movement path.

## 19. Golden Rules

Keep these rules visible during an assessment:

```text id="j5v9x2"
Open Port
   ≠
Usable Service

Usable Service
   ≠
Successful Authentication

Successful Authentication
   ≠
Authorization

Authorization
   ≠
Administrative Access

Remote Access
   ≠
New Useful Position
```

And:

```text id="p7m2q4"
Choose the service based on the access objective.
Choose the authentication method based on the service.
Choose the target based on evidence.
```

## 20. Complete Service Decision Loop

```text id="v4n8p6"
Target
  ↓
Service
  ↓
Port
  ↓
Reachability
  ↓
Authentication
  ↓
Authorization
  ↓
Access
  ↓
New Position
  ↓
Re-enumeration
  ↓
New Target
  ↓
Repeat
```

This is the central service-selection loop for the repository.

## What Next?

Section 12 — **Decision Framework** is now complete.

Continue to `13-Detection-and-Defense/README.md`.

That section will add the defensive perspective: what lateral-movement activity can generate in Windows logs, authentication telemetry, and network monitoring, while keeping detection knowledge tied to the offensive workflow.
