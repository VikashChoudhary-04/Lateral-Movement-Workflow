# Pivoting Concepts

Pivoting begins with understanding a simple problem:

> The current host can reach something that the original assessment position cannot.

A pivot uses that existing network position to provide controlled access to another network or internal service.

The core model is:

```text id="k7u2m3"
Assessment Position
        ↓
Current Host
        ↓
Network Visibility
        ↓
Restricted / Internal Network
        ↓
Internal Target
```

The goal is not to create a tunnel simply because one is possible.

The goal is to answer:

```text id="x5r8k1"
What can the current host reach
that I cannot reach directly?
```

## Objective

Use the current host's network position to determine:

* what networks it can reach
* what networks the assessment machine cannot reach
* whether the host can serve as a pivot
* which internal targets are relevant
* what traffic needs to traverse the pivot
* which pivot mechanism fits the objective
* how to validate the resulting path

## 1. Understand Network Position

Start with the current host.

### Windows

```powershell id="3s7f2n"
ipconfig
Get-NetIPConfiguration
Get-NetRoute
```

Check active connections:

```powershell id="c9v4m1"
Get-NetTCPConnection
```

### Linux

```bash id="f1k8q2"
ip addr
ip route
ip neigh
ss -tunap
```

Record:

```text id="v3m6x8"
Interface:
IP Address:
Subnet:
Gateway:
Routes:
DNS:
Established Connections:
Listening Services:
```

The purpose is to build a network map from the perspective of the compromised host.

## 2. Identify Multiple Network Paths

A potential pivot often has more than one network path.

For example:

```text id="h8n2q5"
WEB01
├── eth0 → 10.10.10.25
└── eth1 → 10.10.20.25
```

The important question is not simply whether two interfaces exist.

Determine whether they provide access to different security or network zones.

For example:

```text id="m4p7s1"
Assessment Network
10.10.10.0/24
        ↓
     WEB01
        ↓
Internal Network
10.10.20.0/24
```

If the assessment machine cannot directly reach `10.10.20.0/24`, WEB01 may provide a useful pivot position.

## 3. Compare Network Visibility

A pivot is meaningful only when there is a difference in visibility.

Compare:

```text id="w2j6r9"
Assessment Machine
        ↓
Reachable Networks
```

with:

```text id="q5v8k3"
Pivot Host
        ↓
Reachable Networks
```

Then identify:

```text id="z1c4n7"
Pivot-Reachable
       -
Assessment-Reachable
       =
Potentially New Network Access
```

This difference is the basis for pivoting.

## 4. Identify the Pivot Host

A useful pivot host generally has:

* an established authorized foothold
* connectivity to the current network
* connectivity to another network
* appropriate forwarding/proxy capability
* sufficient permissions for the selected mechanism

Examples may include:

```text id="s3g6y9"
Web Server
Application Server
Jump Host
Workstation
Dual-Homed Server
VPN-Connected Host
```

The host does not need to be a router in the traditional sense.

Its value comes from its network position.

## 5. Understand Routes

Routes determine where traffic is sent.

Linux:

```bash id="j7k2p5"
ip route
```

Windows:

```cmd id="m3x8c1"
route print
```

Conceptually:

```text id="4f9v2d"
Destination Network
        ↓
Route
        ↓
Interface / Gateway
```

For a pivot:

```text id="r8w1q6"
Internal Network
        ↓
Pivot Interface
        ↓
Internal Target
```

If the pivot host has no route to the target network, the pivot mechanism may not solve the underlying connectivity problem.

## 6. Understand Subnets

Suppose the current host has:

```text id="2h7m4c"
10.10.10.25/24
```

and:

```text id="n9k3s6"
10.10.20.25/24
```

These belong to different `/24` networks:

```text id="j5q8t2"
10.10.10.0/24
10.10.20.0/24
```

The host may therefore have visibility into two network segments.

But this alone does not prove that traffic can freely move between them.

You still need to validate:

```text id="d3v6y8"
Route
 ↓
Firewall
 ↓
Service
 ↓
Application
```

## 7. Understand Network Segmentation

Segmentation can prevent direct communication between networks.

For example:

```text id="u6p2r8"
Assessment Machine
      │
      X
Internal Network
```

while:

```text id="b4m7s9"
Pivot Host
      │
      ✓
Internal Network
```

The pivot may therefore bypass the lack of a direct route while still remaining subject to:

* internal firewall rules
* host firewall rules
* ACLs
* service restrictions
* authentication requirements

Pivoting does not automatically bypass every security control.

## 8. Determine the Required Access

Before choosing a pivot method, identify what you actually need.

### One Internal Service

Example:

```text id="x2n5q8"
Internal Web Server
10.10.20.15:8080
```

A single port forward may be sufficient.

### Multiple Services

Example:

```text id="c7v1m4"
10.10.20.15:80
10.10.20.20:22
10.10.20.30:445
```

A SOCKS proxy or broader routing mechanism may be more appropriate.

### Broad Internal Network Visibility

If the authorized assessment requires access to multiple hosts and services across a network, a routing-based approach may be considered.

The choice should follow the objective rather than the other way around.

## 9. Choose the Smallest Effective Pivot

Use this principle:

```text id="p8m3k6"
Required Access
      ↓
Minimum Pivot Capability
```

For example:

```text id="z5r2t7"
One Host + One Port
       ↓
Port Forward
```

```text id="a4k9v1"
Multiple Hosts + Multiple Ports
       ↓
SOCKS Proxy
```

```text id="n6c2x8"
Broad Network Access
       ↓
Routing-Based Pivot
```

The exact mechanism depends on the environment and authorized tooling.

## 10. Port Forwarding Concept

A port forward maps one endpoint to another.

Conceptually:

```text id="w7f3m9"
Local Endpoint
127.0.0.1:8080
      ↓
Pivot
      ↓
Internal Target
10.10.20.15:80
```

The application connects to:

```text id="s2d8q5"
127.0.0.1:8080
```

while the forwarding path transports the connection toward:

```text id="g4m1v7"
10.10.20.15:80
```

This is useful when the target is known and only one service is needed.

## 11. SOCKS Proxy Concept

A SOCKS proxy provides a general proxy endpoint for applications that support SOCKS.

Conceptually:

```text id="k9p4c6"
Application
     ↓
SOCKS Proxy
     ↓
Pivot Host
     ↓
Internal Target
```

For example:

```text id="y3v8n1"
Browser / Client
      ↓
SOCKS
      ↓
Pivot
      ↓
Internal Web Server
```

The proxy can potentially handle multiple destination hosts and ports.

The application must support the proxy or use an appropriate proxy-routing mechanism.

## 12. Routing-Based Pivot Concept

A routing-based pivot attempts to make an internal network reachable through the pivot as a network path.

Conceptually:

```text id="q6m2s9"
Assessment Machine
       ↓
Route
       ↓
Pivot
       ↓
10.10.20.0/24
```

This is broader than forwarding a single port.

It should therefore be used only when the assessment objective requires broader access.

## 13. Validate the Pivot in Layers

Never treat "tunnel established" as the end of validation.

Use:

```text id="r1k7v4"
1. Pivot Connection
       ↓
2. Forwarding / Proxy
       ↓
3. Target Reachability
       ↓
4. Target Port
       ↓
5. Service Response
       ↓
6. Authentication
       ↓
7. Authorization
```

Each layer answers a different question.

## 14. Example: Internal Web Service

Suppose:

```text id="u8m3x5"
Pivot: 10.10.10.25
Target: 10.10.20.15
Port: 8080
```

The workflow is:

```text id="v4q7n2"
Identify Pivot
      ↓
Confirm Pivot → Target Connectivity
      ↓
Create Appropriate Forward
      ↓
Connect to Local Forward
      ↓
Receive HTTP Response
      ↓
Validate Application
```

If the local forward exists but HTTP fails:

```text id="k2j6p9"
Forwarding Layer ✓
Application Layer ✗
```

Investigate the target service rather than rebuilding the entire pivot.

## 15. Example: Internal SSH Service

Suppose:

```text id="b6w9r3"
Pivot → 10.10.20.30:22
```

The validation chain becomes:

```text id="t5n8c1"
Pivot
 ↓
TCP 22
 ↓
SSH Service
 ↓
Authentication
 ↓
Authorization
 ↓
Shell
```

A successful TCP connection alone does not prove SSH access.

## 16. Example: Internal SMB Service

For SMB:

```text id="p7x4m2"
Pivot
 ↓
10.10.20.40:445
 ↓
SMB
 ↓
Authentication
 ↓
Share Authorization
```

