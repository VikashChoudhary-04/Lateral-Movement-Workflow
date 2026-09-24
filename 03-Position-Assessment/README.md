# Position Assessment

Before attempting lateral movement, build a detailed understanding of the position you currently control.

The objective is not to enumerate everything on the host.

The objective is to answer:

> **What is this system, who am I, what can I access, what is it connected to, and what information can influence my next movement decision?**

This section expands the quick assessment from `02-First-5-Minutes/` into a structured workflow.

---

## Position Assessment Workflow

```text
                    CURRENT FOOTHOLD
                          │
                          ▼
                  Identify Current Host
                          │
                          ▼
                  Identify Current User
                          │
                          ▼
                Determine Effective Access
                          │
                          ▼
                  Map Network Context
                          │
                          ▼
               Identify Domain / Trust Context
                          │
                          ▼
                  Identify Existing Access
                          │
                          ▼
                 Build Position Summary
                          │
                          ▼
                  Select Next Investigation
```

The output of this section should be a **usable position map**.

---

# 1. What Is a Position?

A position is the combination of:

```text
Host
User
Privileges
Network Location
Domain / Trust Context
Reachable Systems
Available Services
Existing Sessions
Available Access
```

For example:

```text
Host: WEB01
User: websvc
Privilege: Limited
Network: 10.10.10.0/24
Domain: CORP.LOCAL
Reachable Network: 10.10.20.0/24
Interesting Host: FILE01
Interesting Service: SMB
Potential Access: Valid account under investigation
```

The individual findings matter, but the **relationship between them** is what creates a movement opportunity.

---

# 2. Assessment Questions

Work through these questions:

### Identity

* What host am I on?
* What user am I?
* Is the account local or domain-based?
* What groups does the account belong to?
* What privileges are active?

### System

* What operating system is running?
* What role might the system have?
* What services are running?
* What other users or sessions are present?

### Network

* What interfaces exist?
* What IP addresses are assigned?
* What routes exist?
* What networks can the host potentially reach?
* What remote systems is it communicating with?

### Environment

* Is the host part of a domain?
* What domain is it associated with?
* What trust relationships are relevant?
* What other systems appear related to it?

### Access

* What credentials or authentication material are available?
* What existing authenticated sessions exist?
* What remote services can potentially be used?
* What accounts appear relevant?

---

# 3. Assessment Order

Use this order as the default:

```text
1. Current Host
       ↓
2. Current User
       ↓
3. Effective Privileges
       ↓
4. Operating System
       ↓
5. Network Context
       ↓
6. Domain / Trust Context
       ↓
7. Existing Sessions
       ↓
8. Existing Connections
       ↓
9. Available Access
       ↓
10. Position Summary
```

This order moves from **identity → system → network → environment → access**.

---

# 4. Why Order Matters

The findings build on one another.

For example:

```text
Current Host
     ↓
Current IP
     ↓
Network
     ↓
Domain
     ↓
Potential Targets
     ↓
Available Services
     ↓
Potential Access
```

Without knowing the current position, target discovery can become noisy.

Without knowing the current account, discovered credentials may be difficult to interpret.

Without knowing network reachability, a discovered target may not be useful.

The workflow therefore builds context before making movement decisions.

---

# 5. Current Host

Determine the identity and role of the system you control.

Detailed procedures:

```text
Current-Host.md
```

Questions to answer:

```text
What is the hostname?
What OS is running?
What version/build is present?
What architecture is being used?
What role might the host have?
What services are running?
```

Do not assume a host's role solely from its hostname.

Treat names as clues and validate them with system information.

---

# 6. Current User

Determine exactly which account represents your current security context.

Detailed procedures:

```text
Current-User.md
```

Questions:

```text
Who am I?
Is this a local or domain account?
What groups am I a member of?
What account identifier is associated with the session?
Are there other relevant accounts?
```

The account identity is critical because it determines which authentication and authorization paths may be available.

---

# 7. Effective Access

Do not stop after identifying the username.

Determine what the current session can actually do.

Consider:

```text
Account
   ↓
Groups
   ↓
Privileges
   ↓
Effective Permissions
```

For example:

```text
User
  │
  ├── Member of Group A
  │
  ├── Member of Group B
  │
  └── Specific Privilege
```

The important question is:

> **What does this combination of identity and privileges allow me to do?**

If higher privileges are required, move into the appropriate PrivEsc workflow.

---

# 8. Network Context

A lateral-movement decision depends heavily on network position.

Determine:

```text
Interfaces
    ↓
IP Addresses
    ↓
Subnets
    ↓
Gateway
    ↓
Routes
    ↓
Reachable Networks
```

Example:

```text
                CURRENT HOST
                     │
          ┌──────────┴──────────┐
          ▼                     ▼
    10.10.10.0/24         10.10.20.0/24
       Network A              Network B
```

A second network may represent a new movement opportunity.

However:

> **A route does not prove that every host on that network is reachable.**

Validate reachability before treating it as an actionable path.

