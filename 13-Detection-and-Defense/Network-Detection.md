# Network Detection

Network telemetry provides the connection-level view of lateral movement.

Where authentication logs answer:

> Who authenticated?

network telemetry can help answer:

> Which system communicated with which other system, over what service, and when?

The most useful model is:

```text id="n7m3q8"
Source Host
     ↓
Source IP
     ↓
Destination IP
     ↓
Destination Port
     ↓
Protocol / Service
     ↓
Timestamp
     ↓
Connection Pattern
```

Network evidence becomes significantly more useful when correlated with authentication and endpoint telemetry.

## Purpose

This file focuses on detecting:

* internal host-to-host communication
* remote administration services
* unusual source-to-target relationships
* repeated connection attempts
* network segmentation violations
* pivoting and proxy paths
* service discovery
* changes in network behavior after compromise

The goal is to reconstruct movement, not simply count open ports.

## Network Detection Model

Use:

```text id="q8m4v2"
Connection
    ↓
Identify Source
    ↓
Identify Destination
    ↓
Identify Service
    ↓
Identify Account
    ↓
Correlate Authentication
    ↓
Correlate Endpoint Activity
    ↓
Determine Context
```

A network connection by itself rarely explains the complete activity.

## 1. Source and Destination

Every relevant connection should ideally provide:

```text id="m3q7v8"
Source IP:
Source Host:
Destination IP:
Destination Host:
Destination Port:
Protocol:
Timestamp:
```

Where available, add:

```text id="x5m8p3"
User / Account
Process
Authentication
Application
Bytes / Duration
```

## 2. Remote Administration Ports

Common ports relevant to lateral movement include:

| Service             | Common Port | Network Investigation                 |
| ------------------- | ----------: | ------------------------------------- |
| SMB                 |         445 | Windows file/remote access            |
| NetBIOS Session     |         139 | Legacy SMB transport                  |
| WinRM               |        5985 | HTTP-based Windows remote management  |
| WinRM over TLS      |        5986 | HTTPS-based Windows remote management |
| RDP                 |        3389 | Windows remote desktop                |
| RPC Endpoint Mapper |         135 | Windows RPC                           |
| SSH                 |          22 | Secure shell                          |
| NFS                 |        2049 | Network filesystem                    |
| HTTP                |          80 | Web/application services              |
| HTTPS               |         443 | Web/application services              |

Ports are common defaults.

Organizations may use non-standard ports, proxies, tunnels, or load balancers.

## 3. SMB Network Detection

A simplified SMB movement pattern is:

```text id="v6m2q8"
Workstation
    ↓
TCP 445
    ↓
Server
    ↓
Authentication
    ↓
SMB Session
    ↓
Share / Resource Access
```

Network monitoring can identify:

* source
* destination
* TCP 445 connection
* connection frequency
* timing
* session duration

Endpoint and authentication logs are required to establish which identity performed the activity.

## 4. WinRM Network Detection

WinRM commonly uses:

```text id="m8q3v7"
TCP 5985
TCP 5986
```

A network-level sequence may appear as:

```text id="p4m8x2"
Host A
 ↓
TCP 5985 / 5986
 ↓
Host B
 ↓
Authentication
 ↓
Remote Management Session
```

Network telemetry can establish that communication occurred.

It may not establish what commands were executed.

For that, correlate:

* authentication events
* WinRM telemetry
* PowerShell logs
* process creation
* endpoint security telemetry

## 5. RDP Network Detection

A typical RDP connection uses:

```text id="q7m3v8"
Source
 ↓
TCP 3389
 ↓
Windows Target
 ↓
Authentication
 ↓
Remote Interactive Session
```

Useful network questions include:

* Is the source expected to use RDP?
* Is the destination an expected RDP server?
* Is the source-to-target relationship normal?
* Is the account expected?
* Did the connection occur at an expected time?

Network telemetry should be correlated with the corresponding authentication and session events.

## 6. RPC Network Detection

RPC commonly begins with:

```text id="z8m4q2"
Source
 ↓
TCP 135
 ↓
Endpoint Mapper
 ↓
Dynamic RPC Connection
 ↓
Target Service
```

Therefore, monitoring only TCP 135 may not provide the complete picture.

Investigate:

```text id="c6m3p8"
135 Connection
      +
Dynamic RPC Connections
      +
Authentication
      +
Endpoint Activity
```

This can help identify RPC-based management activity.

## 7. SSH Network Detection

SSH commonly uses:

```text id="k5m8q3"
TCP 22
```

A simplified movement pattern:

```text id="r7m3v8"
Linux Host A
     ↓
TCP 22
     ↓
Linux Host B
     ↓
SSH Authentication
     ↓
Shell / Session
```

Useful network observations include:

* source host
* destination host
* connection time
* frequency
* session duration
* destination port

Authentication and endpoint logs can provide the account and resulting process context.

## 8. Internal East-West Traffic

Lateral movement often occurs through internal host-to-host communication.

This is sometimes described as:

```text id="w4m8q2"
East-West Traffic
```

The useful question is:

> Which internal systems normally communicate with each other?

A baseline may reveal:

```text id="y7q3m8"
Normal:
Application Server → Database
```

versus:

```text id="v5m2q8"
Unexpected:
User Workstation → Database
```

Unexpected does not automatically mean malicious.

It means the relationship deserves contextual investigation.

## 9. Network Baselines

Build a baseline of:

* normal source-to-destination relationships
* common administrative hosts
* common remote-management paths
* server-to-server communication
* workstation-to-server communication
* service-specific communication

Example:

| Source Type               | Expected Destination | Service           |
| ------------------------- | -------------------- | ----------------- |
| Admin workstation         | Windows servers      | WinRM / RDP / SMB |
| User workstation          | File server          | SMB               |
| Application server        | Database             | Database protocol |
| Linux administration host | Linux servers        | SSH               |

The actual baseline must come from the environment.

## 10. Unusual Source-to-Target Relationships

A potentially useful investigation pattern is:

```text id="m8q4v2"
Previously Unseen Source
        ↓
Previously Unseen Target
        ↓
Remote Administration Service
```

Add:

```text id="p3q7m8"
+
Successful Authentication
+
New Process Activity
```

This provides substantially more context.

## 11. Service Discovery Detection

Network reconnaissance can generate connection patterns.

A host scanning a range may produce:

```text id="c8m3q7"
One Source
   ↓
Many Destinations
   ↓
Same Port(s)
```

Or:

```text id="x4m8p2"
One Source
   ↓
One / Few Targets
   ↓
Many Ports
```

Potential legitimate explanations include:

* vulnerability scanners
* asset discovery
* monitoring systems
* configuration management
* administrative tooling

Therefore, source identity and expected role matter.

## 12. Connection Frequency

Frequency can provide useful context.

Examples:

```text id="n7m4q3"
Single Connection
```

versus:

```text id="q5m8v2"
Hundreds of Connections
```

High frequency may indicate:

* scanning
* automation
* application behavior
* monitoring
* misconfiguration
* repeated authentication
* other administrative activity

Frequency alone is not a verdict.

## 13. Connection Timing

Timing can help correlate network events with authentication and process events.

Example:

```text id="w8m3q5"
10:01:12 Network Connection
10:01:13 Authentication
10:01:14 Remote Session
10:01:15 Process Creation
```

This sequence is more informative than any event viewed independently.

## 14. Network Segmentation

Segmentation creates expected boundaries.

Example:

```text id="k4m8q2"
User Network
     │
     X
Server Network
     │
     ↓
Admin Network
```

If a workstation cannot normally reach an administrative network, a new connection from that workstation should be investigated in context.

For lateral movement, identify:

```text id="p7m3q8"
Current Network
      ↓
Allowed Destinations
      ↓
Blocked Destinations
      ↓
Observed Connections
```

## 15. Pivoting Detection

Pivoting can change the apparent network path.

Example:

```text id="r8m3q5"
Original Source
      ↓
Pivot Host
      ↓
Internal Target
```

The target may observe the pivot host as the direct network source.

Therefore, investigation may require correlation across:

```text id="x6m2q8"
Original Host
+
Pivot Host
+
Target
```

## 16. Port Forwarding

Port forwarding can create a relationship such as:

```text id="m3q8v5"
External / Initial Host
        ↓
Forwarded Connection
        ↓
Pivot Host
        ↓
Internal Service
```

Network monitoring may see only part of this chain depending on where telemetry is collected.

Useful investigation areas include:

* unexpected listening ports
* unusual long-lived connections
* unusual source-to-pivot relationships
* pivot-to-internal-target connections
* process ownership of connections

## 17. SOCKS Proxy Detection

A SOCKS proxy can allow applications to reach internal targets through a pivot.

Conceptually:

```text id="q7m3v8"
Operator
   ↓
Pivot
   ↓
SOCKS
   ↓
Internal Target
```

Detection may focus on:

* long-lived connections
* unusual traffic patterns
* unexpected processes acting as proxy endpoints
* unusual pivot-host connections
* internal targets accessed by unexpected hosts

The exact network evidence depends on where monitoring occurs.

## 18. Network-to-Identity Correlation

Network data becomes much stronger when associated with identity.

Use:

```text id="v4m8q2"
Source IP
    ↓
Source Host
    ↓
Logged-On User
    ↓
Process
    ↓
Destination
    ↓
Service
```

This can transform:

> `10.0.1.15 connected to 10.0.2.20:5985`

into:

> A specific process on a specific host, under a specific account, established a WinRM connection to a specific server.

The latter is far more useful for investigation.

## 19. Process-to-Network Correlation

Endpoint telemetry can identify which process generated a network connection.

Conceptual model:

```text id="m8q3v7"
Process
  ↓
Source Host
  ↓
Network Connection
  ↓
Destination
  ↓
Service
```

This is especially useful for distinguishing:

* expected administrative tools
* applications
* system processes
* scripts
* unexpected processes

Process names alone should not be treated as proof of malicious activity.

## 20. Authentication-to-Network Correlation

Combine:

```text id="p5m8q3"
Network Connection
      +
Authentication Event
      ↓
Movement Context
```

For example:

```text id="y7m4q2"
Host A
 ↓
TCP 5985
 ↓
Host B
 ↓
Successful Authentication
 ↓
Remote Session
```

This is much stronger evidence than either the network event or authentication event alone.

## 21. Detecting Lateral-Movement Chains

A multi-host chain may appear as:

```text id="q8m3v5"
Host A
 ↓
Host B
 ↓
Host C
 ↓
Host D
```

Network telemetry can help reconstruct:

```text id="x5m7q3"
A → B
B → C
C → D
```

Correlate each transition with:

* identity
* service
* authentication
* process
* timing

This allows defenders to reconstruct the movement path.

## 22. Network Detection by Service

| Service    | Network Evidence       | Additional Correlation                          |
| ---------- | ---------------------- | ----------------------------------------------- |
| SMB        | TCP 445/139            | Authentication, share access, endpoint activity |
| WinRM      | TCP 5985/5986          | Authentication, PowerShell, process activity    |
| RDP        | TCP 3389               | Remote-interactive logon, session telemetry     |
| RPC        | TCP 135 + dynamic RPC  | Authentication, WMI/DCOM, process activity      |
| SSH        | TCP 22/custom port     | SSH authentication, shell/process activity      |
| HTTP/HTTPS | TCP 80/443/custom      | Application logs, authentication, process       |
| Database   | Database-specific port | Database logs, account activity                 |
| NFS        | TCP/UDP 2049           | Export/mount activity, endpoint evidence        |

## 23. Network Investigation Workflow

Use:

```text id="b6m8q3"
Identify Connection
      ↓
Identify Source
      ↓
Identify Destination
      ↓
Identify Port / Service
      ↓
Identify Account
      ↓
Correlate Authentication
      ↓
Correlate Process
      ↓
Determine Expected Relationship
      ↓
Investigate Unexpected Activity
      ↓
Reconstruct Movement Chain
```

## 24. Network Detection Checklist

```text id="r4m8q2"
[ ] Source IP available
[ ] Destination IP available
[ ] Source host identifiable
[ ] Destination host identifiable
[ ] Destination port recorded
[ ] Protocol identifiable
[ ] Connection timestamps available
[ ] Relevant internal traffic monitored
[ ] Administrative services monitored
[ ] Authentication data available
[ ] Endpoint/process telemetry available
[ ] Network segmentation documented
[ ] Pivot paths understood
[ ] Logs centralized where appropriate
[ ] Retention is sufficient for investigation
```

## 25. Offensive Assessment Perspective

During an authorized assessment, ask:

```text id="m7q3v8"
I moved from Host A to Host B.
        ↓
Was A → B visible?
        ↓
Was the service visible?
        ↓
Was the source identifiable?
        ↓
Was the target identifiable?
        ↓
Was the identity visible?
        ↓
Could the connection be correlated with process activity?
        ↓
Could defenders reconstruct the movement?
```

This helps identify network-monitoring gaps without assuming that lack of visibility means lack of activity.

## 26. Important Limitations

Network telemetry can be incomplete because of:

* encrypted traffic
* blind spots
* unmanaged network segments
* missing east-west monitoring
* NAT
* proxies
* load balancers
* VPNs
* cloud networking
* sensor placement
* packet loss
* limited retention

Therefore:

> A network sensor that does not observe a connection does not necessarily prove that the connection did not occur.

## 27. Evidence Record

For each relevant network event:

```text id="q8m3v6"
Timestamp:
Source Host:
Source IP:
Source Account:
Destination Host:
Destination IP:
Destination Port:
Protocol:
Service:
Process:
Authentication Evidence:
Network Evidence:
Endpoint Evidence:
Expected / Unexpected:
Related Movement:
Assessment Notes:
```

## What Next?

Continue to [`Defensive-Considerations.md`](Defensive-Considerations.md).

That file brings authentication, endpoint, and network observations together and turns them into practical defensive considerations for reducing lateral-movement risk and improving visibility.
