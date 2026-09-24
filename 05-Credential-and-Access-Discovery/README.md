# Credential and Access Discovery

Credential and access discovery is the process of identifying authentication material, existing sessions, account information, and other forms of legitimate access that may enable movement from the current position to another system.

The objective is not to collect credentials indiscriminately.

The objective is to answer:

> **What authentication material or existing access do I currently possess, what systems or services could it apply to, and what should I validate next?**

The workflow is:

```text id="7n4kq2"
Current Position
      ↓
Identify Current Identity
      ↓
Identify Existing Access
      ↓
Review Sessions and Authentication Context
      ↓
Identify Credential Sources
      ↓
Identify Passwords / Hashes / Tickets / Keys
      ↓
Map Authentication Material
      ↓
Match Material to Services
      ↓
Validate Authorized Access
      ↓
Move or Reassess
```

---

# 1. Where This Fits

The overall lateral-movement workflow is:

```text id="5g8r2m"
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

The previous section answered:

> **Where can I potentially go?**

This section answers:

> **What do I already have that could get me there?**

---

# 2. Credential vs Access

These terms must remain separate.

A credential may be:

```text id="p2m8w4"
Username
Password
Password Hash
Kerberos Ticket
SSH Private Key
API / Service Credential
Authentication Token
```

Access is the ability to successfully use authentication material against a particular service or system.

Therefore:

```text id="r6j1k9"
Credential
    ≠
Access
```

A better model is:

```text id="x4v7n2"
Credential
    ↓
Compatible Authentication
    ↓
Target Service
    ↓
Authorization
    ↓
Validated Access
```

---

# 3. Existing Access Is Also Valuable

You may already have access without discovering a new credential.

Examples include:

```text id="8f3m6q"
Existing SSH Session
Existing SMB Session
Existing Windows Session
Kerberos Session
Authenticated Web Session
Remote Management Session
Mounted Network Resource
```

Therefore, ask:

> **What access already exists before searching for anything new?**

This can be more useful than immediately searching for credentials.

---

# 4. The Credential Discovery Workflow

Use this order:

```text id="q9t5w1"
1. Identify Current Identity
        ↓
2. Identify Existing Sessions
        ↓
3. Identify Authentication Context
        ↓
4. Identify Credential Sources
        ↓
5. Identify Authentication Material
        ↓
6. Determine Ownership / Context
        ↓
7. Match Material to Services
        ↓
8. Validate Access
        ↓
9. Record Result
        ↓
10. Reassess Targets
```

Do not begin by searching every possible file or credential store.

Start with what the current position already tells you.

---

# 5. Current Identity First

Before interpreting any credential material, establish:

```text id="s7k3v2"
Current User
Account Type
Groups
Domain / Workgroup
Privileges
Host
```

Linux:

```bash id="r4n8c1"
whoami
id
```

Windows:

```cmd id="m8q2z5"
whoami
whoami /user
whoami /groups
```

This provides context for everything discovered later.

---

# 6. Existing Sessions

Look for authentication that is already active.

Examples include:

```text id="g2v9x6"
SSH Sessions
SMB Connections
Remote Desktop Sessions
Kerberos Tickets
Mounted Shares
Existing Network Connections
```

The goal is:

```text id="u5k1p8"
Existing Session
      ↓
Identify Identity
      ↓
Identify Target
      ↓
Understand Access
      ↓
Determine Whether It Provides a Movement Path
```

An existing session may reveal more than a static credential.

---

# 7. Credential Categories

This repository separates authentication material into several categories:

```text id="w6j2r4"
Credentials
    │
    ├── Usernames / Account Information
    ├── Passwords
    ├── Password Hashes
    ├── Kerberos Tickets
    ├── Authentication Tokens
    ├── SSH Keys
    └── Existing Access
```

Each category has different:

```text id="y8m3q7"
Discovery Methods
Interpretation
Authentication Compatibility
Movement Paths
Failure Conditions
```

The dedicated files cover them individually.

---

# 8. Credentials vs Passwords

A credential is broader than a password.

For example:

```text id="c5f7n1"
Username + Password
```

is one form of credential.

But:

```text id="v2k8m4"
Username + SSH Key
```

is also a credential.

And:

```text id="p9r3x6"
Kerberos Ticket
```

can represent active authentication material without exposing the user's password.

Therefore, do not structure the workflow around passwords alone.

---

# 9. Password Discovery

Passwords may exist in places such as:

```text id="h4q7s2"
Configuration Files
Application Settings
Scripts
Environment Variables
Credential Stores
Service Configurations
Deployment Files
Documentation
```

The important question is not:

> **"Can I find a password?"**

It is:

> **"What account does this credential belong to, where can it authenticate, and does that authentication create a relevant movement path?"**

A discovered password without account context is incomplete evidence.

---

# 10. Password Hashes

A password hash is not the same thing as a plaintext password.

It may still be relevant to authentication depending on:

```text id="k6m1v9"
Protocol
Authentication Mechanism
Account Type
Target Service
Environment Configuration
```

For example, Windows environments may use NTLM-related authentication material.

This repository treats hash-based authentication as a separate movement topic:

```text id="n3x8q5"
08-Credential-Based-Movement/
```

The current section focuses on:

```text id="j7r4c2"
Finding
Identifying
Contextualizing
and Mapping
```

the authentication material.

---

# 11. Kerberos Authentication Material

In an AD environment, Kerberos tickets may represent active authentication context.

The important distinction is:

```text id="t5q9m3"
Password
    ≠
