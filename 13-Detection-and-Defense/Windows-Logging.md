# Windows Logging

Windows lateral movement can generate telemetry across authentication, process, network, service, and system logging.

The purpose of this file is not to memorize event IDs.

The goal is to understand how Windows telemetry can help reconstruct:

```text id="m4q8x2"
Source Host
     ↓
Identity
     ↓
Remote Service
     ↓
Target Host
     ↓
Authentication
     ↓
Authorization
     ↓
Process / Session
     ↓
Result
```

Exact visibility depends on Windows version, audit-policy configuration, logging configuration, endpoint security products, and centralized collection.

## Logging Mindset

When investigating lateral movement, begin with the question:

> What happened between the source host and target host?

Then collect evidence for:

```text id="r7m3v8"
WHO?
WHERE FROM?
WHERE TO?
USING WHAT?
WHEN?
WHAT ACCESS?
WHAT PROCESS?
WHAT HAPPENED NEXT?
```

## Major Windows Telemetry Sources

Common sources include:

| Source                                   | Useful Context                                                  |
| ---------------------------------------- | --------------------------------------------------------------- |
| Security log                             | Authentication, account, privilege, and security events         |
| System log                               | System and service activity                                     |
| Application log                          | Application-specific activity                                   |
| PowerShell logs                          | PowerShell execution and related activity when configured       |
| Windows Filtering Platform telemetry     | Network connection context when appropriate auditing is enabled |
| Task Scheduler logs                      | Scheduled task activity                                         |
| Windows Defender / security-product logs | Endpoint security detections and observations                   |
| Sysmon                                   | Detailed process, network, and system telemetry when deployed   |
| Remote-management logs                   | Service-specific activity depending on configuration            |

No single source should automatically be treated as complete.

## Security Log

The Windows Security log is particularly important for authentication and account activity.

Commonly useful event categories include:

* successful logon
* failed logon
* explicit credential use
* special privileges
* account activity
* logoff/session activity
* process-related security auditing when configured

A commonly referenced successful logon event is:

```text id="p8m2q5"
4624 — An account was successfully logged on
```

A commonly referenced failed logon event is:

```text id="x4n7v3"
4625 — An account failed to log on
```

The meaning of an event depends on its fields and context.

## Successful Logon Analysis

For a successful logon, investigate fields such as:

* account name
* account domain
* logon type
* source network address
* workstation name when available
* authentication package
* timestamp
* target account
* session identifiers where relevant

The useful model is:

```text id="q6m3v8"
4624
 ↓
Which Account?
 ↓
Which Logon Type?
 ↓
From Which Source?
 ↓
Using Which Authentication Package?
 ↓
Against Which Host?
```

## Logon Type Matters

Windows logon type provides important context.

Common examples include:

| Logon Type | General Meaning   |
| ---------: | ----------------- |
|          2 | Interactive       |
|          3 | Network           |
|          4 | Batch             |
|          5 | Service           |
|          7 | Unlock            |
|          8 | NetworkCleartext  |
|          9 | NewCredentials    |
|         10 | RemoteInteractive |
|         11 | CachedInteractive |

These descriptions are contextual rather than standalone indicators of malicious activity.

For lateral movement, network and remote-interactive logons can be particularly relevant.

## Failed Logons

Event `4625` can help identify unsuccessful authentication attempts.

Investigate:

```text id="v5q8m3"
Account
Source
Logon Type
Failure Reason
Authentication Package
Timestamp
```

A single failed attempt may be normal.

Repeated failures become more useful when correlated with:

* unusual source hosts
* unusual accounts
* unusual destinations
* successful authentication afterward
* other endpoint or network activity

## Explicit Credential Use

Event `4648` can indicate that a process attempted to log on by explicitly specifying credentials.

This can provide useful context when investigating activity involving credentials different from the current logged-on identity.

Investigate:

```text id="k7m3p8"
Subject Account
Target Account
Target Server
Process
Timestamp
```

The event should be correlated with surrounding process and authentication activity.

## Special Privileges

Event `4672` can indicate that special privileges were assigned to a new logon.

This can help identify sessions associated with elevated privileges.

Use the event as a correlation point:

```text id="z4q8m2"
Successful Logon
      +
Special Privileges
      ↓
Investigate Session Context
```

