# Credentials

Credentials are authentication material associated with an identity.

In lateral movement, the important question is not simply:

> **"Can I find a credential?"**

It is:

> **"What identity does this credential represent, where can that identity authenticate, and which discovered targets could potentially accept it?"**

The workflow is:

```text id="k3p8w1"
Credential Material
      ↓
Identify Identity
      ↓
Identify Account Authority
      ↓
Determine Credential Type
      ↓
Determine Intended Use
      ↓
Identify Compatible Services
      ↓
Identify Candidate Targets
      ↓
Validate Authentication
      ↓
Validate Authorization
      ↓
Record Result
```

---

# 1. Where This Fits

The credential-discovery workflow is:

```text id="m7q2v5"
Target Discovery
      ↓
Credential Discovery
      ↓
Credential Type
      ↓
Service Compatibility
      ↓
Target Compatibility
      ↓
Access Validation
      ↓
Lateral Movement
```

This file provides the general credential workflow.

Specific credential types are covered separately:

```text id="d8r4n6"
Passwords.md
Hashes.md
Kerberos.md
Tokens-and-Sessions.md
SSH-Keys.md
Existing-Access.md
```

---

# 2. Credential vs Identity

An identity answers:

> **Who is this?**

A credential answers:

> **What authentication material can represent this identity?**

For example:

```text id="f5x9k2"
Identity:
CORP\alice

Credential:
Password
```

or:

```text id="v3m7q8"
Identity:
backup

Credential:
SSH Private Key
```

or:

```text id="p6c2r4"
Identity:
CORP\alice

Credential:
Kerberos Ticket
```

Keep these concepts separate.

---

# 3. Credential vs Access

A credential may exist without providing access to a particular target.

For example:

```text id="w8n3j6"
Credential:
CORP\alice

Target:
FILE01

Service:
SMB
```

This creates a possibility.

Only after authentication and authorization are validated can you establish:

```text id="q4r7m1"
CORP\alice
     ↓
SMB
     ↓
FILE01
     ↓
Access Confirmed
```

Therefore:

```text id="x9k2p5"
Credential
    ≠
Access
```

---

# 4. Identify the Account Authority

Before using a credential, determine where the account comes from.

Common contexts include:

```text id="g6m1v8"
LOCAL
DOMAIN
SERVICE
APPLICATION
```

Windows examples:

```text id="a2f7c9"
WORKSTATION01\alice
CORP\alice
```

These represent different account authorities.

The distinction matters because the same username can exist in multiple security contexts.

For example:

```text id="n5q8r3"
WORKSTATION01\admin
```

and:

```text id="b7m4x1"
CORP\admin
```

are not necessarily the same identity.

---

# 5. Identify the Credential Source

Whenever credential material is discovered, record where it came from.

Possible sources include:

```text id="u8p3k6"
Current Session
Configuration File
Environment Variable
Application
Service
Script
Credential Store
SSH Configuration
Kerberos Context
Password Database
Assessment Artifact
```

The source provides context.

For example:

```text id="t4m9q2"
Credential found in:
Application configuration

Likely purpose:
Application → Database authentication
```

This does not automatically mean:

```text id="s6r1v7"
Credential → Remote Host Access
```

You need to determine where the identity is authorized.

---

# 6. Identify Credential Type

Classify the material before deciding what to do with it.

Examples:

```text id="c5x8n2"
Plaintext Password
Password Hash
Kerberos Ticket
SSH Private Key
Authentication Token
Existing Session
API Credential
Service Credential
```

The credential type determines which services may be compatible.

---

# 7. Credential-to-Service Mapping

Create a relationship between credential type and authentication mechanism.

Example:

```text id="p3k7m1"
SSH Private Key
      ↓
SSH
```

```text id="v8r2c5"
Windows Account
      ↓
SMB / RDP / WinRM
```

```text id="m4q9x6"
Kerberos Ticket
      ↓
Kerberos-Compatible Service
```

The relationship is a hypothesis until validated.

---

# 8. Credential-to-Target Mapping

Once a compatible service is identified, connect it to a candidate target.

Example:

```text id="w7n3f8"
Credential:
CORP\alice

Service:
SMB

Target:
FILE01
```

The resulting movement hypothesis is:

```text id="g2c6p4"
CORP\alice
     ↓
SMB
     ↓
FILE01
     ↓
Validate
```

This is the core of credential-based movement planning.

---

# 9. Local Credentials

A local credential belongs to a particular host.

For example:

```text id="q9m4r7"
WORKSTATION01\alice
```

This does not automatically mean:

```text id="d6x2k8"
FILE01\alice
```

or:

```text id="y3p7v1"
SERVER01\alice
```

are the same account.

Local account reuse may exist in some environments, but it must be established through evidence and validated within the authorized scope.

---

# 10. Domain Credentials

A domain credential can have a broader authentication context.

Example:

```text id="a5n8q3"
CORP\alice
```