Kerberos Ticket
```

A ticket can provide authentication to compatible services without revealing the underlying password.

When a Kerberos ticket is discovered, determine:

```text id="s8f2k6"
Who does it represent?
What type of ticket is it?
Which service is it for?
Is it currently valid?
What target or service does it relate to?
```

Detailed ticket-based movement belongs in:

```text id="r1v7x4"
08-Credential-Based-Movement/Pass-the-Ticket.md
```

---

# 12. SSH Keys

On Linux and Unix-like systems, SSH keys can provide an important movement path.

Potentially relevant material may include:

```text id="m6p3w8"
Private Keys
Authorized Keys
SSH Configuration
Known Hosts
User Accounts
Service Accounts
```

The key question is:

```text id="f9k2c5"
Which account can this key authenticate as?
```

Then:

```text id="b4x7n1"
Which host?
      ↓
Which service?
      ↓
Is authentication accepted?
      ↓
What authorization is available?
```

Detailed SSH movement is covered later.

---

# 13. Existing Access

Sometimes the most important finding is not a credential.

For example:

```text id="e7r5m2"
Current Host
    │
    ├── Authenticated SMB Session → FILE01
    ├── SSH Session → LINUX02
    └── Kerberos Context → CORP.LOCAL
```

These sessions can reveal:

```text id="v8q1s4"
Target Hosts
Account Context
Authentication Mechanism
Available Services
Potential Movement Paths
```

Always investigate existing access before assuming that new credentials are required.

---

# 14. Credential Ownership Matters

Suppose you discover:

```text id="z2f8k5"
Username:
backup
Password:
********
```

You still need to determine:

```text id="a6m3r9"
Which system owns this account?
Is it local or domain-based?
Is it a service account?
Where is it expected to authenticate?
Is remote access permitted?
What authorization does it have?
```

Never assume:

```text id="u7p4c1"
Credential Found
    =
Credential Works Everywhere
```

---

# 15. Local vs Domain Credentials

This distinction is especially important in Windows environments.

Example:

```text id="g3x8m2"
WORKSTATION01\alice
```

is a local account context.

Whereas:

```text id="n5q7r4"
CORP\alice
```

is a domain account context.

The potential movement scope can be very different.

Always identify:

```text id="j9k2v6"
Account Name
Account Authority
Host
Domain
```

before deciding where to test access.

---

# 16. Service Accounts

Service accounts may be especially relevant because they can be used by applications or automated processes.

Examples:

```text id="w1r6k8"
Database Service
Web Application
Backup System
Scheduled Task
Monitoring System
Deployment System
```

If a credential belongs to a service account, determine:

```text id="c4m9p2"
Where is it used?
What systems does the service interact with?
What remote services accept the identity?
What authorization does it have?
```

Do not assume a service account has administrative access.

Validate the actual authorization.

---

# 17. Configuration Files as Credential Sources

Applications frequently require credentials to connect to other systems.

Possible locations include:

```text id="q8v3n5"
Application Configuration
Database Configuration
Environment Files
Deployment Scripts
Automation Scripts
Backup Configuration
Service Configuration
```

The important workflow is:

```text id="r2k7m4"
Credential Found
      ↓
Identify Application
      ↓
Identify Account
      ↓
Identify Destination
      ↓
Identify Protocol
      ↓
Determine Whether It Supports Movement
```

---

# 18. Environment Variables

Applications and services may receive credentials through environment variables.

For example:

```text id="p5x9c3"
DATABASE_USER
DATABASE_PASSWORD
API_KEY
SERVICE_ACCOUNT
```

A discovered value still requires context.

Record:

```text id="m7q2v8"
Variable:
Application:
Account:
Destination:
Protocol:
Potential Use:
```

Do not treat every secret as a lateral-movement credential.

Some credentials only work against local applications or external services.

---

# 19. Authentication Material Mapping

Once material is identified, build a map.

Example:

```text id="y3f8k1"
CORP\alice
     │
     ├── SMB → FILE01
     ├── RDP → ADMIN01
     └── WinRM → SERVER01
