# Credential Reassessment

After moving to a new host, credential discovery must be repeated from the new position.

The reason is simple:

> A new host may have a different identity, different services, different configuration, different sessions, and access to different authentication material.

Credential reassessment is not a search for "any password."

The objective is to determine:

* what authentication material exists
* who it belongs to
* what authentication protocol it supports
* where it may be valid
* whether the current position can legitimately use it
* what new targets it maps to
* whether it creates a meaningful next movement opportunity

The workflow is:

```text id="9p4m7x"
New Host
   ↓
Identify Current Identity
   ↓
Identify Credential Sources
   ↓
Identify Authentication Material
   ↓
Determine Owner / Context
   ↓
Determine Credential Type
   ↓
Map to Compatible Services
   ↓
Identify Candidate Targets
   ↓
Validate Authentication
   ↓
Validate Authorization
   ↓
Record Result
   ↓
Continue or Stop
```

## Objective

Reassess authentication material from the newly accessed host without assuming that every discovered value is a reusable credential.

Potential material includes:

```text
Passwords
Password Hashes
Kerberos Tickets
SSH Keys
Tokens
Session Information
Credential Files
Application Secrets
Service Credentials
Configuration Values
Existing Authenticated Access
```

Each item must be interpreted in context.

## 1. Start With the New Position

Before looking for credentials, establish the current context.

### Windows

```powershell id="h7k2p9"
hostname
whoami
whoami /user
whoami /groups
```

### Linux

```bash id="v3m8q1"
hostname
whoami
id
```

Record:

```text
Host:
User:
UID / SID:
Groups:
Domain:
Privilege Level:
```

The same credential material can have different implications depending on the current identity and host.

## 2. Identify the Host's Role

Credential sources depend heavily on the host's purpose.

Consider whether the host is:

```text id="q5n9c3"
Workstation
File Server
Application Server
Database Server
Development Host
Management Host
Jump Host
Domain-Joined Server
```

For example, an application server may contain service configuration, while a workstation may contain user-specific authentication material.

Do not search randomly.

First understand what the host is used for.

## 3. Reassess Configuration Files

Configuration files can contain authentication material used by applications or services.

Look for relevant configuration locations appropriate to the operating system and application.

Examples of file types:

```text
.env
.ini
.conf
.yaml
.yml
.json
.xml
.config
```

Search for contextual terms such as:

```text
username
user
password
passwd
secret
token
apikey
api_key
connection
credential
```

The presence of a matching keyword does not prove that the value is an active credential.

Validate the context.

## 4. Understand Credential Context

For every discovered value, determine:

```text id="n4x8m2"
What is it?
   ↓
Who owns it?
   ↓
What system uses it?
   ↓
What protocol uses it?
   ↓
Where could it authenticate?
   ↓
Is it still valid?
```

Example:

```text
Application Config
      ↓
Database Username
      ↓
Database Password
      ↓
Database Service
      ↓
Database Authorization
```

This is more useful than simply reporting:

```text
"Found password."
```

## 5. Reassess Password Material

If a plaintext password is discovered in an authorized assessment, determine:

* associated username
* local/domain/application context
* intended service
* target system
* whether it appears current
* whether reuse is indicated

Do not automatically attempt the password against every account or service.

Use targeted validation against known, in-scope targets.

Record:

```text
Credential Owner:
Credential Type:
Known Service:
Known Target:
Validation Result:
Authorization Result:
```

## 6. Reassess Password Hashes

Hashes should be treated as authentication-related material rather than automatically as passwords.

Determine:

```text
Hash Type:
Associated Account:
Local / Domain Context:
Authentication Protocol:
Known Compatible Service:
```

For example:

```text
NTLM Material
   ↓
Associated Windows Identity
   ↓
NTLM-Compatible Authentication
   ↓
Candidate Target
```

The existence of a hash does not prove that it can be used successfully.

## 7. Reassess Kerberos Material

On domain-joined Windows systems, inspect the current Kerberos context where authorized.

```cmd id="m8q3v6"
klist
```

Determine:

```text
Logon Session:
Principal:
Realm:
Ticket Type:
Service Principal:
Expiration:
```

Conceptually:

```text
Kerberos Material
      ↓
Principal / Identity
      ↓
Ticket
      ↓
Service / SPN
      ↓
Target
```

A ticket is not interchangeable with a plaintext password.

## 8. Reassess SSH Material

On Linux systems, inspect the SSH configuration and authorized access context.

Common locations include:

```text
~/.ssh/
```

Relevant files may include:

```text
config
known_hosts
authorized_keys
```

Private keys, where legitimately accessible within scope, should be handled as sensitive authentication material.

Determine:

```text
Key Owner:
Key Type:
Associated User:
Known Host:
Port:
Authentication Context:
```

The presence of a private key does not automatically establish that the corresponding account remains valid.

## 9. Reassess SSH Agent State

Where an SSH agent is in use, determine whether keys are loaded.

```bash id="r7m2k9"
ssh-add -L
```

If keys are available, determine:

```text
Key Identity:
Key Type:
Potential Associated Account:
Known Target:
```

Do not assume that every loaded key provides useful access.

The key must still map to an authorized target and account.

## 10. Reassess Environment Variables

Applications and services sometimes receive secrets through environment variables.

On Linux:

```bash id="p4v8n3"
env
```

or:

```bash id="c7m2x9"
printenv
```

On Windows PowerShell:

```powershell id="y3q8m5"
Get-ChildItem Env:
```

Look for application-specific configuration.

Treat discovered secrets as sensitive information.

Avoid exposing them unnecessarily in notes, screenshots, terminal recordings, or reports.

## 11. Reassess Application Credentials

If the new host runs an application, identify how that application authenticates to dependencies.

Possible relationships:

```text id="k8n3p5"
Application
   ↓
Database
```

```text id="m4x7q2"
Application
   ↓
API
```

```text id="v9c2r6"
Application
   ↓
LDAP / Active Directory
```

```text id="p6m8x3"
Application
   ↓
File Share
```

Identify:

```text
Application:
Dependency:
Authentication Method:
Credential Source:
Target:
```

The objective is to map credentials to infrastructure relationships.

## 12. Reassess Service Credentials

Services may operate under dedicated identities.

### Windows

Review relevant service configuration where authorized.

```powershell id="x5q9m2"
Get-CimInstance Win32_Service |
    Select-Object Name, StartName, State
```

This identifies service accounts without attempting to expose their passwords.

### Linux

Identify relevant service users through service configuration and process context.

```bash id="n7v3k8"
ps aux
```

Then determine:

```text
Service:
Service User:
Purpose:
Dependencies:
```

A service account name is not itself a credential.

## 13. Reassess Active Sessions

Active sessions can reveal which identities and services are currently in use.

### Windows

Depending on the host and permissions:

```cmd id="q8m4p1"
query user
```

### Linux

```bash id="z5n2v7"
who
```

```bash id="r3k8m6"
w
```

Also consider:

```bash id="c4x9p2"
ss -tunap
```

The objective is to identify session and communication context.

Do not equate an active session with access to the underlying user's credentials.

## 14. Reassess Existing Authentication Access

A new host may already have authenticated access to resources.

Examples:

```text
Mounted SMB Share
Authenticated SSH Agent
Kerberos Tickets
Application Session
Database Connection
Cloud / API Session
```

Ask:

```text id="m9q3v8"
What is already authenticated?
        ↓
Which identity is being used?
        ↓
What service is involved?
        ↓
What target is involved?
        ↓
What authorization exists?
```

Existing access can sometimes be more immediately useful than newly discovered credential material.

## 15. Distinguish Credentials From Sessions

This distinction is important.

### Credential

An authentication secret or reusable authentication material.

Examples:

```text
Password
Private Key
Hash
Authentication Token
```

### Session

An already established authenticated context.

Examples:

```text
SSH Session
Kerberos Logon Context
Web Session
Database Session
```

### Access

An authorization result.

Examples:

```text
Read Share
Remote Shell
Database Query
Application Administration
```

These are related but not equivalent:

```text
Credential
   ↓
Authentication
   ↓
Session
   ↓
Authorization
   ↓
Access
```

## 16. Determine Credential Scope

Every credential should be mapped to its likely scope.