Potentially relevant services may include:

```text id="j7m2x6"
SMB
RDP
WinRM
Kerberos-compatible services
```

But domain membership does not mean universal access.

The workflow remains:

```text id="p4v9c2"
Domain Credential
      ↓
Compatible Service
      ↓
Target
      ↓
Authentication
      ↓
Authorization
```

---

# 11. Service Accounts

A credential may belong to an account used by an application or service.

Examples:

```text id="f8k3m6"
websvc
backupsvc
sqlsvc
deploysvc
```

Do not infer privileges from the name.

Instead determine:

```text id="r2q7n5"
Where is the account used?
Which service uses it?
Which systems does that service communicate with?
Which remote services could accept the identity?
What authorization exists?
```

A service account may have useful access without being an administrator.

---

# 12. Application Credentials

Applications often authenticate to other systems.

For example:

```text id="v6m1x9"
APP01
  │
  └── Application
          │
          └── DATABASE01
```

The application may use:

```text id="c4p8r2"
Database Username
Database Password
API Token
Service Account
Certificate
```

The first question is:

> **What system or service is this credential intended to authenticate to?**

Only then ask whether the same identity is relevant to lateral movement.

---

# 13. Credentials in Configuration

Configuration files may contain authentication material.

Examples include:

```text id="n7w3k5"
Database Configuration
Application Configuration
Service Configuration
Deployment Configuration
Backup Configuration
Automation Scripts
```

When a credential is found, record:

```text id="u2q9f6"
File:
Application:
Account:
Destination:
Protocol:
Credential Type:
Potential Remote Use:
```

This prevents context from being lost.

---

# 14. Environment-Based Credentials

Applications may receive credentials through environment variables.

For example:

```text id="m8r4x1"
DATABASE_USER
DATABASE_PASSWORD
API_TOKEN
SERVICE_USER
SERVICE_PASSWORD
```

Do not immediately classify every value as a movement credential.

Determine:

```text id="z5p2k7"
What application uses it?
What destination does it access?
What protocol is involved?
What identity does it represent?
```

---

# 15. Credential Discovery From Processes and Services

Running applications and services can provide clues about identities being used.

On Linux:

```bash id="w4n8c2"
ps aux
```

On Windows:

```cmd id="q7m3v9"
tasklist
```

PowerShell:

```powershell id="x6r2k5"
Get-CimInstance Win32_Service |
    Select-Object Name, StartName, State
```

The goal is to identify:

```text id="f1p9d4"
Service
   ↓
Running Identity
   ↓
Configuration
   ↓
Remote Dependencies
```

This does not mean extracting secrets from every process.

Start with identity and configuration context.

---

# 16. Existing Authentication Context

Sometimes the current system already contains useful authentication state.

Examples:

```text id="a8k4m2"
Kerberos Tickets
Authenticated SMB Sessions
SSH Sessions
Mounted Shares
Remote Management Sessions
Browser / Application Sessions
```

Treat this as credential context.

The question is:

> **What authenticated relationships already exist from this host?**

---

# 17. Credential Ownership

Every discovered credential should have an owner.

Use:

```text id="r5x7n3"
Credential
   ↓
Identity
   ↓
Authority
   ↓
Intended Service
```

Example:

```text id="p2c8v6"
Credential:
Password

Identity:
CORP\backup

Authority:
CORP.LOCAL

Intended Use:
Backup service

Potential Target:
BACKUP01
```

Without this information, the credential is difficult to use intelligently.

---

# 18. Credential Validation

Validation should follow:

```text id="y9m3k7"
Credential
      ↓
Known Identity
      ↓
Known Service
      ↓
Reachable Target
      ↓
Authentication
      ↓
Authorization
```

Possible outcomes:

```text id="q6v2p8"
Authentication Successful
Authentication Failed
Authorization Denied
Service Unavailable
Network Unreachable
Account Restricted
```

Record the exact result.

---

# 19. Authentication Failure

If authentication fails, do not immediately conclude that the credential is invalid.

Investigate:

```text id="m4r8x2"
Was the correct account used?
Was the correct authority used?
Was the correct service used?
Was the target correct?
Was the credential current?
Is the account allowed to authenticate remotely?
Is the service configured for that authentication method?
```

For example:

```text id="n7c1w5"
CORP\alice
   ↓
RDP → ADMIN01
   ↓
Authentication denied
```

This tells you:

```text id="s3q9k6"
RDP authentication to ADMIN01 was not successful.
```

It does not necessarily prove:

```text id="b8p4m1"
CORP\alice is invalid everywhere.
```

---

# 20. Authorization Failure

Authentication can succeed while authorization fails.

Example:

```text id="x2f7m9"
Credential
   ↓
Authentication SUCCESS
   ↓
Authorization DENIED
```

This is useful information.

It establishes:

```text id="r6k3v8"
Identity is recognized
Authentication works
Required access is not granted
```