---

# 9. Domain and Trust Context

In Windows and Active Directory environments, identify:

```text
Computer
User
Domain
Domain membership
Relevant trust relationships
```

Think of the environment as:

```text
                 DOMAIN
                    │
        ┌───────────┼───────────┐
        ▼           ▼           ▼
      HOST-A      HOST-B      HOST-C
```

The goal at this stage is not to fully map Active Directory.

The goal is to determine:

> **What security environment does this host belong to?**

Detailed Active Directory enumeration belongs in the later workflow where appropriate.

---

# 10. Existing Sessions and Connections

Existing sessions can provide valuable environmental information.

Look for:

```text
Logged-in users
Remote sessions
Active network connections
Connections to servers
Administrative connections
Application connections
```

Represent the findings as relationships:

```text
CURRENT HOST
     │
     ├── Connected to → FILE01
     ├── Connected to → APP01
     └── Session from → USER01
```

These relationships may help identify targets or explain how systems interact.

Do not automatically interpret a connection as an available movement path.

Treat it as **evidence that requires validation**.

---

# 11. Available Access

Now ask:

> **What can I potentially use to authenticate somewhere else?**

Possible categories include:

```text
Passwords
Hashes
Kerberos material
SSH keys
Tokens
Sessions
Service credentials
Existing authenticated connections
```

At this stage, record the existence and type of relevant access.

Detailed investigation belongs in:

```text
05-Credential-and-Access-Discovery/
```

---

# 12. Build the Position Map

Combine your findings.

Use a format such as:

```text
CURRENT POSITION
================

Host:
User:
Privileges:
OS:
IP:
Subnet:
Gateway:
Routes:
Domain:
Other Sessions:
Interesting Connections:
Potential Targets:
Available Access:
```

Then translate it into relationships:

```text
                    CURRENT POSITION
                          │
             ┌────────────┼────────────┐
             ▼            ▼            ▼
           Identity     Network       Access
             │            │            │
             ▼            ▼            ▼
           USER01      NET-A       Credential
             │            │            │
             └────────────┼────────────┘
                          ▼
                    Potential Target
                          │
                          ▼
                       FILE01
                          │
                          ▼
                         SMB
```

This is much more useful than a collection of unrelated command outputs.

---

# 13. From Assessment to Action

The assessment should end with a decision.

Use this model:

```text
                 POSITION ASSESSMENT
                         │
            ┌────────────┼────────────┐
            ▼            ▼            ▼
       Target Found   Access Found   New Network
            │            │            │
            ▼            ▼            ▼
      Target        Credential /     Pivoting /
      Discovery     Access          Internal Access
            │       Discovery
            │            │
            └──────┬─────┘
                   ▼
             Service Discovery
                   │
                   ▼
             Movement Decision
```

### If the host reveals a potential target

Go to:

```text
04-Target-Discovery/
```

### If useful credentials or authentication material are identified

Go to:

```text
05-Credential-and-Access-Discovery/
```

### If a known target and remote service are already identified

Go to:

```text
06-Remote-Access-Services/
```

### If another internal network is visible but not directly reachable

Go to:

```text
09-Pivoting-and-Internal-Network-Access/
```

---

# 14. Avoid Unnecessary Enumeration

More enumeration is not automatically better.

The objective is:

```text
Useful Information
       ↓
Better Decision
       ↓
Better Action
```

Avoid collecting large amounts of information without understanding why it matters.

Before running an enumeration command, ask:

> **What decision will this information help me make?**

If there is no useful answer, consider whether the action belongs in the current stage.

---

# 15. Position Assessment Checklist

```text
[ ] Host identified
[ ] Current user identified
[ ] Local/domain context identified
[ ] Effective privileges identified
[ ] Operating system identified
[ ] Network interfaces identified
[ ] IP addresses identified
[ ] Routes identified
[ ] Reachable networks considered
[ ] Domain/trust context assessed
[ ] Existing sessions reviewed
[ ] Existing connections reviewed
[ ] Relevant services noted
[ ] Available access identified
[ ] Potential targets noted
[ ] Position summary created
[ ] Next workflow branch selected
```

---

# 16. Position Assessment Decision Rule

The section is complete when you can clearly state:

```text
I am on: __________

I am authenticated as: __________

My effective privileges are: __________

This host is connected to: __________

I can potentially reach: __________

The relevant domain/environment is: __________

I have or may have access to: __________

The most relevant target currently identified is: __________

The next thing I need to investigate is: __________
```

If you cannot answer these questions, continue assessing the current position.

If you can answer them, move into the appropriate next workflow.

---

# 17. What Comes Next?

Position assessment produces the context required for the next stages.

```text
Current Position
       │
       ▼
Target Discovery
       │
       ▼
Credential / Access Discovery
       │
       ▼
Remote Service Discovery
       │
       ▼
Movement
```

The next section is:

```text
04-Target-Discovery/
```

That section answers:

> **"Now that I understand where I am, what systems can I potentially move to?"**
