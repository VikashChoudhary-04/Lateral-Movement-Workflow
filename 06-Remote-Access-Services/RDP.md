# RDP

Remote Desktop Protocol (RDP) provides interactive graphical access to Windows systems.

For lateral movement, RDP is important because it can turn valid Windows authentication and appropriate remote-logon authorization into an interactive session on another Windows host.

The practical question is not:

> "Is port 3389 open?"

It is:

> **Can the current identity establish an authorized interactive RDP session to the target, and does that session provide useful access for the next stage?**

The workflow is:

```text id="q6r2m8"
Candidate Windows Target
        ↓
Identify RDP
        ↓
Check Reachability
        ↓
Identify Authentication
        ↓
Identify Account
        ↓
Determine Remote-Logon Authorization
        ↓
Validate Authentication
        ↓
Establish Interactive Session
        ↓
Validate New Position
        ↓
Continue Movement
```

---

## 1. Where RDP Fits

RDP is one of the remote-access services covered by Section 06:

```text id="f8m3x7"
Remote Access Services
      ↓
SMB
WinRM
RDP
WMI
RPC
SSH
```

RDP is primarily relevant to:

```text id="w4p9k2"
Windows
Active Directory
Administrative Workstations
Remote Desktop Servers
Jump Hosts
Management Systems
```

---

## 2. RDP Port

The standard RDP port is:

```text id="m7c2x9"
3389/TCP
```

However:

```text id="p5r8n1"
3389 Open
    ≠
RDP Confirmed
```

and:

```text id="j3k6q4"
RDP Confirmed
    ≠
RDP Authentication
```

and:

```text id="v9m2c7"
RDP Authentication
    ≠
Remote Desktop Authorization
```

Each stage must be validated.

---

## 3. The RDP Movement Model

The basic movement relationship is:

```text id="x6q1m8"
Identity
   ↓
Authentication
   ↓
RDP
   ↓
Remote Desktop Authorization
   ↓
Interactive Session
   ↓
Target
```

The result is normally an interactive graphical session rather than a command-line management session.

---

## 4. Identify the Target

Before testing RDP, establish:

```text id="r4n8p2"
Target Host
IP Address
Hostname
Domain
Network Location
```

For example:

```text id="c7m3x9"
Target:
WORKSTATION02.CORP.LOCAL

IP:
10.10.20.25
```

The target should come from evidence collected during:

```text id="q5p1v7"
Target Discovery
Network Enumeration
Service Discovery
Kerberos
Existing Sessions
Previous Movement
```

---

## 5. Check RDP Reachability

From Windows:

```powershell id="m2r8x4"
Test-NetConnection WORKSTATION02 -Port 3389
```

From Linux, where appropriate:

```bash id="p7c3n9"
nc -vz WORKSTATION02 3389
```

This establishes network-level connectivity.

It does not establish:

```text id="k6m1q8"
RDP Authentication
```

or:

```text id="w3x9r5"
Remote Desktop Authorization
```

---

## 6. Confirm the Service

An authorized service-discovery scan can help confirm RDP:

```bash id="z8m4p2"
nmap -p 3389 -sV WORKSTATION02
```

The objective is:

```text id="v6q2c8"
Target
  ↓
3389/TCP
  ↓
RDP Service
```

If the service is unavailable, investigate the network or service layer before changing credentials.

---

## 7. Identify the Current Identity

Before testing RDP, establish your current identity.

On Windows:

```cmd id="n5x8m2"
whoami
whoami /user
whoami /groups
```

For domain context:

```cmd id="r3q7k1"
echo %USERDOMAIN%
echo %USERDNSDOMAIN%
```

The objective is to establish:

```text id="c8m4p6"
Identity
Account Authority
Group Context
Authentication Context
```

---

## 8. RDP Authentication

RDP authentication can operate with Windows authentication mechanisms depending on configuration.

Relevant concepts include:

```text id="x7m2n9"
Domain Authentication
Local Account Authentication
Kerberos
NTLM
Network Level Authentication
```

The actual mechanism depends on:

```text id="q4r8c1"
Target Configuration
Account
Domain
Security Policy
Client / Server Capabilities
```

Therefore:

```text id="p6m3x9"
Credential
    ≠
RDP Access
```

---

## 9. Network Level Authentication

Network Level Authentication (NLA) allows authentication to occur before the full remote desktop session is established.

Conceptually:

```text id="m8c2r5"
Client
  ↓
Authentication
  ↓
NLA
  ↓
Remote Desktop Session
```

This changes the connection workflow but does not remove the need for proper authorization.

A successful network-level authentication does not automatically mean:

```text id="j4p9x6"
Administrative Access
```

---

## 10. RDP Authorization

