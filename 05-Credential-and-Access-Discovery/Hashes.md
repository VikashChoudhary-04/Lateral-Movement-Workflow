# Hashes

Password hashes are authentication-related material produced from passwords or other authentication secrets.

In lateral movement, a hash can be relevant because some authentication protocols can use hash-derived material without requiring the original plaintext password.

The objective of this file is not to teach indiscriminate password cracking.

The objective is to answer:

> **What hash material exists, which identity does it represent, which authentication mechanisms can use it, and which authorized targets may be relevant?**

The workflow is:

```text id="q8m3v6"
Hash Material Found
      ↓
Identify Hash Type
      ↓
Identify Account
      ↓
Identify Account Authority
      ↓
Determine Authentication Mechanism
      ↓
Identify Compatible Services
      ↓
Identify Candidate Targets
      ↓
Validate Authorized Authentication
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

```text id="h5r2n8"
Credentials
      ↓
Passwords
      ↓
Hashes
      ↓
Kerberos
      ↓
Tokens and Sessions
      ↓
SSH Keys
      ↓
Existing Access
```

Hashes are therefore one branch of the broader credential workflow.

The important relationship is:

```text id="m7p4x1"
Hash
 ↓
Identity
 ↓
Authentication Mechanism
 ↓
Compatible Service
 ↓
Target
 ↓
Validated Access
```

---

# 2. Hash vs Password

A password hash is not the plaintext password.

For example:

```text id="x3k8q5"
Password:
CorrectHorseBattery...

Hash:
<hash value>
```

The hash may or may not be usable directly for authentication.

This depends on:

```text id="v9m2r6"
Hash Type
Authentication Protocol
Account Type
Target Service
Protocol Configuration
```

Therefore:

```text id="j4n7c2"
Hash Found
    ≠
Plaintext Password Found
```

---

# 3. Why Hashes Matter to Lateral Movement

Some Windows authentication scenarios can use NTLM-related hash material.

Conceptually:

```text id="p6w3k9"
Identity
   +
Authentication Material
   ↓
Authentication Protocol
   ↓
Remote Service
   ↓
Target
```

This can create a movement path without first recovering the plaintext password.

The specific credential-based movement technique is covered later in:

```text id="b8r5m1"
08-Credential-Based-Movement/Pass-the-Hash.md
```

This file focuses on:

```text id="y2q7v4"
Discovery
Identification
Context
Compatibility
Validation
```

---

# 4. Identify the Hash Type

Before deciding what a hash means, identify what kind of hash material you have.

Possible examples include:

```text id="f6m1x8"
NTLM-related Hash
Password Database Hash
Application Hash
Cryptographic Digest
API / Token Digest
```

Not every hash is useful for authentication.

For example:

```text id="r3v9k5"
Web Application Password Hash
```

may be intended only for application-side password verification.

Do not assume that every hash can be used for remote authentication.

---

# 5. Identify the Account

A hash without an associated identity is incomplete information.

Record:

```text id="n8p4c2"
Username:
Account Authority:
Host:
Hash Type:
Source:
```

Example:

```text id="w5k7m1"
Username:
alice

Authority:
CORP.LOCAL

Hash Type:
NTLM-related

Source:
Authorized assessment artifact
```

Now the authentication material has context.

---

# 6. Local vs Domain Hashes

In Windows environments, determine whether the account is:

```text id="q2m8x4"
Local
```

or:

```text id="c7r3n9"
Domain
```

For example:

```text id="f1v6k5"
WORKSTATION01\alice
```

versus:

```text id="j9p2m7"
CORP\alice
```

The authentication scope and potential movement paths can differ.

Never assume the same username represents the same account across systems.

---

# 7. Hash Source Matters

Hash material can originate from different locations.

Examples include:

```text id="a4n8q6"
Local Credential Database
Domain Credential Material
Authorized Memory Analysis
Backup
Configuration
Application Data
Assessment Artifact
```

The source can help determine:

```text id="z7m3p1"
Which account?
Which host?
Which authentication mechanism?
Which target?
```

Always preserve the source in your notes.

---

# 8. Windows Account Context

For a Windows identity, first establish the account context.

Useful commands include:

```cmd id="m5q8x2"
whoami
whoami /user
whoami /groups
```

If relevant:

```cmd id="r3n7c6"
echo %USERDOMAIN%
```

The objective is to distinguish:

```text id="t6p1v9"
Current Identity
Account Authority
Group Context
```

from the hash material itself.

---

# 9. NTLM-Related Hashes

In Windows environments, NTLM authentication may involve an NT hash.

The important movement concept is:

```text id="g8x2m5"
NTLM-Compatible Authentication Material
          ↓
NTLM Authentication
          ↓
