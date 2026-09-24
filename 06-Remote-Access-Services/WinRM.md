# WinRM

Windows Remote Management (WinRM) is Microsoft's implementation of the WS-Management protocol for remotely managing Windows systems.

For lateral movement, the practical question is not simply:

> "Is WinRM running?"

The useful question is:

> **Can an identity with an available authentication context establish authorized remote management access to this Windows target through WinRM?**

The workflow is:

```text id="f3p7k2"
Candidate Windows Target
        ↓
Identify WinRM
        ↓
Check Reachability
        ↓
Identify Authentication Method
        ↓
Identify Account
        ↓
Determine Remote-Management Authorization
        ↓
Validate Authentication
        ↓
Validate Remote Access
        ↓
Enumerate New Position
        ↓
Continue Movement
```

---

## 1. Where WinRM Fits

WinRM is one of the remote-access paths in Section 06:

```text id="x8m4q1"
Remote Access Services
      ↓
SMB
WinRM
RDP
WMI
RPC
SSH
```

WinRM is primarily relevant to:

```text id="r6n2v9"
Windows
Active Directory
Windows Administration
Remote PowerShell
Remote Management
```

It is particularly useful when the current identity already has:

```text id="p4k7m1"
Valid Windows Credentials
Kerberos Authentication Context
NTLM-Compatible Authentication
Authorized Remote-Management Rights
```

---

## 2. WinRM Ports

Common WinRM listeners use:

```text id="w3c8x5"
5985/TCP
```

for HTTP and:

```text id="m7q2n9"
5986/TCP
```

for HTTPS.

These are defaults, not guarantees.

A target may:

```text id="j5r1v8"
Use a non-standard configuration
Expose only HTTPS
Restrict listeners
Filter the port
Have WinRM disabled
```

Therefore:

```text id="a9p3x6"
Port
   ↓
WinRM Listener
   ↓
Authentication
   ↓
Authorization
```

must be verified.

---

## 3. WinRM vs PowerShell Remoting

These concepts are closely related but should not be treated as identical.

WinRM provides the remote management transport.

PowerShell remoting can use WinRM to establish remote PowerShell sessions.

Conceptually:

```text id="k8m4r2"
PowerShell Remoting
        ↓
WinRM
        ↓
Windows Target
```

Therefore a successful PowerShell remoting session generally requires the underlying WinRM path and appropriate authorization.

---

## 4. First Establish the Target

Before testing WinRM, identify:

```text id="v6n2p8"
Target Host
Target IP
Domain
Network Location
```

For example:

```text id="c3m7x1"
Target:
SERVER01.CORP.LOCAL

IP:
10.10.20.15
```

The target should come from previous evidence rather than blind scanning.

---

## 5. Check Reachability

Before investigating authentication, establish that the WinRM endpoint is reachable.

Conceptually:

```text id="q5r8m2"
Current Host
      ↓
Target
      ↓
5985 / 5986
      ↓
Reachable?
```

On Windows, a useful test is:

```powershell id="t7n3k9"
Test-NetConnection SERVER01 -Port 5985
```

For HTTPS:

```powershell id="p2m6x4"
Test-NetConnection SERVER01 -Port 5986
```

The result tells you about network connectivity to the port.

It does **not** prove:

```text id="e8c1v7"
WinRM Authentication
```

or:

```text id="n4q9m2"
Remote Authorization
```

---

## 6. Identify the WinRM Listener

On a Windows system where you have authorized local access, inspect the WinRM configuration.

A useful command is:

```powershell id="z6x3p8"
winrm enumerate winrm/config/listener
```

This can help identify:

```text id="m2r7k4"
Transport
Listening Address
Port
HTTPS / HTTP
Certificate Configuration
```

The objective is to understand how the target exposes WinRM.

---

## 7. Check the WinRM Service

On Windows:

```powershell id="c5n8v2"
Get-Service WinRM
```

Possible states include:

```text id="q3m7x9"
Running
Stopped
```

A stopped service generally means the local WinRM service is not currently providing its normal management functionality.

However, remote availability should still be assessed from the source host.

---