The account must be permitted to log on through Remote Desktop Services.

Conceptually:

```text id="w6m3q8"
Identity
   ↓
Authentication
   ↓
Remote Desktop Logon Authorization
   ↓
Interactive Session
```

Authorization may depend on:

```text id="v2r7n4"
Local Groups
Domain Groups
Local Security Policy
Group Policy
Account Restrictions
Session Configuration
```

Always record the actual access result.

---

## 11. Remote Desktop Users

Windows systems commonly use the:

```text id="f8q2m5"
Remote Desktop Users
```

group to provide appropriate remote-logon rights to non-administrative users.

However:

```text id="a7n3x9"
Member of Remote Desktop Users
    ≠
Administrator
```

The group primarily relates to remote desktop authorization.

---

## 12. Administrator RDP Access

An authorized administrator may also have RDP access.

The important distinction is:

```text id="x5m8q2"
RDP Authorization
```

versus:

```text id="c3p7n9"
Administrative Privileges
```

A user can have:

```text id="q6r2m4"
RDP Access
```

without:

```text id="v8m3x1"
Local Administrator Rights
```

---

## 13. RDP and Domain Accounts

A domain account may be able to log in to multiple Windows systems.

Do not assume:

```text id="m4p9c7"
Domain User
    =
RDP Access Everywhere
```

Each target can have different:

```text id="x8n2r5"
Group Membership
Local Policy
GPO
Account Restrictions
RDP Configuration
```

Validate each target.

---

## 14. RDP and Local Accounts

Local accounts can also be used for RDP where policy permits.

For example:

```text id="k7m3p1"
WORKSTATION02\admin
```

is different from:

```text id="r4x8n6"
CORP\admin
```

The account authority matters.

Record the complete identity.

---

## 15. Kerberos and RDP

In an AD environment, Kerberos may participate in authentication.

Conceptually:

```text id="n6p2x8"
Domain Identity
      ↓
Kerberos
      ↓
RDP
      ↓
Windows Target
```

Authentication can depend on:

```text id="c5m9r2"
Correct Hostname
DNS
Domain Connectivity
Time Synchronization
Authentication Policy
```

If Kerberos-related RDP authentication behaves unexpectedly, investigate the environment rather than assuming the account is invalid.

---

## 16. Existing RDP Sessions

Before establishing a new RDP session, inspect whether sessions already exist.

On Windows:

```cmd id="q8m4x2"
query user
```

or:

```cmd id="v3p7n9"
qwinsta
```

This can reveal:

```text id="r6m2c8"
User
Session ID
Session State
Logon Time
```

Existing sessions can provide useful environmental evidence.

---

## 17. Existing Session vs New RDP Access

Distinguish:

```text id="j9p3m6"
Existing RDP Session
```

from:

```text id="x4c8r2"
Potential New RDP Login
```

An existing session may reveal:

```text id="n7m1q5"
Who uses the system
Administrative Activity
Management Workflows
Current System Usage
```

But it does not automatically provide reusable authentication.

---

## 18. RDP Target Prioritization

An RDP target may be particularly relevant if it is:

```text id="p8x2m4"
Administrative Workstation
Jump Host
Management Server
High-Value Server
User Workstation
```

However, target priority should be based on evidence from the current assessment.

Consider:

```text id="q5m9c3"
Reachability
Authentication Compatibility
Authorization
Host Role
Movement Objective
Information Gain
```

---

## 19. RDP and Jump Hosts

A jump host may sit between:

```text id="k6r3m8"
User Network
      ↓
Jump Host
      ↓
Restricted Network
```

If RDP access to the jump host is authorized, it may change the network position.

After accessing the jump host:

```text id="v4m8p1"
Re-enumerate Network
```

because the new host may have access to networks that the original system could not reach.

Pivoting is covered later in:

```text id="j2n7c5"
09-Pivoting-and-Internal-Network-Access/
```

---

## 20. RDP and Network Segmentation

RDP may be allowed only from specific networks.

For example:

```text id="m8p2x6"
Workstation
   ↓
RDP
   ↓
Management Server
```

while:

```text id="c5r9n3"
Internet
   ↓
RDP
   ↓
Management Server
```

may be blocked.

Therefore, a failed RDP connection from one host does not necessarily prove that RDP is unavailable everywhere.

The current network position matters.

---

## 21. RDP Session Validation

After successful RDP authentication, establish:

```text id="w7m3q8"
Who am I?
What host am I on?
What privileges do I have?
What domain am I in?
What networks can I reach?
What services are available?
```

Use:

```cmd id="p4n8x2"
whoami
hostname
ipconfig /all
```

Then continue with the normal position-assessment workflow.

---

## 22. RDP as a New Position

A successful RDP session is not merely:

