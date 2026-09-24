# Pass-the-Hash

Pass-the-Hash (PtH) is a Windows authentication technique in which an NTLM hash is used as authentication material instead of the corresponding plaintext password.

The important workflow is not simply:

```text
Hash Found → Use Hash
```

Instead:

```text
NTLM Hash Identified
        ↓
Identify Associated Account
        ↓
Determine Local / Domain Context
        ↓
Determine Compatible Authentication Path
        ↓
Identify Candidate Target
        ↓
Check Reachability
        ↓
Validate Authentication
        ↓
Validate Authorization
        ↓
Establish Authorized Access
        ↓
Validate New Position
        ↓
Re-enumerate
```

This workflow should be used only within systems and accounts covered by the authorized assessment.

## Objective

The objective is to determine whether recovered NTLM authentication material can provide authorized access to another Windows system.

The key questions are:

* What account does the hash belong to?
* Is the account local or domain-based?
* Is the hash actually an NTLM authentication value?
* What service can use the authentication method?
* Which target is relevant?
* Is the target reachable?
* Does authentication succeed?
* What authorization does the account have?
* What new position is obtained?

## What Is Being Passed?

In a simplified Windows authentication context:

```text
Account
  ↓
NTLM Authentication Material
  ↓
Authentication Attempt
  ↓
Target Service
```

The plaintext password does not need to be known for the authentication workflow to potentially succeed.

The hash therefore represents **authentication material** and should be handled as sensitive information.

## 1. Identify the Hash

Before attempting anything, determine what the recovered value represents.

Potential sources include:

* authorized credential stores
* forensic evidence
* security-testing artifacts
* configuration or deployment material
* password-database exports provided for the assessment
* lab environments

Do not assume every hexadecimal string is an NTLM hash.

Record:

```text
Hash Type:
Associated Account:
Source:
Local / Domain Context:
Collection Time:
```

Where the format is uncertain, verify the hash type before proceeding.

## 2. Identify the Associated Account

A hash without account context is much less useful.

Determine:

```text
Username:
Domain:
Computer:
Account Type:
```

For example:

```text
DOMAIN\alice
```

is different from:

```text
WORKSTATION01\alice
```

The same username can exist in multiple security contexts.

The account context affects:

* where the credential may be valid
* which authentication authority is involved
* which targets may recognize the identity
* what authorization may be available

## 3. Determine Local vs Domain Context

This is one of the most important decisions.

### Local Account

A local account is associated with a specific Windows host.

Conceptually:

```text
HOST-A\user
```

Its credentials are normally associated with that host's local security authority.

### Domain Account

A domain account belongs to an Active Directory domain.

Conceptually:

```text
DOMAIN\user
```

Its authentication is handled within the domain security context.

Do not assume that a local administrator hash can automatically authenticate against every system.

## 4. Identify Candidate Targets

Use target information already gathered.

Prioritize systems where there is evidence that the account may have access.

Useful evidence includes:

* known administrative relationships
* domain membership
* workstation/server roles
* previous authentication
* service exposure
* known account ownership
* documented administrative responsibilities
* network reachability

Avoid broad, indiscriminate authentication attempts.

Use:

```text
Hash
 ↓
Account
 ↓
Evidence
 ↓
Relevant Target
```

rather than:

```text
Hash
 ↓
Every Host
```

## 5. Determine the Authentication Path

Pass-the-Hash is associated with NTLM authentication.

The selected service must therefore support an appropriate authentication path.

Common Windows services involved in lateral movement can include:

* SMB
* RPC/DCOM
* WMI
* other Windows services that accept NTLM authentication

The exact behavior depends on:

* operating-system version
* security configuration
* authentication policy
* service configuration
* account privileges
* network restrictions

Do not assume that because one service accepts the authentication material, every service will.

## 6. Check Target Reachability

Before troubleshooting the hash, verify the network path.

For SMB:

```powershell id="6jv5eh"
Test-NetConnection TARGET -Port 445
```

For RPC:

```powershell id="g0t8f8"
Test-NetConnection TARGET -Port 135
```

From Linux:

```bash id="a4wyv2"
nc -vz TARGET 445
nc -vz TARGET 135
```

Interpret the result carefully.

```text
Port unreachable
      ↓
Network / firewall / service issue

Port reachable
      ↓
Continue to authentication testing
```

A network failure is not evidence that the hash is invalid.

## 7. Validate SMB as an Authentication Path

SMB is a common Windows lateral-movement service.

First determine whether the target exposes SMB:

```powershell id="3m8m1q"
Test-NetConnection TARGET -Port 445
```

Then use an authorized tool or framework capable of testing the NTLM authentication material against the target.

The important evidence is:

```text
Network Reachability
        ↓
SMB Service
        ↓
NTLM Authentication
        ↓
Account Authorization
```

Do not interpret a successful SMB authentication as automatic command execution.

Determine what the account can actually access.

## 8. Understand Administrative Shares

Windows systems may expose administrative shares such as:

```text
C$
ADMIN$
IPC$
```

Their presence alone does not prove that the current identity can use them.

The decision is:

```text
Administrative Share Exists
        ↓
Can Current Identity Authenticate?
        ↓
Is Current Identity Authorized?
        ↓
Can Share Be Accessed?
```

This distinction prevents:

```text
Share Exists
   =
Administrator Access
```

from being incorrectly assumed.

## 9. WMI and RPC Considerations

Remote WMI commonly relies on Windows RPC/DCOM infrastructure.

Relevant connectivity can include:

```text
TCP 135
+
Dynamic RPC
```

Therefore, if a PtH attempt involving WMI or RPC fails, classify the failure carefully.

Possible causes include:

* target unreachable
* TCP 135 blocked
* dynamic RPC unavailable
* authentication rejected
* insufficient authorization
* service configuration
* Windows security policy

Do not immediately conclude that the NTLM material is invalid.

## 10. Authentication vs Authorization

Always separate these results.

### Authentication

```text
NTLM Material
      ↓
Target Service
      ↓
Identity Accepted
```

This establishes that the authentication path accepted the identity.

### Authorization

```text
Authenticated Identity
      ↓
Requested Action
      ↓
Allowed / Denied
```

An identity may authenticate successfully but lack permission to:

* access an administrative share
* perform remote management
* execute the required administrative operation
* access a specific resource

Record these separately.

## 11. Validate the New Position

If authorized movement succeeds, establish exactly what access was obtained.

On the target, validate:

```powershell id="qpmz9q"
whoami
hostname
ipconfig
route print
```

For additional identity context:

```powershell id="5ahc0s"
whoami /user
whoami /groups
whoami /priv
```

The objective is to answer:

```text
Who am I?
Which host am I on?
Which network interfaces exist?
Which routes are available?
What privileges are present?
```

## 12. Re-enumerate the Target

The target is now a new position.

Repeat the appropriate discovery workflow:

```text
New Host
   ↓
Current User
   ↓
Network Context
   ↓
Trust / Domain Context
   ↓
Services
   ↓
Credentials / Access
   ↓
New Targets
```

Do not assume that the previous host and new host have the same visibility.

The new host may expose:

* another subnet
* internal servers
* additional domain relationships
* different credentials
* different services
* different administrative access

## 13. Common Failure Scenarios

### Hash Is Rejected

Possible causes:

* incorrect hash
* incorrect account association
* wrong local/domain context
* authentication restrictions
* target does not accept the selected authentication path
* security policy prevents the attempted authentication

Verify the account and authentication context before changing targets repeatedly.

### Target Is Unreachable

Possible causes:

* firewall
* routing
* segmentation
* VPN/interface configuration
* incorrect IP or hostname

Test connectivity separately.

### SMB Works but Administrative Access Fails

Possible interpretation:

```text
Authentication ✓
Authorization ✗
```

The account may be valid without having administrative rights.

### WMI/RPC Fails but SMB Works

Do not immediately conclude that the credential material is invalid.

The services may have different:

* authorization requirements
* firewall exposure
* RPC dependencies
* security policies

### Authentication Works but the Result Is Not Useful

A valid authentication path may still produce little lateral-movement value.

For example:

```text
Authentication ✓
Remote Access ✗
```

or:

```text
Authentication ✓
Limited Resource Access ✓
Administrative Access ✗
```

Record the actual result and reassess the target.

## 14. Evidence Recording

For every PtH validation, record:

```text
Source Host:
Hash Type:
Associated Account:
Local / Domain Context:
Target Host:
Target IP:
Target Service:
Authentication Result:
Authorization Result:
Access Obtained:
New User:
New Host:
New Network Visibility:
Next Target:
```

Example:

```text
Source Host: WORKSTATION-01
Account: DOMAIN\adminuser
Credential Type: NTLM Hash
Target: SERVER-01
Service: SMB
Authentication: Successful
Authorization: Administrative share access confirmed
New Position: SERVER-01
Next Step: Re-enumerate SERVER-01
```

Do not store the actual hash in general-purpose notes unless the assessment requires it and the storage is appropriately protected.

## 15. Decision Workflow

Use this decision tree:

```text
NTLM Hash Available
        ↓
Identify Account
        ↓
Local or Domain?
        ↓
Identify Relevant Targets
        ↓
Check Reachability
        ↓
Identify Compatible Service
        ↓
Attempt Authorized Authentication
        │
        ├── Network Failure
        │       ↓
        │   Troubleshoot Connectivity
        │
        ├── Service Failure
        │       ↓
        │   Troubleshoot Service / RPC
        │
        ├── Authentication Failure
        │       ↓
        │   Validate Account / Hash / Context
        │
        └── Authentication Success
                ↓
          Validate Authorization
                │
                ├── Denied
                │     ↓
                │  Record Limitation
                │
                └── Allowed
                      ↓
                 Establish Access
                      ↓
                 Validate Position
                      ↓
                 Re-enumerate
```

## 16. What Not to Assume

Avoid these assumptions:

```text
Hash Found
   ≠
Administrator Access
```

```text
Authentication Success
   ≠
Full Remote Control
```

```text
Local Administrator
   ≠
Domain Administrator
```

```text
SMB Access
   ≠
WinRM Access
```

```text
One Successful Target
   ≠
Every Target
```

```text
Failed Service
   ≠
Invalid Credential
```

Each conclusion requires evidence.

## 17. Operational Rules

### Use Only Authorized Authentication Material

Only test hashes obtained within the approved assessment scope.

### Minimize Attempts

Use evidence-driven target selection instead of broad authentication attempts.

### Protect Hashes

An NTLM hash can represent reusable authentication material. Treat it as sensitive.

### Record Context

Always document the account, domain/local context, target, service, and result.

### Re-enumerate

Every successful movement creates a new position and potentially a new set of targets.

## What Next?

Continue to [`Pass-the-Ticket.md`](Pass-the-Ticket.md).

That workflow moves from NTLM authentication material to Kerberos ticket-based authentication in Active Directory environments.