The event itself does not prove malicious activity.

## Process Creation

Process telemetry can help answer:

> What happened after the remote session was established?

Security process-creation auditing can provide Event `4688` when configured.

Useful fields can include:

* new process
* creator process
* account
* command line when available and configured
* process ID
* parent process ID

The investigation model becomes:

```text id="m8q3v5"
Remote Authentication
      ↓
Logon Session
      ↓
Process Creation
      ↓
Command / Application Activity
```

## PowerShell Logging

PowerShell can produce additional telemetry depending on configuration.

Useful sources may include:

* PowerShell operational logging
* script block logging
* module logging
* transcription
* process creation telemetry

For remote PowerShell activity, correlate:

```text id="n7m4q2"
Authentication
      ↓
WinRM / Remote Session
      ↓
PowerShell Process
      ↓
PowerShell Activity
```

The availability and detail of this telemetry depend on policy and configuration.

## WinRM and PowerShell

A simplified investigative chain is:

```text id="b5q8m3"
Source Host
      ↓
TCP 5985 / 5986
      ↓
WinRM
      ↓
Authentication
      ↓
Remote Session
      ↓
PowerShell / Process Activity
```

Defensive investigation should correlate the network, authentication, and endpoint layers.

A WinRM connection alone does not establish what actions occurred after authentication.

## RDP Logging

RDP activity can produce several forms of telemetry.

Useful investigation areas include:

* successful and failed authentication
* remote-interactive logons
* Terminal Services / Remote Desktop Services logs
* session connection and disconnection
* endpoint process activity

A useful model is:

```text id="c6m2p8"
RDP Network Connection
      ↓
Authentication
      ↓
RemoteInteractive Logon
      ↓
Desktop Session
      ↓
User Activity
```

Event `4624` with Logon Type `10` is commonly associated with a successful remote-interactive logon.

The event should be correlated with other session and endpoint telemetry.

## SMB Logging

SMB activity can generate evidence at multiple layers.

Investigate:

```text id="v8q3m5"
Network Connection
      ↓
Authentication
      ↓
SMB Session
      ↓
Share Access
      ↓
Resource Access
```

Depending on audit configuration, useful evidence may come from:

* Security logs
* object-access auditing
* SMB server/client operational logs
* network telemetry
* endpoint telemetry

Do not assume that a Security log alone will show every file or share operation.

## WMI and RPC

WMI and RPC investigations often require correlation.

A simplified chain is:

```text id="j4m8q2"
Source
 ↓
RPC Endpoint Mapper
 ↓
Dynamic RPC
 ↓
Authentication
 ↓
WMI / DCOM Activity
 ↓
Process / Management Activity
```

Network telemetry can establish the connection path.

Endpoint telemetry can help identify the resulting process activity.

Authentication telemetry can establish the account context.

## Sysmon

Sysmon can provide detailed endpoint telemetry when deployed and appropriately configured.

Depending on configuration and version, useful event categories can include:

* process creation
* network connections
* process access
* image loading
* DNS activity
* file creation
* registry activity

For lateral movement, process and network telemetry can be especially useful.

A common investigative correlation is:

```text id="w6q3n8"
Sysmon Network Event
      +
Process Creation
      +
Windows Authentication Event
      ↓
Movement Investigation
```

Sysmon is an additional telemetry source, not a replacement for Windows Security auditing.

## Windows Filtering Platform

Windows Filtering Platform auditing can provide network connection context.

Depending on enabled auditing, investigators may obtain information about:

* source
* destination
* process
* protocol
* port

This can help connect:

```text id="q5m8v3"
Process
 ↓
Network Connection
 ↓
Target
```

That relationship can be valuable when investigating remote services.

## Scheduled Tasks

Scheduled tasks can be relevant to both administration and lateral movement investigations.

Investigate:

* task creation
* task modification
* task execution
* task account
* process creation associated with the task

The important distinction is:

```text id="m3q8p5"
Task Activity
    ≠
Malicious Activity
```

Context determines significance.

## Service Activity

Windows services can also provide useful telemetry.

Investigate:

```text id="v7m4x2"
Service
 ↓
Account
 ↓
Start / Stop
 ↓
Process
 ↓
Network Activity
```

Service-related activity should be correlated with the account and process involved.

