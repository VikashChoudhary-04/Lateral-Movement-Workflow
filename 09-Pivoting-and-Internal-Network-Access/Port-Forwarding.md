# Port Forwarding

Port forwarding is a focused pivoting technique that transports traffic from one endpoint through a pivot host to another endpoint.

It is especially useful when the assessment requires access to **one known internal service** rather than broad access to an entire network.

The core workflow is:

```text id="4v7m2p"
Identify Internal Service
        ↓
Identify Pivot Host
        ↓
Confirm Pivot → Target Connectivity
        ↓
Choose Forwarding Direction
        ↓
Create Forward
        ↓
Validate Local Endpoint
        ↓
Validate Target Port
        ↓
Validate Service
        ↓
Authenticate
        ↓
Validate Authorization
        ↓
Record Result
```

This workflow is intended for authorized assessments and controlled labs.

## Objective

The objective is to make a specific internal service reachable through an existing authorized position.

Before forwarding anything, establish:

* source host
* pivot host
* target host
* target port
* protocol
* required traffic direction
* appropriate forwarding mechanism
* expected result

A useful representation is:

```text id="n2q6s8"
Source
  ↓
Pivot
  ↓
Internal Target:Port
```

## Port Forwarding vs Pivoting

Port forwarding is one implementation of the broader pivoting concept.

```text id="w5r8k3"
Pivoting
= Using an existing position to reach another network location
```

```text id="m1v7q4"
Port Forwarding
= Transporting a specific connection through that position
```

For example:

```text id="z8c3p6"
Assessment Machine
       ↓
127.0.0.1:8080
       ↓
Pivot
       ↓
10.10.20.15:80
```

The local port represents the forwarding endpoint. The actual service remains on the internal target.

## 1. Define the Target

Start with a known target.

Record:

```text id="h4n9s2"
Target Host:
Target IP:
Target Port:
Protocol:
Expected Service:
Reason for Access:
```

Example:

```text id="b7m3x8"
Target: 10.10.20.15
Port: 8080
Protocol: HTTP
Service: Internal Web Application
```

Avoid creating forwards to arbitrary internal addresses when the target is not relevant to the assessment.

## 2. Identify the Pivot

The pivot must be able to reach the target.

Conceptually:

```text id="r6k2v9"
Assessment Machine
       ↓
Pivot Host
       ↓
Target
```

The critical connection is:

```text id="j3p8m5"
Pivot → Target
```

If the pivot cannot reach the target, forwarding through that pivot will not solve the problem.

## 3. Confirm Pivot-to-Target Connectivity

Test connectivity from the pivot host itself.

For Windows:

```powershell id="q9v4c1"
Test-NetConnection TARGET -Port PORT
```

For Linux:

```bash id="t6m2y8"
nc -vz TARGET PORT
```

For example:

```bash id="p4w7n3"
nc -vz 10.10.20.15 8080
```

Interpret the result:

```text id="f8k3q6"
Pivot → Target Port
       │
   ┌───┴────┐
  Open    Closed
   ↓         ↓
Continue   Investigate
```

This test should happen before troubleshooting the forwarding mechanism.

## 4. Determine the Forwarding Direction

Direction matters.

### Local Forward

A local endpoint on the assessment machine forwards through the pivot toward the internal target.

```text id="c2m7x9"
Assessment Machine
127.0.0.1:LOCAL_PORT
        ↓
      Pivot
        ↓
Internal Target:TARGET_PORT
```

This is useful when the assessment machine needs to consume the internal service.

### Remote Forward

A listener is created on the remote/pivot side and forwards traffic toward another destination.

Conceptually:

```text id="v5n8p2"
Remote / Pivot Listener
        ↓
   Forwarding Path
        ↓
Destination
```

The correct direction depends on which side can accept incoming connections and which side can reach the destination.

## 5. Local Port Forwarding

A local forward creates a local endpoint representing an internal service.

For example:

```text id="m8q3w6"
127.0.0.1:8080
       ↓
     Pivot
       ↓
10.10.20.15:80
```

The application can then connect to:

```text id="k4v9r1"
127.0.0.1:8080
```

while the forwarding mechanism transports the connection to the internal service.

The local endpoint does not mean the service actually runs on the assessment machine.

## 6. Example Local Forward Workflow

