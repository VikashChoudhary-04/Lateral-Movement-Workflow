# Kerberos

Kerberos is a network authentication protocol commonly used in Active Directory environments.

For lateral movement, the important question is not simply:

> "What is Kerberos?"

The practical question is:

> **What Kerberos authentication context exists from my current position, which identities and services does it relate to, and does it create a legitimate path to another system?**

The workflow is:

```text
Kerberos Context
      ↓
Identify Current Identity
      ↓
Identify Domain
      ↓
Identify Tickets
      ↓
Identify Service / SPN Context
      ↓
Identify Candidate Targets
      ↓
Determine Authentication Path
      ↓
Validate Authorized Access
      ↓
Record Result
      ↓
Reassess Movement Paths
```

---

## 1. Where This Fits

Kerberos is one branch of credential and authentication discovery:

```text
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

It is especially relevant when the current host is:

```text
Domain Joined
      or
Domain Integrated
      or
Using Kerberos Authentication
```

Kerberos analysis should remain connected to the movement objective.

Do not turn this file into a general Active Directory enumeration guide.

---

## 2. Kerberos in the Movement Workflow

A simplified movement path can look like:

```text
Current Host
      ↓
Domain Identity
      ↓
Kerberos Authentication Context
      ↓
Service
      ↓
Target Host
      ↓
Authentication
      ↓
Authorization
      ↓
Remote Access
```

The important relationship is:

```text
Identity
   +
Kerberos Authentication
   +
Service
   +
Target
   =
Potential Movement Path
```

The path must still be validated.

---

## 3. Minimum Kerberos Concepts

You only need a few concepts to work with Kerberos during lateral movement.

### Kerberos Realm / Domain

In Active Directory, the Kerberos realm normally corresponds to the AD domain namespace.

Example:

```text
CORP.LOCAL
```

### Principal

A principal identifies an account or service within the Kerberos environment.

Examples:

```text
alice@CORP.LOCAL
```

or a service principal such as:

```text
HTTP/web01.corp.local
```

### Ticket

A ticket is authentication material issued by the Kerberos infrastructure.

Common concepts include:

```text
TGT
Service Ticket
```

### TGT

A Ticket Granting Ticket is used to request service tickets.

Conceptually:

```text
Identity
   ↓
TGT
   ↓
Service Ticket
   ↓
Service
```

### Service Ticket

A service ticket is associated with a particular service.

Conceptually:

```text
User
 ↓
Service Ticket
 ↓
Service
 ↓
Target
```

---

## 4. Why Kerberos Matters for Lateral Movement

Kerberos can provide an authentication path without repeatedly entering a plaintext password.

For example:

```text
CORP\alice
      ↓
Kerberos Authentication Context
      ↓
CIFS/server01.corp.local
      ↓
SMB Service
```

This does not automatically mean:

```text
Administrator Access
```

or even:

```text
Remote Access
```

It only establishes a potential authentication relationship.

---

## 5. First Determine Whether Kerberos Is Relevant

Before investigating tickets, establish the environment.

Ask:

```text
Is this host domain joined?
Is the current user a domain account?
Is a Kerberos realm configured?
Is a domain controller reachable?
Are Kerberos tickets present?
Are Kerberos-authenticated services being used?
```

If the environment is not using Kerberos, move to another credential branch.

---

## 6. Windows: Identify Domain Context

Start with the current identity:

```cmd
whoami
```

Then:

```cmd
whoami /user
whoami /groups
```

Check the domain context:

```cmd
echo %USERDOMAIN%
echo %USERDNSDOMAIN%
```

You can also inspect the computer's domain relationship:

```powershell
Get-CimInstance Win32_ComputerSystem |
    Select-Object Name, Domain, PartOfDomain
```

The objective is to determine:

```text
Current User
      ↓
Account Authority
      ↓
Domain
      ↓
Kerberos Relevance
```

---

## 7. Windows: Inspect Kerberos Tickets

The standard Windows command for viewing the current Kerberos ticket cache is:

```cmd
klist
```

This can provide information such as:

```text
Logon Session
Cached Tickets
Client Identity
Service Principal
Ticket Times
```

The exact output depends on the current security context.

---

## 8. What to Look for in `klist`

Do not simply run:

```cmd
klist
```

and stop.

Interpret the result.

Look for:

```text
Client:
Which identity owns the ticket?

Service:
Which service is the ticket for?

Server:
Which system provides that service?

Validity:
Is the ticket currently valid?

Authentication Context:
Which logon session owns it?
```

A useful finding might look conceptually like:

```text
Identity:
CORP\alice

