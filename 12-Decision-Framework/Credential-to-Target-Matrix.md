# Credential-to-Target Matrix

This matrix helps connect authentication material to potential lateral-movement targets.

The purpose is not to assume that possession of a credential means access.

Instead, use the following chain:

```text
Credential / Access Material
          ↓
Identity / Account Context
          ↓
Authentication Protocol
          ↓
Compatible Service
          ↓
Target
          ↓
Authentication
          ↓
Authorization
          ↓
Effective Access
```

A credential becomes a movement opportunity only after the relevant authentication and authorization conditions are validated.

## How to Use This Matrix

Start with material discovered on the current host.

Ask:

1. Who does it belong to?
2. Is it local, domain, service, application, or another identity?
3. What authentication protocol can use it?
4. Which services accept that authentication?
5. Which targets expose those services?
6. Can the current position reach those targets?
7. Is the target in scope?
8. What authorization might the identity have?
9. What access can be safely validated?

## Quick Matrix

| Authentication Material         | Identity Context                     | Common Authentication Context      | Potential Services                           | Potential Target Types              |
| ------------------------------- | ------------------------------------ | ---------------------------------- | -------------------------------------------- | ----------------------------------- |
| Username + password             | Local account                        | Local authentication               | SMB, RDP, WinRM, SSH where configured        | Windows/Linux host                  |
| Username + password             | Domain account                       | Domain authentication              | SMB, WinRM, RDP, SSH where configured        | Domain workstation/server           |
| NTLM hash                       | Windows local/domain account         | NTLM                               | SMB, RPC/WMI and other NTLM-capable services | Windows hosts                       |
| Kerberos TGT                    | Domain principal                     | Kerberos                           | Kerberos-aware services                      | Domain resources                    |
| Kerberos service ticket         | Domain principal                     | Kerberos                           | Specific SPN-backed service                  | Matching service/host               |
| SSH private key                 | Linux/Unix or configured SSH account | Public-key authentication          | SSH                                          | Linux/Unix or SSH-enabled host      |
| SSH agent identity              | SSH account context                  | Public-key authentication          | SSH                                          | SSH-enabled host                    |
| Existing authenticated session  | Existing account/session             | Already-established authentication | Depends on session/service                   | Existing service/resource           |
| Application credential          | Application/service identity         | Application-specific               | Application/API/database/service             | Application infrastructure          |
| Service account credential      | Service identity                     | Depends on account configuration   | SMB, WinRM, SSH, application, database, etc. | Systems where account is authorized |
| Configuration-stored credential | Depends on configuration             | Depends on protocol                | Depends on credential use                    | Associated infrastructure           |

The matrix is a starting point, not proof of access.

## 1. Username and Password

### Local Account

```text
Local Username + Password
          ↓
Local Account Context
          ↓
Compatible Remote Service
          ↓
Target Windows Host
```

Potential services may include:

* SMB
* RDP
* WinRM
* other configured remote services

The important question is whether the account is recognized and authorized on the target.

### Domain Account

```text
Domain Username + Password
          ↓
Domain Identity
          ↓
Domain Authentication
          ↓
Compatible Service
          ↓
Domain-Joined Target
```

Potential services include:

* SMB
* WinRM
* RDP
* other domain-integrated services

Do not assume that domain membership gives administrative access.

### Decision

```text
Password
  ↓
Which Account?
  ↓
Local or Domain?
  ↓
Which Target?
  ↓
Which Service?
  ↓
Authentication?
  ↓
Authorization?
```

## 2. NTLM Hash

An NTLM hash may represent authentication material associated with a Windows account.

Reason about it as:

```text
NTLM Hash
    ↓
Account
    ↓
NTLM-Compatible Authentication
    ↓
Compatible Service
    ↓
Target
```

Potential services may include:

* SMB
* RPC-related Windows management workflows
* WMI workflows where the authentication configuration permits NTLM

The important questions are:

```text
Who owns the hash?
Is the account local or domain-based?
Does the target support the required authentication?
Is the account authorized on the target?
```

A hash does not automatically provide administrative access.

## 3. Kerberos TGT

A Ticket Granting Ticket represents a Kerberos authentication context for a principal.

Reason about it as:

```text
TGT
 ↓
Kerberos Principal
 ↓
Service Ticket Request
 ↓
Target Service / SPN
 ↓
Authentication
 ↓
Authorization
```

Useful contextual information includes:

* principal
* realm/domain
* ticket validity
* domain controller
* target service
* SPN
* DNS
* time synchronization

A TGT is not itself equivalent to access to every domain resource.

