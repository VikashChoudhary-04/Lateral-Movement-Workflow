# Overpass-the-Hash

Overpass-the-Hash (OPtH) is a Windows/Active Directory authentication technique in which NTLM hash material is used as the starting authentication material for obtaining or establishing a Kerberos authentication context.

The important distinction is:

```text
Pass-the-Hash
NTLM hash → NTLM-based authentication

Overpass-the-Hash
NTLM hash → Kerberos authentication context → Kerberos service
```

The workflow is therefore:

```text
NTLM Hash Available
        ↓
Identify Associated Account
        ↓
Determine Domain Context
        ↓
Establish Authentication Context
        ↓
Obtain / Use Kerberos Authentication
        ↓
Identify Target Service
        ↓
Check Target Reachability
        ↓
Validate Kerberos Authentication
        ↓
Validate Authorization
        ↓
Establish Authorized Access
        ↓
Validate New Position
        ↓
Re-enumerate
```

This workflow is intended for authorized assessments and controlled labs.

## Objective

The objective is to understand whether NTLM authentication material associated with a domain identity can be used to establish a Kerberos authentication context and provide authorized access to another service or host.

The key questions are:

* What account does the hash belong to?
* Is it a domain account?
* Is the account's domain context known?
* Can the authentication material establish the required Kerberos context?
* Which Kerberos service is the intended target?
* Is the target reachable?
* Is the resulting authentication accepted?
* Is the account authorized?
* What new position is obtained?

## Core Concept

The technique bridges two authentication contexts:

```text
NTLM Authentication Material
          ↓
   Domain Identity
          ↓
Kerberos Authentication Context
          ↓
   Service Ticket
          ↓
 Target Kerberos Service
```

This differs from directly using the NTLM hash against an NTLM-compatible service.

Conceptually:

```text
Pass-the-Hash
Hash → NTLM → Target Service
```

versus:

```text
Overpass-the-Hash
Hash → Kerberos Context → Service Ticket → Target Service
```

The exact implementation depends on the Windows and Active Directory environment.

## 1. Confirm the Authentication Material

First determine that the available material is an NTLM hash associated with an identity.

Record:

```text
Hash Type:
Associated Username:
Domain:
Source:
Account Type:
```

Do not assume that every hash-like value is an NTLM credential.

The identity associated with the material is just as important as the material itself.

## 2. Confirm the Account Context

Overpass-the-Hash is primarily relevant when the identity participates in an Active Directory/Kerberos environment.

Establish:

```text
Username:
Domain:
Domain Controller:
Kerberos Realm:
```

On Windows:

```cmd id="e6plg4"
whoami
echo %USERDOMAIN%
echo %USERDNSDOMAIN%
```

Domain-controller discovery can be performed with:

```cmd id="tq2w7r"
nltest /dsgetdc:DOMAIN
```

The purpose is to establish the authentication authority before attempting a Kerberos workflow.

## 3. Understand the Authentication Transition

The central idea is:

```text
NTLM Hash
    ↓
Authenticate as Domain Identity
    ↓
Kerberos Authentication Context
    ↓
Request / Use Service Ticket
```

This does **not** mean that the NTLM hash itself is a Kerberos ticket.

Instead, the hash is used as authentication material in a workflow that establishes Kerberos credentials for the associated identity.

Therefore:

```text
NTLM Hash
   ≠
Kerberos Ticket
```

and:

```text
NTLM Hash
   ↓
Kerberos Authentication Context
```

is the important transition.

## 4. Verify Domain Infrastructure

Kerberos depends on Active Directory infrastructure.

Check:

* domain name
* DNS
* domain-controller availability
* hostname resolution
* system time
* Kerberos configuration

Useful Windows checks include:

```cmd id="xw3slb"
nltest /dsgetdc:DOMAIN
```

and:

```cmd id="r0m9zq"
nslookup DOMAIN
```

Check the current Kerberos cache:

```cmd id="w9h2qx"
klist
```

The exact ticket state depends on the current logon and authentication context.

## 5. Check Time Synchronization

Kerberos is sensitive to clock differences.

If the authentication workflow fails unexpectedly, verify system time.

On Windows:

```cmd id="bq1x5k"
w32tm /query /status
```

The important reasoning is:

```text
Kerberos Failure
      ↓
Check Time
      ↓
Check DNS
      ↓
Check Domain / Realm
      ↓
Check Account
```