Suppose:

```text id="g2n7x4"
Pivot: 10.10.10.25
Internal Target: 10.10.20.15
Target Service: HTTP
Target Port: 80
Local Port: 8080
```

The workflow is:

```text id="d6p3s8"
Identify Target
      ↓
Identify Pivot
      ↓
Confirm Pivot → 10.10.20.15:80
      ↓
Create Local Forward
      ↓
Connect to 127.0.0.1:8080
      ↓
Validate HTTP Response
```

The exact forwarding command depends on the authorized tool and operating environment.

The important part is understanding the traffic path rather than memorizing a command.

## 7. Validate the Local Listener

After establishing the forward, confirm that the local endpoint is listening.

On Windows:

```powershell id="s4m8q1"
Get-NetTCPConnection -State Listen
```

On Linux:

```bash id="y7c2n5"
ss -lnt
```

The expected result should show the chosen local port.

For example:

```text id="x9v3k6"
127.0.0.1:8080
```

A listener confirms the local forwarding endpoint exists.

It does **not** yet prove that the internal target is reachable.

## 8. Validate the Forwarded Connection

The next layer is:

```text id="q5r2m8"
Local Listener
      ↓
Forwarding Path
      ↓
Internal Target
```

For HTTP, test the application endpoint through the local port.

For example:

```text id="a3k7v1"
http://127.0.0.1:8080
```

For TCP services, use an appropriate client for that protocol.

The important evidence is:

```text id="e6p9s4"
Local Endpoint ✓
      ↓
Target Connection ✓
      ↓
Service Response ✓
```

## 9. Separate Forwarding From Application Access

A forward can exist while the application remains inaccessible.

For example:

```text id="j8m4q2"
Local Listener ✓
      ↓
Target Port ✗
```

or:

```text id="c5v9x7"
Target Port ✓
      ↓
Application ✗
```

Classify the failure before changing the pivot.

## 10. Port Forwarding for SSH

Suppose an internal SSH service exists:

```text id="u7n3m6"
10.10.20.30:22
```

A local forward might expose it as:

```text id="p2r8k4"
127.0.0.1:2222
```

The conceptual path becomes:

```text id="f6w1q9"
SSH Client
   ↓
127.0.0.1:2222
   ↓
Pivot
   ↓
10.10.20.30:22
```

After the transport works, SSH authentication remains a separate step.

```text id="n4v7c2"
TCP Connectivity
      ↓
SSH Service
      ↓
Authentication
      ↓
Authorization
```

## 11. Port Forwarding for SMB

An internal SMB service may be exposed through a local forwarding endpoint.

Conceptually:

```text id="k3x8m5"
SMB Client
   ↓
Local Forward
   ↓
Pivot
   ↓
Internal Server:445
```

The workflow remains:

```text id="r9q2v6"
Forward
  ↓
TCP 445
  ↓
SMB
  ↓
Authentication
  ↓
Share Authorization
```

A successful TCP connection does not establish SMB access by itself.

## 12. Port Forwarding for Web Applications

Web applications are a common reason to use focused forwarding.

Example:

```text id="w6p3n8"
Internal Web Server
10.10.20.15:8080
```

The workflow is:

```text id="s2m7q4"
Identify Web Service
      ↓
Confirm Pivot → Target:8080
      ↓
Create Forward
      ↓
Open Local Endpoint
      ↓
Receive HTTP Response
      ↓
Enumerate Application
```

Once the application is reachable, continue with the appropriate web-application assessment workflow.

## 13. Remote Port Forwarding

Remote forwarding reverses the traffic relationship.

Conceptually:

```text id="g5k1v9"
Remote Listener
       ↓
Forwarding Path
       ↓
Destination on Another Side
```

This can be useful when:

* inbound connections to the assessment machine are restricted
* the pivot can initiate outbound connections
* the desired traffic path requires the listener to exist on the remote side

Always determine:

```text id="v8n4c2"
Who can initiate?
Who can listen?
Who can reach the destination?
Which firewall rules apply?
```

before choosing the direction.

## 14. Choosing the Local Port

A local port should:

* not conflict with an existing service
* be easy to identify
* be appropriate for the assessment
* not expose unnecessary interfaces

Prefer a loopback binding when remote access to the forwarded service is unnecessary.

