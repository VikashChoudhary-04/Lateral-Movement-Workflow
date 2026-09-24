# Host Discovery

Host discovery is the process of identifying systems that appear to exist and be reachable from the current position.

The objective is not to scan every possible address blindly.

The objective is to answer:

> **Which hosts can I potentially reach from where I currently am?**

The workflow is:

```text
Current Position
      ↓
Identify Candidate Network
      ↓
Determine Address Range
      ↓
Perform Host Discovery
      ↓
Identify Potentially Live Hosts
      ↓
Validate Findings
      ↓
Record Candidate Targets
      ↓
Move to Service Discovery
```

---

# 1. Where This Fits

Target discovery follows this sequence:

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

Host discovery answers:

> **What systems appear to exist?**

It does not answer:

> **What services are running?**

That is handled by:

```text
Service-Discovery.md
```

---

# 2. Start With the Current Network

Before discovering hosts, identify the networks that the current system can potentially reach.

Review:

```text
IP Address
Subnet
Network Interface
Default Gateway
Routing Table
Additional Routes
```

Examples:

### Linux

```bash
ip addr
ip route
```

### Windows

```cmd
ipconfig /all
route print
```

PowerShell:

```powershell
Get-NetIPConfiguration
Get-NetRoute
```

The goal is to determine the candidate network ranges.

For example:

```text
Current Host:
10.10.10.15/24

Candidate Network:
10.10.10.0/24
```

Do not automatically assume every private network is reachable.

---

# 3. Understand the Address Range

If the host has:

```text
10.10.10.15/24
```

the directly connected network is commonly:

```text
10.10.10.0/24
```

This gives a candidate range for discovery.

However:

```text
Candidate Network
      ≠
Confirmed Reachable Network
```

Routing information should be considered before scanning.

---

# 4. Host Discovery Questions

For every candidate network, ask:

```text
What network am I investigating?
Why is this network relevant?
Which addresses are candidates?
Which hosts respond?
Are the results consistent?
What systems should I investigate next?
```

This keeps discovery evidence-driven.

---

# 5. Linux Host Discovery

Linux systems can use several approaches depending on the environment and available tools.

A common first option is Nmap host discovery:

```bash
nmap -sn 10.10.10.0/24
```

The `-sn` option performs host discovery without performing a normal port scan.

Example output may identify:

```text
Nmap scan report for 10.10.10.1
Nmap scan report for 10.10.10.15
Nmap scan report for 10.10.10.20
Nmap scan report for 10.10.10.30
```

Record these as candidate hosts.

---

# 6. Interpret Host Discovery Results

Suppose the result is:

```text
10.10.10.1
10.10.10.20
10.10.10.30
```

Do not immediately conclude:

```text
All three are valid movement targets.
```

Instead:

```text
10.10.10.1
    ↓
Candidate Host
    ↓
Need Identity / Service Information

10.10.10.20
    ↓
Candidate Host
    ↓
Need Identity / Service Information

10.10.10.30
    ↓
Candidate Host
    ↓
Need Identity / Service Information
```

Host discovery only establishes the first layer of evidence.

---

# 7. Windows Host Discovery

Windows environments may provide useful information through local network configuration and existing connections.

Review:

```cmd
ipconfig /all
```

and:

```cmd
arp -a
```

The ARP cache may contain systems that the host has recently communicated with.

PowerShell:

```powershell
Get-NetNeighbor
```

can provide neighbor information.

These results can reveal candidate systems, but they are not a complete inventory.

---

# 8. ARP-Based Discovery

On a directly connected IPv4 network, ARP information can be useful.

Linux:

```bash
ip neigh
```

Windows:

```cmd
arp -a
```

PowerShell:

```powershell
Get-NetNeighbor
```

You may find entries such as:

```text
10.10.10.1
10.10.10.20
10.10.10.30
```

These are useful clues.

But remember:

```text
ARP Entry
   ≠
Complete Network Inventory
```

A host may not appear in the local ARP cache simply because the current machine has not communicated with it.

---

# 9. Discovery Through Existing Connections

Existing connections can provide valuable host information.

Linux:

```bash
ss -tun
```

Windows:

```cmd
netstat -ano
```

PowerShell:

```powershell
Get-NetTCPConnection
```

For example:

```text
Current Host
     │
     ├── 10.10.10.20:445
     ├── 10.10.10.30:22
     └── 10.10.10.40:443
```

These systems are particularly interesting because communication is already occurring.

