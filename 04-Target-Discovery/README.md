# Target Discovery

Target discovery is the process of identifying systems that may be reachable and relevant for lateral movement from the current position.

The objective is not to scan everything blindly.

The objective is to move from:

```text
Current Position
      ↓
Known Network Context
      ↓
Potential Hosts
      ↓
Reachable Hosts
      ↓
Available Services
      ↓
Potential Access Paths
      ↓
Prioritized Targets
```

A useful target is not simply a host that exists.

A useful target is a host where there is a **reasonable path from the current position to meaningful access**.

---

# 1. Where This Fits

The overall lateral-movement workflow is:

```text
Initial Foothold
      ↓
Position Assessment
      ↓
Target Discovery
      ↓
Credential / Access Discovery
      ↓
Remote Access Services
      ↓
Movement Technique
      ↓
Access Validation
      ↓
Post-Movement Enumeration
      ↓
Continue the Chain
```

`03-Position-Assessment/` answered:

> **Where am I?**

This section answers:

> **What systems can I potentially move to?**

---

# 2. Target Discovery Workflow

Use the following order:

```text
1. Review Current Position
        ↓
2. Identify Candidate Networks
        ↓
3. Discover Candidate Hosts
        ↓
4. Determine Reachability
        ↓
5. Enumerate Services
        ↓
6. Identify Access Opportunities
        ↓
7. Understand Target Role
        ↓
8. Prioritize Targets
        ↓
9. Select Next Investigation
```

Do not skip directly from:

```text
Network
   ↓
Movement
```

There are several decisions between those two points.

---

# 3. What Makes a Target Interesting?

A target becomes more interesting when multiple useful conditions overlap.

For example:

```text
Target
  │
  ├── Reachable
  ├── Relevant Role
  ├── Useful Service
  ├── Known Authentication Method
  ├── Existing Credentials
  └── Reasonable Access Path
```

A target does **not** need every condition to be interesting.

For example:

```text
Reachable Host
      +
SSH
      +
Known Valid Credential
```

may already provide a strong movement hypothesis.

Similarly:

```text
Reachable Windows Host
      +
SMB
      +
Known Domain Credential
```

may justify further validation.

---

# 4. Do Not Confuse Discovery With Access

Finding a host does not mean you can access it.

Keep these states separate:

```text
Host Exists
    ↓
Host Is Reachable
    ↓
Service Is Available
    ↓
Authentication Is Possible
    ↓
Authorization Is Sufficient
    ↓
Access Is Validated
```

Each step provides stronger evidence.

For example:

```text
10.10.10.20 responds
```

only establishes that a host appears reachable.

It does not establish:

```text
SSH access
SMB access
RDP access
Administrative access
```

---

# 5. Start With What You Already Know

Before performing new discovery, review the information collected during position assessment.

You may already know:

```text
Current IP:
Subnet:
Gateway:
Routes:
DNS:
Domain:
Current User:
Existing Connections:
Listening Services:
```

Use that information to create initial target hypotheses.

Example:

```text
Current Host:
10.10.10.15/24

Network:
10.10.10.0/24

Potential Hosts:
10.10.10.1
10.10.10.10
10.10.10.20
10.10.10.30
```

The important question becomes:

> **Which of these systems are relevant to my current objective?**

---

# 6. Candidate Networks

A compromised system may have access to more than one network.

For example:

```text
                Current Host
                     │
          ┌──────────┴──────────┐
          ▼                     ▼
    10.10.10.0/24         172.16.20.0/24
       Network A              Network B
```

This can be an important lateral-movement clue.

The second network may contain systems that cannot be reached directly from the original attacking position.

This is where later:

```text
09-Pivoting-and-Internal-Network-Access/
```

may become relevant.

For now, record the network.

Do not automatically pivot.

---

# 7. Host Discovery

Host discovery determines which candidate IP addresses appear to correspond to active systems.

The process is:

```text
Candidate Network
       ↓
Host Discovery
       ↓
Potential Live Hosts
       ↓
Validate Individually
```

Useful discovery methods vary by operating system, network configuration, and assessment environment.

The dedicated file:

```text
Host-Discovery.md
```

covers this process.

