# Current User

After identifying the current host, determine exactly **who you are operating as**.

The objective is not simply to obtain a username.

The objective is to understand:

```text
Who am I?
Is the account local or domain-based?
What groups am I a member of?
What privileges does the account have?
What security context is the session using?
What systems or resources might this identity be able to access?
```

Identity is one of the primary inputs into every lateral-movement decision.

---

# 1. The Identity Assessment Workflow

```text id="1z0vqr"
Current Session
      │
      ▼
Identify Account
      │
      ▼
Determine Local / Domain Context
      │
      ▼
Identify Account Identifier
      │
      ▼
Identify Group Membership
      │
      ▼
Identify Effective Privileges
      │
      ▼
Understand Security Context
      │
      ▼
Determine Movement Implications
```

The goal is to turn:

```text
"I am user X."
```

into:

```text
"I am user X, in this security context, with these groups
and privileges, and this identity may have access to these
types of systems or services."
```

---

# 2. Identify the Current Account

## Linux

Start with:

```bash id="6r8nps"
whoami
```

Then:

```bash id="v0v9hj"
id
```

`id` provides additional information about the account and its group memberships.

You can also inspect the current shell identity:

```bash id="1l3m52"
echo $USER
```

---

## Windows

Start with:

```cmd id="0v3r4p"
whoami
```

Then:

```cmd id="y3xv4x"
whoami /user
```

PowerShell:

```powershell id="7u8njk"
[System.Security.Principal.WindowsIdentity]::GetCurrent().Name
```

### Record

```text id="5m2gdo"
Username:
Account Identifier:
```

---

# 3. Determine Local vs Domain Identity

This distinction is extremely important on Windows.

A user may be:

```text id="l2x0qs"
LOCALHOST\user
```

or:

```text id="7v9l6c"
DOMAIN\user
```

These represent different security contexts.

### Windows

```cmd id="h9l4z5"
whoami
```

You can also inspect:

```cmd id="trzvte"
echo %USERDOMAIN%
```

PowerShell:

```powershell id="2c9d9y"
$env:USERDOMAIN
```

### Interpretation

```text id="v1z4cp"
Local Account
     │
     └── Primarily tied to the local system

Domain Account
     │
     └── May have access across multiple domain systems
```

Do not assume a domain account can access every domain system.

Authentication and authorization still need to be validated.

---

# 4. Why Account Type Matters

Consider:

```text id="8c8g8n"
WORKSTATION01
     │
     └── Local User
```

versus:

```text id="0l6yyj"
WORKSTATION01
     │
     └── CORP\user
             │
             ├── FILE01
             ├── APP01
             └── Other domain resources
```

A domain identity may participate in a much broader set of authentication relationships.

Therefore, after determining that an account is domain-based, investigate the surrounding domain context.

This does **not** mean immediately attempting access to every system.

First identify where the identity is relevant.

---

# 5. Identify Group Membership

Group membership can significantly affect what an account can do.

## Linux

```bash id="bbr2nz"
id
```

```bash id="j28qgl"
groups
```

You can also inspect group information associated with the current account.

### Look for

```text id="blt8q5"
Administrative groups
Privileged groups
Service-related groups
Groups associated with remote access
```

---

## Windows

```cmd id="4m2wq5"
whoami /groups
```

This shows groups associated with the current security token.

You can also inspect local group membership where appropriate:

```cmd id="c7m9m7"
net user %USERNAME%
```

For domain context:

```cmd id="f5k5l4"
whoami /groups
```

### Look For

Examples include:

```text id="6u6jgm"
Administrators
Remote Desktop Users
Remote Management Users
Domain Users
Domain Admins
Other privileged or application-specific groups
```

Group membership is evidence of potential authorization.

It is not by itself proof that every associated action will succeed.

---

# 6. Understand the Security Token / Context

On Windows, the current process operates with a security token containing identity, groups, privileges, and other security information.

Start with:

```cmd id="i3j9wq"
whoami /all
```

This combines useful identity information into one view.

It can show:

```text id="r0a2k7"
User
SID
Groups
Privileges
Authentication information
```

### Why?

Two sessions using similarly named accounts can behave differently depending on their security context.

For example:

```text id="x0c3la"
Same User
   │
   ├── Different Session
   ├── Different Token
   └── Different Effective Access
```

Always reason from the **actual security context**, not just the username.

---

# 7. Identify Effective Privileges

Group membership is only part of the picture.

### Windows

```cmd id="i9v1qk"
whoami /priv
```

Or:

```cmd id="1g5g5j"
whoami /all
```

Look for privileges that may affect what the current account can do.

Examples include:

```text id="3tq7b0"
SeDebugPrivilege
SeImpersonatePrivilege
SeBackupPrivilege
SeRestorePrivilege
```

The presence of a privilege does not automatically mean it can be successfully abused.

The important point here is:

> **Determine whether the current security context is sufficient for the next action.**

If you identify a privilege-escalation opportunity rather than a lateral-movement opportunity, switch to the appropriate PrivEsc workflow.

---

# 8. Linux Privilege Context

For Linux:

```bash id="0i3yyh"
id
```

Check sudo permissions where authorized:

```bash id="s0o2ut"
sudo -l
```

Check groups:

```bash id="e6y0b0"
groups
```

### Look For

```text id="p9yp92"
root
sudo
wheel
Administrative groups
Application-specific privileged groups
```

### Decision

If the account has useful local privileges:

```text id="f0kj2g"
Investigate local privilege escalation
```

If the account is limited:

```text id="0f8sp4"
Continue assessing:
- Network access
- Credentials
- Remote services
- Other systems
```

---

# 9. Determine Whether the Account Is a Service Account

Not every account represents a human user.

You may encounter accounts such as:

```text id="y7j2re"
websvc
sqlsvc
backup
jenkins
appuser
svc-web
```

Treat the naming as a clue, not proof.

Ask:

```text id="8t9wcb"
Is this account used by a service?
What process uses it?
What systems might depend on it?
What permissions does it have?
```

### Why This Matters

Service accounts may have access relationships that differ from ordinary user accounts.

For example:

```text id="e9n9df"
SERVICE ACCOUNT
      │
      ├── Application
      ├── Database
      └── Internal Service
```

That can reveal potential movement paths.

---

# 10. Identify Other Logged-In Users

The current account isn't necessarily the only useful identity on the system.

## Linux

```bash id="1p8w3g"
who
```

```bash id="36ox6c"
w
```

## Windows

```cmd id="s6m8y0"
query user
```

Depending on the environment, additional session information may be available through Windows administrative tooling.

### What to Record

```text id="p6e9q5"
Username
Session Type
Session State
Source / Origin if visible
```

### Why?

Other sessions can reveal:

* Administrative users
* Domain users
* Remote connections
* Active operational accounts
* Relationships between systems

Do not automatically attempt to interact with another user's session.

Treat the information as **environmental evidence** unless you have explicit authorization and a specific assessment objective.

---

# 11. Identity → Access Mapping

Now connect identity information to lateral movement.

Think:

```text id="s9r1k5"
Current Identity
      │
      ▼
Groups
      │
      ▼
Privileges
      │
      ▼
Authorization
      │
      ▼
Potential Remote Access
      │
      ▼
Potential Targets
```

For example:

```text id="z9aq42"
CORP\alice
    │
    ├── Domain Users
    ├── Remote Management Users
    └── Application Group
             │
             ▼
       Potential Access
             │
             ▼
        Candidate Hosts
```

This is much more useful than simply recording:

```text
Username = alice
```

---

# 12. Do Not Assume Group Membership Equals Access

A common mistake is:

```text id="m2g7hd"
User is in Group X
        ↓
Therefore access to HOST-B exists
```

That conclusion may be wrong.

Instead:

```text id="u7jv6b"
Group Membership
       ↓
Potential Authorization
       ↓
Identify Target
       ↓
Validate Service
       ↓
Attempt Authorized Authentication
       ↓
Confirm Actual Access
```

The workflow always separates:

**identity → potential authorization → validated access**

---

# 13. Identity and Credential Discovery

Your current identity can guide later credential investigation.

For example:

```text id="j6qk0y"
Current Account
      │
      ├── Local
      ├── Domain
      ├── Service
      └── Privileged
             │
             ▼
      Relevant Credential
      Discovery Paths
```

Do not assume every credential found belongs to the current user.

Always determine:

```text
Who does this credential belong to?
Where is it valid?
What authentication mechanism does it represent?
What systems could potentially accept it?
```

Detailed credential investigation belongs in:

```text
05-Credential-and-Access-Discovery/
```

---

# 14. Identity → Movement Decision

At the end of identity assessment, classify your position.

### Case 1 — Local low-privileged user

```text id="xwqf6e"
Local User
    ↓
Limited Privileges
    ↓
Investigate:
- Network
- Remote services
- Credentials
- Other targets
```

### Case 2 — Domain user

```text id="3w5q9n"
Domain User
    ↓
Identify Domain Context
    ↓
Identify Relevant Targets
    ↓
Investigate Available Authentication
```

### Case 3 — Privileged account

```text id="jv4y7m"
Privileged Identity
    ↓
Determine What That Privilege Actually Provides
    ↓
Identify Authorized Remote Access
```

Do not assume that a privileged identity automatically gives unrestricted access.

### Case 4 — Service account

```text id="2i6p4s"
Service Account
    ↓
Identify Associated Service
    ↓
Identify Systems / Resources
    ↓
Investigate Relevant Access Paths
```

---

# 15. Identity Assessment Checklist

```text id="3exjkn"
[ ] Current username identified
[ ] Account identifier identified
[ ] Local/domain context identified
[ ] Group membership identified
[ ] Effective privileges identified
[ ] Security context assessed
[ ] Service-account possibility considered
[ ] Other logged-in users reviewed
[ ] Potential authorization relationships identified
[ ] Relevant credential-discovery paths identified
[ ] Potential movement implications recorded
[ ] Next workflow selected
```

---

# 16. Final Identity Summary

Finish with a concise summary:

```text id="m38f4p"
CURRENT IDENTITY
================

Account:
Account Type:
Domain / Local:
Groups:
Effective Privileges:
Security Context:
Service Account?:
Other Relevant Sessions:

Potential Access:
Potential Targets:

Next Investigation:
```

The objective is to be able to state:

> **"I know exactly which identity I control, what security context it operates under, what authorization it may have, and how that identity changes my lateral-movement options."**

---

# 17. What Comes Next?

Identity alone does not tell you where you can move.

Combine it with network and environmental information:

```text id="e3p5jh"
Current Identity
      │
      +
Network Context
      │
      +
Domain / Trust Context
      │
      ▼
Potential Movement Opportunities
```

Continue with:

```text id="9w2q2h"
03-Position-Assessment/Network-Context.md
```

The next file answers:

> **"What networks, interfaces, routes, and connections are available from my current position, and how do they affect where I can potentially move?"**
