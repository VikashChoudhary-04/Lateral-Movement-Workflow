# Network Enumeration

Network enumeration is the process of understanding how discovered systems relate to the current host through interfaces, routes, subnets, gateways, DNS, and existing communication.

The objective is not to build a perfect network diagram.

The objective is to answer:

> **How can the systems I discovered communicate with my current position, and which network paths are relevant to lateral movement?**

The workflow is:

```text
Current Position
      ↓
Review Interfaces
      ↓
Review Routes
      ↓
Identify Networks
      ↓
Identify Gateways
      ↓
Review DNS Context
      ↓
Review Existing Connections
      ↓
Map Candidate Hosts
      ↓
Determine Reachability
      ↓
Identify Additional Networks
      ↓
Choose Next Investigation
```

---

# 1. Where This Fits

The target-discovery workflow is:

```text
Position Assessment
       ↓
Host Discovery
       ↓
Network Enumeration
       ↓
Service Discovery
       ↓
Target Prioritization
```

`Host-Discovery.md` answered:

> **What systems appear to exist?**

This file answers:

> **How are those systems connected to my current position?**

---

# 2. Network Enumeration Is Not the Same as Host Discovery

These activities are related but different.

### Host Discovery

```text
Which hosts appear to exist?
```

### Network Enumeration

```text
Which networks exist?
How are they connected?
Which routes are available?
Which hosts belong to which networks?
What paths exist from the current host?
```

For example:

```text
Host Discovery
    ↓
10.10.10.20
10.10.10.30
10.10.10.40
```

Network enumeration adds:

```text
10.10.10.0/24
      │
      ├── 10.10.10.20
      ├── 10.10.10.30
      └── 10.10.10.40
```

and potentially:

```text
172.16.20.0/24
      │
      ├── 172.16.20.10
      └── 172.16.20.20
```

---

# 3. The Three Important States

Always distinguish:

```text
Route
  ↓
Reachability
  ↓
Access
```

These are not equivalent.

### Route

The operating system knows where to send traffic.

### Reachability

Traffic can actually reach the destination.

### Access

You can successfully interact with the required service using valid authentication and authorization.

Therefore:

```text
Route
  ≠
Reachability
  ≠
Access
```

This distinction is fundamental to lateral movement.

---

# 4. Start With Network Interfaces

Identify every network interface.

## Linux

```bash
ip addr
```

or:

```bash
ip link
```

Look for:

```text
Interface
IP Address
Prefix
Interface State
```

Example:

```text
eth0
10.10.10.15/24

eth1
172.16.20.15/24
```

This immediately suggests that the current host participates in two networks.

---

# 5. Windows Interfaces

Use:

```cmd
ipconfig /all
```

PowerShell:

```powershell
Get-NetIPConfiguration
```

For adapter details:

```powershell
Get-NetAdapter
```

Record:

```text
Adapter
IPv4 Address
Subnet Prefix
Default Gateway
DNS Servers
Interface State
```

Example:

```text
Ethernet0
10.10.10.15/24
Gateway: 10.10.10.1
```

---

# 6. Build the Network Map

Convert raw interface information into a simple map.

Example:

```text
Current Host
10.10.10.15
     │
     ▼
10.10.10.0/24
     │
     ├── 10.10.10.1
     ├── 10.10.10.20
     ├── 10.10.10.30
     └── 10.10.10.40
```

If another interface exists:

```text
Current Host
     │
     ├──────────────┐
     ▼              ▼
10.10.10.0/24   172.16.20.0/24
     │              │
     ▼              ▼
  Targets        Targets
```

This is more useful than keeping isolated IP addresses.

---

# 7. Review the Routing Table

Routing information tells the operating system where traffic should be sent.

## Linux

```bash
ip route
```

Example:

```text
default via 10.10.10.1 dev eth0
10.10.10.0/24 dev eth0
172.16.20.0/24 via 10.10.10.1
```

