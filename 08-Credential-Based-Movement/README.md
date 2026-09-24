# Credential-Based Movement

Credential-based movement focuses on situations where access to another host is enabled by authentication material already available from the current position.

The goal is not simply to find credentials. The goal is to determine:

* What identity or authentication material is available?
* What type of authentication material is it?
* Which systems or services can use it?
* Which targets are relevant?
* Does the material authenticate successfully?
* Is the authenticated identity authorized to perform the required action?
* Does successful movement provide a useful new position?

The workflow is:

```text
Current Position
      ↓
Identify Identity / Authentication Material
      ↓
Determine Credential Type
      ↓
Map to Compatible Protocol / Service
      ↓
Identify Candidate Targets
      ↓
Validate Authentication
      ↓
Validate Authorization
      ↓
Establish Movement
      ↓
Validate New Position
      ↓
Re-enumerate
```

## What This Section Covers

This section covers four major credential-based movement paths:

| Technique         | Primary Material                    | Typical Context          |
| ----------------- | ----------------------------------- | ------------------------ |
| Valid Accounts    | Username + authentication material  | Local or domain accounts |
| Pass-the-Hash     | NTLM hash                           | Windows authentication   |
| Pass-the-Ticket   | Kerberos ticket                     | Active Directory         |
| Overpass-the-Hash | NTLM hash → Kerberos authentication | Active Directory         |

The techniques are related, but they should not be treated as interchangeable.

The first task is always to identify **what authentication material you actually have**.

## Credential Material vs Access

Finding authentication material does not automatically mean lateral movement is possible.

For example:

```text
Username + Password
        ↓
Can authenticate?
        ↓
Is target reachable?
        ↓
Is the account authorized?
        ↓
Can the selected service provide remote access?
```

Likewise:

```text
NTLM Hash
   ↓
Compatible authentication mechanism?
   ↓
Target supports required authentication?
   ↓
Account authorized?
   ↓
Movement possible?
```

And:

```text
Kerberos Ticket
      ↓
Correct domain / realm?
      ↓
Correct service?
      ↓
Ticket valid?
      ↓
Account authorized?
      ↓
Movement possible?
```

A credential should therefore be treated as an **enabler**, not as proof of access.

## Local vs Domain Identity

Before attempting movement, determine whether the identity is:

* a local account
* a domain account
* a service account
* a machine account
* another identity supported by the environment

This matters because the same username can exist in different security contexts.

For example:

```text
HOST-A\administrator
```

and:

```text
DOMAIN\administrator
```

are not necessarily the same account.

Always establish the security context before interpreting authentication results.

## Credential-to-Service Mapping

Different authentication materials work with different services and protocols.

Use the following reasoning:

```text
Authentication Material
        ↓
What protocol can use it?
        ↓
What service exposes that protocol?
        ↓
Which target exposes that service?
        ↓
Is the identity authorized there?
```

Common relationships include:

| Authentication Material                   | Possible Service / Protocol                             |
| ----------------------------------------- | ------------------------------------------------------- |
| Username + password                       | SMB, WinRM, RDP, SSH and other authenticated services   |
| SSH private key                           | SSH                                                     |
| NTLM hash                                 | NTLM-compatible Windows authentication                  |
| Kerberos ticket                           | Kerberos-enabled services                               |
| Kerberos-capable identity + NTLM material | Kerberos authentication through an appropriate workflow |

The exact options depend on the environment, authentication configuration, and account privileges.

## Authentication Is Not Authorization

Always separate these two questions:

```text
Authentication
"Who are you?"
```

from:

```text
Authorization
"What are you allowed to do?"
```

A successful login does not automatically mean:

* administrative access
* remote command execution
* access to a particular share
* RDP access
* WinRM access
* access to another service
* access to another host

After authentication succeeds, explicitly determine what the account can do.

## Target Selection

Do not test every discovered host indiscriminately.

Prioritize targets based on evidence such as:

* known administrative roles
* server function
* domain-controller status
* previously identified trust relationships
* services known to be reachable
* known account ownership
* current network position
* existing access paths
* the purpose of the assessment

A useful decision pattern is:

```text
Credential Material Found
        ↓
Identify Account
        ↓
Identify Target Group
        ↓
Check Reachability
        ↓
Check Compatible Service
        ↓
Validate Authentication
        ↓
Check Authorization
```

This keeps movement evidence-driven rather than speculative.

## Valid Accounts

Valid-account movement begins with an account and usable authentication material.

The workflow is:

```text
Account Identified
      ↓
Determine Local / Domain Context
      ↓
Identify Authentication Method
      ↓
Identify Candidate Targets
      ↓
Check Reachability
      ↓
Select Compatible Service
      ↓
Authenticate
      ↓
Validate Authorization
      ↓
Move
```

See [`Valid-Accounts.md`](Valid-Accounts.md).

## Pass-the-Hash

Pass-the-Hash uses an NTLM hash as authentication material instead of requiring the corresponding plaintext password.

The important workflow is:

```text
NTLM Hash Available
      ↓
Identify Associated Account
      ↓
Determine Local / Domain Context
      ↓
Identify Compatible Authentication Path
      ↓
Identify Candidate Target
      ↓
Validate Reachability
      ↓
Attempt Authorized Authentication
      ↓
Validate Result
      ↓
Re-enumerate
```

