# Pivoting and Internal Network Access

Pivoting is the process of using an already-accessible host to reach systems, services, or network segments that are not directly reachable from the original position.

The important distinction is:

```text
Lateral Movement
= Moving from one host to another

Pivoting
= Using one host or access path to reach a network location that was previously inaccessible
```

In real environments, these concepts can overlap.

The workflow is:

```text
Current Host
     ↓
Understand Network Position
     ↓
Identify Unreachable / Restricted Network
     ↓
Identify Pivot Host
     ↓
Determine Reachable Interface / Route
     ↓
Select Pivot Method
     ↓
Establish Forwarding / Proxy
     ↓
Validate Internal Connectivity
     ↓
Access Internal Service
     ↓
Validate Result
     ↓
Re-enumerate Internal Network
```

This section focuses on authorized assessments and controlled labs.

## Objective

The goal is to determine whether an existing foothold can provide access to additional internal network resources.

The key questions are:

* What networks can the current host reach?
* What networks cannot the assessment machine reach directly?
* Which host has access to the restricted network?
* What interfaces and routes does that host have?
* Which pivot mechanism is appropriate?
* What traffic needs to traverse the pivot?
* Can the resulting path be validated?
* What internal services become visible?
* What new targets should be investigated?

## Pivoting vs Port Forwarding

These concepts are related but should remain distinct.

### Pivoting

Pivoting is the broader concept:

```text
Attacker
   ↓
Compromised Host
   ↓
Internal Network
   ↓
Internal Target
```

The compromised host acts as a network position through which internal resources become reachable.

### Port Forwarding

Port forwarding exposes or transports a specific service through another host.

Conceptually:

```text
Attacker:LOCAL_PORT
       ↓
Pivot Host
       ↓
Internal Target:TARGET_PORT
```

Port forwarding can therefore be one mechanism used to accomplish pivoting.

## Why Pivoting Matters

A compromised host may have network visibility that the original assessment machine does not.

For example:

```text
Assessment Machine
      │
      │ Directly reachable
      ↓
Web Server
      │
      │ Internal interface
      ↓
10.10.20.0/24
      │
      ├── Database
      ├── File Server
      └── Management Server
```

The assessment machine may not have a direct route to `10.10.20.0/24`.

The web server may.

That difference creates a potential pivot point.

## 1. Establish the Current Network Position

Before creating any tunnel or proxy, understand the current host.

### Windows

```powershell
ipconfig
route print
Get-NetIPConfiguration
Get-NetRoute
```

Check listening and established connections:

```powershell
Get-NetTCPConnection
```

### Linux

```bash
ip addr
ip route
ip neigh
ss -tunap
```

The goal is to identify:

```text
Interfaces
Routes
Gateways
Reachable Networks
Listening Services
Established Connections
```

Do not start pivoting before understanding the existing network position.

## 2. Identify Additional Networks

Look for evidence of networks that are not directly reachable from the original assessment position.

For example:

```text
Current Host
├── 10.10.10.25
└── 10.10.20.25
```

This suggests the host may have access to both:

```text
10.10.10.0/24
10.10.20.0/24
```

If the assessment machine can only reach:

```text
10.10.10.0/24
```

then the second interface may represent a potential pivot path.

The important question is not:

> "Does this host have two interfaces?"

It is:

> "Does this host provide network reachability that I do not currently have?"

## 3. Identify the Pivot Host

A useful pivot host typically has:

* access to the current network
* access to another network
* an active session under the authorized assessment
* suitable routing or forwarding capability
* sufficient permissions for the selected pivot mechanism

Conceptually:

```text
Network A
    ↓
Pivot Host
    ↓
Network B
```

The pivot host does not necessarily need administrative privileges for every pivot method.

The required privileges depend on the mechanism and operating system.

## 4. Confirm the Internal Target

Before establishing forwarding, identify the actual resource you need to reach.

Examples:

```text
Internal Web Server
Internal SSH Server
Internal SMB Server
Internal Database
Internal Management Interface
```

Record:

```text
Target IP / Hostname:
Target Port:
Protocol:
Expected Service:
Reason for Access:
```

This prevents creating unnecessary tunnels to arbitrary internal systems.

## 5. Select the Pivot Method

The method should be chosen based on the requirement.

Common approaches include:

| Method                 | Primary Purpose                                        |
| ---------------------- | ------------------------------------------------------ |
| Port Forwarding        | Expose one internal service                            |
| Local Port Forwarding  | Bring a remote service to the local machine            |
| Remote Port Forwarding | Expose a local service through a remote host           |
| SOCKS Proxy            | Route multiple application connections through a pivot |
| Internal Routing       | Provide broader network-level reachability             |
| Proxy-aware Tools      | Send selected application traffic through a pivot      |