## 8. WinRM Requires More Than an Open Port

Consider:

```text id="j8p4r1"
5985 Open
```

This tells you:

```text id="v5m9x2"
Something is reachable on TCP/5985.
```

It does not automatically establish:

```text id="q7c3n6"
WinRM
```

and even confirmed WinRM does not establish:

```text id="x2m8r5"
Authorized Remote Management
```

The evidence chain is:

```text id="a6p1k9"
Port
 ↓
WinRM Listener
 ↓
Authentication
 ↓
Authorization
 ↓
Remote Session
```

---

## 9. Identify the Current Identity

Before choosing an authentication path:

```cmd id="n4x8m2"
whoami
```

Then:

```cmd id="r7p3k5"
whoami /user
whoami /groups
```

For domain context:

```cmd id="v9m1q6"
echo %USERDOMAIN%
echo %USERDNSDOMAIN%
```

The objective is to determine:

```text id="w2k8c4"
Local Account
or
Domain Account
```

and:

```text id="j5p9n3"
Current Authentication Context
```

---

## 10. Authentication Mechanisms

WinRM can operate with different authentication mechanisms depending on configuration.

Common Windows environments may involve:

```text id="f6m2r8"
Kerberos
NTLM
Certificate-Based Authentication
Other Configured Mechanisms
```

The relevant question is:

```text id="c8q3v5"
Which authentication mechanism can this identity
actually use against this target?
```

Do not assume that a credential compatible with one Windows service is automatically accepted by WinRM.

---

## 11. Kerberos and WinRM

In a domain environment, Kerberos can provide a natural authentication path.

Conceptually:

```text id="m7x2p4"
Domain Identity
      ↓
Kerberos
      ↓
WinRM
      ↓
Windows Target
```

The target must be:

```text id="r5n8c1"
Reachable
Kerberos-Compatible
Correctly Named
Properly Configured
```

DNS and time synchronization can therefore matter.

---

## 12. NTLM and WinRM

Depending on configuration, WinRM may also use NTLM authentication.

The conceptual path is:

```text id="q4m7x2"
Windows Identity
      ↓
NTLM Authentication
      ↓
WinRM
      ↓
Target
```

Whether this is accepted depends on:

```text id="v8p3n6"
Target Configuration
Authentication Policy
Account Restrictions
Network Configuration
```

Do not assume that NTLM-related material automatically provides WinRM access.

---

## 13. WinRM Authorization

Authentication alone is not enough.

The target must authorize the account for the required remote-management operation.

Conceptually:

```text id="x6m2r9"
Identity
   ↓
Authentication
   ↓
Remote Management Authorization
   ↓
WinRM Session
```

A valid domain account may still receive:

```text id="z3p7k5"
Access Denied
```

if the account lacks the necessary remote-management rights.

---

## 14. Remote Management Groups

Windows environments may use groups and local policy to control remote management access.

For example, an account may have permissions through:

```text id="j9r4m1"
Administrators
Remote Management Users
Configured Delegation / Policy
```

The exact authorization model depends on the target configuration.

Therefore, record the actual result rather than assuming group membership guarantees access.

---

## 15. Test the WinRM Path

Where you are authorized to test remote management, PowerShell provides a native workflow.

For example:

```powershell id="w5k8p2"
Test-WSMan SERVER01
```

This helps determine whether the WS-Management endpoint responds.

The result primarily validates:

```text id="m3q7x9"
WinRM / WS-Man Availability
```

It does not by itself prove:

```text id="r8c2n6"
Successful User Authentication
```

or:

```text id="v4p1m7"
Administrative Authorization
```

---

## 16. Test Remote PowerShell Access

Where the account is authorized, a PowerShell remoting session can be tested using:

```powershell id="c9m4x2"
Enter-PSSession -ComputerName SERVER01
```

This attempts to establish an interactive PowerShell remoting session.

The outcome should be classified as:

```text id="q7n3p5"
Network Failure
WinRM Failure
Authentication Failure
Authorization Failure
Successful Session
```

Do not treat every failure as a credential problem.

---

## 17. Non-Interactive Remote Commands

