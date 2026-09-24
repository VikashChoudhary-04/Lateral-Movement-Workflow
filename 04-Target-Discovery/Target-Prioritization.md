# Target Prioritization

Target prioritization is the process of deciding which discovered system should receive the next investigation based on the evidence collected so far.

The objective is not to create a universal ranking of hosts.

The objective is to answer:

> **Given what I currently know, which target gives me the most justified next investigation?**

A useful prioritization process combines:

```text id="f6zj9k"
Reachability
+
Host Role
+
Available Services
+
Authentication
+
Available Credentials
+
Current Objective
+
Expected Information Gain
+
Uncertainty
```

---

# 1. Where This Fits

The target-discovery workflow is:

```text id="5h1y9d"
Position Assessment
       ↓
Host Discovery
       ↓
Network Enumeration
       ↓
Service Discovery
       ↓
Target Prioritization
       ↓
Credential / Access Discovery
       ↓
Remote Access
```

The previous files established:

```text id="m0lqwj"
Host Discovery
→ What systems appear to exist?

Network Enumeration
→ How do those systems relate to my current position?

Service Discovery
→ What can I communicate with?
```

This file answers:

> **Which of those systems should I investigate next?**

---

# 2. Prioritization Is a Decision, Not a Score

Avoid thinking:

```text id="qzv7g8"
Host A = 10/10
Host B = 7/10
Host C = 4/10
```

A numerical score can hide the reasoning that actually matters.

Instead, make the evidence explicit:

```text id="2f3n9m"
Target
  ↓
Why is it relevant?
  ↓
What evidence do I have?
  ↓
What is still unknown?
  ↓
What action can resolve that uncertainty?
```

The result should be a defensible next action.

---

# 3. The Core Prioritization Model

For every candidate target, ask:

```text id="t3xv8b"
1. Can I reach it?
2. What is it?
3. What services does it expose?
4. What authentication does it use?
5. Do I have potentially relevant access?
6. Does it matter to the current assessment objective?
7. What can I learn by investigating it?
8. What is still uncertain?
9. What is the smallest useful next action?
```

This keeps prioritization evidence-driven.

---

# 4. First Filter: Reachability

Start with:

```text id="n2m5fj"
Can I communicate with the target?
```

Classify the target:

```text id="1h6b5e"
Confirmed Reachable
      ↓
Continue Investigation
```

```text id="b4y9p2"
Uncertain
      ↓
Resolve Reachability
```

```text id="f7g8r3"
No Evidence of Reachability
      ↓
Deprioritize / Record
```

Do not spend significant effort on a target when the basic communication path has not been established unless resolving that uncertainty is itself important.

---

# 5. Second Filter: What Is the Host?

Identify the likely role.

Examples:

```text id="u0m8a7"
Workstation
File Server
Application Server
Database Server
Management Server
Linux Server
Domain Controller
Jump Host
Unknown
```

Host role provides context.

For example:

```text id="j5d2p1"
FILE01
   ↓
Likely File Server
   ↓
SMB is especially relevant
```

Whereas:

```text id="s4n9f3"
APP01
   ↓
Likely Application Server
   ↓
HTTP/HTTPS may be especially relevant
```

Role should guide investigation, not determine it automatically.

---

# 6. Third Filter: What Services Exist?

Build a service map.

Example:

| Target  | Role               | Services    |
| ------- | ------------------ | ----------- |
| WS01    | Workstation        | SMB, RDP    |
| FILE01  | File Server        | SMB         |
| APP01   | Application Server | HTTP, HTTPS |
| ADMIN01 | Management Host    | RDP, WinRM  |

Now ask:

```text id="q7u4s0"
Which services can potentially be used with the access I already have?
```

This is where service discovery becomes useful.

---

# 7. Fourth Filter: What Authentication Is Relevant?

Suppose:

```text id="x3q8v6"
Current Identity:
CORP\alice
```

and:

```text id="m6h0s2"
Target:
FILE01
Service:
SMB
Authentication:
Windows authentication
```

This creates a meaningful movement hypothesis.

Compare that with:

```text id="y9r2b5"
Target:
APP01
Service:
Custom Web Application
Authentication:
Unknown
```

The second target may still be important, but more information is required before selecting a movement path.

---