```

Another example:

```text id="d6m2r9"
SSH Key A
     │
     └── user@LINUX02
```

This is much more useful than a list such as:

```text id="h8p4w7"
password.txt
key.pem
ticket.kirbi
```

The goal is to connect:

```text id="q1n5x3"
Authentication Material
        ↓
Identity
        ↓
Service
        ↓
Target
        ↓
Authorization
```

---

# 20. Credential-to-Target Matrix

Use a working matrix:

| Identity / Material | Target  | Service  | Authentication | Authorization | Status   |
| ------------------- | ------- | -------- | -------------- | ------------- | -------- |
| CORP\alice          | FILE01  | SMB      | Windows        | Unknown       | Validate |
| CORP\alice          | ADMIN01 | WinRM    | Windows        | Unknown       | Validate |
| SSH Key A           | LINUX02 | SSH      | Key            | Unknown       | Validate |
| Ticket A            | APP01   | Kerberos | Kerberos       | Unknown       | Validate |

This prevents credentials from becoming disconnected from their possible use.

---

# 21. Authentication Compatibility

Before attempting access, check:

```text id="w5j8c2"
Does the credential type match the service?
```

Examples:

```text id="m9r3x6"
SSH Private Key
    ↓
SSH
```

```text id="c7k2p4"
Windows Account
    ↓
SMB / RDP / WinRM
```

```text id="v8n1q5"
Kerberos Ticket
    ↓
Kerberos-Compatible Service
```

Compatibility does not prove authorization.

It only establishes that the authentication path is technically relevant.

---

# 22. Credential Validation

Validation should be deliberate.

Use:

```text id="f2q6m9"
Credential
    ↓
Compatible Service
    ↓
Reachable Target
    ↓
Authentication Attempt
    ↓
Access Result
```

Possible outcomes:

```text id="j5x8r1"
Successful Authentication
Authentication Failure
Authorization Failure
Network Failure
Service Failure
Account Restriction
```

Record the result.

---

# 23. Authentication Success vs Authorization Success

A successful login does not necessarily mean administrative access.

For example:

```text id="n7c4v2"
Authentication
      ↓
SUCCESS
      ↓
Authorization
      ↓
LIMITED ACCESS
```

You may successfully authenticate but only receive:

```text id="m1p6k9"
Read Access
User Shell
Limited Share Access
Application Access
```

Therefore:

```text id="x3r8w5"
Authentication Success
      ≠
Administrative Access
```

This distinction is critical.

---

# 24. What If a Credential Fails?

Do not immediately discard it.

Classify the failure.

```text id="q8m4c7"
Authentication Failed
        │
        ├── Wrong Account?
        ├── Wrong Password / Key?
        ├── Wrong Target?
        ├── Wrong Service?
        ├── Account Restricted?
        ├── Authentication Method Unsupported?
        └── Network / Service Problem?
```

Then update the credential map.

For example:

```text id="z5n2k8"
CORP\alice
SMB → FILE01
FAILED

Do not conclude:
CORP\alice is useless.

Instead record:
SMB authentication to FILE01 failed.
Reason requires further investigation.
```

---

# 25. Credential Reuse

A credential may potentially work across multiple systems.

However:

```text id="a7f3m9"
Credential Reuse
      ≠
Credential Universality
```

Validate only where the assessment scope and evidence justify it.

The workflow is:

```text id="c2k8v5"
Known Credential
      ↓
Identify Candidate Targets
      ↓
Match Service
      ↓
Validate
      ↓
Record Result
```

---

# 26. Avoid Credential Spraying by Default

Finding a credential does not justify blindly testing it against every discovered host.

A better approach is:

```text id="h4m7q2"
Credential
   ↓
Understand Account Context
   ↓
Identify Likely Targets
   ↓
Identify Compatible Services
   ↓
Select Specific Validation
```

This reduces unnecessary activity and keeps the assessment evidence-driven.

---

# 27. Existing Sessions vs Static Credentials

Compare:

```text id="w8p3k6"
Static Credential
```

with:

```text id="n2r7x4"
Existing Authenticated Session
```

A session may already provide:

```text id="q6m1v9"
Authenticated Context
Target Relationship
Service Relationship
Session Identity
```

Therefore:

> **Always inspect existing access before assuming that new credential discovery is necessary.**

---

# 28. Credential Discovery and Target Discovery Form a Loop

These two sections should not be treated as one-way stages.

The workflow is:

```text id="s4k9p2"
Targets
  ↓
Credential Discovery
  ↓
Credential Found
  ↓
Re-evaluate Targets
  ↓
New Candidate
  ↓
Validate
  ↓
New Access
  ↓
