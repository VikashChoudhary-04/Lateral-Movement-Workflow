# Passwords

Passwords are one form of authentication material that may enable access to another system or service.

The objective of password discovery is not to collect passwords indiscriminately.

The objective is to determine:

> **What password-based credentials are available, which identities do they belong to, where are they intended to work, and which authorized targets could potentially accept them?**

The workflow is:

```text id="u7m4q2"
Identify Password Source
      ↓
Identify Account
      ↓
Identify Account Authority
      ↓
Determine Intended Service
      ↓
Identify Candidate Targets
      ↓
Validate Authentication
      ↓
Validate Authorization
      ↓
Record Result
      ↓
Reassess Movement Paths
```

---

# 1. Where This Fits

The credential-discovery workflow is:

```text id="g3x8n5"
Credential Discovery
      ↓
Passwords
      ↓
Hashes
      ↓
Kerberos
      ↓
Tokens / Sessions
      ↓
SSH Keys
      ↓
Existing Access
```

Passwords are only one form of authentication material.

A password should always be associated with:

```text id="k5p2v8"
Identity
Account Authority
Intended Service
Potential Target
Authentication Result
Authorization Result
```

---

# 2. Password vs Credential

A password is authentication material.

A credential is the broader concept of authentication information associated with an identity.

For example:

```text id="m8r4c1"
Identity:
CORP\alice

Password:
********
```

Together, these form a usable credential.

But:

```text id="v6n2q9"
Password
   ≠
Access
```

Access still depends on:

```text id="w3f7m5"
Target
Service
Authentication
Authorization
Account Restrictions
```

---

# 3. Start With Account Context

Before evaluating a password, identify the account.

Example:

```text id="j4k8p2"
Username:
alice

Authority:
CORP.LOCAL

Account:
CORP\alice
```

or:

```text id="q7m3x5"
Username:
alice

Authority:
WORKSTATION01

Account:
WORKSTATION01\alice
```

These are different identities.

Always determine the authority before deciding where the password may apply.

---

# 4. Password Sources

Password material may appear in many legitimate assessment contexts.

Common sources include:

```text id="r5v1n8"
Configuration Files
Application Settings
Environment Variables
Scripts
Service Configuration
Deployment Files
Backup Configuration
Credential Stores
Documentation
User-Provided Assessment Data
```

The source provides important context.

For example:

```text id="c8m2q6"
Database Configuration
      ↓
Database Account
      ↓
Database Server
```

This does not automatically imply:

```text id="f9x4k7"
SSH / SMB / RDP Access
```

---

# 5. Configuration Files

Applications often require passwords to connect to other services.

Potential examples include:

```text id="n3p7w5"
Database Password
Service Account Password
SMTP Password
API Secret
Application Credential
Backup Credential
```

When one is discovered, record:

```text id="x6r1m9"
Source File:
Application:
Username:
Account Authority:
Destination:
Protocol:
Password Type:
Potential Remote Use:
```

The goal is to preserve context.

---

# 6. Application Credentials

Consider:

```text id="v4q8c2"
APP01
  │
  └── Application
         │
         └── DB01
```

The application may use:

```text id="j7m3p9"
dbuser
password
```

The first question is:

> **What is this password intended to authenticate to?**

If the answer is:

```text id="b5x2k8"
DB01
```

then the immediate movement hypothesis is:

```text id="a9r4n6"
APP01
   ↓
Database Credential
   ↓
DB01
```

Whether that database access constitutes lateral movement depends on the environment and what access the database provides.

---

# 7. Environment Variables

Applications may receive passwords through environment variables.

For example:

```text id="y2m8q4"
DB_USER
DB_PASSWORD
SERVICE_USER
SERVICE_PASSWORD
```

If such a value is discovered, determine:

```text id="k6p3v1"
Which process uses it?
Which account does it represent?
What service does it authenticate to?
What destination does the service use?
```

Do not assume the password works against unrelated services.

---

# 8. Service Configuration

Services may run under specific identities.

Windows PowerShell:

```powershell id="m5r7x2"
Get-CimInstance Win32_Service |
    Select-Object Name, StartName, State
```

This can reveal service account context.

Linux:

```bash id="q8n4c6"
systemctl list-units --type=service
```

and:

```bash id="p2v9k5"
ps aux
```

can help identify services and their running identities.

The objective is to establish:

```text id="f7m3x8"
Service
   ↓
Identity
   ↓
Configuration
   ↓
Remote Dependency
```

---

# 9. Scripts and Automation

Automation frequently requires credentials.

Potential examples:

```text id="w1c6r9"
Backup Scripts
Deployment Scripts
Maintenance Scripts
Monitoring Scripts
Scheduled Tasks
Administration Scripts
```

When a password is found in a script, determine:

```text id="g4q8m2"
Who executes the script?
What account does the credential belong to?
What destination does the script connect to?
Which protocol is used?
```

A password in a script is not automatically a remote-login credential.