Service:
cifs/server01.corp.local

Target:
server01.corp.local
```

This creates a candidate movement relationship:

```text
alice
 ↓
Kerberos
 ↓
CIFS / SMB
 ↓
SERVER01
```

---

## 9. TGT vs Service Ticket

A TGT and a service ticket answer different questions.

### TGT

Conceptually:

```text
Who can request additional Kerberos service tickets?
```

### Service Ticket

Conceptually:

```text
Which service is this authentication context associated with?
```

For movement analysis:

```text
TGT
 ↓
Potential Service Access

Service Ticket
 ↓
Specific Service Relationship
```

Do not treat every ticket as equivalent.

---

## 10. Identify the Service

A Kerberos ticket may reference a service principal.

Examples include:

```text
CIFS
HTTP
HOST
LDAP
MSSQLSvc
WSMAN
```

The service name provides a clue about the target functionality.

For example:

```text
CIFS/server01.corp.local
```

suggests:

```text
SMB-related service
      ↓
SERVER01
```

Whereas:

```text
HTTP/web01.corp.local
```

suggests:

```text
Web service
      ↓
WEB01
```

The service principal is therefore useful for target discovery.

---

## 11. SPNs

A Service Principal Name (SPN) associates a service instance with an account.

Conceptually:

```text
Service
   ↓
SPN
   ↓
Service Account
   ↓
Host / Application
```

Examples:

```text
HTTP/web01.corp.local
MSSQLSvc/db01.corp.local
CIFS/files01.corp.local
```

During lateral movement analysis, an SPN can help answer:

```text
Which service exists?
Which host provides it?
Which account is associated with it?
```

---

## 12. SPN Is Not Automatically an Access Path

Finding:

```text
MSSQLSvc/db01.corp.local
```

does not prove:

```text
Database Access
```

It tells you that a Kerberos-aware service relationship exists.

You still need:

```text
Reachability
      ↓
Service Availability
      ↓
Authentication
      ↓
Authorization
```

---

## 13. Linux Kerberos Context

Linux systems can participate in Kerberos authentication through mechanisms such as:

```text
SSSD
realmd
Samba
Kerberos client configuration
Other domain-integration mechanisms
```

First determine whether the system is domain integrated.

Useful commands include:

```bash
realm list
```

and:

```bash
hostname -f
```

Depending on the environment:

```bash
realm discover
```

may provide information about a discoverable realm.

---

## 14. Linux: Identify the Current Identity

Start with:

```bash
whoami
```

Then:

```bash
id
```

If domain accounts are relevant:

```bash
id user@domain
```

The objective is to determine:

```text
Local Account
or
Domain Account
```

before interpreting Kerberos material.

---

## 15. Linux: Inspect Kerberos Tickets

Where a Kerberos client is configured, use:

```bash
klist
```

This may show:

```text
Default principal
Ticket cache
Valid starting time
Expiration time
Service principal
```

The default principal is especially useful.

For example:

```text
alice@CORP.LOCAL
```

indicates the identity associated with the current Kerberos cache.

---

## 16. Interpret Linux Ticket Output

Suppose `klist` shows a service such as:

```text
cifs/server01.corp.local@CORP.LOCAL
```

Interpret it as:

```text
Identity:
alice@CORP.LOCAL

Protocol:
Kerberos

Service:
CIFS

Target:
server01.corp.local
```

This creates a candidate movement path.

It does not prove authorization.

---

## 17. Ticket Lifetime Matters

Tickets are time-sensitive.

Record:

```text
Valid From
Expiration
Current Time
```

Conceptually:

```text
Ticket
  │
  ├── Not Yet Valid
  ├── Currently Valid
  └── Expired
```

An expired ticket should not be treated as current authentication material.

---

## 18. Clock Synchronization Matters

Kerberos is sensitive to time differences between participating systems.

A failure may therefore occur because of:

```text
Clock Skew
```

rather than:

```text
Invalid Credentials
```

When Kerberos authentication fails, consider:

```text
Identity
Ticket
Target
Service
DNS
Time Synchronization
Domain Connectivity
Policy
```

before concluding that the credential is invalid.

---

## 19. DNS Is Important

Kerberos relies heavily on correct service and domain naming.

A target may be identified as:

```text
server01.corp.local
```

rather than simply:

```text
10.10.10.20
```

Therefore check:

```text
Hostname Resolution
Domain DNS
Service Naming
Target FQDN
```

If a Kerberos request fails unexpectedly, DNS should be part of the troubleshooting process.

---

## 20. Target Mapping From Tickets

Create a mapping like:

| Identity   | Ticket / Service | Target   | Service | Reachable | Access  |
| ---------- | ---------------- | -------- | ------- | --------- | ------- |
| CORP\alice | CIFS/server01    | SERVER01 | SMB     | Yes       | Unknown |
| CORP\alice | HTTP/web01       | WEB01    | HTTP    | Yes       | Unknown |
| CORP\alice | MSSQLSvc/db01    | DB01     | MSSQL   | Unknown   | Unknown |

This converts ticket information into movement hypotheses.

---

## 21. Kerberos + SMB

A common relationship is:

```text
Kerberos
   ↓
