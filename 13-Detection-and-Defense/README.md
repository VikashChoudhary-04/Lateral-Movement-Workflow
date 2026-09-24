# Detection and Defense

Lateral movement is not only an offensive workflow.

Every movement decision can produce authentication, process, network, service, and system telemetry.

Understanding that telemetry helps an operator:

* recognize what their actions may generate
* validate whether an assessment activity was observed
* understand defensive controls
* distinguish expected activity from suspicious activity
* communicate findings clearly in a report

This section keeps detection knowledge tied to the same movement workflow used throughout the repository.

## Purpose

The offensive workflow is:

```text id="d3f8k2"
Current Position
      ↓
Target
      ↓
Service
      ↓
Authentication
      ↓
Authorization
      ↓
Movement
      ↓
New Position
```

The defensive view is:

```text id="q7m4x9"
Movement Attempt
      ↓
Authentication Telemetry
      ↓
Network Telemetry
      ↓
Process / Service Telemetry
      ↓
Authorization / Access Telemetry
      ↓
Correlation
      ↓
Detection
      ↓
Investigation
```

The two perspectives describe the same activity from different sides.

## Why Detection Matters

A lateral-movement workflow should account for more than whether access succeeded.

Ask:

```text id="m8q3v5"
What happened?
      ↓
Which identity performed it?
      ↓
From which host?
      ↓
Against which target?
      ↓
Using which service?
      ↓
At what time?
      ↓
What process generated it?
      ↓
What authorization occurred?
      ↓
What changed afterward?
```

This produces a much more useful assessment record than simply stating that movement succeeded.

## Detection Layers

Lateral movement can be observed across several layers.

| Layer           | What It Can Show                                |
| --------------- | ----------------------------------------------- |
| Network         | Connections between source and destination      |
| Authentication  | Successful and failed authentication            |
| Authorization   | Access granted or denied                        |
| Process         | Processes involved in remote activity           |
| Service         | Remote-management or service activity           |
| Endpoint        | Logon sessions, processes, connections          |
| File / Resource | Access to shares or other resources             |
| Domain          | Kerberos and directory-related activity         |
| Application     | Application-specific authentication and actions |

No single telemetry source necessarily provides the complete picture.

## Source → Target Model

A useful detection model is:

```text id="z6p2m8"
Source Host
     ↓
Source Identity
     ↓
Network Connection
     ↓
Target Host
     ↓
Target Service
     ↓
Authentication
     ↓
Authorization
     ↓
Process / Session
     ↓
Result
```

For example:

```text id="x5q8m3"
WORKSTATION-01
      ↓
user@example
      ↓
TCP connection
      ↓
SERVER-01
      ↓
WinRM
      ↓
Authentication
      ↓
Remote session
      ↓
PowerShell
```

The exact telemetry depends on operating-system configuration, logging policy, security products, and the technique used.

## Authentication Is a Key Signal

Authentication events can provide:

* account identity
* source system
* destination system
* authentication type
* success/failure
* timestamp
* session information

Repeated authentication activity can become more meaningful when correlated with:

* unusual source hosts
* unusual destinations
* unusual accounts
* unusual authentication methods
* unusual time periods
* subsequent process activity

A single authentication event should not automatically be interpreted as malicious.

## Network Telemetry

Network monitoring can help identify:

```text id="k8n3p6"
Source
 ↓
Destination
 ↓
Port
 ↓
Protocol
 ↓
Timestamp
 ↓
Connection Pattern
```

For lateral movement, useful observations may include connections involving:

* SMB
* WinRM
* RDP
* RPC
* SSH
* internal web services
* database services

Network telemetry becomes more useful when combined with identity and endpoint data.

## Endpoint Telemetry

Endpoint telemetry can provide context that network logs cannot.

Examples include:

* process creation
* parent/child process relationships
* logon sessions
* service activity
* PowerShell activity
* remote-management processes
* network connections
* security events

The key question is:

> What process or session caused the network activity?

## Authentication vs Authorization

Detection should distinguish:

```text id="r4m8q2"
Authentication
    ↓
Who authenticated?
```

from:

```text id="c7n3v5"
Authorization
    ↓
What was the identity allowed to do?
```

A successful authentication event does not prove that the account obtained administrative access.

Similarly:

```text id="y8m4p6"
Connection
   ≠
Authentication
   ≠
Authorization
   ≠
Command Execution
```

## Detection by Movement Technique

Different movement methods can produce different telemetry.

| Technique               | Useful Detection Areas                                                            |
| ----------------------- | --------------------------------------------------------------------------------- |
| SMB                     | Network connections, authentication, share access, process activity               |
| WinRM                   | Authentication, WinRM activity, PowerShell/process telemetry, network connections |
| RDP                     | Remote interactive logon, authentication, session activity, network connection    |
| WMI                     | Authentication, RPC activity, process creation, WMI-related telemetry             |
| RPC                     | Network/RPC connections, authentication, dependent management activity            |
| SSH                     | SSH authentication, network connections, shell/process activity                   |
| Kerberos-based movement | Kerberos authentication, service-ticket activity, source/target relationships     |
| Pivoting                | Connections originating through an intermediate host, forwarding/proxy activity   |