Investigate why the connection exists.

---

# 10. DNS-Based Host Discovery

In domain environments, DNS can provide useful host information.

You may already know the domain:

```text
corp.example
```

and encounter names such as:

```text
dc01.corp.example
file01.corp.example
app01.corp.example
```

DNS results should be treated as candidate information.

A resolved hostname does not automatically mean:

```text
Host is reachable
```

Validate the system from the current position.

---

# 11. Hostname Resolution

When a candidate IP address is discovered, attempt to determine whether it has a useful hostname.

Linux:

```bash
getent hosts 10.10.10.20
```

Windows:

```cmd
nslookup 10.10.10.20
```

PowerShell:

```powershell
Resolve-DnsName 10.10.10.20
```

You may discover:

```text
10.10.10.20 → FILE01.corp.example
```

This provides additional context.

Now the candidate is:

```text
IP:
10.10.10.20

Hostname:
FILE01

Domain:
corp.example

Role:
Unknown
```

The role will be investigated later through service discovery and other evidence.

---

# 12. When a Host Does Not Respond

A failed response does not always mean:

```text
Host does not exist.
```

Possible explanations include:

```text
Host Offline
Firewall
ICMP Filtering
Network Filtering
Incorrect Route
Segmentation
Host Discovery Method Not Supported
```

Therefore:

```text
No Response
    ↓
Do Not Immediately Discard
    ↓
Check Routing
    ↓
Check Relevant Services
    ↓
Use Another Appropriate Discovery Method
```

The exact next step depends on the network and authorization scope.

---

# 13. ICMP Is Not the Whole Story

A common mistake is assuming:

```text
Ping Fails
   ↓
Host Is Down
```

This is unreliable.

A host may block ICMP while still allowing application traffic.

Therefore:

```text
ICMP Response
      ≠
Only Proof of Host Existence
```

and:

```text
No ICMP Response
      ≠
Proof That the Host Is Offline
```

When appropriate, validate the candidate through other authorized network observations.

---

# 14. Discovery From a Multi-Homed Host

A compromised host may have multiple interfaces.

For example:

```text
             CURRENT HOST
              /        \
             /          \
            ▼            ▼
     10.10.10.15      172.16.20.15
          │                 │
          ▼                 ▼
   10.10.10.0/24      172.16.20.0/24
```

This is an important finding.

The second interface may expose a network that was not directly visible from the original system.

Record:

```text
Interface:
IP:
Subnet:
Gateway:
Potential Network:
```

Do not automatically assume that the second network contains useful targets.

Investigate it systematically.

---

# 15. Discovery Through Routing

A routing table may reveal additional candidate networks.

Linux:

```bash
ip route
```

Windows:

```cmd
route print
```

PowerShell:

```powershell
Get-NetRoute
```

Example:

```text
10.10.10.0/24
172.16.20.0/24
```

The workflow becomes:

```text
Route Found
    ↓
Candidate Network Identified
    ↓
Determine Reachability
    ↓
Perform Appropriate Discovery
    ↓
Record Candidate Hosts
```

This becomes particularly important when pivoting is later required.

---

# 16. Domain Environment Clues

In an AD environment, host discovery may reveal systems with names such as:

```text
DC01
FILE01
APP01
SQL01
WS01
```

Do not rely solely on naming conventions.

For example:

```text
DC01
```

may suggest a Domain Controller, but the name itself is not proof.

Treat the hostname as a clue:

```text
Hostname
   ↓
Hypothesized Role
   ↓
Service Discovery
   ↓
Role Validation
```

---

# 17. Build a Candidate Host List

Do not keep discovery results only in your terminal.

Create a working list.

Example:

| IP          | Hostname | Network       | Discovery Source    | Reachability | Initial Notes         |
| ----------- | -------- | ------------- | ------------------- | ------------ | --------------------- |
| 10.10.10.1  | Unknown  | 10.10.10.0/24 | Nmap                | Observed     | Gateway candidate     |
| 10.10.10.20 | FILE01   | 10.10.10.0/24 | Nmap/DNS            | Observed     | Possible file server  |
| 10.10.10.30 | APP01    | 10.10.10.0/24 | Existing connection | Observed     | Application candidate |

This makes the next stage much easier.

---

# 18. Validate Important Candidates

Once a candidate appears interesting, validate it.

Ask:

```text
Does the IP still respond?
Does the hostname resolve?
Is the host on the expected network?
Does the host expose relevant services?
Does the host appear to match the expected role?
```