Conceptually:

```text id="m3q7x9"
127.0.0.1:LOCAL_PORT
```

is more controlled than:

```text id="h6v2k8"
0.0.0.0:LOCAL_PORT
```

when only the assessment machine needs access.

The exact behavior depends on the forwarding tool.

## 15. Failure Classification

### Local Listener Does Not Exist

```text id="r1n5c7"
Forward Setup
     ↓
No Local Listener
```

Investigate:

* forwarding configuration
* selected port
* process state
* tool errors
* port conflicts

### Local Listener Exists but Connection Fails

```text id="z4m8p2"
Local Listener ✓
      ↓
Target Connection ✗
```

Check:

* pivot-to-target connectivity
* target port
* firewall
* forwarding direction
* target address

### Target Port Works but Service Fails

```text id="q7c3v9"
TCP ✓
Service ✗
```

Check:

* protocol
* application configuration
* TLS requirements
* service-specific behavior

### Service Works but Authentication Fails

```text id="t2k6m8"
Network ✓
Service ✓
Authentication ✗
```

Return to credential and access discovery.

### Authentication Works but Authorization Fails

```text id="b5x9q3"
Authentication ✓
Authorization ✗
```

The account may not have the required permissions.

## 16. Multi-Hop Port Forwarding

Some environments require more than one pivot:

```text id="j6p2w8"
Assessment Machine
       ↓
Pivot A
       ↓
Pivot B
       ↓
Internal Target
```

The workflow becomes:

```text id="n8v4m1"
Validate Pivot A
      ↓
Validate Pivot A → Pivot B
      ↓
Establish Second Forward
      ↓
Validate Pivot B → Target
      ↓
Access Target Service
```

Do not assume that because Pivot A can reach Pivot B, Pivot B can reach the final target.

Each hop must be validated.

## 17. Keep the Forwarding Path Documented

For every forward, document:

```text id="c4y7p2"
Local Listener:
Forwarding Host:
Intermediate Hosts:
Target:
Target Port:
Protocol:
Direction:
Validation:
Authentication:
Authorization:
Purpose:
```

Example:

```text id="m7k3x9"
Local Listener: 127.0.0.1:8080
Pivot: WEB01
Target: 10.10.20.15:80
Protocol: HTTP
Direction: Local → Pivot → Target
Validation: HTTP response received
Authentication: Not required
Purpose: Authorized internal web assessment
```

## 18. Security and Scope Considerations

### Bind Locally When Appropriate

If only the assessment machine needs the service, use a loopback listener where supported.

### Minimize Exposure

Do not expose an internal service to additional systems unnecessarily.

### Forward Only Required Ports

Avoid creating broad forwarding rules when one service is sufficient.

### Remove Temporary Access

When the assessment no longer requires the forward, close it.

### Protect Sensitive Traffic

Understand whether the forwarded connection contains credentials, session tokens, or other sensitive information.

## 19. Decision Workflow

Use this workflow:

```text id="u3r8m5"
Known Internal Service
        ↓
Identify Pivot
        ↓
Can Pivot Reach Target?
        │
   ┌────┴────┐
  No        Yes
  ↓           ↓
Find another  Determine Direction
pivot             ↓
             One Service?
              │       │
             Yes     No
              ↓       ↓
        Port Forward  Consider
                      SOCKS / Routing
              ↓
       Establish Forward
              ↓
       Validate Listener
              ↓
       Validate Target Port
              ↓
       Validate Service
              ↓
       Authenticate
              ↓
       Validate Authorization
              ↓
       Record Result
```

## 20. What Not to Assume

```text id="k5q9m2"
Local Listener
   ≠
Target Reachable
```

```text id="p8v3x6"
Target Port Reachable
   ≠
Service Working
```

```text id="r4m7n1"
Service Working
   ≠
Authentication Success
```

```text id="c2w6q8"
Authentication Success
   ≠
Authorization
```

```text id="j9s3v5"
Forward Established
   ≠
Target Compromised
```

The forward provides a network path. Everything above that path must still be validated.

## What Next?

Continue to [`SOCKS-Proxies.md`](SOCKS-Proxies.md).

That file covers situations where multiple internal hosts or services need to be accessed through the same pivot rather than forwarding each individual port.