CIFS
   ↓
SMB
   ↓
File Server
```

If a valid Kerberos service ticket is associated with:

```text
CIFS/server01.corp.local
```

then SERVER01 becomes a relevant candidate.

The next step is to validate the authorized SMB access path.

See:

```text
06-Remote-Access-Services/SMB.md
```

---

## 22. Kerberos + WinRM

WinRM can use Kerberos in domain environments.

Conceptually:

```text
Kerberos
   ↓
WSMAN / WinRM
   ↓
Windows Target
```

If the environment supports this authentication path, the target may become a movement candidate.

Do not infer administrative rights from the presence of a Kerberos relationship.

See:

```text
06-Remote-Access-Services/WinRM.md
```

---

## 23. Kerberos + HTTP

Kerberos can also be associated with web services.

For example:

```text
HTTP/web01.corp.local
```

may indicate a Kerberos-aware web service.

This is primarily useful for:

```text
Service Identification
Target Discovery
Authentication Context
```

It does not automatically mean that the current user can access privileged application functionality.

---

## 24. Kerberos + Database Services

An SPN such as:

```text
MSSQLSvc/db01.corp.local
```

can reveal:

```text
Database Service
Target Host
Service Account Context
```

This can help expand the movement map.

The correct workflow is:

```text
SPN
 ↓
Identify Service
 ↓
Identify Host
 ↓
Check Reachability
 ↓
Identify Authentication
 ↓
Validate Authorization
```

---

## 25. Kerberos and Service Accounts

Kerberos service relationships can reveal service-account context.

Conceptually:

```text
Service
 ↓
SPN
 ↓
Service Account
 ↓
Host / Application
```

This is useful for understanding the environment.

However, discovering a service account does not automatically provide its credentials or administrative access.

Keep:

```text
Service Discovery
```

separate from:

```text
Credential Acquisition
```

---

## 26. Ticket Ownership Matters

Always ask:

```text
Who owns this ticket?
```

A ticket belonging to:

```text
CORP\alice
```

should not be recorded as:

```text
Administrator access
```

unless authorization has actually been established.

Likewise, a ticket in one logon session may not automatically be usable from another security context.

---

## 27. Existing Ticket vs New Authentication

There are two different situations:

### Existing Authentication Context

```text
Current Session
      ↓
Existing Kerberos Tickets
      ↓
Potential Service Access
```

### New Authentication

```text
Known Identity / Credential
      ↓
Kerberos Authentication
      ↓
New Service Ticket
      ↓
Target Service
```

The first situation should be investigated before creating unnecessary new authentication attempts.

---

## 28. Do Not Destroy Useful Context

During assessment, avoid unnecessary actions that invalidate or overwrite useful authentication context.

Before changing authentication state, record:

```text
Current Identity
Current Logon Session
Ticket Cache
Relevant Service Tickets
Target Relationships
```

The objective is evidence preservation and controlled testing.

---

## 29. Kerberos Failure Analysis

When Kerberos authentication fails, classify the failure.

### Identity Problem

```text
Wrong Principal
Wrong Domain
Wrong Account
```

### Ticket Problem

```text
Expired Ticket
Not-Yet-Valid Ticket
Wrong Service Ticket
Missing Ticket
```

### Service Problem

```text
Service Unavailable
Incorrect SPN
Service Not Kerberos-Compatible
```

### Environment Problem

```text
DNS Failure
Domain Connectivity
Clock Skew
Routing
Firewall
```

### Policy Problem

```text
Authentication Restricted
Account Restrictions
Protocol Restrictions
```

This prevents random troubleshooting.

---

## 30. Kerberos Decision Tree

```text
Kerberos Relevant?
       │
   ┌───┴───┐
   │       │
  No      Yes
   │       │
   ▼       ▼