The next step may therefore involve:

```text id="c5m1q7"
Another Service
Another Target
Another Account
Credential Discovery
Privilege Assessment
```

---

# 21. Credential Reuse

Credential reuse may create additional movement opportunities.

Example:

```text id="j4n8p2"
CORP\alice
    │
    ├── FILE01 → SMB → Valid
    ├── APP01 → Web → Unknown
    └── ADMIN01 → WinRM → Not Tested
```

Do not blindly test every system.

Use the evidence already collected to identify plausible targets.

---

# 22. Credential Reassessment

After validating a credential against one target, update the map.

Example:

```text id="v8q2m5"
Before:

CORP\alice
   ↓
Potential SMB Access

After:

CORP\alice
   ↓
SMB → FILE01
   ↓
Authentication SUCCESS
   ↓
Authorization LIMITED
```

This is stronger evidence.

It may change which targets should be investigated next.

---

# 23. Credential-to-Target Matrix

Maintain a matrix such as:

| Identity   | Credential Type | Target  | Service | Authentication | Authorization | Status    |
| ---------- | --------------- | ------- | ------- | -------------- | ------------- | --------- |
| CORP\alice | Password        | FILE01  | SMB     | Success        | Limited       | Validated |
| CORP\alice | Password        | ADMIN01 | WinRM   | Unknown        | Unknown       | Candidate |
| backup     | SSH Key         | LINUX02 | SSH     | Success        | User          | Validated |
| svc-web    | Password        | APP01   | HTTPS   | Unknown        | Unknown       | Candidate |

This creates a practical movement map.

---

# 24. Credential Discovery Decision Tree

```text id="g5m9r2"
Credential Material Found
          │
          ▼
Identify Identity
          │
          ▼
Identify Account Authority
          │
          ▼
Identify Credential Type
          │
          ▼
Determine Intended Use
          │
          ▼
Identify Compatible Services
          │
          ▼
Identify Candidate Targets
          │
          ▼
Is Access Plausible?
       │        │
      No       Yes
       │        │
       ▼        ▼
Gather More   Validate
Context        Access
                  │
                  ▼
             Authentication
                  │
             ┌────┴────┐
             ▼         ▼
          Success    Failure
             │         │
             ▼         ▼
       Check Access  Classify
       / Continue    Failure
```

---

# 25. Common Mistakes

## Mistake 1: Treating a Username as a Credential

```text id="k7v2p9"
Username
   ≠
Password
```

A username identifies an account but does not authenticate by itself.

---

## Mistake 2: Treating a Credential as Universal

A credential may only work:

```text id="f3m8q1"
On One Host
For One Service
For One Account Context
For One Authentication Mechanism
```

Validate before making broader assumptions.

---

## Mistake 3: Ignoring Account Authority

Always distinguish:

```text id="r9c4x6"
LOCAL\user
DOMAIN\user
```

---

## Mistake 4: Ignoring Intended Use

A database credential may not provide SSH, SMB, RDP, or WinRM access.

Identify its intended destination first.

---

## Mistake 5: Confusing Authentication With Authorization

```text id="n6p2w8"
Login Successful
    ≠
Required Privilege Granted
```

---

## Mistake 6: Testing Everywhere

Use target and service evidence to select specific validation attempts.

---

# 26. Credential Record Template

Use this format for important findings:

```text id="y3k7m2"
Identity:
____________________

Authority:
____________________

Credential Type:
____________________

Source:
____________________

Host / Application:
____________________

Intended Service:
____________________

Potential Target:
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

# 27. Completion Criteria

Credential identification for the current position is sufficiently complete when you can answer:

```text id="p8r4v1"
Who does the credential belong to?
What authority owns the account?
What type of authentication material is available?
Where did it come from?
What service was it intended for?
Which discovered targets may accept it?
Has authentication been validated?
Has authorization been validated?
What happened when validation failed?
What should happen next?
```

The objective is not:

```text id="m2x9c5"
Find as many credentials as possible.
```

The objective is:

```text id="q7k3n8"
Understand the authentication material available
and map it to realistic movement paths.
```

---

# 28. What Next?

You now have the general credential model.

The next question is:

> **"Are there passwords or password-like secrets available that can be mapped to specific identities and services?"**

Continue with:

```text id="w5n1r6"
05-Credential-and-Access-Discovery/
└── Passwords.md
```

The workflow becomes:

```text id="x8q4m2"
Credential Identification
        ↓
Password Discovery
        ↓
Identify Account
        ↓
Identify Intended Service
        ↓
Identify Candidate Targets
        ↓
Validate Authentication
        ↓
Validate Authorization
        ↓
Update Movement Map
```

The core principle is:

> **A credential becomes useful for lateral movement only when its identity, authority, authentication mechanism, compatible service, target, and authorization context are understood. Treat every credential as a relationship to be validated, not as a standalone secret.**