## Authentication Package

Authentication events may contain an authentication package or mechanism.

This can help distinguish authentication contexts such as:

* Kerberos
* NTLM
* Negotiate
* other configured mechanisms

Use the authentication package together with:

* account
* source
* target
* logon type
* timing

rather than treating it as an independent verdict.

## Source Address

When available, the source network address is an important field.

Use:

```text id="p8m3q7"
Account
+
Source IP
+
Destination
+
Timestamp
```

to reconstruct movement.

Be aware of complications such as:

* NAT
* proxies
* load balancers
* VPNs
* jump hosts
* IPv6
* address reuse
* missing or unavailable fields

The logged source address may not always directly identify the original human endpoint.

## Workstation Name

Some authentication events may include a workstation name.

When available, compare:

```text id="k4q7m3"
Workstation Name
+
Source IP
+
Account
```

Inconsistencies can provide useful investigative leads, but they should be validated against authoritative network and endpoint data.

## Timestamp Correlation

Time is essential for reconstructing movement.

Use:

```text id="r6m3q8"
T1: Network Connection
T2: Authentication
T3: Remote Session
T4: Process Creation
T5: Resource Access
```

Then ask:

> Does the sequence make sense?

Clock differences between systems can complicate correlation, so centralized logging and time synchronization are important.

## Practical Investigation Example

Suppose a server shows an unexpected remote-management session.

Start with:

```text id="u5m8q3"
1. Identify the destination server
2. Identify the source
3. Identify the account
4. Find the authentication event
5. Determine the logon type
6. Determine the authentication package
7. Identify related process activity
8. Check network telemetry
9. Check PowerShell / WinRM telemetry if applicable
10. Reconstruct the sequence
```

The objective is to build an evidence chain, not to rely on one event.

## Windows Lateral-Movement Investigation Model

```text id="x7m2q8"
             SOURCE
                │
                ▼
        Network Connection
                │
                ▼
          Authentication
                │
                ▼
          Target Service
                │
        ┌───────┴───────┐
        ▼               ▼
   Authorization      Session
        │               │
        └───────┬───────┘
                ▼
        Process Activity
                │
                ▼
          Resource Access
                │
                ▼
             RESULT
```

## Detection Coverage Checklist

Use this checklist when evaluating a Windows environment:

```text
[ ] Successful logons are collected
[ ] Failed logons are collected
[ ] Relevant logon details are available
[ ] Process creation is audited
[ ] PowerShell telemetry is available where appropriate
[ ] Remote-management telemetry is collected
[ ] Network telemetry is available
[ ] Security logs are centralized
[ ] Endpoint telemetry is retained
[ ] Timestamps can be correlated
[ ] Source and destination can be reconstructed
[ ] Account identity can be reconstructed
[ ] Relevant administrative activity can be investigated
```

Actual coverage depends on configuration and retention.

## Offensive Assessment Perspective

During an authorized assessment, ask:

```text id="h3q8m5"
I performed lateral movement.
        ↓
What Windows telemetry should exist?
        ↓
Does it exist?
        ↓
Can the source be identified?
        ↓
Can the account be identified?
        ↓
Can the target be identified?
        ↓
Can the access method be identified?
        ↓
Can the session/process activity be reconstructed?
```

This can help identify visibility gaps.

## Important Limitations

Windows logs are not guaranteed to provide complete visibility.

Possible limitations include:

* auditing not enabled
* insufficient log retention
* centralized collection gaps
* overwritten logs
* endpoint unavailable
* missing command-line data
* incomplete network telemetry
* security-product configuration differences
* time synchronization problems

Therefore:

> Absence of an observed event does not automatically prove that the activity did not occur.

## Evidence Record

For a Windows lateral-movement investigation, record:

```text id="n6q2v8"
Timestamp:
Source Host:
Source IP:
Account:
Logon Type:
Authentication Package:
Destination Host:
Destination IP:
Service:
Process:
Related Event IDs:
Network Evidence:
Authentication Evidence:
Process Evidence:
Resource Evidence:
Result:
```

## What Next?

Continue to [`Authentication-Events.md`](Authentication-Events.md).

That file focuses specifically on authentication telemetry and how to reason about successful and failed authentication across Windows hosts and domain environments.