These are investigation categories, not standalone detection rules.

## Baseline Matters

A detection system needs context.

For example:

```text id="p2q7m4"
Normal:
Admin workstation → Server
```

may be expected.

But:

```text id="h8m3v6"
Unusual workstation
      ↓
Unexpected server
      ↓
Administrative service
      ↓
Rare account
```

may deserve investigation.

The same network connection can have different meaning depending on the environment.

## Detection Questions

For each lateral-movement event, investigate:

### Identity

> Which account performed the activity?

### Source

> Which host initiated it?

### Destination

> Which host received it?

### Service

> Which service was accessed?

### Authentication

> How did the account authenticate?

### Authorization

> What access was granted?

### Process

> Which process generated the activity?

### Timing

> When did it occur?

### Sequence

> What happened immediately before and after it?

### Scope

> Is the source-to-target relationship expected?

## Correlation

Strong detection generally comes from combining multiple signals.

For example:

```text id="v6q2m8"
Network Connection
      +
Authentication Event
      +
Remote Service Activity
      +
Process Creation
      ↓
Higher-Context Investigation
```

Another example:

```text id="m3p8q5"
New Source Host
      +
Valid Credential
      +
Rare Target
      +
Remote Management Service
      ↓
Investigate
```

Correlation provides context that individual events may lack.

## Offensive Validation Perspective

During an authorized assessment, detection knowledge can help answer:

```text id="w7n4q2"
Did the movement generate telemetry?
       ↓
Which telemetry?
       ↓
Was the source identifiable?
       ↓
Was the identity identifiable?
       ↓
Was the target identifiable?
       ↓
Was the access method identifiable?
       ↓
Could defenders correlate the activity?
```

This can be useful when assessing defensive visibility.

## Detection Does Not Equal Prevention

Keep these concepts separate:

```text id="n5q8m3"
Detection
   ≠
Prevention
   ≠
Response
```

A system may detect a lateral-movement event without preventing it.

Likewise, a preventive control may block movement without producing a useful detection signal.

A mature defensive assessment considers all three.

## Defensive Control Areas

Defensive controls can include:

* least privilege
* strong authentication
* network segmentation
* restricted administrative protocols
* credential protection
* endpoint monitoring
* centralized logging
* authentication monitoring
* service hardening
* application security
* privileged-access management
* incident-response procedures

Controls should be evaluated according to the environment and threat model.

## Logging Coverage

A useful assessment should determine whether the environment can answer:

```text id="f4m7x2"
Who?
 ↓
From Where?
 ↓
To Where?
 ↓
Using What?
 ↓
When?
 ↓
What Happened?
```

If one of these is consistently unavailable, investigation may become more difficult.

## Evidence Collection

When documenting detection observations, record:

```text id="q8m3v6"
Timestamp:
Source Host:
Source IP:
Source Identity:
Destination Host:
Destination IP:
Service:
Authentication Type:
Relevant Event / Telemetry:
Process:
Result:
Correlated Evidence:
Detection Outcome:
```

Avoid collecting unnecessary sensitive information.

## Detection and the Movement Chain

The movement chain can be represented from both perspectives.

### Offensive

```text id="s7m2q4"
P1
 ↓
Authentication
 ↓
P2
 ↓
Authentication
 ↓
P3
```

### Defensive

```text id="d4n8p6"
P1
 ↓
Network Event
 ↓
Authentication Event
 ↓
Remote Service Event
 ↓
P2
 ↓
New Network Event
 ↓
Authentication Event
 ↓
P3
```

The defender attempts to reconstruct the same chain from telemetry.

## What This Section Covers

The remaining files focus on four areas:

### Windows Logging

[`Windows-Logging.md`](Windows-Logging.md)

Introduces Windows logging sources relevant to lateral movement and explains how to connect logs to the movement workflow.

### Authentication Events

[`Authentication-Events.md`](Authentication-Events.md)

Explains authentication telemetry and how to reason about source, destination, identity, authentication type, and result.

### Network Detection

[`Network-Detection.md`](Network-Detection.md)

Covers network-level visibility for remote services, internal connections, segmentation, and pivoting.

### Defensive Considerations

[`Defensive-Considerations.md`](Defensive-Considerations.md)

Connects the observations above to practical defensive controls and assessment conclusions.

## Practical Detection Workflow

Use:

```text id="m9q4v7"
Movement Observed
      ↓
Identify Source
      ↓
Identify Identity
      ↓
Identify Destination
      ↓
Identify Service
      ↓
Identify Authentication
      ↓
Identify Process / Session
      ↓
Correlate Events
      ↓
Determine Expected vs Unusual
      ↓
Investigate
      ↓
Respond / Improve Controls
```

## Golden Rule

> A lateral-movement event becomes much more useful to defenders when identity, source, destination, service, authentication, and process context can be correlated.

The goal of this section is therefore not to memorize event IDs.

It is to understand the evidence required to reconstruct movement.

## What Next?

Continue to [`Windows-Logging.md`](Windows-Logging.md).

That file begins the platform-specific detection workflow with Windows logging sources relevant to SMB, WinRM, RDP, WMI, RPC, and other remote-management activity.