# 8. Fifth Filter: What Access Do You Already Have?

Available access can dramatically change target priority.

Potential access material may include:

```text id="g8k5j3"
Credentials
Password
Password Hash
Kerberos Ticket
SSH Key
Existing Session
Access Token
Configuration-Stored Credentials
Previously Validated Account
```

The important relationship is:

```text id="k2w6r4"
Available Access
      ↓
Compatible Authentication
      ↓
Compatible Service
      ↓
Candidate Target
```

Detailed credential discovery is covered in:

```text id="8n7f3w"
05-Credential-and-Access-Discovery/
```

---

# 9. Strong Movement Hypothesis

A target becomes especially interesting when several pieces of evidence align.

For example:

```text id="x5d1k9"
Target:
FILE01

Reachability:
Confirmed

Service:
SMB

Authentication:
Domain authentication

Credential:
Potentially valid domain credential

Role:
File Server

Objective:
Move deeper into internal server network
```

This is a strong reason to investigate the target further.

The correct next step is still:

```text id="c8m2s4"
Validate Access
```

not:

```text id="e4q7v9"
Assume Access
```

---

# 10. Information Gain

Sometimes you do not have credentials for a target.

That does not necessarily make the target irrelevant.

Consider:

```text id="m7r5x2"
Target A
Unknown role
Unknown services
Unknown authentication
```

versus:

```text id="s3q9k1"
Target B
Known role
Known services
Known authentication
No credential
```

Target B may be easier to reason about.

But Target A may provide more useful information if investigating it can clarify the environment.

Therefore ask:

> **What will this next action teach me?**

This is information gain.

---

# 11. Prioritize Actions, Not Just Hosts

A useful distinction is:

```text id="u8w4g2"
Target Priority
```

versus:

```text id="a9p6k5"
Action Priority
```

For example:

```text id="m1d7r3"
Target:
FILE01

Possible Actions:
1. Validate SMB access
2. Perform broader service enumeration
3. Search for credentials
```

The best next action is the one that resolves the most important uncertainty with the least unnecessary work.

---

# 12. Use the Smallest Useful Action

Suppose you already know:

```text id="q5x8m0"
FILE01
445/tcp
SMB
```

and you have a potentially valid domain credential.

You probably do not need another broad host-discovery scan.

The next question is narrower:

```text id="r3f7k1"
Can the credential authenticate to the relevant service?
```

This is more efficient:

```text id="c7z2n8"
Known Evidence
     ↓
Smallest Useful Test
     ↓
New Evidence
     ↓
Next Decision
```

---

# 13. Candidate Target Matrix

Use a working matrix.

| Target  | Reachability | Role        | Service   | Authentication | Potential Access     | Unknown              | Next Action |
| ------- | ------------ | ----------- | --------- | -------------- | -------------------- | -------------------- | ----------- |
| FILE01  | Confirmed    | File Server | SMB       | Domain         | Credential available | Authorization        | Validate    |
| APP01   | Confirmed    | App Server  | HTTPS     | Application    | None                 | Auth model           | Enumerate   |
| ADMIN01 | Confirmed    | Management  | WinRM/RDP | Windows        | Credential available | Remote authorization | Validate    |
| WS03    | Uncertain    | Workstation | SMB/RDP   | Windows        | None                 | Reachability         | Validate    |

This makes the reasoning visible.

---

# 14. Decision Categories

Each target can be placed into a temporary investigation state:

```text id="z2k8p1"
READY TO VALIDATE
```

Evidence already supports a specific access test.

```text id="j4m7c6"
NEEDS INFORMATION
```

Important information is missing.

```text id="w5n3x9"
DEPRIORITIZED
```

There is currently little evidence that the target provides a useful next step.

```text id="a8q2v4"
REASSESS LATER
```

The target may become relevant after gaining a new position or credential.

These are workflow states, not permanent judgments.

---

# 15. Example: Windows Environment

Suppose discovery produces:

```text id="s7k4f2"
WS01
10.10.10.20
SMB + RDP

FILE01
10.10.10.30
SMB

APP01
10.10.10.40
HTTP + HTTPS

ADMIN01
10.10.10.50
RDP + WinRM
```

And the current position has:

```text id="h5m8q1"
Domain Identity:
CORP\alice

Potential Credential:
Available

Current Objective:
Establish another internal host position
```