This reveals that the current host may have a route toward:

```text
172.16.20.0/24
```

That network should become a candidate for further investigation.

---

# 8. Windows Routing Table

Use:

```cmd
route print
```

PowerShell:

```powershell
Get-NetRoute
```

Look for:

```text
Destination
Prefix
Next Hop
Interface
Metric
```

Example:

```text
10.10.10.0/24
172.16.20.0/24
0.0.0.0/0
```

The default route is not evidence that every private network is reachable.

Specific routes provide more useful context.

---

# 9. What Does a Route Tell You?

Suppose you find:

```text
172.16.20.0/24 via 10.10.10.1
```

This tells you:

```text
Current Host
     │
     ▼
10.10.10.1
     │
     ▼
172.16.20.0/24
```

It does **not** prove that:

```text
172.16.20.10
```

is online.

It also does not prove that you can authenticate to any service there.

The correct progression is:

```text
Route Found
     ↓
Candidate Network
     ↓
Discover Hosts
     ↓
Validate Reachability
     ↓
Enumerate Services
     ↓
Validate Access
```

---

# 10. Identify Gateways

Gateways can explain how traffic leaves the local subnet.

Linux:

```bash
ip route
```

Windows:

```cmd
ipconfig
```

or:

```cmd
route print
```

PowerShell:

```powershell
Get-NetIPConfiguration
```

Record:

```text
Gateway:
Interface:
Network:
```

Example:

```text
Network: 10.10.10.0/24
Gateway: 10.10.10.1
```

A gateway is an important network boundary, but it is not automatically a pivot point.

---

# 11. Review DNS Configuration

DNS can provide valuable environmental context.

## Linux

```bash
cat /etc/resolv.conf
```

Depending on the distribution:

```bash
resolvectl status
```

## Windows

```cmd
ipconfig /all
```

PowerShell:

```powershell
Get-DnsClientServerAddress
```

Look for:

```text
DNS Server
DNS Search Domain
DNS Suffix
Domain Namespace
```

---

# 12. Why DNS Matters

Suppose the current host uses:

```text
DNS:
10.10.10.10

Search Domain:
corp.example
```

This gives useful context:

```text
Current Host
     │
     ▼
DNS Infrastructure
     │
     ▼
corp.example
     │
     ├── dc01
     ├── file01
     ├── app01
     └── sql01
```

DNS does not automatically provide access to these systems.

It provides names and environmental clues that can guide further discovery.

---

# 13. Existing Network Connections

Existing connections can reveal active communication paths.

## Linux

```bash
ss -tun
```

For process information:

```bash
ss -tunp
```

## Windows

```cmd
netstat -ano
```

PowerShell:

```powershell
Get-NetTCPConnection
```

You may find:

```text
10.10.10.15 → 10.10.10.20:445
10.10.10.15 → 10.10.10.30:443
10.10.10.15 → 172.16.20.10:22
```

These connections are particularly useful because they provide evidence that communication already exists.

---

# 14. Investigate Existing Connections

For every interesting connection, ask:

```text
Who initiated it?
What destination is involved?
Which port is being used?
What process owns the connection?
Is the connection persistent?
Why does the communication exist?
```

Linux:

```bash
ss -tunp
```

Windows:

```cmd
netstat -ano
```

Then correlate Windows PIDs when needed:

```cmd
tasklist /FI "PID eq <PID>"
```

PowerShell:

```powershell
Get-Process -Id <PID>
```

The objective is not just to collect connections.

It is to understand **why they exist**.

---

# 15. Identify Network Segmentation

A network may be divided into multiple segments.

For example:

```text
                 Network Boundary
                       │
          ┌────────────┴────────────┐
          ▼                         ▼
   User Network                Server Network
   10.10.10.0/24              172.16.20.0/24
          │                         │
        WS01                      APP01
        WS02                      FILE01
```

If your current host can reach both networks, it may have a valuable position.