---

# 10. Password Context

A password should be recorded with its context.

Bad record:

```text id="x3n7p5"
Password:
********
```

Useful record:

```text id="m8k2v6"
Account:
CORP\backup

Password:
[redacted in notes]

Source:
Backup configuration

Destination:
BACKUP01

Protocol:
SMB

Potential Use:
Remote backup administration

Status:
Not validated
```

The second record is actionable because the identity and intended use are known.

---

# 11. Local Passwords

A local password belongs to a local account.

Example:

```text id="q5r9c3"
SERVER01\backup
```

The password should initially be associated with:

```text id="j6m2v8"
SERVER01
```

Do not assume it works on:

```text id="w4p7n1"
SERVER02
```

or:

```text id="f8x3k5"
DOMAIN\backup
```

unless evidence supports that relationship.

---

# 12. Domain Passwords

A domain password belongs to a domain identity.

Example:

```text id="t2k8m4"
CORP\alice
```

Potential services may include:

```text id="v7n3q6"
SMB
RDP
WinRM
Kerberos-Compatible Services
```

But service authorization remains separate.

The correct model is:

```text id="c4m9x1"
Domain Password
      ↓
Domain Identity
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

# 13. Service Account Passwords

Service accounts may have passwords used by applications or Windows services.

Example:

```text id="p8q3r7"
CORP\websvc
```

Potential use:

```text id="y5m1k9"
Web Application
      ↓
Database
```

The account may also have permissions elsewhere, but that must be determined from evidence.

Do not assume:

```text id="d7x4n2"
Service Account
   =
Administrator
```

---

# 14. Password Reuse

Password reuse can create movement opportunities.

For example:

```text id="s2k7m5"
CORP\alice
Password X
```

may authenticate successfully to more than one authorized system.

However, do not treat reuse as guaranteed.

The correct workflow is:

```text id="n8q4v1"
Known Password
      ↓
Known Account
      ↓
Relevant Candidate Target
      ↓
Compatible Service
      ↓
Specific Validation
      ↓
Record Result
```

---

# 15. Avoid Blind Password Testing

Do not turn password discovery into indiscriminate authentication attempts.

Instead:

```text id="r6m3x8"
Password Found
      ↓
Identify Account
      ↓
Identify Intended Use
      ↓
Identify Compatible Target
      ↓
Validate
```

This keeps the assessment controlled and evidence-driven.

---

# 16. Authentication Result

When validating a password, classify the result.

Possible outcomes include:

```text id="k9p2w5"
Authentication Successful
Authentication Failed
Account Locked / Restricted
Authentication Method Unsupported
Service Unavailable
Network Unreachable
```

Record exactly what happened.

---

# 17. Authentication Success

Suppose:

```text id="c3v7m1"
CORP\alice
      ↓
Password
      ↓
SMB
      ↓
FILE01
```

and authentication succeeds.

You now know:

```text id="q8n4r6"
The credential can authenticate to the SMB service on FILE01.
```

You still need to determine:

```text id="x5m2k9"
What authorization was granted?
What resources are accessible?
What useful position does this provide?
```

---

# 18. Authorization After Authentication

For example:

```text id="f7r3c8"
Authentication:
SUCCESS

Authorization:
Read-only
```

This is different from:

```text id="j2n6v4"
Authentication:
SUCCESS

Authorization:
Administrative
```

Therefore:

```text id="p9m5x1"
Authentication Success
       ↓
Determine Access Level
       ↓
Determine Movement Value
```

---

# 19. Authentication Failure

Suppose:

```text id="w4q8m2"
CORP\alice
      ↓
RDP → ADMIN01
      ↓
Authentication Failed
```

Possible explanations include:

```text id="n7k3v5"
Wrong Password
Wrong Account Authority
Remote Logon Restricted
RDP Not Permitted
Account Disabled
Credential Expired
Target Policy
```

Do not immediately conclude that the password is invalid everywhere.

Record:

```text id="x2m9r6"
Credential:
CORP\alice

Target:
ADMIN01

Service:
RDP

Result:
Authentication failed

Known Cause:
Unknown
```

---

# 20. Password-to-Target Matrix

Use a matrix to maintain context.

| Account    | Password Source    | Target   | Service  | Authentication | Authorization | Status    |
| ---------- | ------------------ | -------- | -------- | -------------- | ------------- | --------- |
| CORP\alice | Application config | FILE01   | SMB      | Success        | Read          | Validated |
| CORP\alice | User context       | ADMIN01  | RDP      | Failed         | Unknown       | Failed    |
| backup     | Script             | BACKUP01 | SMB      | Unknown        | Unknown       | Candidate |
| dbsvc      | App config         | DB01     | Database | Unknown        | Unknown       | Candidate |

This makes password findings usable for movement planning.

---

# 21. Passwords and Service Compatibility

A password does not have a universal service.

For example:

```text id="a5k9q3"
Database Password
      ↓
