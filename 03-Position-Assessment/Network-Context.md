# Network Context

A compromised host is useful for lateral movement partly because of **where it can communicate**.

The objective of network-context assessment is to determine:

```text
What interfaces exist?
What IP addresses are assigned?
What subnets are connected?
What routes exist?
What gateways are used?
What DNS infrastructure is available?
What systems is the host already communicating with?
What networks may be reachable from this position?
```

The goal is not to produce a huge network inventory.

The goal is to understand:

> **What does the current host allow me to see and potentially reach that I could not see from my original position?**

---

# 1. Network Assessment Workflow

```text id="9x3l8w"
Current Host
     │
     ▼
Identify Interfaces
     │
     ▼
Identify IP Addresses
     │
     ▼
Identify Subnets
     │
     ▼
Identify Gateways
     │
     ▼
Identify Routes
     │
     ▼
Identify DNS Context
     │
     ▼
Review Existing Connections
     │
     ▼
Identify Potentially Reachable Networks
     │
     ▼
Identify Candidate Targets
     │
     ▼
Choose Next Investigation
```

---

# 2. Identify Network Interfaces

Start by identifying all network interfaces.

## Linux

```bash id="m1u0pf"
ip addr
```

Short form:

```bash id="h8c7c2"
ip a
```

You can also list interfaces:

```bash id="x4u4f0"
ip link
```

### Look For

```text id="t4m3xj"
Interface name
IP address
Subnet mask / prefix
MAC address
Interface state
```

---

## Windows

```cmd id="6a2x0s"
ipconfig /all
```

PowerShell:

```powershell id="ry78x7"
Get-NetIPConfiguration
```

For interfaces:

```powershell id="x7z4a5"
Get-NetAdapter
```

### Record

```text id="p6i2q8"
Interface:
IP:
Prefix / Subnet:
Gateway:
DNS:
State:
```

---

# 3. Why Multiple Interfaces Matter

A host may have more than one network interface.

For example:

```text id="i2g5s7"
                 COMPROMISED HOST
                    /         \
                   /           \
                  ▼             ▼
             Network A      Network B
             10.10.10.0    10.10.20.0
```

This can be significant.

The compromised host may have access to a network that the original attacker position cannot directly reach.

### Decision

If multiple interfaces exist:

```text id="1n4u7m"
Identify each network
        ↓
Determine what each interface connects to
        ↓
Inspect routing
        ↓
Determine which networks are actually reachable
```

Do not assume every interface provides unrestricted access to its connected network.

---

# 4. Identify the Local Subnet

The IP address alone is not enough.

You need to understand the network prefix.

Example:

```text id="c0c5ax"
IP:      10.10.10.15
Prefix:  /24
Network: 10.10.10.0/24
```

This tells you the local network context.

### Linux

The prefix is shown by:

```bash id="04fl57"
ip addr
```

### Windows

```cmd id="z2c93y"
ipconfig
```

The subnet mask can be converted into the corresponding prefix/network as needed.

### Why?

The local subnet often provides the first set of systems worth investigating.

```text id="q7sj4j"
10.10.10.0/24
       │
       ├── HOST-A
       ├── HOST-B
       ├── FILE01
       └── APP01
```

However:

> **Being on the same subnet does not automatically mean a host is reachable or accessible.**

---

# 5. Identify the Default Gateway

The gateway shows where traffic destined for other networks may be sent.

## Linux

```bash id="7s48p5"
ip route
```

Look for:

```text id="w0xg7w"
default via <gateway>
```

## Windows

```cmd id="rj1p2m"
ipconfig
```

or:

```cmd id="w5h0l2"
route print
```

### Why?

The gateway helps explain how the current host communicates outside its local subnet.

For example:

```text id="18l1q0"
10.10.10.0/24
       │
       ▼
   Gateway
       │
       ▼
10.10.20.0/24
```

This may reveal additional network paths.

---

# 6. Inspect the Routing Table

The routing table is one of the most useful pieces of network context.

## Linux

```bash id="u4f0ev"
ip route
```

## Windows

```cmd id="4y3fkg"
route print
```

PowerShell:

```powershell id="f7g9bm"
Get-NetRoute
```

### Look For

```text id="uqw9v4"
Local networks
Remote networks
Default route
Specific routes
Unexpected internal networks
Multiple gateways
```

Example:

```text id="5q4x3g"
10.10.10.0/24
10.10.20.0/24
10.10.30.0/24
0.0.0.0/0
```