Compatible Remote Service
```

Potential services may include Windows remote-access protocols where NTLM authentication is supported.

However, the exact authentication behavior depends on configuration and policy.

Do not assume that an NTLM-related hash works against every Windows service.

---

# 10. Hashes and SMB

SMB can use Windows authentication mechanisms including NTLM in applicable configurations.

Therefore:

```text id="k4m9r2"
Hash
 ↓
Windows Identity
 ↓
SMB
 ↓
Target
```

may form a legitimate movement hypothesis.

Before attempting access, establish:

```text id="v6q1n8"
Target Reachability
SMB Availability
Account Context
Authentication Compatibility
Assessment Authorization
```

Detailed SMB movement belongs in:

```text id="y3c7p5"
06-Remote-Access-Services/SMB.md
```

---

# 11. Hashes and Other Windows Services

Hash-based authentication may be relevant to other Windows remote-access protocols depending on configuration.

Examples include:

```text id="w8n3k6"
SMB
WinRM
RPC / WMI-related authentication
Other NTLM-capable services
```

Do not assume compatibility solely because the target is Windows.

The workflow remains:

```text id="q5m2r8"
Hash
 ↓
Authentication Mechanism
 ↓
Service
 ↓
Target
 ↓
Validation
```

---

# 12. Pass-the-Hash Is Not Hash Cracking

These are different workflows.

### Hash Cracking

```text id="p8r4x2"
Hash
 ↓
Attempt to Recover Password
 ↓
Plaintext Password
```

### Pass-the-Hash

```text id="n6k1v9"
NTLM-Related Hash
 ↓
Compatible Authentication
 ↓
Remote Service
 ↓
Target
```

The second workflow does not require recovering the plaintext password first.

Detailed Pass-the-Hash movement is covered in:

```text id="c3m7q5"
08-Credential-Based-Movement/Pass-the-Hash.md
```

---

# 13. Hashes Do Not Automatically Provide Access

Suppose:

```text id="r9x2k4"
CORP\alice
NTLM-related hash
```

and:

```text id="v7m3n8"
FILE01
SMB
```

This creates:

```text id="a5q6p1"
Potential Authentication Path
```

It does not prove:

```text id="j8c4m2"
SMB Access
```

Access must still be validated.

---

# 14. Authorization Still Matters

Even if authentication succeeds:

```text id="m2r7x5"
Authentication
      ↓
SUCCESS
      ↓
Authorization
      ↓
?
```

The account may have:

```text id="n4k8q1"
Read Access
Limited Share Access
Remote Logon
Administrative Access
```

The result must be recorded accurately.

---

# 15. Hash-to-Target Mapping

Use a matrix:

| Identity    | Hash Type    | Target   | Service | Authentication | Authorization | Status    |
| ----------- | ------------ | -------- | ------- | -------------- | ------------- | --------- |
| CORP\alice  | NTLM-related | FILE01   | SMB     | Unknown        | Unknown       | Candidate |
| CORP\alice  | NTLM-related | ADMIN01  | WinRM   | Unknown        | Unknown       | Candidate |
| LOCAL\admin | NTLM-related | SERVER01 | SMB     | Unknown        | Unknown       | Candidate |

This keeps the hash associated with its identity and possible movement path.

---

# 16. Credential Material Context

When a hash is found, record:

```text id="w3p9m6"
Identity:
____________________

Authority:
____________________

Hash Type:
____________________

Source:
____________________

Host:
____________________

Potential Authentication:
____________________

Compatible Services:
____________________

Candidate Targets:
____________________

Authentication Status:
____________________

Authorization Status:
____________________

Next Action:
____________________
```

Do not store sensitive material unnecessarily in public notes or repositories.

---

# 17. Local Account Hashes

Suppose you identify:

```text id="x6k2r8"
SERVER01\Administrator
```

and corresponding hash material.

Initially associate it with:

```text id="q4m7n1"
SERVER01
```

Do not assume the same local administrator credential works on:

```text id="p8v3c5"
SERVER02
```

unless the environment provides evidence of shared credentials.

Credential reuse must be validated.

---

# 18. Domain Account Hashes

For a domain identity:

```text id="m9r5x2"
CORP\alice
```

the relevant authentication context may extend across multiple domain-connected systems.

But:

```text id="v4k8q6"
Domain Identity
   ≠
Access Everywhere
```

The target must still support the authentication mechanism and the account must have appropriate authorization.

---

# 19. Hashes From Memory

Authorized memory analysis can sometimes expose authentication-related material.

If such material is discovered:

```text id="c2p7m4"
Identify Identity
      ↓
Identify Material
      ↓
Determine Validity
      ↓
Determine Authentication Compatibility
      ↓