The reasoning becomes:

```text id="x6j3b9"
FILE01
  ↓
SMB
  ↓
Domain Authentication
  ↓
Potential Credential
  ↓
Validate

ADMIN01
  ↓
RDP / WinRM
  ↓
Domain Authentication
  ↓
Potential Credential
  ↓
Validate

APP01
  ↓
Web Services
  ↓
Authentication Unknown
  ↓
Gather More Information
```

The important part is not which host receives a label.

The important part is that each next action is supported by evidence.

---

# 16. Example: Linux Environment

Suppose:

```text id="r4c8y2"
Current Host:
Linux01

Candidate Targets:

Linux02
  └── SSH

Linux03
  └── HTTP

Linux04
  └── SSH + NFS
```

You also have:

```text id="p6n1d5"
Potential SSH Key
```

The reasoning becomes:

```text id="k8v2m4"
Linux02
  ↓
SSH
  ↓
Potentially compatible key
  ↓
Validate authorized access
```

Meanwhile:

```text id="q7s3f9"
Linux03
  ↓
HTTP
  ↓
No known authentication path
  ↓
Investigate application context
```

And:

```text id="b5x9j2"
Linux04
  ↓
SSH + NFS
  ↓
Potentially useful services
  ↓
Gather additional access information
```

---

# 17. Example: AD Environment

Suppose the environment contains:

```text id="j6r1z8"
WS01
FILE01
APP01
DC01
```

Current context:

```text id="v3q7m5"
Domain:
CORP.LOCAL

Current Account:
CORP\alice
```

Do not automatically treat:

```text id="dc01"
DC01
```

as the next target.

Instead consider:

```text id="e9k4s6"
What access exists?
Which services are reachable?
Which credentials are available?
What authentication paths are supported?
What is the assessment objective?
```

A Domain Controller can be highly relevant to an assessment while still requiring a different access path from the current position.

---

# 18. When No Target Has a Clear Path

This is a normal result.

Use:

```text id="g1m6w8"
No Clear Movement Path
        │
        ▼
Review Credentials
        │
        ▼
Review Existing Sessions
        │
        ▼
Review Service Enumeration
        │
        ▼
Review Network Context
        │
        ▼
Review Additional Networks
        │
        ▼
Reassess Targets
```

Do not manufacture a movement path simply because movement is the objective.

Sometimes the correct next step is:

```text id="u2p9c4"
05-Credential-and-Access-Discovery/
```

---

# 19. When Multiple Targets Look Useful

Use a comparison based on evidence.

Ask:

```text id="n4f8x1"
Which target has:
- confirmed reachability?
- a relevant service?
- compatible authentication?
- available credentials?
- a meaningful role?
- a clear validation step?
- information that could improve the next decision?
```

Then choose the next **investigation**.

After the result, reassess the remaining targets.

---

# 20. Do Not Overcommit to One Target

A lateral-movement assessment is iterative.

Example:

```text id="c7r2m9"
Target A
   ↓
Investigation
   ↓
No Useful Access
   ↓
Return to Candidate List
```

Or:

```text id="w8d4q5"
Target A
   ↓
Access Confirmed
   ↓
New Position
   ↓
Re-enumerate
   ↓
New Targets
```

The target list should remain dynamic.

---

# 21. Prioritization After New Evidence

New information can change the order of investigation.

For example:

```text id="k5v1s8"
Initial:
FILE01 → Interesting
APP01 → Unknown
ADMIN01 → Interesting
```

Then you discover:

```text id="p3x7n2"
Credential compatible with WinRM
```

Now:

```text id="q9m4c6"
ADMIN01
   ↓
WinRM
   ↓
Compatible Credential
   ↓
New Movement Hypothesis
```

The target's investigation state changes because the evidence changed.

Therefore:

> **Prioritization must be revisited whenever new credentials, sessions, routes, services, or host information are discovered.**

---

# 22. Prioritization Record

For the selected target, record:

```text id="w7k3f9"
Selected Target:
____________________

Why It Is Relevant:
____________________

Reachability Evidence:
____________________

Host Role:
____________________

Relevant Service:
____________________

Authentication:
____________________

Potential Access:
____________________

Known Unknowns:
____________________

Next Action:
____________________

Expected Result:
____________________

What I Will Do If It Succeeds:
____________________

What I Will Do If It Fails:
____________________
```