If it can reach only one, the second network may require another path.

This is where pivoting may eventually become relevant.

---

# 16. Multi-Homed Hosts

A multi-homed host has multiple network interfaces.

Example:

```text
                 CURRENT HOST
                /             \
               /               \
              ▼                 ▼
       10.10.10.15         172.16.20.15
              │                 │
              ▼                 ▼
       USER NETWORK        SERVER NETWORK
```

This can create an important movement opportunity because the host may have visibility into networks that the original position cannot directly reach.

But do not assume:

```text
Two Interfaces
     =
Automatic Pivot
```

First determine:

```text
Which networks?
Which routes?
Which destinations?
Which services?
Which access?
```

---

# 17. Route a Destination Through the Current Host

If you need to understand whether a specific destination is routed through a particular path, use appropriate route inspection.

Linux:

```bash
ip route get <destination-ip>
```

Example:

```bash
ip route get 172.16.20.10
```

This can show which interface and next hop the operating system would use.

Windows:

```cmd
tracert <destination-ip>
```

or PowerShell/network diagnostics as appropriate.

The important question is:

> **What path does the current system use to attempt communication with this destination?**

---

# 18. Reachability Testing

Once a candidate host has been identified, test whether the current system can communicate with it.

Do not rely on one test.

For example:

```text
Candidate Host
      ↓
Route Exists?
      ↓
Communication Attempt
      ↓
Response?
      ↓
Relevant Service?
```

A failed ping does not automatically mean the host is unreachable.

A successful ping does not prove service access.

---

# 19. Compare Multiple Evidence Sources

Suppose:

```text
Ping:
No response

Route:
Exists

DNS:
Resolves

TCP:
Potentially reachable
```

The conclusion should not be:

```text
Host is offline.
```

Instead:

```text
Host remains a candidate.
ICMP response is unavailable.
Additional service-level validation is required.
```

This is a better assessment.

---

# 20. Identify Network Boundaries

Look for:

```text
Different Subnets
Different VLANs
Additional Interfaces
Routing Entries
Gateways
Firewall Boundaries
VPN Interfaces
Tunnel Interfaces
```

These may explain why:

```text
Target A
```

is directly reachable while:

```text
Target B
```

is not.

Understanding the boundary is more useful than simply recording that Target B failed.

---

# 21. Network Enumeration and Pivoting

Suppose:

```text
Current Host
10.10.10.15
```

can reach:

```text
172.16.20.0/24
```

but your original assessment position cannot.

The situation may look like:

```text
Attacker
   │
   ▼
Current Host
10.10.10.15
   │
   │
   ▼
172.16.20.0/24
   │
   ├── 172.16.20.10
   ├── 172.16.20.20
   └── 172.16.20.30
```

This is the kind of evidence that can justify investigating:

```text
09-Pivoting-and-Internal-Network-Access/
```

Do not configure a pivot merely because multiple networks exist.

First establish that the additional network is relevant.

---

# 22. Network Enumeration in AD

In an Active Directory environment, network information can often be combined with domain information.

Example:

```text
Domain:
CORP.LOCAL

Current Host:
WS01

Networks:
10.10.10.0/24
172.16.20.0/24

Potential Infrastructure:
DC01
FILE01
APP01
```

This gives a more useful map:

```text
                 CORP.LOCAL
                     │
          ┌──────────┴──────────┐
          ▼                     ▼
   10.10.10.0/24          172.16.20.0/24
          │                     │
        WS01                  FILE01
        WS02                  APP01
                              DC01
```

The map still represents hypotheses until services and roles are validated.

---

# 23. Network Enumeration Record

Maintain a compact network record.

```text
Current Host:
Primary IP:

Interfaces:
- 
- 

Networks:
- 
- 

Gateways:
- 
- 

Routes:
- 
- 

DNS:
- 
- 

Existing Connections:
- 
- 

Candidate Internal Networks:
- 

Potential Pivot Points:
- 

Relevant Targets:
- 

Next Investigation:
```

