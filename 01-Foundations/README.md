# Lateral Movement Foundations

Lateral movement begins after an initial foothold has been obtained.

The objective is to use existing access, credentials, authentication material, trusted relationships, or reachable services to move from the current position to another system or security context.

This section establishes the minimum concepts required to operate the lateral movement workflow.

---

## 1. The Core Idea

The simplest model is:

```text
Current Position
      │
      │ Existing Access
      ▼
Target System
      │
      ▼
New Position
```

The target could be:

* Another workstation
* A server
* A Linux host
* A Windows host
* An Active Directory system
* Another account or security context

The important point is that lateral movement changes **where you have access**, not necessarily how privileged you are.

---

## 2. Lateral Movement vs Privilege Escalation

These are different operations.

### Privilege Escalation

The goal is to obtain greater privileges while remaining in the current security context or host.

```text
HOST-A

Low Privilege
     │
     ▼
Higher Privilege
```

Example:

```text
Windows user
     ↓
Local Administrator
```

or:

```text
Linux user
     ↓
root
```

### Lateral Movement

The goal is to reach another system or security context.

```text
HOST-A
   │
   ▼
HOST-B
```

You may move without gaining higher privileges.

### Combined Workflow

In an actual engagement, both commonly interact:

```text
Initial Foothold
      ↓
Enumerate
      ↓
Privilege Escalation
      ↓
Credential / Access Discovery
      ↓
Lateral Movement
      ↓
New Host
      ↓
Enumerate Again
      ↓
Privilege Escalation
      ↓
Lateral Movement
      ↓
...
```

### Practical Rule

Ask:

> **Am I trying to become more privileged, or am I trying to reach somewhere else?**

If the answer is:

```text
More privilege → Privilege Escalation
Another system → Lateral Movement
```

---

## 3. Lateral Movement vs Pivoting

These concepts are related but different.

### Lateral Movement

Moving from one system to another:

```text
HOST-A ─────────► HOST-B
```

### Pivoting

Using a compromised system as a path into a network or system that the original attacker position cannot directly reach:

```text
Attacker
    │
    ▼
COMPROMISED HOST
    │
    │ Pivot
    ▼
Internal Network
    │
    ├── HOST-B
    ├── HOST-C
    └── HOST-D
```

Pivoting can therefore **enable** lateral movement.

A useful distinction is:

> **Lateral movement changes the destination. Pivoting changes the path available to reach that destination.**

---

## 4. The Four Things You Need to Move

Successful lateral movement generally depends on understanding four things:

```text
             Lateral Movement
                    │
        ┌───────────┼───────────┐
        ▼           ▼           ▼
      Target      Access      Service
        │           │           │
        └───────────┼───────────┘
                    ▼
              Reachability
```

### 4.1 Target

You need to know **where you want to go**.

Examples:

```text
FILE01
APP01
DB01
DC01
```

---

### 4.2 Access

You need some form of authentication or authorization.

Examples:

* Valid username and password
* Password hash
* Kerberos authentication material
* SSH private key
* Existing authenticated session
* Other legitimate access mechanisms

---

### 4.3 Remote Service

The target must expose a service that can provide remote access.

Examples:

| Service | Common Environment |
| ------- | ------------------ |
| SMB     | Windows            |
| WinRM   | Windows            |
| RDP     | Windows            |
| WMI     | Windows            |
| RPC     | Windows            |
| SSH     | Linux / Unix       |

---

### 4.4 Reachability

Your current position must be able to communicate with the target, either directly or through an authorized network path.

```text
Current Host
     │
     │ Network Path
     ▼
Target
```

If the target is not reachable, possessing credentials alone does not solve the problem.

---

## 5. Authentication vs Authorization

These concepts matter throughout the workflow.

### Authentication

Answers:

> **Who are you?**

Examples:

```text
Username + Password
Kerberos
SSH Key
```

### Authorization

Answers:

> **What are you allowed to do?**

For example:

```text
User
 │
 ├── Can authenticate to HOST-B
 │
 ├── Can access SMB
 │
 └── Cannot use administrative functions
```

Therefore:

```text
Successful Authentication
        ≠
Full Administrative Access
```

Always distinguish between:

1. Can I authenticate?
2. Can I establish a session?
3. What can I actually do after authentication?

---

## 6. Credentials Are Not the Same as Access

Finding a credential does not automatically mean you can move laterally.

For example:

```text
Credential discovered
        ↓
Which account?
        ↓
Where is the account valid?
        ↓
What systems can it access?
        ↓
Which services accept it?
        ↓
What privileges does it have there?
```

The workflow therefore treats credential discovery as the **beginning of an investigation**, not the end.

---

## 7. Remote Services Are Movement Paths

Think of remote services as possible doors into another system.

```text
                 TARGET
                   │
        ┌──────────┼──────────┐
        ▼          ▼          ▼
       SMB        WinRM       RDP
        │          │          │
        └──────────┼──────────┘
                   ▼
             Remote Access
```

The presence of a service does not automatically mean access is possible.

You still need to determine:

* Is the service reachable?
* Is authentication required?
* Do you have suitable credentials?
* Is your account authorized?
* What level of access will it provide?

---

## 8. The Lateral Movement Decision Loop

The core workflow can be reduced to a repeating loop:

```text
1. Where am I?
       ↓
2. What access do I have?
       ↓
3. What systems can I reach?
       ↓
4. What services are available?
       ↓
5. What authentication can I use?
       ↓
6. Which movement paths are possible?
       ↓
7. Can I establish access?
       ↓
8. Did I reach the target?
       ↓
9. What did I gain?
       ↓
10. What can I discover from the new position?
       ↓
11. Repeat
```