This should remain consistent with the broader lateral-movement workflow.

## 17. Identify Internal Targets

Once the pivot is validated, use targeted discovery.

Potential targets may include:

* domain controllers
* file servers
* application servers
* database servers
* management systems
* internal web applications
* SSH servers
* Windows administration services

Prioritize based on assessment scope and evidence.

A useful process is:

```text id="c5m8q2"
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

## 18. Avoid Unnecessary Internal Scanning

A pivot can dramatically increase the visible attack surface.

Do not automatically scan every internal address.

Instead:

```text id="n4v7k1"
Known Internal Range
       ↓
Assessment Scope
       ↓
Relevant Hosts
       ↓
Relevant Ports
       ↓
Focused Enumeration
```

This reduces noise and keeps the assessment controlled.

## 19. Pivot Chains

Sometimes the first pivot does not directly reach the final target.

Example:

```text id="m9c3r7"
Assessment Machine
       ↓
WEB01
       ↓
APP01
       ↓
DB01
```

Here:

* WEB01 is the first pivot
* APP01 may become a second pivot
* DB01 is the internal target

After reaching APP01:

```text id="f2x6v8"
Reassess Network Position
        ↓
Identify New Interfaces / Routes
        ↓
Identify Additional Network
        ↓
Determine Whether Another Pivot Is Required
```

Every new host creates a new network perspective.

## 20. Pivoting Decision Tree

Use this workflow:

```text id="w5n2j8"
Current Host
      ↓
Does It Reach Another Network?
      │
 ┌────┴────┐
 No        Yes
 ↓           ↓
Continue     Is Target Directly Reachable?
Other Work       │
             ┌───┴────┐
            Yes       No
             ↓          ↓
       No Pivot Needed  Use Pivot
                          ↓
                  How Much Access?
                    │        │
                  One      Multiple
                    ↓        ↓
              Port Forward  SOCKS
                    │        │
                    └────┬─────┘
                         ↓
                  Validate Target
                         ↓
                   Validate Service
                         ↓
                 Authenticate
                         ↓
                 Check Authorization
                         ↓
                  Re-enumerate
```

## 21. Failure Classification

### No Additional Network

The current host may not provide useful pivoting capability.

Return to normal target discovery.

### Additional Network Exists but Is Unreachable

Investigate:

* routes
* firewall
* interface state
* ACLs
* segmentation

### Pivot Works but Target Does Not

Investigate the internal path and target service separately.

### Target Port Works but Application Fails

The network path is probably functioning.

Investigate the application protocol.

### Service Works but Authentication Fails

Return to credential and access discovery.

### Authentication Works but Authorization Fails

The identity is valid but does not have the required permissions.

## 22. Evidence Recording

Record the network relationship:

```text id="a7k4p9"
Current Host:
Pivot Host:
Pivot Interface:
Pivot IP:
Internal Network:
Target:
Target IP:
Target Port:
Protocol:
Pivot Method:
Connectivity Result:
Service Result:
Authentication Result:
Authorization Result:
Next Step:
```

This makes multi-stage pivoting easier to understand and reproduce.

## 23. Operational Rules

### Understand Before Tunneling

Map interfaces and routes first.

### Choose the Method From the Objective

Do not create broad network access when a single forwarded service is sufficient.

### Validate Each Layer

Always separate network, transport, service, authentication, and authorization.

### Keep Scope Controlled

Only access internal systems covered by the assessment.

### Record the Path

Document:

```text id="v9m2x5"
Source
 ↓
Pivot
 ↓
Network
 ↓
Target
 ↓
Service
```

### Re-enumerate

After internal access is established, the newly reachable network becomes part of the assessment position.

## What Not to Assume

```text id="g3q8w1"
Dual-Homed Host
   ≠
Automatic Pivot
```

```text id="c6m1r9"
Route Exists
   ≠
Service Reachable
```

```text id="z8v4k2"
Port Reachable
   ≠
Application Access
```

```text id="s5n7p3"
Pivot Established
   ≠
Target Compromised
```

```text id="d2x9m6"
Authentication
   ≠
Authorization
```

The pivot is a network-access mechanism. What happens after reaching the target must still be assessed separately.

## What Next?

Continue to [`Port-Forwarding.md`](Port-Forwarding.md).

That file turns the pivoting concepts above into a focused workflow for exposing a specific internal service through a controlled forwarded connection.