```text id="f2m7c9"
"Login succeeded."
```

It is:

```text id="r8p3x5"
New Host
+
New Interactive Session
+
New Network Position
+
Potential New Credentials
+
Potential New Targets
```

Therefore:

```text id="v6n2m8"
RDP Success
   ↓
Position Assessment
```

---

## 23. RDP Failure Classification

When RDP fails, classify the failure.

### Network Failure

```text id="m3x7p1"
Timeout
No Route
Filtered Port
Connection Refused
```

### Service Failure

```text id="q8r2n5"
RDP Disabled
No Listener
Wrong Port
Service Unavailable
```

### Authentication Failure

```text id="c6m9x3"
Invalid Credential
Unsupported Authentication
Kerberos Failure
NLA Failure
Account Problem
```

### Authorization Failure

```text id="w4p7m2"
Authentication Successful
but Remote Desktop Logon Denied
```

### Policy Failure

```text id="n5x8c1"
Group Policy
Account Restrictions
Logon Rights
Session Restrictions
```

---

## 24. Troubleshooting Workflow

If:

```text id="r2m7q5"
Test-NetConnection
```

fails:

```text id="j8p3n6"
Investigate Network
```

If the port is reachable but RDP does not respond correctly:

```text id="v4c9m2"
Investigate RDP Service / Listener
```

If RDP responds but authentication fails:

```text id="x7n1p8"
Investigate Authentication
```

If authentication succeeds but logon is denied:

```text id="m5q8r3"
Investigate Authorization / Policy
```

If the session succeeds:

```text id="c2p6x9"
Validate New Position
```

---

## 25. RDP Authentication vs Authorization

Always record the two separately.

Example:

```text id="k3m8p1"
Authentication:
Successful

Authorization:
Denied
```

means:

```text id="w7q2n5"
The identity was recognized,
but the target did not permit the required RDP logon.
```

This is different from:

```text id="r4c9m6"
Authentication:
Failed
```

where the failure occurred earlier.

---

## 26. RDP Access Levels

Record the actual result:

```text id="p8m3x7"
No Access
Authentication Rejected
Remote Logon Denied
User-Level Interactive Session
Administrative Interactive Session
```

Do not use vague terms such as:

```text id="q5n9c2"
"RDP access"
```

without specifying what was confirmed.

---

## 27. RDP and Credential Matching

Use the credential-to-service model:

| Authentication Context         | RDP Compatibility       | Notes                           |
| ------------------------------ | ----------------------- | ------------------------------- |
| Domain account                 | Possible                | Depends on target authorization |
| Local Windows account          | Possible                | Depends on local policy         |
| Kerberos context               | Possible                | Common in AD environments       |
| NTLM-compatible authentication | Configuration-dependent | Depends on policy               |
| SSH private key                | No                      | Not an RDP credential           |

This describes compatibility, not guaranteed access.

---

## 28. Practical Windows Workflow

From a Windows foothold:

```text id="m8q3x5"
1. Identify current identity
2. Identify domain context
3. Identify candidate target
4. Test TCP/3389
5. Confirm RDP
6. Determine authentication context
7. Determine RDP authorization
8. Validate authorized authentication
9. Establish the RDP session
10. Validate the new position
11. Enumerate network context
12. Discover credentials / existing access
13. Identify next targets
14. Continue the movement chain
```

Useful commands include:

```powershell id="c4m7p2"
Test-NetConnection WORKSTATION02 -Port 3389
```

and:

```cmd id="n8x3q6"
query user
```

---

## 29. Practical Linux Assessment Workflow

A Linux system may be used as the assessment source when testing an authorized Windows RDP target.

The workflow remains:

```text id="r5m2x8"
1. Identify target
2. Verify network reachability
3. Confirm RDP
4. Identify authorized Windows identity
5. Determine authentication method
6. Establish authorized RDP session
7. Validate identity
8. Validate privileges
9. Enumerate new position
10. Continue lateral movement
```

The important point is that the source operating system does not change the target's authorization requirements.

---

## 30. Example: RDP User Access

Suppose:

```text id="q7m3n5"
Target:
WORKSTATION02

Identity:
CORP\alice

RDP:
Reachable

Authentication:
Successful

Authorization:
Remote Desktop User

Result:
Interactive Session
```

The correct interpretation is:

```text id="v2x8m1"
Authorized Interactive Access
```

It does not automatically mean:

```text id="k5p9r3"
Administrator
```

The new position must be assessed.

---

## 31. Example: Authentication Success, Authorization Failure

Suppose:

```text id="c8m4q7"
Identity:
CORP\bob

Authentication:
Successful

RDP Logon:
Denied
```

The next investigation should focus on:

```text id="n6p2x5"
Remote Desktop Logon Rights
Group Membership
Local Policy
Group Policy
Account Restrictions
```

Changing the password would not necessarily solve the problem.

---

## 32. Example: RDP Reveals a New Network

Suppose:

```text id="w4m8p2"
Initial Host
   ↓
RDP
   ↓
Jump Host
```

After login:

```text id="r7c3n9"
ipconfig /all
route print
```

reveals:

```text id="m5p1x8"
10.10.30.0/24
```

which was not reachable from the original host.

This changes the movement map:

```text id="q8n2m6"
Initial Host
      ↓
Jump Host
      ↓
New Network
      ↓
New Targets
```

The next step is target discovery from the new position.

---

## 33. RDP and Post-Movement Enumeration

After a successful RDP session:

```cmd id="p3m7x9"
whoami
hostname
ipconfig /all
route print
```

Then investigate:

```text id="k6r2n8"
Domain Context
Current Sessions
Network Connections
Services
Credentials
SSH Keys
Existing Access
Remote Services
```

Return to:

```text id="v4m9x1"
03-Position-Assessment/
```

and repeat the workflow.

---

## 34. Common Mistakes

### Mistake 1: Treating Port 3389 as Proof of RDP

Confirm the actual service.

### Mistake 2: Treating Authentication as Authorization

A valid account can still be denied RDP logon.

### Mistake 3: Assuming Domain Users Can RDP Everywhere

RDP authorization is target-specific.

### Mistake 4: Assuming RDP Means Administrator

A normal authorized user can have an interactive session without administrative privileges.

### Mistake 5: Ignoring NLA

NLA changes when authentication occurs and can affect troubleshooting.

### Mistake 6: Ignoring DNS and Domain Context

Kerberos-related authentication can depend on correct host and domain information.

### Mistake 7: Ignoring Network Position

A jump host may expose networks unavailable from the original system.

### Mistake 8: Skipping Re-enumeration

A successful RDP session creates a new position.

---

## 35. RDP Investigation Record

Use this structure:

```text id="x7m2p5"
Target:
____________________

Hostname:
____________________

IP:
____________________

Port:
____________________

RDP Confirmed:
____________________

Current Identity:
____________________

Account Authority:
____________________

Authentication:
____________________

NLA:
____________________

RDP Authorization:
____________________

Session Result:
____________________

Access Level:
____________________

New Network Context:
____________________

New Position:
____________________

Failure Reason:
____________________

Next Action:
____________________
```

---

## 36. RDP Decision Tree

```text id="w4m8q2"
Candidate Target
      │
      ▼
Is TCP/3389 Reachable?
      │
 ┌────┴────┐
No        Yes
 │          │
 ▼          ▼
Check     Confirm
Network   RDP
             │
             ▼
       Identify Identity
             │
             ▼
       Authentication Compatible?
          │          │
         No         Yes
          │          │
          ▼          ▼
     Continue      Validate
     Other Path    Authentication
                        │
                        ▼
                 RDP Authorization?
                    │         │
                   No        Yes
                    │         │
                    ▼         ▼
                Record /   Establish
                Reassess   Session
                              │
                              ▼
                       Validate Position
                              │
                              ▼
                       Re-enumerate
                              │
                              ▼
                       Continue Chain
```

---

## 37. Completion Criteria

RDP investigation is sufficiently complete when you can answer:

```text id="m2q8x5"
[ ] Is the target reachable?
[ ] Is RDP actually available?
[ ] Which port / listener is being used?
[ ] Which identity is being used?
[ ] Which authentication mechanism is relevant?
[ ] Is NLA involved?
[ ] Is the identity authorized for RDP?
[ ] Did authentication succeed?
[ ] Did the RDP logon succeed?
[ ] What privilege level was obtained?
[ ] What network position was established?
[ ] What caused any failure?
[ ] What new targets are visible?
[ ] Has the new position been re-enumerated?
[ ] What is the next movement path?
```

The objective is not:

```text id="v8p3m1"
Connect to every RDP server.
```

It is:

```text id="q6m2r8"
Determine whether a known identity and authentication context
can establish useful, authorized interactive access to a Windows target.
```

---

## 38. What Next?

Continue with:

```text id="n5x8q2"
06-Remote-Access-Services/
└── WMI.md
```

The workflow changes from:

```text id="j4m7p9"
Interactive Remote Desktop
```

to:

```text id="c8r2x5"
Remote Windows Management
      ↓
WMI
      ↓
RPC
      ↓
Authorization
      ↓
Management Action
```

The core principle is:

> **RDP is a lateral-movement path when a reachable Windows target, compatible authentication context, and appropriate Remote Desktop authorization combine to create a validated interactive session. A successful login creates a new position that must immediately be re-enumerated.**
