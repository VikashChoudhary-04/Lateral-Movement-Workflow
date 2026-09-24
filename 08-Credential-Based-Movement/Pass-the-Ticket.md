# Pass-the-Ticket

Pass-the-Ticket (PtT) is a Kerberos-based lateral-movement technique in which captured or otherwise obtained Kerberos ticket material is used to authenticate to services without necessarily requiring the user's plaintext password.

The workflow is:

```text
Kerberos Ticket Identified
        ↓
Identify Ticket Type
        ↓
Identify Principal
        ↓
Identify Domain / Realm
        ↓
Identify Target Service
        ↓
Check Ticket Validity
        ↓
Identify Compatible Target
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

This workflow is intended for authorized security assessments and controlled labs.

## Objective

The objective is to determine whether available Kerberos ticket material can provide authorized access to another system or service.

The key questions are:

* What ticket do I have?
* Is it a TGT or service ticket?
* Which principal does it represent?
* Which domain/realm issued it?
* Which service is the ticket for?
* Is the ticket currently valid?
* Which target exposes the corresponding service?
* Does authentication succeed?
* What authorization does the identity have?
* What new position is obtained?

## Kerberos Context

Kerberos authentication involves several important identities and artifacts.

A simplified flow is:

```text
User / Principal
      ↓
Kerberos Authentication
      ↓
TGT
      ↓
Service Ticket
      ↓
Target Service
```

A **Ticket Granting Ticket (TGT)** is used to obtain service tickets.

A **service ticket** is associated with a particular service.

Therefore:

```text
TGT
≠
Service Ticket
```

and:

```text
Service Ticket for Service A
≠
Automatic Access to Service B
```

Understanding the ticket's intended use is essential.

## 1. Identify Available Tickets

On Windows, inspect the current Kerberos ticket cache with:

```cmd id="u4r8j1"
klist
```

Useful information includes:

* client principal
* server principal
* ticket type
* start time
* expiry time
* renew time
* encryption type

The purpose is to understand the authentication context before attempting movement.

## 2. Identify the Principal

Determine which identity the ticket represents.

A ticket may contain a principal such as:

```text id="n1x6gm"
user@DOMAIN.LOCAL
```

or be associated with a service principal such as:

```text id="9k4m3c"
cifs/server.domain.local
```

The principal matters because it determines the identity and service context of the authentication material.

Record:

```text
Principal:
Domain / Realm:
Ticket Type:
Service Principal:
Validity:
```

## 3. Identify the Ticket Type

The first major decision is:

```text
What ticket do I have?
```

### TGT

A TGT is associated with the Kerberos ticket-granting service.

Conceptually:

```text
TGT
 ↓
Request Service Ticket
 ↓
Access Kerberos Service
```

### Service Ticket

A service ticket is intended for a particular service.

For example:

```text
cifs/server.domain.local
```

represents a service context for SMB/CIFS.

Other service principals may represent services such as:

```text
HTTP/server.domain.local
HOST/server.domain.local
LDAP/server.domain.local
```

Do not assume a service ticket for one service can simply be substituted for another.

## 4. Check Domain and Realm Context

Kerberos depends heavily on domain and realm relationships.

Determine:

```text
Client Domain:
Kerberos Realm:
Ticket Realm:
Target Domain:
```

In Active Directory environments, DNS is also important.

Check the current domain context:

```cmd id="w5w4fm"
echo %USERDOMAIN%
echo %USERDNSDOMAIN%
```

Domain-controller discovery can be useful:

```cmd id="6z2h1j"
nltest /dsgetdc:DOMAIN
```

A mismatch in domain, realm, or target naming can cause authentication failures.

## 5. Check Ticket Validity

Kerberos tickets have time boundaries.

Inspect:

```text
Start Time
End Time
Renew Until
```

A ticket outside its validity period may not work.

Kerberos also depends on reasonably synchronized system clocks.

If authentication fails unexpectedly, consider:

```text
Ticket Validity
      ↓
Clock Synchronization
      ↓
DNS / Domain Resolution
```

Do not immediately assume the ticket itself is unusable.

## 6. Identify the Target Service

A service ticket points toward a service.

For example:

```text
cifs/server01.domain.local
```

suggests SMB/CIFS.

The workflow becomes:

```text
Ticket
  ↓
Service Principal
  ↓
Service Type
  ↓
Target Host
  ↓
Network Reachability
```

This prevents choosing an unrelated target merely because it is reachable.

## 7. Check Target Reachability

For SMB:

```powershell id="ddj4u5"
Test-NetConnection SERVER01 -Port 445
```

For LDAP:

```powershell id="q8o5xn"
Test-NetConnection SERVER01 -Port 389
```

For HTTPS-based services:

```powershell id="i3j4l9"
Test-NetConnection SERVER01 -Port 443
```

The exact port depends on the service represented by the ticket.

A network failure should be separated from Kerberos authentication failure.

## 8. Understand SPNs

Service Principal Names (SPNs) associate Kerberos services with identities.

A simplified representation is:

```text
SERVICE/HOST:PORT
```

Examples include:

```text
cifs/server01.domain.local
HTTP/web01.domain.local
LDAP/dc01.domain.local
```

The service principal helps determine which service the ticket is intended for.

The important reasoning is:

```text
Ticket
 ↓
SPN
 ↓
Service
 ↓
Host
```

An SPN mismatch can cause authentication problems even when the underlying account is valid.

## 9. Validate SMB / CIFS Access

For a CIFS/SMB service ticket, the target service is typically SMB.

First verify the service is reachable:

```powershell id="n4l0gx"
Test-NetConnection SERVER01 -Port 445
```

Then use an authorized Kerberos-capable SMB workflow to validate the available ticket material.

The result should be interpreted in layers:

```text
Network Reachability
        ↓