This tells you that the host has routes associated with several networks.

---

# 7. Route ≠ Reachability

This distinction is critical.

Suppose you see:

```text id="f2a9di"
10.10.30.0/24
```

in the routing table.

That does **not** automatically mean:

```text id="2o6s2r"
Every host in 10.10.30.0/24
is reachable.
```

A route means the system has a path for traffic destined for that network.

Actual communication may still be affected by:

* Firewalls
* ACLs
* Host-based filtering
* Network segmentation
* Service availability
* Authentication requirements

Therefore:

```text id="2x0y9r"
Route discovered
      ↓
Validate reachability
      ↓
Identify responsive systems
      ↓
Identify available services
```

---

# 8. Identify DNS Context

DNS can provide valuable environmental information.

## Linux

Check resolver configuration:

```bash id="r6l7i1"
cat /etc/resolv.conf
```

Depending on the environment:

```bash id="y3yq6f"
resolvectl status
```

## Windows

```cmd id="4c2z3r"
ipconfig /all
```

PowerShell:

```powershell id="f3d6q2"
Get-DnsClientServerAddress
```

### Look For

```text id="q6f6cm"
DNS servers
DNS search domains
Internal domain names
```

For example:

```text id="8o9v9w"
DNS:
10.10.10.10

Search Domain:
corp.local
```

This can help establish the internal naming and domain context.

---

# 9. DNS Is an Information Source

DNS can help turn an IP address into a useful identity.

For example:

```text id="0d3u6y"
10.10.10.20
      ↓
fileserver.corp.local
      ↓
Potential FILE01
```

Likewise, a domain name may reveal:

```text id="w0g2px"
corp.local
internal.example
dev.corp.local
```

These names can help organize later target discovery.

Do not assume that DNS records represent every live system.

Treat them as **environmental information that can guide further validation**.

---

# 10. Inspect Existing Network Connections

The current host may already communicate with important internal systems.

## Linux

```bash id="pm6w3u"
ss -tunap
```

If available:

```bash id="f7ot1p"
netstat -tunap
```

## Windows

```cmd id="6sywzk"
netstat -ano
```

PowerShell:

```powershell id="8zh8by"
Get-NetTCPConnection
```

### Look For

```text id="d7w0q1"
Remote IP
Remote Port
Local Port
Protocol
Connection State
Associated Process
```

---

# 11. Turn Connections Into Relationships

Suppose you discover:

```text id="w07s4x"
WEB01
  │
  ├──→ 10.10.20.15:445
  ├──→ 10.10.20.20:1433
  └──→ 10.10.10.10:53
```

Instead of recording three unrelated connections, interpret them:

```text id="zv4l8f"
WEB01
 │
 ├──→ SMB
 │
 ├──→ Database Service
 │
 └──→ DNS
```

Now you have hypotheses about how the system interacts with the environment.

The next question becomes:

> **Which of these relationships can provide a legitimate movement path?**

---

# 12. Identify Listening Services

The current host's own listening services are also relevant.

### Linux

```bash id="8djf6h"
ss -lntup
```

### Windows

```cmd id="l6jjfw"
netstat -ano
```

PowerShell:

```powershell id="5g4d8a"
Get-NetTCPConnection -State Listen
```

This can help identify services that other systems may use to communicate with the current host.

It also helps validate your understanding of the host's role.

---

# 13. Understand Network Segmentation

A common enterprise layout may look like:

```text id="v9t8q2"
                 User Network
                 10.10.10.0/24
                       │
                       ▼
                    WEB01
                       │
                       ▼
              Application Network
                 10.10.20.0/24
                       │
              ┌────────┴────────┐
              ▼                 ▼
            APP01              DB01
                                │
                                ▼
                         Restricted Network
                            10.10.30.0/24
```

From the original attacker position, perhaps only:

```text id="8k7h7r"
10.10.10.0/24
```

was reachable.

After compromising `WEB01`, you may discover:

```text id="t2jv6c"
10.10.20.0/24
```

and possibly a path toward:

```text id="1trm5r"
10.10.30.0/24
```

This is where network context becomes directly relevant to lateral movement.

---

# 14. Identify Candidate Networks

Create a simple map:

```text id="h3y3l6"
CURRENT HOST
     │
     ├── Local Network
     │      └── 10.10.10.0/24
     │
     ├── Routed Network
     │      └── 10.10.20.0/24
     │
     └── Potentially Restricted Network
            └── 10.10.30.0/24
```