---

# 8. Network Enumeration

Host discovery alone does not explain the environment.

Network enumeration helps answer:

```text
What network am I connected to?
What routes exist?
What other networks are visible?
What hosts appear relevant?
What communication paths already exist?
```

This builds on the earlier:

```text
03-Position-Assessment/Network-Context.md
```

The dedicated file:

```text
Network-Enumeration.md
```

focuses on turning that information into target-discovery decisions.

---

# 9. Service Discovery

Once a host is identified, determine what services are exposed.

For example:

```text
HOST
 │
 ├── 22/tcp   SSH
 ├── 80/tcp   HTTP
 ├── 445/tcp  SMB
 ├── 3389/tcp RDP
 └── 5985/tcp WinRM
```

The important question is not:

> **"Which ports are open?"**

It is:

> **"Which available services provide a plausible path to authenticated remote access?"**

Service discovery therefore connects directly to:

```text
06-Remote-Access-Services/
```

and:

```text
08-Credential-Based-Movement/
```

---

# 10. Target Role Matters

A host's role changes its relevance.

Examples:

```text
Workstation
File Server
Application Server
Database Server
Web Server
Management Server
Domain Controller
Linux Server
Jump Host
```

Consider:

```text
Target A
  └── User workstation

Target B
  └── File server

Target C
  └── Domain Controller
```

These are not equivalent targets.

However, this repository does not assign an automatic priority based solely on role.

Instead, combine:

```text
Role
+
Reachability
+
Services
+
Available Authentication
+
Current Objective
```

---

# 11. Target Prioritization

After discovery, create a target list.

Example:

| Target | Role        | Reachable | Relevant Service | Known Access      | Next Action           |
| ------ | ----------- | --------- | ---------------- | ----------------- | --------------------- |
| HOST-A | Workstation | Yes       | SMB              | Domain credential | Validate              |
| HOST-B | File Server | Yes       | SMB/SSH          | None yet          | Credential review     |
| HOST-C | App Server  | Yes       | HTTP             | None              | Service investigation |
| HOST-D | Unknown     | Yes       | None identified  | None              | Low priority          |

The purpose of the table is not to assign a universal ranking.

It is to make the reasoning visible.

---

# 12. Evidence-Based Target Selection

For every candidate target, ask:

```text
1. Does the host appear reachable?
2. What is its role?
3. Which services are exposed?
4. What authentication methods are relevant?
5. Do I already possess potentially useful credentials?
6. Does my current identity suggest possible access?
7. Is there a reason this host matters to the assessment?
```

Then classify the next action.

```text
Strong Evidence
      ↓
Validate Access

Partial Evidence
      ↓
Gather Missing Information

Weak Evidence
      ↓
Deprioritize / Record

No Evidence
      ↓
Move to Another Candidate
```

---

# 13. The Target Discovery Decision Loop

Use this loop repeatedly:

```text
Find Host
   ↓
Is It Reachable?
   │
   ├── No ──► Record / Move On
   │
   ▼
What Services Exist?
   │
   ▼
What Is the Host's Role?
   │
   ▼
What Authentication Is Relevant?
   │
   ▼
Do I Have Potential Access?
   │
   ├── No ──► Credential / Access Discovery
   │
   └── Yes
        ↓
     Validate
        ↓
   Access Confirmed?
      │       │
     No      Yes
      │       │
      ▼       ▼
 Reassess   Move
            └──► Post-Movement Enumeration
```

---

# 14. Avoid Blind Scanning

Target discovery should answer a question.

Bad workflow:

```text
Scan Everything
      ↓
Collect Thousands of Results
      ↓
Try Everything
```

Better workflow:

```text
Understand Position
      ↓
Identify Relevant Network
      ↓
Discover Candidate Hosts
      ↓
Identify Useful Services
      ↓
Select Relevant Targets
      ↓
Validate Specific Paths
```

This reduces noise and makes the assessment easier to reason about.

---

# 15. Discovery Does Not End After Finding One Target

You may discover:

```text
HOST-A
```

and successfully move to it.

That does **not** mean target discovery is finished.

Your new position may reveal:

```text
New Network
New Routes
New Hosts
New Services
New Domain Context
New Credentials
New Sessions
```

Therefore:

```text
Target Discovery
      ↓
Movement
      ↓
New Position
      ↓
Target Discovery Again
```

Lateral movement is iterative.

---

# 16. What If Nothing Interesting Is Found?

Do not immediately assume that lateral movement is impossible.

Ask:

```text
Did I enumerate the correct network?
Did I identify all reachable networks?
Did I identify services correctly?
Did I understand the current identity?
Did I check existing sessions?
Did I review available credentials?
Did I identify the target's role?
```

If important information is missing:

```text
Return to the Missing Evidence
```

If the evidence genuinely shows no useful path:

```text
Document the Result
      ↓
Return to Credential / Access Discovery
      ↓
Reassess Later
```

---

# 17. Common Discovery Mistakes

## Mistake 1: Treating Every Host as a Target

A host may exist without being relevant.

**Better approach:**

```text
Existence
+
Reachability
+
Service
+
Access Path
+
Objective
```

---

## Mistake 2: Treating an Open Port as Access

For example:

```text
445/tcp open
```

does not mean:

```text
SMB access confirmed
```

It means:

```text
SMB service appears available
```

Authentication and authorization still need to be validated.

---

## Mistake 3: Ignoring Host Roles

A workstation and a database server may expose completely different movement opportunities.

Always try to understand the role.

---

## Mistake 4: Ignoring Multiple Networks

A host with multiple interfaces or routes may provide access to an otherwise unreachable network.

Record the information even if pivoting is not immediately required.

---

## Mistake 5: Stopping After the First Interesting Host

One successful target may expose a much better position.

Always perform post-movement enumeration.

---

# 18. Target Discovery Evidence

For each useful target, record:

```text
Target:
IP:
Hostname:
Network:
Role:
Reachability:
Open Services:
Authentication Options:
Potential Credentials:
Current Evidence:
Access Status:
Why It Matters:
Next Action:
```

Example:

```text
Target: FILE01
IP: 10.10.10.30
Hostname: FILE01
Network: 10.10.10.0/24
Role: File Server

Reachability:
Confirmed

Services:
SMB

Authentication:
Domain authentication appears relevant

Potential Credentials:
CORP\alice

Current Evidence:
SMB service reachable

Access Status:
Not yet validated

Why It Matters:
Potentially useful internal file server

Next Action:
Validate authorized SMB access
```

---

# 19. Section Structure

This section is divided into four focused stages:

```text
Host-Discovery.md
        │
        ▼
Identify candidate systems
        │
        ▼
Network-Enumeration.md
        │
        ▼
Understand network relationships
        │
        ▼
Service-Discovery.md
        │
        ▼
Identify remote services
        │
        ▼
Target-Prioritization.md
        │
        ▼
Choose the most justified next investigation
```

Each file should answer a different question.

### Host Discovery

> **What systems appear to exist?**

### Network Enumeration

> **How are those systems connected to my current position?**

### Service Discovery

> **What remote services can I interact with?**

### Target Prioritization

> **Which targets provide the most justified next movement opportunities based on the evidence I have?**

---

# 20. Target Discovery Completion Criteria

Target discovery is sufficiently complete for the current position when you can identify:

```text
[ ] Candidate networks
[ ] Candidate hosts
[ ] Reachable hosts
[ ] Relevant host roles
[ ] Available remote services
[ ] Potential authentication paths
[ ] Potential credentials / access
[ ] Candidate movement targets
[ ] Missing information
[ ] Next investigation
```

You do not need a perfect map of the entire environment.

You need enough evidence to make the **next movement decision**.

---

# 21. What Next?

Once a target has been identified, the next question becomes:

> **"What access do I already have that could work against this target?"**

That leads to:

```text
05-Credential-and-Access-Discovery/
```

The overall transition is:

```text
Position Assessment
        ↓
Target Discovery
        ↓
Candidate Target
        ↓
Credential / Access Discovery
        ↓
Remote Service
        ↓
Movement Path
```

The core principle is:

> **Discover targets based on your current position, validate what is actually reachable, identify the services that create movement opportunities, and prioritize only when the evidence supports further investigation.**