New Position
```

New credentials can change which targets are relevant.

New targets can change which credentials are relevant.

---

# 29. Credential Discovery Record

For every potentially useful authentication material, record:

```text id="j8v5m3"
Credential / Material:
____________________

Identity:
____________________

Account Authority:
____________________

Source:
____________________

Host:
____________________

Credential Type:
____________________

Potential Services:
____________________

Potential Targets:
____________________

Authentication Status:
____________________

Authorization Status:
____________________

Evidence:
____________________

Next Action:
____________________
```

---

# 30. Example Credential Record

```text id="r6x2n9"
Credential / Material:
SSH Private Key

Identity:
backup

Account Authority:
LINUX02

Source:
Authorized assessment artifact

Host:
LINUX01

Credential Type:
SSH Key

Potential Service:
SSH

Potential Target:
LINUX02

Authentication Status:
Not yet validated

Authorization Status:
Unknown

Evidence:
Private key corresponds to a known backup account

Next Action:
Validate authorized SSH authentication to LINUX02
```

---

# 31. Common Mistakes

## Mistake 1: Treating Every Secret as a Movement Credential

A secret may only work against:

```text id="x4n7q2"
Local Application
External API
Database
Specific Service
```

Always identify its intended context.

---

## Mistake 2: Ignoring Account Authority

Always distinguish:

```text id="p8m3k6"
LOCAL\user
DOMAIN\user
```

The authentication scope may differ.

---

## Mistake 3: Treating Authentication as Authorization

Successful authentication does not prove administrative rights.

---

## Mistake 4: Testing Credentials Everywhere

Use evidence to identify plausible targets.

---

## Mistake 5: Ignoring Existing Sessions

An existing authenticated session may already provide a movement path.

---

## Mistake 6: Collecting Without Mapping

A list of:

```text id="z5q1r8"
Passwords
Hashes
Tickets
Keys
```

is less useful than:

```text id="c9m4v7"
Identity
   ↓
Authentication Material
   ↓
Service
   ↓
Target
   ↓
Authorization
```

---

# 32. Credential Discovery Decision Tree

```text id="u7k3p5"
Current Position
      │
      ▼
Existing Access?
   │         │
  Yes       No
   │         │
   ▼         ▼
Analyze    Identify
Sessions   Credential Sources
   │         │
   └────┬────┘
        ▼
Identify Authentication Material
        │
        ▼
Determine Account Context
        │
        ▼
Match to Services
        │
        ▼
Match to Targets
        │
        ▼
Potential Access?
      │       │
     No      Yes
      │       │
      ▼       ▼
Gather      Validate
More Info    Access
                │
                ▼
          Access Result
                │
                ▼
          Reassess Targets
```

---

# 33. Completion Criteria

Credential and access discovery for the current position is sufficiently complete when you can identify:

```text id="e3m8q6"
[ ] Current authentication context
[ ] Existing sessions
[ ] Existing remote access
[ ] Relevant account identities
[ ] Potential passwords
[ ] Potential hashes
[ ] Kerberos material where applicable
[ ] SSH keys where applicable
[ ] Other relevant authentication material
[ ] Credential ownership / account authority
[ ] Compatible services
[ ] Candidate targets
[ ] Authentication status
[ ] Authorization status
[ ] Failed validation results
[ ] Next movement investigation
```

You do not need to find every credential on the system.

You need enough evidence to identify a **realistic and authorized access path**.

---

# 34. Section Structure

This section is divided into:

```text id="v5n2k8"
Credentials.md
Passwords.md
Hashes.md
Kerberos.md
Tokens-and-Sessions.md
SSH-Keys.md
Existing-Access.md
```

Each file focuses on a different form of authentication or access.

The distinction is:

```text id="p6x3r9"
Credentials
→ What identities exist?

Passwords
→ What password-based authentication material exists?

Hashes
→ What hash-based authentication material exists?

Kerberos
→ What Kerberos authentication material exists?

Tokens and Sessions
→ What active authentication contexts exist?

SSH Keys
→ What key-based SSH access exists?

Existing Access
→ What remote access already works?
```

---

# 35. What Next?

The first question in this section is:

> **"What credential and account information exists that could be relevant to movement?"**

Continue with:

```text id="c7m1v4"
05-Credential-and-Access-Discovery/
└── Credentials.md
```

The workflow is now:

```text id="k9q3w6"
Target Discovery
      ↓
Credential / Account Discovery
      ↓
Passwords / Hashes / Tickets / Keys
      ↓
Existing Access
      ↓
Credential-to-Target Mapping
      ↓
Remote Access Validation
```

The core principle is:

> **Do not treat credentials as isolated secrets. Identify the account they belong to, understand the authentication mechanism they support, map them to compatible services and targets, validate the resulting access, and use the result to update the movement plan.**