For authorized administration or assessment, PowerShell can also execute a command remotely:

```powershell id="v2m8r6"
Invoke-Command -ComputerName SERVER01 -ScriptBlock {
    hostname
}
```

The purpose in this workflow is validation.

If successful, the simplest useful validation is:

```text id="x5p9k3"
Who am I?
Where am I?
```

rather than immediately performing unrelated actions.

---

## 18. Validate the New Position

After successful WinRM access, immediately establish the new position.

For example:

```powershell id="m7c2q8"
whoami
hostname
ipconfig
```

Then investigate:

```text id="r4n8x1"
Privileges
Network
Domain
Services
Existing Sessions
Credentials
Remote Access
```

Return to:

```text id="j6p3v9"
03-Position-Assessment/
```

and repeat the workflow.

---

## 19. WinRM as a Movement Path

A successful movement chain may look like:

```text id="w8m2q5"
Initial Host
      ↓
CORP\alice
      ↓
WinRM Authentication
      ↓
SERVER01
      ↓
Remote PowerShell
      ↓
Position Assessment
      ↓
New Targets
```

The new host becomes the next observation point.

---

## 20. WinRM and Domain Naming

Kerberos-based WinRM commonly depends on correct naming.

Prefer evidence such as:

```text id="q6p3m8"
SERVER01.CORP.LOCAL
```

when the environment expects the fully qualified hostname.

If authentication behaves unexpectedly, investigate:

```text id="c4n9x2"
DNS
Hostname
SPN
Domain Connectivity
Time
```

rather than immediately replacing the credential.

---

## 21. WinRM and IP Addresses

An IP address can be useful for connectivity testing:

```text id="k8m1r6"
Test-NetConnection 10.10.20.15 -Port 5985
```

But Kerberos-based authentication may rely on the target's hostname and associated service identity.

Therefore:

```text id="v5p2x9"
IP Reachability
    ≠
Kerberos Authentication
```

This distinction is important when troubleshooting.

---

## 22. HTTPS WinRM

WinRM can also use HTTPS:

```text id="n3q7m1"
5986/TCP
```

HTTPS changes the transport security characteristics but does not automatically change authorization.

The workflow remains:

```text id="f8m2c5"
Reachability
 ↓
WinRM Listener
 ↓
Authentication
 ↓
Authorization
 ↓
Remote Management
```

Certificate configuration may introduce additional validation considerations.

---

## 23. WinRM Failure Classification

When WinRM access fails, classify the failure.

### Network Failure

```text id="s6x3m8"
Timeout
No Route
Filtered Port
Connection Refused
```

### Service Failure

```text id="p4k9v2"
WinRM Not Running
No Listener
Wrong Port
Incorrect Transport
```

### Authentication Failure

```text id="r7m3x5"
Invalid Credential
Unsupported Authentication
Kerberos Failure
NTLM Restriction
Certificate Problem
```

### Authorization Failure

```text id="c8n1q6"
Account Authenticated
but Remote Management Not Authorized
```

### Environment Failure

```text id="w2m7p4"
DNS
Clock Skew
Domain Connectivity
Policy
```

---

## 24. Decision Tree

```text id="y6q2m8"
Candidate Windows Target
        │
        ▼
Is WinRM Reachable?
        │
   ┌────┴────┐
  No        Yes
   │          │
   ▼          ▼
Check       Confirm
Network     WinRM
              │
              ▼
       Identify Authentication
              │
              ▼
       Compatible Credential?
          │          │
         No         Yes
          │          │
          ▼          ▼
   Continue Other   Validate
   Credential Path  Authentication
                        │
                        ▼
                  Authorized?
                   │       │
                  No      Yes
                   │       │
                   ▼       ▼
              Record /   Establish
              Reassess   Session
                           │
                           ▼
                    Validate Position
                           │
                           ▼
                    Re-enumerate
```

---

## 25. Practical Windows Workflow

From a Windows foothold:

```text id="k4m8p1"
1. Identify current identity
2. Identify domain context
3. Identify candidate target
4. Check 5985 / 5986 reachability
5. Confirm WinRM
6. Determine authentication mechanism
7. Determine remote-management authorization
8. Validate WinRM
9. Establish authorized remote session
10. Validate identity on target
11. Enumerate the new position
12. Record the movement path
13. Continue the lateral-movement workflow
```

Useful validation commands include:

```powershell id="p9x3m7"
Test-NetConnection SERVER01 -Port 5985
Test-WSMan SERVER01
Enter-PSSession -ComputerName SERVER01
```

Use only the actions required to validate the authorized movement hypothesis.

---

## 26. Practical Troubleshooting Workflow

If:

```text id="w3m7q1"
Test-NetConnection
```

fails:

```text id="f8n2c5"
Investigate Network
```

If the port is reachable but:

```text id="r6p3x9"
Test-WSMan
```

fails:

```text id="m4q8k2"
Investigate WinRM Listener / Service / Configuration
```

If WinRM responds but authentication fails:

```text id="x7n1v5"
Investigate Authentication
```

If authentication succeeds but the session is denied:

```text id="c2m9p4"
Investigate Authorization / Policy
```

If the session succeeds:

```text id="v8k3r6"
Validate New Position
```

This is much more efficient than repeatedly changing credentials.

---

## 27. WinRM Access Record

Use this structure:

```text id="q5m8x2"
Target:
____________________

Hostname:
____________________

IP:
____________________

Port:
____________________

Transport:
HTTP / HTTPS

Current Identity:
____________________

Authentication:
____________________

Reachability:
____________________

WinRM Availability:
____________________

Authorization:
____________________

Session Result:
____________________

New Position:
____________________

Failure Reason:
____________________

Next Action:
____________________
```

---

## 28. Common Mistakes

### Mistake 1: Assuming Port 5985 Means WinRM

Verify the actual service.

---

### Mistake 2: Treating `Test-WSMan` as Proof of User Access

It primarily validates the WS-Management endpoint.

Authentication and authorization still need to be established.

---

### Mistake 3: Ignoring Domain Context

Kerberos-based authentication can depend heavily on:

```text
DNS
Hostname
Domain
Time
SPN
```

---

### Mistake 4: Assuming Any Domain User Can Use WinRM

Remote management authorization must be established.

---

### Mistake 5: Treating Authentication Success as Administrative Access

A successful session does not automatically mean administrator-level access.

---

### Mistake 6: Ignoring Failure Type

A network timeout and an authorization failure require completely different next actions.

---

### Mistake 7: Continuing Without Re-enumeration

A successful WinRM connection creates a new position.

Restart the position-assessment workflow.

---

## 29. Completion Criteria

WinRM investigation is sufficiently complete when you can answer:

```text id="m6x2q8"
[ ] Is the target reachable?
[ ] Is WinRM actually available?
[ ] Which listener / transport is used?
[ ] Which authentication mechanism is relevant?
[ ] Which identity is being tested?
[ ] Is the identity authorized for remote management?
[ ] Did authentication succeed?
[ ] Did remote management succeed?
[ ] What access level was obtained?
[ ] What new position was established?
[ ] What caused any failure?
[ ] Has the new host been re-enumerated?
[ ] What is the next movement path?
```

The objective is not:

```text id="p7n3c5"
Connect to every WinRM server.
```

It is:

```text id="x4m8q1"
Determine whether a known identity and authentication context
can establish useful, authorized remote management access to a target.
```

---

## 30. What Next?

The next service is:

```text id="h2v7m4"
06-Remote-Access-Services/RDP.md
```

RDP changes the movement model from:

```text id="n8q3p5"
Remote Management
```

to:

```text id="r6m2x9"
Interactive Graphical Session
```

The workflow becomes:

```text id="c4p8m1"
Windows Target
      ↓
RDP Available
      ↓
Reachability
      ↓
Authentication
      ↓
Remote Desktop Authorization
      ↓
Interactive Session
      ↓
Validate Identity
      ↓
Re-enumerate
      ↓
Continue Movement
```

The core principle is:

> **WinRM is useful for lateral movement only when a reachable target, compatible authentication mechanism, and appropriate remote-management authorization combine to create a validated remote management session.**