Database Service
```

is a stronger hypothesis than:

```text id="r7m2x6"
Database Password
      ↓
SSH
```

unless there is additional evidence that the same identity is permitted to use SSH.

Always identify the intended authentication context first.

---

# 22. Passwords in Web Applications

Web applications may contain credentials used for:

```text id="g8c4n1"
Database
Internal API
SMTP
LDAP
Cloud Service
Background Service
```

The application may therefore provide information about other systems.

The workflow is:

```text id="m6p2v9"
Application Credential
      ↓
Identify Destination
      ↓
Identify Account
      ↓
Identify Protocol
      ↓
Determine Whether Destination Is a Movement Target
```

Not every application secret creates host-to-host movement.

---

# 23. Passwords and Databases

A database credential may provide access to another server or service.

However, distinguish:

```text id="k4r8x2"
Database Access
```

from:

```text id="w1m7q5"
Operating System Access
```

A successful database login does not automatically provide a shell on the database server.

Determine what the authenticated database account can actually do.

---

# 24. Passwords and Remote Administration

Some passwords may be directly relevant to remote administration.

Examples:

```text id="p3v9n6"
Windows Account
    ↓
RDP / WinRM / SMB

Linux Account
    ↓
SSH
```

This is where password discovery connects directly to:

```text id="q7m2x4"
06-Remote-Access-Services/
```

---

# 25. Password Discovery Failure Handling

If no passwords are discovered:

```text id="z5n8k1"
No Password Found
      │
      ▼
Check Existing Sessions
      │
      ▼
Check Other Credential Types
      │
      ├── Hashes
      ├── Kerberos
      ├── SSH Keys
      └── Tokens
      │
      ▼
Review Credential Sources
      │
      ▼
Continue Movement Assessment
```

A lack of passwords does not mean there is no movement path.

Other authentication material may exist.

---

# 26. Common Mistakes

## Mistake 1: Treating Any Secret as a Password

API keys, tokens, hashes, certificates, and tickets have different authentication semantics.

Classify the material first.

---

## Mistake 2: Ignoring Account Authority

Always distinguish:

```text id="m7c2x9"
LOCAL\user
DOMAIN\user
```

---

## Mistake 3: Treating Password Reuse as Guaranteed

Reuse must be validated.

---

## Mistake 4: Ignoring Intended Destination

A database password should initially be mapped to its database service.

---

## Mistake 5: Treating Authentication as Administrative Access

A successful login may provide limited authorization.

---

## Mistake 6: Testing Passwords Everywhere

Use evidence to select candidate targets.

---

# 27. Password Discovery Record

Use:

```text id="q4x8m2"
Account:
____________________

Authority:
____________________

Password Source:
____________________

Intended Application / Service:
____________________

Destination:
____________________

Potential Additional Targets:
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

# 28. Password Decision Tree

```text id="v6p1k9"
Password Found
      │
      ▼
Identify Account
      │
      ▼
Identify Authority
      │
      ▼
Identify Intended Service
      │
      ▼
Identify Destination
      │
      ▼
Is Remote Access Relevant?
      │        │
     No       Yes
      │        │
      ▼        ▼
Record       Identify
Context      Candidate Target
                   │
                   ▼
             Validate Access
                   │
             ┌─────┴─────┐
             ▼           ▼
          Success      Failure
             │           │
             ▼           ▼
       Check Access   Classify
          Level       Failure
             │           │
             └─────┬─────┘
                   ▼
             Update Map
```

---

# 29. Completion Criteria

Password discovery is sufficiently complete for the current position when you can identify:

```text id="n8r3m6"
[ ] Relevant password sources
[ ] Account identities
[ ] Account authorities
[ ] Intended services
[ ] Intended destinations
[ ] Candidate movement targets
[ ] Authentication results
[ ] Authorization results
[ ] Failed validation attempts
[ ] Password reuse evidence where applicable
[ ] Remaining unknowns
[ ] Next movement decision
```

The objective is not:

```text id="t4m7q2"
Find every password.
```

The objective is:

```text id="c9x2p5"
Find password-based authentication material
that can be meaningfully mapped to a validated movement path.
```

---

# 30. What Next?

Passwords are only one form of authentication material.

The next question is:

> **"Are there password hashes or other hash-based authentication material that could be relevant to movement?"**

Continue with:

```text id="r7k3m8"
05-Credential-and-Access-Discovery/
└── Hashes.md
```

The workflow becomes:

```text id="p2n6v9"
Credential Discovery
      ↓
Password Discovery
      ↓
Hash Discovery
      ↓
Identify Account
      ↓
Identify Authentication Mechanism
      ↓
Identify Compatible Service
      ↓
Identify Candidate Target
      ↓
Validate Authorized Access
```

The core principle is:

> **A discovered password is useful only when its identity, authority, intended service, target, and authorization context are understood. Map the password to a specific authentication path instead of treating it as a universally reusable secret.**