Other    Identify
Branch   Identity
           │
           ▼
       Identify Domain
           │
           ▼
       Inspect Tickets
           │
           ▼
       Ticket Found?
        │         │
       No        Yes
        │         │
        ▼         ▼
   Investigate  Identify
   Other Auth   Service
                   │
                   ▼
               Identify
                Target
                   │
                   ▼
              Check Reachability
                   │
                   ▼
            Validate Authentication
                   │
             ┌─────┴─────┐
             ▼           ▼
          Success      Failure
             │           │
             ▼           ▼
      Check Authorization
                         │
                         ▼
                    Classify Failure
             │           │
             └─────┬─────┘
                   ▼
             Update Movement Map
```

---

## 31. Kerberos Investigation Record

Use this structure:

```text
Identity:
____________________

Domain / Realm:
____________________

Host:
____________________

Ticket Type:
____________________

Service Principal:
____________________

Target:
____________________

Service:
____________________

Ticket Validity:
____________________

Reachability:
____________________

Authentication Result:
____________________

Authorization Result:
____________________

Failure / Restrictions:
____________________

Next Action:
____________________
```

---

## 32. Practical Windows Workflow

From a Windows foothold:

```text
1. whoami
2. whoami /user
3. echo %USERDOMAIN%
4. echo %USERDNSDOMAIN%
5. Check domain membership
6. klist
7. Identify ticket owners
8. Identify service principals
9. Map services to target hosts
10. Check target reachability
11. Validate the relevant service
12. Validate authorization
13. Record result
14. Reassess movement paths
```

The important part is not the command sequence.

The important part is the reasoning:

```text
Ticket
 ↓
Service
 ↓
Target
 ↓
Reachability
 ↓
Authentication
 ↓
Authorization
```

---

## 33. Practical Linux Workflow

From a Linux foothold:

```text
1. whoami
2. id
3. realm list
4. hostname -f
5. Check domain integration
6. klist
7. Identify default principal
8. Identify service tickets
9. Map services to target hosts
10. Check DNS and reachability
11. Validate the relevant service
12. Validate authorization
13. Record result
14. Reassess movement paths
```

If Kerberos is not configured or no relevant ticket context exists, continue with another credential/access branch.

---

## 34. Common Mistakes

### Mistake 1: Treating Every Ticket as Administrative Access

A ticket proves authentication context, not privilege level.

---

### Mistake 2: Ignoring the Service Principal

The service principal often tells you which host/service relationship matters.

---

### Mistake 3: Ignoring DNS

Kerberos commonly depends on correct domain and service naming.

---

### Mistake 4: Ignoring Time

Clock skew can produce authentication failures that look like credential problems.

---

### Mistake 5: Treating an Expired Ticket as Current

Always inspect ticket validity.

---

### Mistake 6: Assuming a Domain Account Can Access Every Host

Domain membership does not equal universal authorization.

---

### Mistake 7: Jumping Directly to Credential Abuse

First determine whether an existing legitimate authentication context already provides the required access.

---

## 35. Completion Criteria

Kerberos investigation is sufficiently complete for the current position when you can answer:

```text
[ ] Is Kerberos relevant?
[ ] What identity am I using?
[ ] What domain / realm am I in?
[ ] Are Kerberos tickets present?
[ ] Who owns the tickets?
[ ] Which tickets are currently valid?
[ ] Which services are represented?
[ ] Which target hosts do those services identify?
[ ] Are those targets reachable?
[ ] Which remote-access paths are relevant?
[ ] Has authentication been validated?
[ ] Has authorization been validated?
[ ] Have failures been classified?
[ ] Has the movement map been updated?
[ ] What is the next action?
```

The objective is not:

```text
Find as many tickets as possible.
```

The objective is:

```text
Understand the current Kerberos authentication context
and determine whether it creates a useful, authorized movement path.
```

---

## 36. What Next?

After Kerberos, investigate another form of authentication context:

> **"Are there active sessions, tokens, cached authentication contexts, or other logged-in identities that could change what I can access?"**

Continue with:

```text
05-Credential-and-Access-Discovery/
└── Tokens-and-Sessions.md
```

The workflow becomes:

```text
Kerberos Context
      ↓
Tokens and Sessions
      ↓
Identify Active Contexts
      ↓
Identify Associated Identities
      ↓
Identify Accessible Services / Hosts
      ↓
Validate Access
      ↓
Update Movement Map
```

The core principle is:

> **A Kerberos ticket is evidence of an authentication context, not proof of universal access. Identify its owner, service, target, validity, and authorization before treating it as a lateral-movement path.**
