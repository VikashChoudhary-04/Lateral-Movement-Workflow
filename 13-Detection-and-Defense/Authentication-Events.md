# Authentication Events

Authentication is one of the most important telemetry layers for understanding lateral movement.

A movement attempt often creates a sequence involving:

```text id="p7m3q8"
Source Host
      ↓
Account
      ↓
Authentication Attempt
      ↓
Target Host
      ↓
Authentication Result
      ↓
Session / Access
```

The objective is to reconstruct that sequence accurately.

Authentication telemetry should be interpreted together with network, endpoint, service, and authorization evidence.

## Authentication vs Authorization

Always separate:

```text id="m4q8v2"
Authentication
    ↓
Who are you?
```

from:

```text id="x7n3p6"
Authorization
    ↓
What are you allowed to do?
```

A successful authentication does not automatically mean:

* administrative access
* file access
* remote shell access
* RDP access
* WinRM access
* access to every resource on the target

## Authentication Investigation Model

Use:

```text id="c8m3q7"
Account
  ↓
Source
  ↓
Destination
  ↓
Authentication Method
  ↓
Authentication Result
  ↓
Logon Type
  ↓
Authorization
  ↓
Session / Activity
```

## Windows Authentication Events

Several Windows Security events are particularly useful during authentication investigations.

| Event ID | General Meaning                                                |
| -------: | -------------------------------------------------------------- |
|     4624 | Successful account logon                                       |
|     4625 | Failed account logon                                           |
|     4634 | Account logoff                                                 |
|     4647 | User-initiated logoff                                          |
|     4648 | Logon attempted using explicit credentials                     |
|     4672 | Special privileges assigned to a new logon                     |
|     4768 | Kerberos authentication service ticket-granting ticket request |
|     4769 | Kerberos service ticket request                                |
|     4771 | Kerberos pre-authentication failed                             |
|     4776 | Domain controller authentication validation                    |

The exact availability and fields depend on audit policy, Windows version, domain configuration, and log collection.

## 1. Event 4624 — Successful Logon

Event `4624` records a successful logon.

For lateral-movement investigation, examine:

* account name
* account domain
* logon type
* source network address
* workstation name where available
* authentication package
* timestamp
* target system
* other session identifiers

The basic model is:

```text id="u4q8m2"
4624
 ↓
Account
 ↓
Source
 ↓
Logon Type
 ↓
Authentication Package
 ↓
Target
```

## Logon Type

Logon type can provide important context.

Common values include:

| Type | General Context   |
| ---: | ----------------- |
|    2 | Interactive       |
|    3 | Network           |
|    4 | Batch             |
|    5 | Service           |
|    7 | Unlock            |
|    8 | NetworkCleartext  |
|    9 | NewCredentials    |
|   10 | RemoteInteractive |
|   11 | CachedInteractive |

For lateral movement, Type 3 and Type 10 can be particularly relevant because they may correspond to network and remote-interactive access respectively.

However, the event must be interpreted in context.

## Event 4625 — Failed Logon

Event `4625` records a failed logon.

Useful fields include:

* account
* source
* logon type
* failure reason
* authentication package
* timestamp
* target system

Use it to investigate:

```text id="q6m3v8"
Who attempted authentication?
       ↓
From where?
       ↓
Against what?
       ↓
Using what context?
       ↓
Why did it fail?
```

## Failed Authentication Patterns

One failed authentication may be expected.

More useful patterns can involve:

```text id="r8m4p2"
Repeated Failures
      +
Same Account
      +
Same / Multiple Targets
```

or:

```text id="x3q7m8"
Failure
   ↓
Successful Authentication
   ↓
Remote Service Activity
```

These sequences can justify further investigation.

Do not infer malicious activity from failed authentication alone.

## Event 4648 — Explicit Credentials

Event `4648` can indicate that a process attempted authentication using explicitly supplied credentials.

Useful fields may include:

* subject account
* target account
* target server
* process information
* timestamp

The investigation chain is:

```text id="m7q3v8"
4648
 ↓
Which Process?
 ↓
Which Account?
 ↓
Which Target?
 ↓
What Happened Next?
```

This can help explain authentication performed by a process rather than the currently logged-on identity.

## Event 4672 — Special Privileges

Event `4672` indicates that special privileges were assigned to a new logon.

Use it to add privilege context:

```text id="p4m8q2"
4624 Successful Logon
        +
4672 Special Privileges
        ↓
Investigate Session
```

The event should not be treated as proof of suspicious activity.