Then ask:

```text id="6es2v4"
Which networks are actually reachable?
Which systems exist there?
Which services are exposed?
Which systems are relevant?
```

This naturally leads into target discovery.

---

# 15. Test Reachability Before Assuming Access

Once you identify a potential target, distinguish:

```text id="x1q7a0"
Network Reachability
        ↓
Service Reachability
        ↓
Authentication
        ↓
Authorization
        ↓
Actual Access
```

These are different stages.

For example:

```text id="g8r5ik"
Can reach host
      ↓
Can reach TCP/445
      ↓
SMB responds
      ↓
Credentials accepted
      ↓
Authorized access
```

A failure at any stage changes the next action.

---

# 16. Avoid Blind Network Scanning

The goal is not:

```text id="0f8f8q"
Scan everything
      ↓
Collect thousands of results
```

Instead:

```text id="r4r0l3"
Understand current network position
      ↓
Identify relevant networks
      ↓
Identify likely targets
      ↓
Validate reachability
      ↓
Enumerate relevant services
```

This produces much more useful information.

---

# 17. Network Context → Movement Decision

At the end of network assessment, classify the situation.

### Case 1 — Only local network visible

```text id="7udgk0"
Current Host
     │
     ▼
Local Network
     │
     ▼
Target Discovery
```

### Case 2 — Additional routed network discovered

```text id="l4zzn0"
Current Host
     │
     ▼
Additional Network
     │
     ▼
Validate Reachability
     │
     ▼
Identify Targets
```

### Case 3 — Network exists but is not directly reachable

```text id="8z9u4h"
Current Host
     │
     ▼
Restricted Network
     │
     ▼
Investigate Pivoting / Internal Access
```

Continue to:

```text id="y2s6cm"
09-Pivoting-and-Internal-Network-Access/
```

when the situation requires a different network path.

---

# 18. Build the Network Map

At the end of assessment, create a simple representation:

```text id="31b3xz"
CURRENT POSITION
================

Host:
IP:
Interface(s):

Local Network:
Gateway:

Additional Networks:
Routes:

DNS:
Domain:

Known Connections:
Known Services:

Potential Targets:
Potential Restricted Networks:

Next Investigation:
```

Example:

```text id="6s5e6s"
Host: WEB01
IP: 10.10.10.15

Local Network:
10.10.10.0/24

Gateway:
10.10.10.1

Additional Network:
10.10.20.0/24

DNS:
10.10.10.10

Domain:
corp.local

Known Connection:
10.10.20.20:445

Potential Target:
10.10.20.20

Next Investigation:
Validate target reachability and enumerate available services.
```

---

# 19. Network Assessment Checklist

```text id="q6nq2p"
[ ] Interfaces identified
[ ] IP addresses identified
[ ] Subnets identified
[ ] Default gateway identified
[ ] Routing table reviewed
[ ] Additional networks identified
[ ] DNS servers identified
[ ] DNS/domain context identified
[ ] Existing network connections reviewed
[ ] Listening services reviewed
[ ] Potential target systems identified
[ ] Reachability distinguished from access
[ ] Restricted networks identified
[ ] Pivoting requirement considered
[ ] Network map created
[ ] Next workflow selected
```

---

# 20. Final Network Assessment

You should finish with the ability to state:

```text id="s8j1l2"
I am connected to:
____________________

My local network is:
____________________

My gateway is:
____________________

I have routes to:
____________________

The relevant DNS/domain context is:
____________________

This host communicates with:
____________________

Potentially reachable targets include:
____________________

The most interesting network relationship is:
____________________

The next thing I need to investigate is:
____________________
```

The objective is not to know the entire network.

The objective is to understand:

> **How does my current network position affect where I can potentially move?**

---

# 21. What Comes Next?

Network context gives us the **where**.

The next major question is the **environmental relationship**:

```text id="h4i6xj"
Current Host
     │
     ├── Identity
     │
     ├── Network
     │
     └── Domain / Trust Context
                 │
                 ▼
          Potential Targets
```

Continue to:

```text id="v4r0s5"
03-Position-Assessment/Trust-and-Domain-Context.md
```

The next file focuses on Windows and Active Directory environments and answers:

> **"What domain, workgroup, trust, and identity relationships surround my current position, and how can they influence lateral movement?"**