The evidence should become progressively stronger.

```text
Candidate IP
    ↓
Reachable
    ↓
Hostname
    ↓
Services
    ↓
Role
    ↓
Potential Access
```

---

# 19. Discovery Noise

Not every discovered address represents a useful system.

You may encounter:

```text
Gateway
Network Appliance
Printer
Broadcast Address
Infrastructure Device
Monitoring System
Unknown Host
```

Record unusual findings when relevant, but do not automatically treat every device as a lateral-movement target.

The question remains:

> **Does this system provide a plausible next step from my current position?**

---

# 20. Common Host Discovery Mistakes

## Mistake 1: Scanning Without Understanding the Network

Bad:

```text
Run a large scan immediately
```

Better:

```text
Understand Current Network
       ↓
Identify Relevant Range
       ↓
Discover Hosts
```

---

## Mistake 2: Treating Ping as Proof

Bad:

```text
Ping failed → Host is offline
```

Better:

```text
Ping failed
    ↓
Check other evidence
    ↓
Determine whether the host remains relevant
```

---

## Mistake 3: Treating Every Live Host as a Target

A live host is simply a candidate.

Service and role information are still required.

---

## Mistake 4: Ignoring Existing Connections

Existing connections may immediately identify systems that are communicating with the current host.

Always review them.

---

## Mistake 5: Ignoring Additional Routes

A second route may expose an entirely different environment.

Always review the routing table during initial discovery.

---

# 21. If Host Discovery Produces No Results

Use the following reasoning:

```text
No Hosts Found
      │
      ▼
Is the Network Correct?
      │
      ├── No → Reassess Network Context
      │
      ▼
Is the Route Valid?
      │
      ├── No → Reassess Routing
      │
      ▼
Could Discovery Traffic Be Filtered?
      │
      ├── Yes → Use Appropriate Alternative Evidence
      │
      ▼
Are Existing Connections Available?
      │
      ├── Yes → Investigate Them
      │
      ▼
Are DNS / Neighbor Records Available?
      │
      ├── Yes → Investigate Candidates
      │
      ▼
No Reliable Candidates
      │
      ▼
Record Result and Reassess Later
```

Failure is information.

Do not compensate for uncertainty by blindly increasing scan volume.

---

# 22. Host Discovery Record

For each useful candidate, record:

```text
Target IP:
Hostname:
Network:
Discovery Method:
Observed Reachability:
DNS Information:
Existing Connections:
Initial Role Hypothesis:
Relevant Notes:
Next Action:
```

Example:

```text
Target IP:
10.10.10.20

Hostname:
FILE01.corp.example

Network:
10.10.10.0/24

Discovery Method:
Host discovery + DNS

Observed Reachability:
Confirmed

Initial Role Hypothesis:
File server

Relevant Notes:
SMB connection observed

Next Action:
Service discovery
```

---

# 23. Host Discovery Decision Tree

```text
Identify Candidate Network
          │
          ▼
   Discover Candidate Hosts
          │
          ▼
     Host Found?
       │       │
      No      Yes
       │       │
       ▼       ▼
 Reassess   Validate
 Network       │
               ▼
        Identify Hostname
               │
               ▼
        Record Candidate
               │
               ▼
       Continue to Service
          Discovery
```

---

# 24. Completion Criteria

Host discovery for the current network is sufficiently complete when you have:

```text
[ ] Identified the relevant network
[ ] Reviewed routing information
[ ] Identified candidate hosts
[ ] Considered existing connections
[ ] Considered DNS information
[ ] Considered neighbor information
[ ] Distinguished non-response from confirmed absence
[ ] Recorded interesting candidates
[ ] Avoided treating every host as a target
[ ] Identified the next investigation
```

You do not need a perfect inventory.

You need enough information to move to the next decision.

---

# 25. What Next?

Once candidate hosts have been identified, the next question is:

> **"What networks and communication paths connect these hosts to my current position?"**

Continue with:

```text
04-Target-Discovery/
└── Network-Enumeration.md
```

The workflow becomes:

```text
Host Discovery
      ↓
Candidate Hosts
      ↓
Network Enumeration
      ↓
Understand Connectivity
      ↓
Service Discovery
      ↓
Identify Movement Paths
```

The core principle is:

> **Host discovery identifies candidate systems; it does not establish access. Find the systems that are relevant, validate the evidence you have, record them, and then determine how they communicate with your current position.**