## 4. Kerberos Service Ticket

A service ticket is associated with a specific Kerberos service.

Reason about it as:

```text
Service Ticket
      ↓
Service Principal
      ↓
SPN
      ↓
Matching Service
      ↓
Target
```

For example:

```text
Kerberos Context
      ↓
cifs/server.example.local
      ↓
SMB Service
      ↓
SERVER
```

The ticket's service context matters.

Do not treat a service ticket as a universal credential.

## 5. SSH Private Key

An SSH private key may provide public-key authentication for a particular account.

Reason about it as:

```text
Private Key
     ↓
Public Key / Account Association
     ↓
SSH
     ↓
Target
```

Before using it, establish:

* associated username
* target host
* SSH port
* key permissions
* whether the target is in scope
* whether the account is authorized

Useful discovery sources include:

```bash id="1f7n3q"
ls -la ~/.ssh
cat ~/.ssh/config
cat ~/.ssh/known_hosts
ssh-add -L
```

Do not assume that a private key works on every SSH server.

## 6. SSH Agent Identity

An SSH agent can contain identities without exposing the private key directly.

The workflow is:

```text
SSH Agent
    ↓
Available Identity
    ↓
Known / Candidate SSH Target
    ↓
SSH Authentication
    ↓
Account Authorization
```

Useful validation:

```bash id="2f0t4r"
ssh-add -L
```

If an identity is available, determine its relationship to an authorized target before attempting authentication.

## 7. Existing Sessions

An existing session can provide useful authentication or access context.

Examples include:

* active SSH sessions
* Windows logon sessions
* authenticated SMB connections
* application sessions
* domain authentication context

Reason about it as:

```text
Existing Session
      ↓
Identity
      ↓
Authentication Context
      ↓
Accessible Resources
      ↓
Potential Target
```

A session is not automatically reusable authentication material.

First determine what access the session actually provides.

## 8. Application Credentials

Application credentials can include:

* API credentials
* database credentials
* service credentials
* application usernames/passwords
* configuration-stored secrets

The workflow is:

```text
Application Credential
        ↓
Application / Service
        ↓
Identity
        ↓
Accessible Resource
        ↓
Potential Infrastructure Relationship
```

An application credential should not automatically be tested against unrelated infrastructure.

First determine its intended context.

## 9. Service Account Credentials

Service accounts can have access across multiple systems depending on their configuration.

Use:

```text
Service Account
      ↓
Known Role
      ↓
Known Services
      ↓
Known Targets
      ↓
Authorized Access
```

Useful contextual questions:

* Is the account local or domain-based?
* What service uses it?
* Where is the service deployed?
* Which systems does it interact with?
* What remote services accept the account?
* What permissions does it have?

Avoid assuming that a service account is privileged merely because it is used by an important service.

## 10. Configuration-Stored Credentials

Credentials may appear in:

* application configuration
* deployment files
* scripts
* service definitions
* connection strings
* environment variables
* automation configuration

The workflow is:

```text
Configuration
      ↓
Credential
      ↓
Owner / Application
      ↓
Intended Service
      ↓
Associated Target
```

The associated application or service is usually the first context to investigate.

## 11. Credential Context Matrix

| Material                | First Question          | Next Question                              |
| ----------------------- | ----------------------- | ------------------------------------------ |
| Password                | Who owns it?            | Where is the account valid?                |
| NTLM hash               | Which account?          | Which NTLM-compatible service?             |
| Kerberos TGT            | Which principal?        | Which services can the principal access?   |
| Kerberos service ticket | Which SPN?              | Which target provides that service?        |
| SSH key                 | Which account?          | Which SSH target accepts it?               |
| SSH agent identity      | Which identity?         | Which authorized SSH target?               |
| Existing session        | Which user?             | What access does the session already have? |
| Application credential  | What application?       | What resource does it access?              |
| Service account         | What service uses it?   | Which systems depend on that account?      |
| Configuration secret    | What component uses it? | What infrastructure is associated with it? |

## 12. Target Selection Matrix

| Target Evidence          | Authentication Material     | Next Investigation                            |
| ------------------------ | --------------------------- | --------------------------------------------- |
| SMB on Windows host      | Domain account              | Validate SMB authentication and authorization |
| WinRM on Windows host    | Appropriate Windows account | Validate remote management authorization      |
| RDP on Windows host      | Account permitted for RDP   | Validate interactive logon authorization      |
| SSH on Linux host        | SSH key/password            | Validate SSH authentication                   |
| Kerberos-enabled service | Kerberos context            | Identify SPN and target service               |
| Internal-only service    | Pivot access                | Establish authorized pivot path               |
| Application endpoint     | Application credential      | Validate application-level authorization      |
| Database service         | Database credential         | Validate database authentication and role     |