Do not immediately conclude that the hash is invalid.

## 6. Identify the Target Service

Once a Kerberos authentication context is available, determine which service you are trying to access.

Common examples include:

```text
SMB / CIFS
HTTP
LDAP
HOST
```

For example:

```text id="9n9v8b"
cifs/server01.domain.local
```

indicates an SMB/CIFS service.

The decision becomes:

```text
Kerberos Identity
       ↓
Target Service
       ↓
SPN
       ↓
Target Host
```

## 7. Identify the SPN

The Service Principal Name connects the Kerberos service identity to the target service.

Examples:

```text
cifs/server01.domain.local
HTTP/web01.domain.local
LDAP/dc01.domain.local
```

The service ticket must correspond to the intended service.

Do not assume:

```text
cifs/server01
```

and:

```text
HTTP/server01
```

are interchangeable.

They represent different service principals.

## 8. Check Target Reachability

Before troubleshooting Kerberos, verify the network path.

For SMB:

```powershell id="w7i8h3"
Test-NetConnection SERVER01 -Port 445
```

For LDAP:

```powershell id="o7l8m0"
Test-NetConnection DC01 -Port 389
```

For HTTPS:

```powershell id="5x3q6d"
Test-NetConnection SERVER01 -Port 443
```

A network failure should be handled separately from an authentication failure.

## 9. Establish the Kerberos Context

The exact mechanism depends on the assessment environment and the authorized tooling being used.

The conceptual process is:

```text
NTLM Hash
     ↓
Associated Domain Identity
     ↓
Kerberos Authentication
     ↓
Ticket-Granting Context
     ↓
Service Ticket
```

After establishing the context, inspect the ticket cache:

```cmd id="k9z3c5"
klist
```

Look for evidence that the expected Kerberos identity and relevant tickets are present.

Do not treat the presence of any ticket as proof that the desired service is accessible.

## 10. Validate the Service Ticket

Once the Kerberos context is established, identify whether the required service ticket exists.

Conceptually:

```text
TGT
 ↓
Service Request
 ↓
Service Ticket
 ↓
Target Service
```

For an SMB target:

```text
TGT
 ↓
CIFS Service Ticket
 ↓
SERVER01
```

The ticket should correspond to the service you are attempting to access.

## 11. Validate SMB / CIFS Access

If the target is SMB:

```powershell id="3j8y4v"
Test-NetConnection SERVER01 -Port 445
```

Then validate access using an authorized Kerberos-capable SMB workflow.

Interpret the result in layers:

```text
Network Reachability
        ↓
SMB Available
        ↓
Kerberos Authentication
        ↓
Account Authorization
        ↓
Share Access
```

A successful Kerberos authentication does not automatically imply administrative access.

## 12. Authentication vs Authorization

As with every lateral-movement workflow, separate these questions.

### Authentication

```text
Kerberos Context
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
SMB Access              ✓
Administrative Share    ✗
```

This indicates an authorization limitation.

## 13. Validate the New Position

If movement succeeds, establish the resulting position.

On Windows:

```cmd id="6s8v8q"
whoami
hostname
ipconfig
route print
```

Then inspect Kerberos state:

```cmd id="d8m4pb"
klist
```

And identity/privilege context:

```cmd id="w9v7d3"
whoami /groups
whoami /priv
```

The purpose is to answer:

```text
Who am I?
Where am I?
Which domain am I using?
What privileges are available?
What networks can I reach?
What Kerberos tickets are present?
```

## 14. Re-enumerate the New Host

Do not continue from the old host's assumptions.

Start the position-assessment workflow again:

```text
New Host
   ↓
Current User
   ↓
Network Context
   ↓
Domain / Trust Context
   ↓
Services
   ↓
Credentials / Access
   ↓
Target Discovery
```

The new host may expose:

* additional subnets
* different servers
* domain controllers
* different service accounts
* different Kerberos service relationships
* additional authentication material

## 15. Failure Handling

### Hash Is Invalid or Incorrectly Associated

Possible causes:

* incorrect NTLM value
* wrong username
* incorrect domain
* wrong account context
* corrupted evidence

Verify the account and authentication material before proceeding.

### Domain Controller Cannot Be Reached

Possible causes:

* network segmentation
* DNS problems
* firewalling
* incorrect domain configuration
* unavailable domain controller

The Kerberos workflow depends on domain infrastructure.