---

# 24. Example Network Assessment

```text
Current Host:
WS01

Primary IP:
10.10.10.15

Interfaces:
Ethernet0 → 10.10.10.15/24

Routes:
10.10.10.0/24
172.16.20.0/24 via 10.10.10.1

DNS:
10.10.10.10

Existing Connections:
10.10.10.15 → 10.10.10.20:445

Candidate Internal Network:
172.16.20.0/24

Potential Target:
FILE01 / 172.16.20.20

Next Investigation:
Determine which hosts and services are reachable on the additional network.
```

Notice that this does **not** claim:

```text
FILE01 is accessible.
```

It only establishes:

```text
FILE01 is a candidate requiring further validation.
```

---

# 25. Common Network Enumeration Mistakes

## Mistake 1: Treating a Route as Access

```text
Route exists
    ≠
Target accessible
```

Always validate.

---

## Mistake 2: Treating a Gateway as a Pivot

A gateway routes traffic.

That does not mean it can be used as an interactive pivot.

---

## Mistake 3: Ignoring Secondary Interfaces

A second interface can reveal an entirely different network.

Always inspect all interfaces.

---

## Mistake 4: Ignoring Existing Connections

Existing communication can reveal important internal systems and services.

---

## Mistake 5: Treating DNS as an Inventory

DNS can provide useful names and records, but it should not automatically be treated as a complete list of active systems.

---

## Mistake 6: Declaring a Host Unreachable Too Early

A failed ICMP test is not sufficient evidence by itself.

Consider routing, filtering, and service-level evidence.

---

# 26. If the Network Map Is Incomplete

Use the following reasoning:

```text
Incomplete Network Map
        │
        ▼
Review Interfaces
        │
        ▼
Review Routes
        │
        ▼
Review DNS
        │
        ▼
Review Existing Connections
        │
        ▼
Review Neighbor Information
        │
        ▼
Compare With Discovered Hosts
        │
        ▼
Identify Missing Relationships
```

Do not attempt to fill every gap immediately.

Focus on gaps that affect the next movement decision.

---

# 27. Network Enumeration Decision Tree

```text
Candidate Host
      │
      ▼
Is There a Route?
   │          │
  No         Yes
   │          │
   ▼          ▼
Reassess   Test Communication
Network        │
Context        ▼
          Reachable?
           │      │
          No     Yes
           │      │
           ▼      ▼
      Investigate   Identify
      Filtering /   Services
      Segmentation      │
                        ▼
                 Potential Access
```

---

# 28. Completion Criteria

Network enumeration for the current position is sufficiently complete when you can identify:

```text
[ ] Active interfaces
[ ] Local networks
[ ] Relevant routes
[ ] Gateways
[ ] DNS context
[ ] Existing communication
[ ] Additional reachable networks
[ ] Candidate network boundaries
[ ] Potential pivot-relevant paths
[ ] Relationship between discovered hosts and networks
[ ] Missing network information
[ ] Next investigation
```

The objective is not a perfect network diagram.

The objective is:

> **Enough network context to understand how candidate targets relate to the current position.**

---

# 29. What Next?

You now know:

```text
Which hosts exist
        +
Which networks they belong to
        +
How they relate to the current position
```

The next question is:

> **"What services are exposed by these candidate targets?"**

Continue with:

```text
04-Target-Discovery/
└── Service-Discovery.md
```

The workflow becomes:

```text
Host Discovery
      ↓
Candidate Hosts
      ↓
Network Enumeration
      ↓
Connectivity / Network Context
      ↓
Service Discovery
      ↓
Remote Access Opportunities
      ↓
Target Prioritization
```

The core principle is:

> **Network enumeration turns a list of hosts into a map of possible communication paths. Distinguish routes, reachability, and access, and use the resulting network context to decide which services and targets deserve investigation next.**