The correct choice depends on what traffic needs to cross the pivot.

## 6. Port Forwarding

Port forwarding is useful when only one service needs to be accessed.

Conceptually:

```text
Local Machine
127.0.0.1:LOCAL_PORT
        ↓
   Pivot Host
        ↓
Internal Target:TARGET_PORT
```

For example:

```text
127.0.0.1:8080
        ↓
Pivot
        ↓
10.10.20.15:80
```

The application connects to:

```text
127.0.0.1:8080
```

while the forwarding mechanism transports the connection to the internal target.

### When to Prefer It

Use port forwarding when:

* only one or a few services are required
* the target and port are known
* broad internal network access is unnecessary
* you want a small and easily understood traffic path

## 7. SOCKS Proxies

A SOCKS proxy is useful when multiple applications need to access internal resources through the pivot.

Conceptually:

```text
Application
     ↓
SOCKS Proxy
     ↓
Pivot Host
     ↓
Internal Network
```

The advantage is that different target connections can be routed through the same proxy without creating a separate local forward for every service.

Applications must support the proxy directly or use an appropriate proxy-routing mechanism.

## 8. Internal Network Access

Once the pivot exists, the workflow becomes:

```text
Assessment Machine
       ↓
Pivot
       ↓
Internal Network
       ↓
Target
       ↓
Service
```

Do not immediately assume the tunnel works.

Validate it in stages.

```text
Pivot Reachable
      ↓
Forwarding / Proxy Active
      ↓
Target Reachable Through Pivot
      ↓
Target Port Reachable
      ↓
Service Responds
      ↓
Application-Level Access
```

## 9. Validate Connectivity

Validation should happen at the same layer as the intended access.

For example, if the goal is an HTTP service:

```text
Network Path
    ↓
TCP Port
    ↓
HTTP Response
```

If the goal is SSH:

```text
Network Path
    ↓
TCP 22
    ↓
SSH Service
    ↓
Authentication
```

If the goal is SMB:

```text
Network Path
    ↓
TCP 445
    ↓
SMB
    ↓
Authentication
    ↓
Authorization
```

This layered validation helps identify exactly where a pivot is failing.

## 10. Failure Classification

When internal access fails, classify the failure.

### Pivot Host Unreachable

```text
Assessment Machine
      X
Pivot Host
```

Investigate the original connection first.

### Pivot Established but Target Unreachable

```text
Assessment Machine
       ↓
     Pivot
       X
Internal Target
```

Investigate:

* pivot routing
* interface configuration
* internal firewall
* target address
* network segmentation

### Target Port Unreachable

```text
Internal Target
      ↓
Required Port
      X
```

The service may be:

* stopped
* filtered
* listening elsewhere
* protected by host firewall

### Port Reachable but Service Fails

```text
TCP Connection ✓
      ↓
Application ✗
```

Investigate the service protocol and application configuration.

### Service Works but Authentication Fails

```text
Network ✓
Service ✓
Authentication ✗
```

Return to credential and access discovery.

## 11. Avoid Confusing Pivoting With New Host Access

A pivot does not necessarily mean you have obtained an interactive shell on another host.

For example:

```text
Pivot
  ↓
Internal HTTP Service
```

provides network access to the HTTP service.

It does not automatically mean:

```text
Internal HTTP Service
  =
Shell Access
```

Likewise:

```text
SOCKS Proxy
  ↓
Internal SMB
```

provides network-level access to SMB.

Authentication and authorization are still separate steps.

## 12. Re-enumerate the Internal Network

Once internal access is validated, perform focused discovery.

Determine:

```text
What hosts are reachable?
What services are exposed?
Which systems are interesting?
Which identities may access them?
Which services provide legitimate next steps?
```

Prioritize evidence-based targets.

For example:

```text
Internal Network
      ↓
Host Discovery
      ↓
Service Discovery
      ↓
Target Prioritization
      ↓
Credential / Access Mapping
```

Avoid broad scanning when a smaller, targeted assessment is sufficient.

## 13. Pivot Chain

A pivot can create a multi-stage path.

Example:

```text
Assessment Machine
        ↓
Web Server
        ↓
Internal Network
        ↓
Application Server
        ↓
Second Internal Network
        ↓
Database Server
```

Each stage becomes a new position.

At every stage:

```text
Validate
   ↓
Enumerate
   ↓
Identify New Network
   ↓
Identify New Targets
   ↓
Decide Whether Another Pivot Is Required
```

## 14. Pivoting and Lateral Movement Together

The two workflows can combine:

```text
Initial Host
     ↓
Lateral Movement
     ↓
Pivot Host
     ↓
Internal Network Access
     ↓
Internal Target
     ↓
Lateral Movement
     ↓
New Pivot
```

This is common in segmented environments.

The distinction remains:

```text
Lateral Movement
→ obtaining access to another host

Pivoting
→ using an existing position to reach another network location
```

## 15. Traffic Direction

Understand the direction of the connection before creating a tunnel.

### Local Forward

Conceptually:

```text
Local Machine
     ↓
Pivot
     ↓
Internal Target
```

The local machine connects to a local listening port that forwards toward the internal target.

### Remote Forward

Conceptually:

```text
Remote / Pivot Side
        ↓
Forwarding Listener
        ↓
Local / Assessment Service
```

The direction matters because firewalls and network controls may allow one direction but block another.

## 16. Minimal Pivoting Principle

Prefer the smallest access path that satisfies the assessment objective.

For example:

```text
Need One Internal Web Service
        ↓
Port Forward
```

rather than:

```text
Need One Web Service
        ↓
Full Network Tunnel
```

Similarly:

```text
Need Several Internal Services
        ↓
SOCKS Proxy
```

may be more appropriate than creating many individual forwards.

The objective is controlled, understandable access.

## 17. Evidence Recording

Record each pivot path:

```text
Source:
Pivot Host:
Pivot Interface:
Internal Network:
Target:
Target Port:
Protocol:
Pivot Method:
Validation Result:
Authentication Result:
Authorization Result:
New Access:
Next Target:
```

Example:

```text
Source: Assessment Machine
Pivot Host: WEB01
Pivot Interface: 10.10.20.5
Internal Network: 10.10.20.0/24
Target: DB01
Target Port: 5432
Protocol: PostgreSQL
Pivot Method: SOCKS Proxy
Connectivity: Successful
Authentication: Not Tested
Next Step: Assess authorized database access
```

## 18. Decision Framework

Use this workflow:

```text
Current Host
      ↓
Multiple Interfaces / Routes?
      │
 ┌────┴────┐
 No        Yes
 ↓           ↓
Continue     Identify Additional Network
Normal             ↓
Workflow       Is Network Directly Reachable?
                    │
               ┌────┴────┐
              Yes        No
               ↓           ↓
        No Pivot Needed   Identify Pivot Host
                              ↓
                       Identify Internal Target
                              ↓
                       One Service or Many?
                         │           │
                    One Service   Multiple
                         ↓           ↓
                   Port Forward  SOCKS / Routing
                         │           │
                         └─────┬─────┘
                               ↓
                       Validate Connectivity
                               ↓
                         Validate Service
                               ↓
                      Validate Authentication
                               ↓
                      Validate Authorization
                               ↓
                       Re-enumerate Network
```

## 19. Operational Rules

### Understand the Network First

Do not establish a tunnel before identifying the network relationship.

### Use Evidence-Based Targets

Know why an internal target is relevant before accessing it.

### Validate in Layers

Separate:

```text
Network
Service
Authentication
Authorization
Application
```

### Minimize Tunnel Scope

Expose only the access required for the authorized objective.

### Protect Tunnel Credentials and Configuration

Proxy and forwarding configurations may reveal sensitive network information.

### Re-enumerate

A pivot changes what the assessment can see.

Treat the newly reachable network as a new enumeration position.

## What Not to Assume

```text
Two Network Interfaces
   ≠
Automatic Pivot
```

```text
Pivot Established
   ≠
Target Service Access
```

```text
Target Reachable
   ≠
Authentication Success
```

```text
Authentication Success
   ≠
Authorization
```

```text
SOCKS Proxy
   ≠
Interactive Shell
```

```text
Internal Network Access
   ≠
Access to Every Internal Host
```

Each layer must be validated independently.

## Section Structure

The remaining files cover the main pivoting mechanisms:

| File                                                       | Focus                                                      |
| ---------------------------------------------------------- | ---------------------------------------------------------- |
| [`Pivoting-Concepts.md`](Pivoting-Concepts.md)             | Network position, pivot hosts, routes, and decision-making |
| [`Port-Forwarding.md`](Port-Forwarding.md)                 | Single-service forwarding workflows                        |
| [`SOCKS-Proxies.md`](SOCKS-Proxies.md)                     | Multi-service application-level proxying                   |
| [`Internal-Network-Access.md`](Internal-Network-Access.md) | Validation and enumeration after internal access           |

## What Next?

Continue to [`Pivoting-Concepts.md`](Pivoting-Concepts.md).

That file establishes the network reasoning required before choosing a specific pivot mechanism.