The hash should be treated as sensitive authentication material.

See [`Pass-the-Hash.md`](Pass-the-Hash.md).

## Pass-the-Ticket

Pass-the-Ticket involves the use of Kerberos ticket material to authenticate to services in an Active Directory environment.

The workflow is:

```text
Kerberos Ticket Available
        ↓
Identify Ticket Type
        ↓
Identify Principal
        ↓
Identify Realm / Domain
        ↓
Identify Service
        ↓
Check Ticket Validity
        ↓
Identify Target
        ↓
Validate Service Access
        ↓
Move
        ↓
Re-enumerate
```

Do not assume that possession of a ticket means access to every service or host.

See [`Pass-the-Ticket.md`](Pass-the-Ticket.md).

## Overpass-the-Hash

Overpass-the-Hash uses NTLM authentication material as the starting point for obtaining or using Kerberos authentication in an Active Directory environment.

Conceptually:

```text
NTLM Hash
    ↓
Authenticate as the Associated Identity
    ↓
Obtain / Establish Kerberos Authentication Context
    ↓
Use Kerberos-Compatible Service
    ↓
Validate Access
    ↓
Move
```

The important distinction is that the technique bridges authentication material associated with NTLM and a Kerberos-based authentication workflow.

See [`Overpass-the-Hash.md`](Overpass-the-Hash.md).

## Failure Classification

When credential-based movement fails, do not immediately assume the credential is invalid.

Classify the failure first.

### Target Unreachable

```text
Credential
   ↓
Target
   ↓
No Network Connectivity
```

Investigate:

* routing
* firewalling
* segmentation
* VPN/interface state
* incorrect target address

### Service Unavailable

```text
Target Reachable
      ↓
Required Service Unavailable
```

Investigate:

* listening ports
* service state
* host firewall
* incorrect protocol assumption
* non-standard service ports

### Authentication Failure

```text
Service Reachable
      ↓
Authentication Rejected
```

Investigate:

* username
* credential type
* domain/local context
* expired or invalid material
* authentication protocol
* ticket validity
* clock synchronization where Kerberos is involved

### Authorization Failure

```text
Authentication Successful
      ↓
Requested Action Denied
```

The credential may be valid while the account lacks the required rights.

This is an authorization problem, not necessarily a credential problem.

### Wrong Credential-to-Service Mapping

A credential can be valid but unsuitable for the selected service.

For example:

```text
Valid Kerberos Material
        ↓
Selected Service
        ↓
Service Does Not Accept That Authentication Path
```

Return to the service and authentication mapping before changing credentials.

## Evidence Collection

For every successful movement path, record:

```text
Source Host:
Source User:
Credential / Authentication Material Type:
Target Host:
Target IP:
Target Service:
Authentication Method:
Authorization Result:
New User:
New Host:
New Network Position:
Next Enumeration Step:
```

This makes the movement chain reproducible.

A useful chain may look like:

```text
Host A
  ↓
Credential discovered
  ↓
Account identified
  ↓
Host B reachable
  ↓
SMB authentication succeeds
  ↓
Authorized remote access confirmed
  ↓
Host B
  ↓
Re-enumerate
  ↓
New credential / target discovered
  ↓
Host C
```

## Operational Rules

### 1. Do Not Spray Indiscriminately

Do not blindly test discovered credentials against large numbers of systems.

Use evidence to select relevant targets.

### 2. Confirm Scope

Only perform credential-based movement against systems and accounts covered by the authorized assessment.

### 3. Minimize Authentication Attempts

Prefer a small number of justified validation attempts over broad testing.

### 4. Protect Authentication Material

Treat passwords, hashes, tickets, private keys, and tokens as sensitive data.

Do not unnecessarily copy, expose, or store them.

### 5. Validate Before Escalating

A successful authentication should be followed by authorization validation and position assessment.

### 6. Re-enumerate After Movement

A new host changes the attacker's visibility.

Do not continue using the old host's assumptions.

## Decision Framework

Use this simplified decision process:

```text
Do I have authentication material?
        │
       No
        ↓
Return to Credential / Access Discovery
        │
       Yes
        ↓
What type?
        │
        ├── Account + Password
        │       ↓
        │   Valid Accounts
        │
        ├── NTLM Hash
        │       ↓
        │   Pass-the-Hash
        │       │
        │       └── AD + Kerberos objective
        │               ↓
        │           Overpass-the-Hash
        │
        └── Kerberos Ticket
                ↓
            Pass-the-Ticket
```

Then:

```text
Technique Selected
        ↓
Compatible Service?
        │
   ┌────┴────┐
  No        Yes
  ↓           ↓
Find another  Target reachable?
service           │
               ┌──┴──┐
              No    Yes
              ↓       ↓
        Find another  Authenticate
        target             ↓
                      Authorized?
                       │
                  ┌────┴────┐
                 No        Yes
                 ↓           ↓
          Record limitation  Move
                             ↓
                        Validate
                             ↓
                       Re-enumerate
```

## What Next?

Start with [`Valid-Accounts.md`](Valid-Accounts.md).

It establishes the foundational workflow for using legitimate account authentication as a lateral-movement path before moving into hash- and ticket-based techniques.