| Scope       | Example                     |
| ----------- | --------------------------- |
| Local       | Local Windows/Linux account |
| Domain      | Active Directory identity   |
| Service     | Application/service account |
| Application | Web application identity    |
| Database    | DB-specific account         |
| SSH         | Key-based account access    |
| API         | API token                   |
| Kerberos    | Domain principal / ticket   |

The same username can exist in different authentication contexts.

For example:

```text
administrator
```

does not by itself tell you whether it is:

```text
Local Administrator
```

or:

```text
DOMAIN\Administrator
```

Always record the context.

## 17. Map Credential to Target

Use this workflow:

```text id="w6p2n9"
Credential
   ↓
Owner
   ↓
Authentication Type
   ↓
Compatible Service
   ↓
Known Target
   ↓
Reachability
   ↓
Authentication
   ↓
Authorization
```

Example:

```text
DOMAIN\analyst
      ↓
Kerberos
      ↓
SMB
      ↓
FILE-SERVER-01
      ↓
Reachable
      ↓
Authentication Successful
      ↓
Share Access Confirmed
```

This creates an evidence-based movement opportunity.

## 18. Validate Existing Access Before Searching Further

If the new host already has useful authenticated access, validate that access first.

For example:

```text id="a3m7q8"
Existing SMB Access
        ↓
Identify Shares
        ↓
Determine Permissions
        ↓
Identify Useful Resources
```

or:

```text id="p9x4c6"
Existing Kerberos Context
        ↓
Identify Relevant Services
        ↓
Determine Authorized Access
```

Do not unnecessarily search for additional credentials when an existing access path already answers the assessment objective.

## 19. Identify Credential Reuse Carefully

Credential reuse can be significant evidence.

For example:

```text id="t8q3m5"
Credential A
   ↓
Host 1
   ↓
Authentication Successful
   ↓
Host 2
   ↓
Authentication Successful
```

Record the reuse relationship.

Do not perform broad authentication attempts against unrelated systems.

Use known, authorized targets.

## 20. Validate Authentication and Authorization Separately

For each candidate:

```text id="r5m8v2"
Credential Valid?
      ↓
Service Accepts It?
      ↓
Authentication Successful?
      ↓
Requested Resource Authorized?
      ↓
Useful Access?
```

Example:

```text
Authentication ✓
Authorization ✗
```

means:

> The credential is valid for the service, but the identity does not have the required permission.

This is different from:

```text
Authentication ✗
```

## 21. Build a Credential-to-Target Map

Use a table such as:

| Credential / Identity | Type     | Scope       | Compatible Service | Candidate Target | Auth | Authz |
| --------------------- | -------- | ----------- | ------------------ | ---------------- | ---- | ----- |
| DOMAIN\user           | Account  | Domain      | SMB                | FILE-01          |      |       |
| Key A                 | SSH Key  | User        | SSH                | LINUX-01         |      |       |
| Ticket A              | Kerberos | Domain      | CIFS               | SERVER-01        |      |       |
| DB Account            | Database | Application | DB                 | DB-01            |      |       |

Fill only what has been established.

Do not mark a target as accessible merely because the credential appears compatible.

## 22. Credential Reassessment by Platform

### Windows

Focus on:

```text
Domain Context
Kerberos Tickets
Current Sessions
Application Configuration
Service Accounts
Environment Variables
Existing Network Access
```

### Linux

Focus on:

```text
SSH Configuration
SSH Keys
SSH Agent
Application Configuration
Environment Variables
Service Users
Existing Sessions
Network Connections
```

### Mixed / AD Environment

Focus on:

```text
Domain Identity
Kerberos Context
SMB Access
Windows Management Services
Application Dependencies
Service Accounts
Cross-Host Credential Reuse
```

## 23. Sensitive Data Handling

Credential material should be treated as sensitive evidence.

Do not unnecessarily place:

```text
Passwords
Private Keys
Tokens
Session Cookies
Hashes
API Secrets
```

directly into public notes or Git repositories.

For the workflow repository, document the **process and interpretation**, not real secrets.

Use placeholders:

```text
<USERNAME>
<PASSWORD>
<DOMAIN>
<TARGET>
<TOKEN>
<PRIVATE_KEY>
```