Map to Targets
```

Do not assume that every value extracted from memory is currently valid.

Tickets, tokens, credentials, and other material may have expiration or contextual restrictions.

---

# 20. Hashes From Backups or Credential Stores

A backup may contain authentication material associated with an older system state.

Therefore, ask:

```text id="f8n3r5"
When was this material created?
Which host did it belong to?
Which account did it represent?
Is the account still active?
Is the credential still valid?
```

Historical credential material should not automatically be treated as current.

---

# 21. Hash Validation

When attempting authorized authentication using hash-derived material, record:

```text id="k7q2m9"
Target:
Service:
Identity:
Authentication Method:
Result:
Authorization:
Evidence:
```

Example:

```text id="z4x8c1"
Target:
FILE01

Service:
SMB

Identity:
CORP\alice

Authentication Method:
NTLM-related credential material

Result:
Authentication successful

Authorization:
Limited share access

Next Action:
Determine whether the access provides a useful movement path
```

---

# 22. Authentication Failure

If hash-based authentication fails:

```text id="m5p1r8"
Authentication Failed
```

consider:

```text id="q9k4x6"
Incorrect Identity
Incorrect Material
Expired / Changed Credential
Target Does Not Support Mechanism
Protocol Restrictions
Account Restrictions
Network / Service Failure
```

Do not immediately discard the hash.

Classify the failure.

---

# 23. Security Policy Matters

Modern Windows environments may restrict or disable certain authentication paths.

Therefore:

```text id="w2n7c5"
Hash Available
      ↓
Protocol Supports It?
      ↓
Policy Allows It?
      ↓
Service Accepts It?
      ↓
Account Authorized?
```

A technically valid credential may still fail because of security policy.

---

# 24. Do Not Confuse Hashes With Cracked Passwords

Keep the evidence precise.

Bad:

```text id="r6m2x9"
Password Found
```

when you actually have:

```text id="j8q4v1"
NTLM-related Hash Found
```

Better:

```text id="c3n7p5"
NTLM-related authentication material associated with CORP\alice was identified.
```

This distinction matters when deciding what authentication methods are available.

---

# 25. Common Mistakes

## Mistake 1: Assuming Every Hash Is an Authentication Credential

Some hashes are only used for verification or application-specific purposes.

Identify the hash type first.

---

## Mistake 2: Automatically Cracking Every Hash

Hash cracking is a separate activity.

Determine whether direct authentication analysis is more relevant to the movement objective.

---

## Mistake 3: Assuming NTLM Material Works Everywhere

Authentication compatibility depends on:

```text id="y7m3q1"
Service
Protocol
Configuration
Policy
Account
```

---

## Mistake 4: Ignoring Account Authority

Always determine:

```text id="p4x8n6"
Local vs Domain
```

---

## Mistake 5: Confusing Authentication With Authorization

A successful authentication does not prove administrative access.

---

## Mistake 6: Testing Every Target

Use the target and service evidence already collected.

---

# 26. Hash Decision Tree

```text id="n3k7r5"
Hash Material Found
       │
       ▼
Identify Hash Type
       │
       ▼
Identify Account
       │
       ▼
Identify Authority
       │
       ▼
Determine Authentication Mechanism
       │
       ▼
Identify Compatible Services
       │
       ▼
Identify Candidate Targets
       │
       ▼
Is Authentication Plausible?
      │          │
     No         Yes
      │          │
      ▼          ▼
Record /      Validate
Reassess       Access
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
           Level      Failure
             │         │
             └────┬────┘
                  ▼
             Update Map
```

---

# 27. Completion Criteria

Hash discovery is sufficiently complete for the current position when you can identify:

```text id="f5m9x2"
[ ] Hash material discovered
[ ] Hash type identified
[ ] Associated account identified
[ ] Account authority identified
[ ] Source documented
[ ] Authentication mechanism understood
[ ] Compatible services identified
[ ] Candidate targets identified
[ ] Authentication result recorded
[ ] Authorization result recorded
[ ] Failed attempts classified
[ ] Relevant security-policy considerations recorded
[ ] Next action identified
```

The objective is not:

```text id="w8q3k6"
Collect every hash.
```

It is:

```text id="m1r7p4"
Determine whether identified hash material creates
a realistic and authorized authentication path.
```

---

# 28. What Next?

The next credential type is particularly important in Active Directory environments:

> **"What Kerberos authentication material or ticket context already exists, and what services or identities does it relate to?"**

Continue with:

```text id="x6k2v9"
05-Credential-and-Access-Discovery/
└── Kerberos.md
```

The workflow becomes:

```text id="c8m4r1"
Credential Discovery
      ↓
Hash Discovery
      ↓
Kerberos Context
      ↓
Identify Ticket / Identity
      ↓
Identify Service / SPN Context
      ↓
Identify Candidate Target
      ↓
Validate Authentication
      ↓
Update Movement Map
```

The core principle is:

> **A hash is authentication-related material, not automatically a universal credential. Identify its type, owner, authentication mechanism, compatible services, and target before treating it as a lateral-movement opportunity.**