## 13. Authentication vs Authorization

Always separate:

### Authentication

> Can this identity prove who it is?

### Authorization

> What is this identity allowed to do?

Example:

```text
Valid Password
     ↓
Authentication Success
     ↓
SMB Session
     ↓
Share Access?
     ↓
Read?
     ↓
Write?
     ↓
Administrative Access?
```

Each step can produce a different result.

## 14. Credential-to-Service Decision

Use this sequence:

```text
I found authentication material.
        ↓
Who owns it?
        ↓
What authentication protocol does it support?
        ↓
Which services accept that protocol?
        ↓
Which targets expose those services?
        ↓
Can I reach those targets?
        ↓
Is the target in scope?
        ↓
Can I authenticate?
        ↓
What am I authorized to do?
```

## 15. Credential-to-Target Ledger

For practical assessments, maintain a ledger:

| Credential / Material | Identity | Protocol | Target | Service | Reachable | Authenticated | Authorized | Result |
| --------------------- | -------- | -------- | ------ | ------- | --------- | ------------- | ---------- | ------ |
|                       |          |          |        |         |           |               |            |        |

This prevents repeatedly testing the same combination and makes the movement chain easier to reconstruct.

## 16. When the Credential Does Not Work

Do not immediately conclude that the credential is invalid.

Check:

```text
Credential
   ↓
Correct Identity?
   ↓
Correct Authentication Protocol?
   ↓
Correct Target?
   ↓
Correct Service?
   ↓
Correct Account Context?
   ↓
Account Allowed?
```

Possible explanations include:

* wrong username
* wrong target
* unsupported authentication method
* expired credential
* disabled account
* incorrect domain/realm
* Kerberos DNS issue
* Kerberos time issue
* service-specific authorization
* network restriction

## 17. Avoid Broad Credential Testing

Use targeted validation.

Prefer:

```text
Known Credential
+
Known Identity
+
Known Target
+
Known Service
```

Avoid:

```text
Credential
+
Many Unrelated Targets
+
Many Unrelated Services
```

This reduces unnecessary authentication attempts and keeps the assessment evidence-driven.

## 18. Credential Lifecycle After Movement

After reaching a new host:

```text
New Host
   ↓
Credential Discovery
   ↓
Credential Classification
   ↓
Identity Mapping
   ↓
Service Mapping
   ↓
Target Mapping
   ↓
Authorization Validation
   ↓
Potential Movement
```

This creates the recurring lateral-movement loop.

## 19. Important Distinctions

Keep these distinctions explicit:

```text
Password
   ≠
Hash
   ≠
Kerberos Ticket
   ≠
SSH Key
   ≠
Session
   ≠
Authorization
   ≠
Administrative Access
```

Similarly:

```text
Open Port
   ≠
Usable Service
   ≠
Successful Authentication
   ≠
Authorized Access
   ≠
Interactive Shell
   ≠
Administrative Access
```

These distinctions prevent incorrect conclusions during an assessment.

## 20. Practical Example

Suppose a Linux host contains an SSH private key.

Do not immediately assume:

```text
Key → Any Server
```

Instead:

```text
SSH Key
   ↓
Identify Associated Account
   ↓
Inspect SSH Configuration / Known Hosts
   ↓
Identify Candidate Target
   ↓
Confirm Target Reachability
   ↓
Confirm SSH Service
   ↓
Attempt Authorized Authentication
   ↓
Validate User
   ↓
Validate Privilege
   ↓
Re-enumerate
```

The key becomes useful only when the surrounding context supports the movement path.

## 21. Decision Summary

When authentication material is discovered, use:

```text
WHAT IS IT?
     ↓
WHO DOES IT BELONG TO?
     ↓
WHAT PROTOCOL DOES IT USE?
     ↓
WHAT SERVICE ACCEPTS IT?
     ↓
WHAT TARGET EXPOSES THAT SERVICE?
     ↓
CAN I REACH THE TARGET?
     ↓
CAN I AUTHENTICATE?
     ↓
WHAT AM I AUTHORIZED TO DO?
     ↓
WHAT ACCESS DID I GAIN?
     ↓
WHAT DID THE NEW POSITION REVEAL?
```

## What Next?

Continue to [`Service-to-Technique-Matrix.md`](Service-to-Technique-Matrix.md).

That file maps discovered remote services to the corresponding lateral-movement workflow and validation path.