## 24. Failure Handling

### Credential Found but Authentication Fails

Possible causes:

* expired credential
* incorrect account context
* wrong authentication protocol
* disabled account
* service does not accept that credential type
* target is incorrect

Do not immediately assume the credential is invalid.

First verify the authentication context.

### Authentication Succeeds but Authorization Fails

The identity is valid but lacks the required permission.

Record:

```text
Authentication: Successful
Authorization: Denied
```

Then identify whether another authorized use exists.

### Credential Has No Known Target

Record it as unvalidated material.

Do not attempt broad guessing.

### Key Exists but SSH Fails

Check:

```text
Key Format
Username
Target Host
Target Port
SSH Configuration
Authorized-Key Relationship
```

### Kerberos Material Exists but Service Access Fails

Check:

```text
Principal
Realm
Service Principal
DNS
Time Synchronization
Target Service
Authorization
```

## 25. Decision Framework

### Credential discovered?

```text
Yes
 ↓
Identify Type
 ↓
Identify Owner
 ↓
Identify Scope
```

### Compatible service known?

```text
Yes → Map credential to that service.
No  → Record as unvalidated material.
```

### Candidate target known?

```text
Yes → Check reachability and scope.
No  → Continue controlled target discovery.
```

### Authentication succeeds?

```text
Yes → Validate authorization.
No  → Troubleshoot authentication context.
```

### Authorization succeeds?

```text
Yes → Validate useful access.
No  → Record the permission boundary.
```

### Useful new access obtained?

```text
Yes → Continue the movement chain.
No  → Reassess other evidence.
```

## 26. Credential Reassessment Checklist

```text id="u8r3m5"
[ ] Current identity confirmed
[ ] Host role identified
[ ] Configuration sources identified
[ ] Password material reviewed
[ ] Hash material reviewed
[ ] Kerberos context reviewed
[ ] SSH material reviewed
[ ] SSH agent reviewed
[ ] Environment variables reviewed
[ ] Application credentials considered
[ ] Service identities reviewed
[ ] Active sessions reviewed
[ ] Existing authenticated access reviewed
[ ] Credential scope identified
[ ] Credential-to-target mapping created
[ ] Authentication validated
[ ] Authorization validated
[ ] Credential reuse considered
[ ] Sensitive material handled safely
[ ] Next movement opportunity identified
```

## 27. Evidence Record

For each meaningful discovery, record:

```text id="p4m9x7"
Source Host:
Source User:
Credential / Access Type:
Credential Owner:
Authentication Context:
Compatible Service:
Candidate Target:
Target Reachability:
Authentication Result:
Authorization Result:
Effective Access:
Evidence:
Next Action:
```

Example:

```text id="q7v2n8"
Source Host: SERVER-01
Source User: DOMAIN\analyst
Credential Type: Existing Kerberos Context
Target: FILE-02
Service: SMB
Reachability: Confirmed
Authentication: Successful
Authorization: Read access to approved share
Effective Access: SMB Resource Access
Next Action: Validate share contents within scope
```

## 28. When to Stop Credential Reassessment

Stop when:

* relevant credential sources have been reviewed
* discovered credentials have been mapped
* known targets have been validated
* no additional meaningful authentication material is identified
* further searching would create unnecessary exposure
* the assessment objective has been reached
* the next useful action is target discovery or movement rather than more credential collection

Credential discovery should support the movement workflow, not become an objective by itself.

## Golden Rule

> Do not ask only, "What credentials are on this host?" Ask, "What authentication material does this host expose, who does it belong to, where can it legitimately authenticate, what authorization does it provide, and what new position would that access create?"

The useful result is not the secret itself.

The useful result is the validated relationship:

```text
Credential
   ↓
Identity
   ↓
Service
   ↓
Target
   ↓
Authentication
   ↓
Authorization
   ↓
Access
   ↓
New Position
```

## What Next?

Continue to [`Continue-the-Chain.md`](Continue-the-Chain.md).

That file turns the validated access, new-host enumeration, and credential reassessment into a controlled decision process for determining whether and how the lateral-movement chain should continue.