This turns target selection into an explicit decision.

---

# 23. Failure Handling

A failed access attempt should update the model.

Example:

```text id="y4n8p2"
Expected:
Credential → SMB → FILE01

Result:
Authentication failed
```

Do not immediately conclude:

```text id="v6q1r5"
Credential is useless everywhere.
```

Possible explanations include:

```text id="m3z7k9"
Wrong Credential
Wrong Account Context
Account Restriction
Service Restriction
Authorization Failure
Target Policy
Expired Credential
Network Filtering
```

The correct response is:

```text id="x8c4j1"
Classify Failure
      ↓
Update Evidence
      ↓
Reassess Hypothesis
      ↓
Choose Next Action
```

---

# 24. Success Handling

If access succeeds:

```text id="n5w2h7"
Access Confirmed
      ↓
Validate Session
      ↓
Identify New Host / User
      ↓
Enumerate New Position
      ↓
Review Network Context
      ↓
Review Credentials / Sessions
      ↓
Discover New Targets
```

Do not immediately jump to another target without understanding the new position.

This is the beginning of:

```text id="p3j9q6"
10-Post-Movement-Workflow/
```

---

# 25. Target Prioritization Checklist

```text id="h7x3v2"
[ ] Reachability considered
[ ] Host role identified
[ ] Services identified
[ ] Authentication context understood
[ ] Available credentials reviewed
[ ] Existing sessions considered
[ ] Current objective considered
[ ] Unknowns identified
[ ] Expected information gain considered
[ ] Next action is explicit
[ ] Success path defined
[ ] Failure path defined
[ ] Target list remains dynamic
```

---

# 26. Final Target Decision

Before acting on a target, complete this sentence:

> **I am investigating this target because __________.**

Then:

> **The evidence supporting this is __________.**

Then:

> **The next action is __________ because it will tell me __________.**

Finally:

> **If it succeeds, I will __________. If it fails, I will __________.**

If you cannot answer these questions, gather more information before proceeding.

---

# 27. Complete Target Discovery Workflow

The entire section now becomes:

```text id="q8m4t1"
CURRENT POSITION
      │
      ▼
HOST DISCOVERY
      │
      ├── Candidate Hosts
      │
      ▼
NETWORK ENUMERATION
      │
      ├── Networks
      ├── Routes
      ├── Connectivity
      └── Network Boundaries
      │
      ▼
SERVICE DISCOVERY
      │
      ├── Ports
      ├── Protocols
      ├── Services
      └── Authentication Clues
      │
      ▼
TARGET PRIORITIZATION
      │
      ├── Evidence
      ├── Access
      ├── Role
      ├── Objective
      └── Unknowns
      │
      ▼
CANDIDATE MOVEMENT PATH
      │
      ▼
CREDENTIAL / ACCESS DISCOVERY
```

---

# 28. Target Discovery Completion Criteria

The Target Discovery phase is complete for the current position when you can answer:

```text id="r2y7k4"
What hosts exist?
What networks are relevant?
Which hosts are reachable?
What services do they expose?
What authentication mechanisms matter?
What access do I potentially possess?
Which targets have a justified next investigation?
What information is still missing?
What will I do next?
```

You do not need:

```text id="t6m3x8"
A complete network inventory
A complete domain inventory
A complete credential inventory
A successful movement path
```

Those are addressed progressively throughout the workflow.

---

# 29. What Next?

The Target Discovery phase is now complete.

The next question is:

> **"What credentials, authentication material, sessions, or existing access can I use against the targets I discovered?"**

Continue with:

```text id="j4v8p6"
05-Credential-and-Access-Discovery/
```

The next file is:

```text id="w3n7q2"
05-Credential-and-Access-Discovery/README.md
```

The workflow now changes from:

```text id="a8f5m1"
Where can I go?
```

to:

```text id="z6c2r9"
What do I already have that can get me there?
```

The core principle is:

> **Prioritize the next investigation using evidence rather than assumptions. A good target is one where reachability, services, authentication, available access, and the assessment objective combine into a clear next action. Reassess the decision whenever new evidence changes the situation.**