This loop is the foundation of the entire repository.

---

## 9. Evidence Drives the Next Action

Avoid treating lateral movement as:

```text
Run command A
Run command B
Run command C
```

Instead:

```text
Action
  ↓
Observation
  ↓
Interpretation
  ↓
Decision
  ↓
Next Action
```

### Example

Suppose a target exposes SMB.

Do not immediately assume:

> "Use SMB."

Instead:

```text
SMB discovered
      ↓
Is the host reachable?
      ↓
YES
      ↓
Do I have suitable credentials?
      ↓
YES
      ↓
Can those credentials authenticate?
      ↓
Validate
      ↓
Access granted?
      ↓
YES
      ↓
Establish access
      ↓
Re-enumerate
```

If authentication fails:

```text
Authentication failed
      ↓
Determine why
      ↓
Check credentials / account / target / service
      ↓
Try another valid path or return to discovery
```

The result determines what happens next.

---

## 10. Position Matters

The same credential or service can have different value depending on where you currently are.

Consider:

```text
Attacker
   │
   ▼
WEB01
   │
   ├── Network A
   │
   └── Network B
```

After moving to another system:

```text
Attacker
   │
   ▼
WEB01
   │
   ▼
APP01
   │
   └── Internal Network C
```

`APP01` may provide visibility or access that was unavailable from `WEB01`.

Therefore, after every successful movement:

> **Treat the new host as a new starting position.**

---

## 11. Lateral Movement Is a Chain

A single movement may look like:

```text
HOST-A → HOST-B
```

A real environment can become:

```text
HOST-A
  ↓
HOST-B
  ↓
SERVER-A
  ↓
SERVER-B
  ↓
HOST-C
```

At each stage:

```text
New Host
   ↓
Re-enumerate
   ↓
Discover New Information
   ↓
Identify New Access
   ↓
Identify New Targets
   ↓
Move Again
```

This is why **post-movement enumeration** is part of lateral movement rather than an optional extra.

---

## 12. Windows, Linux, and Active Directory

The workflow is cross-platform, but the available movement mechanisms differ.

### Windows

Common remote access mechanisms include:

* SMB
* WinRM
* RDP
* WMI
* RPC

### Linux

A common remote access mechanism is:

* SSH

Other services may also provide remote access depending on the environment.

### Active Directory

Active Directory introduces additional concepts:

* Domain accounts
* Kerberos
* NTLM
* Domain trust relationships
* Administrative privileges
* Service accounts
* Authentication material
* Domain-connected hosts

The underlying workflow remains the same:

```text
Position
   ↓
Access
   ↓
Reachability
   ↓
Service
   ↓
Authentication
   ↓
Movement
   ↓
New Position
```

The available mechanisms simply change.

---

## 13. What Does Successful Movement Mean?

Successful movement should not be defined only as:

> "I got a shell."

Validate the new position.

Determine:

```text
Who am I?
Where am I?
What privileges do I have?
What network am I connected to?
What systems can I reach?
What services are available?
What new information is visible?
What new credentials or access might exist?
```

A successful movement therefore produces a **new position from which the workflow can continue**.

---

## 14. Failure Is Information

A failed movement attempt is not necessarily wasted effort.

For example:

```text
Connection failed
```

may indicate:

* The target is unreachable.
* The service is unavailable.
* A firewall blocks the connection.
* The wrong service was selected.

Or:

```text
Authentication failed
```

may indicate:

* Incorrect credentials.
* Wrong authentication mechanism.
* Account restrictions.
* The account is not valid on the target.

Or:

```text
Access denied
```

may indicate:

* Authentication succeeded.
* Authorization is insufficient.
* The account lacks the required privilege.

The correct response is:

```text
Failure
  ↓
Interpret the failure
  ↓
Update your understanding
  ↓
Choose the next investigation
```

Do not simply repeat the same action blindly.

---

## 15. The Practical Mental Model

When you obtain a foothold, think:

```text
                 CURRENT POSITION
                        │
                        ▼
                  WHO AM I?
                        │
                        ▼
                  WHERE AM I?
                        │
                        ▼
               WHAT CAN I REACH?
                        │
                        ▼
              WHAT TARGETS EXIST?
                        │
                        ▼
              WHAT SERVICES EXIST?
                        │
                        ▼
             WHAT ACCESS DO I HAVE?
                        │
                        ▼
          WHAT AUTHENTICATION CAN I USE?
                        │
                        ▼
             WHICH PATH CAN I TAKE?
                        │
                        ▼
                ESTABLISH ACCESS
                        │
                        ▼
               VALIDATE NEW HOST
                        │
                        ▼
              ENUMERATE AGAIN
                        │
                        └──────────────► REPEAT
```

This is the mental model to carry into every subsequent section.

---

## 16. Before Moving On

You should be able to answer these questions:

* What is lateral movement?
* How is it different from privilege escalation?
* How is it different from pivoting?
* What are the four major things required for movement?
* What is the difference between authentication and authorization?
* Why doesn't finding a credential automatically mean you can move?
* Why are remote services important?
* Why does position matter?
* Why must you re-enumerate after movement?
* Why can a failed attempt still provide useful information?

If you can answer those questions, you have enough foundation to begin the **practical workflow**.

---

## Next

Continue to:

```text
02-First-5-Minutes/
```

The next section changes from **"What is lateral movement?"** to:

> **"I have just obtained a foothold. What do I actually do right now?"**