SMB Service
        ↓
Kerberos Authentication
        ↓
SMB Authorization
        ↓
Resource Access
```

A successful Kerberos authentication does not automatically imply administrative share access.

## 10. Authentication vs Authorization

Separate these two results.

### Authentication

```text
Kerberos Ticket
      ↓
Target Service
      ↓
Identity Accepted
```

### Authorization

```text
Authenticated Identity
      ↓
Requested Resource / Action
      ↓
Allowed / Denied
```

For example:

```text
Kerberos Authentication ✓
SMB Authentication      ✓
Administrative Share    ✗
```

This is an authorization limitation rather than proof that the ticket is invalid.

## 11. TGT vs Service Ticket Decision

Use this simplified decision:

```text
Kerberos Ticket
      ↓
   What type?
      │
 ┌────┴─────┐
 TGT      Service Ticket
  ↓             ↓
Obtain /     Identify
request      SPN
service        ↓
ticket       Target
  ↓             ↓
Service      Validate
access       service
```

The correct next step depends on what ticket material is available.

## 12. Validate the New Position

If authorized movement succeeds, immediately establish the new position.

On Windows:

```cmd id="br5v2u"
whoami
hostname
ipconfig
route print
```

Then inspect Kerberos context:

```cmd id="z3m6p4"
klist
```

For additional privilege information:

```cmd id="p5w1ar"
whoami /groups
whoami /priv
```

The objective is to determine:

```text
Who am I?
Which host am I on?
Which domain context am I in?
Which networks are visible?
Which privileges are available?
Which Kerberos tickets now exist?
```

## 13. Re-enumerate From the New Host

A successful Kerberos-based movement changes the assessment position.

Repeat the relevant workflow:

```text
New Host
   ↓
Current User
   ↓
Network Context
   ↓
Domain Context
   ↓
Kerberos Tickets
   ↓
Services
   ↓
Credentials / Access
   ↓
New Targets
```

Pay particular attention to:

* newly reachable hosts
* domain-controller visibility
* additional service principals
* new authentication material
* administrative relationships
* new network segments

## 14. Common Failure Scenarios

### Ticket Expired

```text
Ticket
  ↓
Validity Check
  X
```

Check:

* start time
* expiry time
* renewal period

### Clock Skew

Kerberos is sensitive to time differences.

If a ticket appears valid but authentication fails, verify that the participating systems have sufficiently synchronized clocks.

### DNS / Naming Problem

Kerberos commonly depends on correct host and service naming.

Check:

```text
Hostname
FQDN
DNS Resolution
SPN
Realm
```

### Wrong Service

A service ticket may be valid but intended for another service.

For example:

```text
HTTP/server01
```

should not automatically be treated as:

```text
CIFS/server01
```

### Authentication Succeeds but Access Is Denied

Interpret as:

```text
Authentication ✓
Authorization ✗
```

The identity may be valid but lack the required permissions.

### Target Unreachable

A valid ticket cannot compensate for a missing network path.

Check:

```text
Routing
Firewall
Segmentation
Service Availability
```

## 15. Kerberos Troubleshooting Order

When a ticket-based movement attempt fails, use this order:

```text
1. Is the target reachable?
        ↓
2. Is the required service available?
        ↓
3. Is the hostname / FQDN correct?
        ↓
4. Is the domain / realm correct?
        ↓
5. Is the SPN appropriate?
        ↓
6. Is the ticket valid?
        ↓
7. Is the authentication accepted?
        ↓
8. Is the account authorized?
```

This avoids changing several variables simultaneously.

## 16. Evidence Recording

Record the ticket context without unnecessarily storing sensitive ticket material.

```text
Source Host:
Principal:
Domain / Realm:
Ticket Type:
Service Principal:
Target Host:
Target Service:
Ticket Validity:
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
Principal: DOMAIN\user
Ticket Type: Service Ticket
SPN: cifs/server01.domain.local
Target: SERVER-01
Service: SMB
Authentication: Successful
Authorization: Share access confirmed
New Position: SERVER-01
Next Step: Re-enumerate SERVER-01
```

## 17. Operational Rules

### Do Not Treat Tickets as Universal Credentials

A ticket has a specific principal, service, realm, and validity context.

### Check Time and DNS

Kerberos failures often involve environmental dependencies rather than simply bad credentials.

### Separate Authentication From Authorization

A valid ticket does not automatically provide administrative access.

### Minimize Exposure

Treat ticket material as sensitive authentication material.

### Re-enumerate

Every successful movement creates a new security and network position.

## 18. Decision Workflow

Use this workflow when Kerberos ticket material is available:

```text
Kerberos Ticket Available
        ↓
Identify Principal
        ↓
Identify Ticket Type
        ↓
Identify Realm / Domain
        ↓
Identify SPN / Service
        ↓
Check Ticket Validity
        ↓
Check DNS / Naming
        ↓
Check Target Reachability
        ↓
Validate Authentication
        │
        ├── Failed
        │     ↓
        │  Check Network / DNS / Time /
        │  Realm / SPN / Ticket
        │
        └── Successful
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

## What Not to Assume

```text
Ticket Found
   ≠
Access to Every Host
```

```text
TGT
   ≠
Service Ticket
```

```text
Valid Ticket
   ≠
Authorization
```

```text
Correct User
   ≠
Correct Service
```

```text
Authentication Failure
   ≠
Invalid Ticket
```

Investigate the complete authentication path before drawing conclusions.

## What Next?

Continue to [`Overpass-the-Hash.md`](Overpass-the-Hash.md).

That workflow covers the transition from NTLM authentication material to Kerberos-based authentication in Active Directory environments.