Administrative accounts can legitimately generate this telemetry.

## Kerberos Authentication

Kerberos is central to many Active Directory environments.

Useful events include:

```text id="v8m3q5"
4768 → TGT request
4769 → Service ticket request
4771 → Kerberos pre-authentication failure
```

These events help reconstruct:

```text id="n5q8m3"
Principal
   ↓
Authentication
   ↓
TGT
   ↓
Service Ticket
   ↓
Target Service
```

## Event 4768 — TGT Request

Event `4768` relates to a Kerberos authentication service ticket-granting ticket request.

Investigate fields such as:

* account/principal
* client address
* domain
* timestamp
* authentication details

This can help identify where a Kerberos authentication context originated.

## Event 4769 — Service Ticket Request

Event `4769` relates to a Kerberos service ticket request.

This is especially useful when investigating access to specific domain services.

The reasoning is:

```text id="g7m4q2"
Account
  ↓
4769
  ↓
Service Principal / Service
  ↓
Target
```

Where available, service-principal information can help connect authentication to the destination service.

## Event 4771 — Kerberos Pre-Authentication Failure

Event `4771` can indicate a Kerberos pre-authentication failure.

Possible investigation areas include:

* account
* client address
* failure code
* timestamp
* authentication context

Repeated failures should be correlated with:

* account activity
* source host
* target services
* successful authentication
* endpoint telemetry

## Event 4776 — Authentication Validation

Event `4776` can provide authentication-validation context, particularly around NTLM authentication in domain environments.

Investigate:

```text id="c3m8q5"
Account
 ↓
Source / Workstation Context
 ↓
Authentication Result
 ↓
Timestamp
```

Correlate it with other events rather than treating it as a complete description of the session.

## NTLM Authentication

NTLM-related authentication can appear in multiple Windows telemetry sources.

The investigation model is:

```text id="k8m3q7"
Source Host
      ↓
NTLM Authentication
      ↓
Account
      ↓
Target
      ↓
Service
      ↓
Authorization
```

The presence of NTLM does not automatically indicate malicious activity.

Some environments legitimately use NTLM for compatibility or specific applications.

## Kerberos vs NTLM

A simplified investigation comparison:

| Attribute           | Kerberos                     | NTLM                                     |
| ------------------- | ---------------------------- | ---------------------------------------- |
| Common environment  | Active Directory             | Windows / compatibility scenarios        |
| Core evidence       | TGT/service tickets          | Authentication validation / logon events |
| Useful events       | 4768, 4769, 4771             | 4776 plus related logon events           |
| Target context      | Service principal / SPN      | Service / target context                 |
| Investigation focus | Principal → ticket → service | Account → authentication → target        |

Actual authentication flows can be more complex than this simplified model.

## Remote Interactive Authentication

Remote interactive access can generate authentication telemetry associated with RDP.

A useful sequence is:

```text id="u6q3m8"
RDP Connection
      ↓
Authentication
      ↓
4624
      ↓
Logon Type 10
      ↓
Remote Desktop Session
```

Correlate with Remote Desktop Services logs and endpoint activity where available.

## Network Logons

Network logons can be associated with remote resource access.

A simplified model:

```text id="f5m8q2"
Source
 ↓
Network Service
 ↓
Authentication
 ↓
Network Logon
 ↓
Resource / Service Access
```

This can be particularly relevant for SMB and other network-access protocols.

## Service Accounts

Authentication involving service accounts requires context.

Investigate:

```text id="m8q4v3"
Account
 ↓
Expected Service
 ↓
Expected Source
 ↓
Expected Target
 ↓
Actual Activity
```

An authentication may be legitimate even when the account is highly privileged.

The useful question is whether the observed activity matches the account's expected role.

## Privileged Accounts

For privileged identities, investigate:

* source host
* destination host
* time
* authentication method
* logon type
* service accessed
* resulting process/session
* whether the source is an expected administrative system

The goal is to establish context, not to label every privileged authentication as suspicious.

## Source-Target Correlation

Build a simple relationship:

```text id="q7m3v8"
SOURCE
  |
  | Authentication
  ↓
TARGET
  |
  | Service
  ↓
ACCESS
```

Then enrich it:

```text id="x4m8q2"
Source Host
+
Source IP
+
Account
+
Authentication Method
+
Target Host
+
Target Service
+
Timestamp
```

## Authentication Timeline

For an individual movement event, create a timeline:

| Time | Evidence                       |
| ---- | ------------------------------ |
| T1   | Source initiates connection    |
| T2   | Authentication attempt         |
| T3   | Authentication succeeds/fails  |
| T4   | Remote service/session appears |
| T5   | Process activity occurs        |
| T6   | Resource accessed              |
| T7   | Session ends                   |

This helps determine whether the events form one activity sequence.

## Detecting Unexpected Lateral Movement

Use multiple indicators together:

```text id="v3m8q6"
Unexpected Source
       +
Unexpected Account
       +
Unexpected Target
       +
Remote Service
       +
Authentication Event
       +
Endpoint Activity
```

This provides more context than any individual signal.

## Failed → Successful Sequence

One useful investigation pattern is:

```text id="n8q3m5"
Authentication Failure
        ↓
Authentication Failure
        ↓
Successful Authentication
        ↓
Remote Service Activity
```

This does not automatically indicate malicious activity.

Investigate:

* account
* source
* target
* authentication method
* timing
* process
* expected administrative activity

## Authentication Spray vs Normal Activity

Large numbers of authentication failures across many accounts or systems may warrant investigation.

However, legitimate causes can include:

* service misconfiguration
* expired credentials
* mapped drives
* scheduled tasks
* applications using old credentials
* administrative automation

Therefore, use source, account, target, timing, and process context before drawing conclusions.

## Domain Controller Perspective

Domain controllers can provide valuable authentication telemetry for domain activity.

Investigate relationships such as:

```text id="c6m3q8"
Client
 ↓
Domain Authentication
 ↓
Domain Controller
 ↓
Service Ticket
 ↓
Target Service
```

The domain controller provides authentication evidence, but endpoint and network telemetry may still be required to reconstruct the complete movement.

## Authentication Evidence Matrix

| Question            | Useful Evidence                         |
| ------------------- | --------------------------------------- |
| Who authenticated?  | Account / principal                     |
| From where?         | Source IP / workstation                 |
| To what?            | Destination / target                    |
| How?                | Authentication package / protocol       |
| What type?          | Logon type                              |
| When?               | Timestamp                               |
| Did it succeed?     | Success/failure event                   |
| What happened next? | Process / session / resource telemetry  |
| Was it authorized?  | Resource/service authorization evidence |

## Investigation Workflow

Use:

```text id="m7q4v8"
Authentication Event
      ↓
Identify Account
      ↓
Identify Source
      ↓
Identify Destination
      ↓
Identify Logon Type
      ↓
Identify Authentication Method
      ↓
Check Success / Failure
      ↓
Correlate Network Activity
      ↓
Correlate Process / Session Activity
      ↓
Determine Authorization / Resource Access
      ↓
Build Timeline
```

## Offensive Assessment Perspective

During an authorized assessment, the same model can be used to assess visibility:

```text id="p8m3q7"
Perform Authorized Movement
        ↓
Expected Authentication Event?
        ↓
Source Identifiable?
        ↓
Account Identifiable?
        ↓
Target Identifiable?
        ↓
Authentication Method Visible?
        ↓
Session / Process Activity Visible?
        ↓
Can the Movement Chain Be Reconstructed?
```

This helps identify authentication-monitoring gaps.

## Important Limitations

Authentication telemetry can be incomplete because of:

* disabled auditing
* missing event forwarding
* short retention periods
* endpoint isolation
* log rotation
* incomplete source-address information
* authentication occurring through intermediary systems
* NAT or proxy infrastructure
* inconsistent time synchronization

Therefore:

> A missing authentication event is not, by itself, proof that authentication did not occur.

## Authentication Investigation Checklist

```text id="u5m8q3"
[ ] Account identified
[ ] Source identified
[ ] Destination identified
[ ] Authentication method identified
[ ] Logon type identified
[ ] Success / failure identified
[ ] Timestamp recorded
[ ] Related network activity checked
[ ] Related process activity checked
[ ] Related service activity checked
[ ] Authorization context checked
[ ] Timeline reconstructed
[ ] Expected vs unusual activity assessed
```

## Evidence Record

```text id="q8m3v6"
Timestamp:
Account:
Domain:
Source Host:
Source IP:
Destination Host:
Destination IP:
Service:
Authentication Method:
Logon Type:
Event ID:
Result:
Related Process:
Related Network Activity:
Authorization Evidence:
Assessment Notes:
```

## What Next?

Continue to [`Network-Detection.md`](Network-Detection.md).

That file shifts from identity-focused telemetry to the network layer and explains how source-to-target connections, remote services, segmentation, and pivoting can be observed.