### DNS Resolution Fails

Check:

```cmd id="0u0x1n"
nslookup DOMAIN
nslookup SERVER01.domain.local
```

Kerberos service naming depends heavily on correct DNS and host resolution.

### Time Synchronization Problem

Check:

```cmd id="j4k7o2"
w32tm /query /status
```

A significant time mismatch can prevent Kerberos authentication.

### Kerberos Authentication Fails

Check:

```text id="f3x8i4"
Account
 ↓
Domain
 ↓
DNS
 ↓
Time
 ↓
Realm
 ↓
SPN
 ↓
Ticket
```

Troubleshoot one layer at a time.

### Authentication Succeeds but Access Is Denied

Interpret as:

```text
Authentication ✓
Authorization ✗
```

The account may be valid without having the required rights.

## 16. Overpass-the-Hash vs Pass-the-Hash

The distinction should remain clear.

| Aspect                      | Pass-the-Hash                    | Overpass-the-Hash                         |
| --------------------------- | -------------------------------- | ----------------------------------------- |
| Starting material           | NTLM hash                        | NTLM hash                                 |
| Primary authentication path | NTLM                             | Kerberos                                  |
| Typical environment         | Windows                          | Active Directory                          |
| Goal                        | Authenticate using NTLM material | Establish Kerberos authentication context |
| Service dependency          | NTLM-compatible service          | Kerberos-compatible service               |
| Ticket context              | Not the primary objective        | Central to the workflow                   |

The same hash can therefore lead to different workflows depending on the target service and authentication requirements.

## 17. Overpass-the-Hash Decision Tree

Use this workflow:

```text
NTLM Hash Available
        ↓
Identify Account
        ↓
Domain Account?
        │
   ┌────┴────┐
  No        Yes
  ↓           ↓
Consider      Check AD / Kerberos
other paths        ↓
              Domain Controller
                    ↓
                 DNS / Time
                    ↓
            Establish Kerberos Context
                    ↓
                 klist
                    ↓
             Identify Service
                    ↓
                Identify SPN
                    ↓
             Check Reachability
                    ↓
          Validate Authentication
                    │
              ┌─────┴─────┐
             Fail        Success
              ↓             ↓
       Troubleshoot      Validate
       auth context      authorization
                            ↓
                       Establish Access
                            ↓
                       Validate Position
                            ↓
                       Re-enumerate
```

## 18. Evidence Recording

Record the authentication chain without unnecessarily storing the actual hash or ticket material.

```text
Source Host:
Associated Account:
Domain:
Authentication Material:
Domain Controller:
Kerberos Realm:
Target Host:
Target Service:
SPN:
Ticket Context:
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
Starting Material: NTLM Hash
Domain: DOMAIN.LOCAL
Target: SERVER-01
Service: SMB / CIFS
Kerberos Context: Established
Authentication: Successful
Authorization: Administrative share access confirmed
New Position: SERVER-01
Next Step: Re-enumerate SERVER-01
```

## 19. Operational Rules

### Confirm Scope

Only use the technique against authorized systems and identities.

### Protect Authentication Material

NTLM hashes and Kerberos tickets are sensitive authentication material.

### Avoid Broad Testing

Select targets based on evidence rather than attempting authentication against every discovered host.

### Keep the Authentication Path Clear

Always know whether the workflow is:

```text
NTLM → NTLM
```

or:

```text
NTLM → Kerberos
```

### Validate Every Layer

Check:

```text
Network
 ↓
Service
 ↓
Authentication
 ↓
Authorization
 ↓
Access
```

### Re-enumerate

A successful movement creates a new position and potentially exposes a new part of the environment.

## What Not to Assume

```text
NTLM Hash
   ≠
Kerberos Ticket
```

```text
Kerberos Context
   ≠
Administrative Access
```

```text
TGT
   ≠
Universal Service Access
```

```text
Correct Account
   ≠
Correct SPN
```

```text
Authentication Failure
   ≠
Invalid Hash
```

Investigate the entire authentication chain before deciding where the failure occurred.

## What Next?

This completes `08-Credential-Based-Movement`.

The next section is `09-Pivoting-and-Internal-Network-Access`, beginning with:

`09-Pivoting-and-Internal-Network-Access/README.md`

That section moves from credential-enabled host access to situations where the newly compromised host provides access to otherwise unreachable internal network resources.
