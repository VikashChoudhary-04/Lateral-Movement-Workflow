# Defensive Considerations

Lateral movement becomes significantly harder when an environment limits unnecessary access, protects authentication material, segments systems, and provides enough telemetry to reconstruct movement.

This file connects the offensive workflow to practical defensive controls.

The objective is not to provide a generic security checklist.

Instead, use the same movement chain from the rest of the repository:

```text id="m8q3v7"
Initial Access
      ↓
Identity
      ↓
Credential
      ↓
Target Discovery
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
      ↓
Re-enumeration
```

For each stage, ask:

> What control could prevent it, limit it, detect it, or help investigate it?

## Defensive Model

Use four broad control categories:

```text id="q4m8p2"
Prevent
  ↓
Limit
  ↓
Detect
  ↓
Respond
```

### Prevent

Stop unnecessary access before it occurs.

### Limit

Reduce what an already-compromised identity or host can reach.

### Detect

Generate and correlate useful telemetry.

### Respond

Contain the account, host, service, or network path involved.

No single control eliminates lateral movement.

## 1. Identity Security

Identity is central to lateral movement.

Defensive objectives include:

* minimize unnecessary privileges
* use separate administrative accounts where appropriate
* protect privileged accounts
* disable unnecessary accounts
* remove stale access
* review service-account permissions
* enforce appropriate authentication controls
* monitor privileged authentication

The core principle is:

```text id="v7m3q8"
Identity
   ↓
Minimum Required Access
```

## 2. Least Privilege

Least privilege reduces the impact of compromised credentials.

Instead of:

```text id="m4q8v2"
Compromised Account
      ↓
Many Hosts
      ↓
Administrative Access
```

prefer:

```text id="x8m3q5"
Compromised Account
      ↓
Limited Systems
      ↓
Limited Permissions
```

This can reduce the number of useful lateral-movement paths.

## 3. Credential Protection

Protect authentication material that could enable movement.

Relevant areas include:

* passwords
* password hashes
* Kerberos credentials
* SSH private keys
* SSH agent identities
* application credentials
* service-account credentials
* secrets stored in configuration
* cached credentials
* session tokens

The objective is:

```text id="p7m3q8"
Reduce Credential Exposure
        ↓
Reduce Credential Reuse
        ↓
Reduce Lateral-Movement Opportunities
```

## 4. Password Management

Defensive considerations include:

* strong authentication policies
* unique credentials where appropriate
* controlled administrative credentials
* removal of unnecessary credential reuse
* protection of privileged credentials
* monitoring for authentication anomalies

Password policy alone is not sufficient if credentials are exposed elsewhere.

## 5. Credential Reuse

Credential reuse can turn one compromised host into access to multiple systems.

Example:

```text id="q8m3v2"
Same Credential
   ↓
Host A
   ↓
Host B
   ↓
Host C
```

Reducing unnecessary reuse limits the movement chain.

Where operationally appropriate, use mechanisms that provide separate or managed administrative credentials.

## 6. Local Administrator Credential Management

Local administrative credentials deserve particular attention because shared local credentials can create direct host-to-host movement opportunities.

Defensive controls can include:

* unique local administrator credentials
* managed rotation
* restricted remote use
* privileged-access controls
* monitoring administrative authentication

The objective is to prevent:

```text id="m5q8v3"
Compromise Host A
      ↓
Recover Shared Local Credential
      ↓
Authenticate to Host B
      ↓
Repeat
```

## 7. Active Directory Security

For domain environments, review:

* privileged groups
* delegated permissions
* service accounts
* administrative logon paths
* Kerberos configuration
* domain-controller security
* trust relationships
* workstation/server administrative relationships
* credential exposure

The goal is to reduce unnecessary paths between identities and systems.

## 8. Administrative Tiering

Separate administrative access according to system sensitivity where appropriate.

Conceptually:

```text id="r7m3q8"
User Systems
      ↓
Server Administration
      ↓
Critical Infrastructure
      ↓
Domain / Identity Infrastructure
```

An identity used for lower-tier systems should not automatically provide unrestricted access to higher-tier systems.

## 9. Network Segmentation

Segmentation limits reachability.

Instead of:

```text id="x4m8q2"
Every Host
   ↕
Every Host
```

prefer controlled relationships:

```text id="p8m3q7"
User Network
     ↓
Approved Services
     ↓
Server Network
     ↓
Restricted Services
     ↓
Critical Network
```

The exact architecture depends on organizational requirements.

## 10. Restrict Remote Administration

Remote-management services should be exposed only where required.

Review:

* SMB
* WinRM
* RDP
* RPC
* SSH
* database services
* administrative web interfaces

Ask:

```text id="m7q3v8"
Who needs this service?
From where?
To which systems?
Using which accounts?
For what purpose?
```

## 11. SMB Controls

Consider:

* restrict SMB exposure
* disable unnecessary legacy protocols
* control administrative share access
* restrict inbound SMB by network policy
* monitor unusual SMB connections
* protect privileged credentials
* use appropriate authentication controls

A useful defensive model is:

```text id="q3m8v5"
Source
  ↓
SMB
  ↓
Target
```

Only expected source-target relationships should normally exist.

## 12. WinRM Controls

WinRM can be useful for administration but should be controlled.

Review:

* which systems accept WinRM
* which source networks can connect
* which accounts can use remote management
* authentication configuration
* logging
* PowerShell visibility

The objective is not necessarily to eliminate WinRM.

It is to limit it to justified administrative paths.

## 13. RDP Controls

Review:

* which systems expose RDP
* which users are allowed to log on
* source-network restrictions
* Network Level Authentication
* session controls
* privileged-account usage
* RDP authentication monitoring

A useful model is:

```text id="v6m3q8"
User
 ↓
Source
 ↓
RDP
 ↓
Target
 ↓
Authorization
```

Each relationship should have a legitimate reason.

## 14. SSH Controls

For SSH environments, review:

* exposed SSH services
* allowed users
* public-key management
* private-key protection
* root login configuration
* source restrictions
* authentication methods
* logging
* key rotation

The objective is to reduce unauthorized reuse of SSH credentials.

## 15. WMI and RPC Controls

Because Windows management can rely on RPC and related mechanisms, review:

* firewall policy
* allowed management hosts
* administrative permissions
* WMI authorization
* RPC exposure
* endpoint telemetry

Avoid treating TCP 135 as the only relevant control point.

## 16. Kerberos Security

For Active Directory environments, monitor:

* unusual Kerberos authentication
* service-ticket activity
* privileged accounts
* unusual source systems
* authentication failures
* service-account behavior
* domain-controller telemetry

The investigative model is:

```text id="m8q3v7"
Principal
 ↓
Authentication
 ↓
Ticket
 ↓
Service
 ↓
Target
```

## 17. NTLM Reduction

Where operationally feasible, organizations may evaluate reducing unnecessary NTLM usage.

Before changing authentication behavior, identify dependencies.

Review:

* legacy applications
* older systems
* services
* authentication dependencies
* compatibility requirements

A defensive change should account for legitimate workloads.

## 18. Service Accounts

Service accounts should have only the permissions required for their function.

Review:

```text id="q7m3v8"
Service Account
      ↓
Used By Which Service?
      ↓
On Which Hosts?
      ↓
What Permissions?
      ↓
What Remote Access?
```

This helps identify unnecessary movement paths.

## 19. Application Secrets

Application credentials should not be unnecessarily exposed in:

* source code
* configuration files
* deployment artifacts
* scripts
* environment configuration
* logs

Use appropriate secret-management mechanisms where available.

The objective is:

```text id="x5m8q2"
Application
    ↓
Controlled Secret
    ↓
Limited Access
```

rather than:

```text id="n7m3q8"
Application
    ↓
Plaintext Credential
    ↓
Reusable Across Infrastructure
```

## 20. Endpoint Detection

Endpoint security can provide visibility into:

* process creation
* network connections
* PowerShell
* WMI
* service activity
* scheduled tasks
* authentication
* suspicious child processes
* credential-related activity

The most useful telemetry connects:

```text id="m3q8v5"
Account
 +
Process
 +
Network
 +
Target
```

## 21. Centralized Logging

Centralized collection helps preserve evidence when a compromised endpoint is modified or becomes unavailable.

Collect appropriate telemetry from:

* domain controllers
* servers
* workstations
* critical applications
* network infrastructure
* security products

The goal is to maintain enough information to reconstruct:

```text id="p8m4q2"
Who
 ↓
From Where
 ↓
To Where
 ↓
Using What
 ↓
When
 ↓
What Happened
```

## 22. Log Retention

Detection quality depends on retention.

If relevant logs disappear before investigation begins:

```text id="v7m3q8"
Activity
 ↓
Log Generated
 ↓
Log Overwritten
 ↓
Investigation
 ↓
Missing Evidence
```

Retention should be aligned with:

* investigation requirements
* incident-response needs
* regulatory requirements
* organizational risk
* available storage

## 23. Network Monitoring

Network visibility should cover important internal communication where practical.

Pay particular attention to:

* workstation-to-server traffic
* administrative services
* server-to-server traffic
* critical infrastructure
* domain-controller communication
* unusual internal connections
* segmentation boundaries

The goal is to identify unexpected relationships rather than simply block ports.

## 24. Baseline Administrative Paths

Document expected administrative relationships.

Example:

```text id="q5m8v3"
ADMIN-01
   ↓
WinRM
   ↓
SERVER-01
```

Then investigate unexpected relationships such as:

```text id="x8m3q7"
USER-PC-17
   ↓
WinRM
   ↓
SERVER-01
```

The second relationship is not automatically malicious.

Its significance depends on whether it is expected and authorized.

## 25. Detection Correlation

Strong detection can combine:

```text id="m4q8v3"
Network
   +
Authentication
   +
Endpoint
   +
Process
   +
Identity
```

Example:

```text id="r7m3q8"
Unusual Source Host
       +
Successful Authentication
       +
Remote Management Service
       +
New Process
       ↓
Higher-Priority Investigation
```

The individual signals should still be validated.

## 26. Response Considerations

When lateral movement is suspected, response may involve:

```text id="p8m3q7"
Identify
  ↓
Contain
  ↓
Investigate
  ↓
Eradicate
  ↓
Recover
  ↓
Review
```

Possible containment actions depend on the incident and organizational procedures.

Examples can include:

* isolating affected endpoints
* disabling or restricting compromised accounts
* rotating exposed credentials
* restricting network paths
* blocking unauthorized remote services
* preserving evidence

Actions should follow established incident-response procedures.

## 27. Credential Rotation

If authentication material is confirmed exposed, determine:

* what account it belongs to
* where it is used
* what systems it can access
* whether it was reused
* whether related credentials are also exposed

Then apply appropriate credential-recovery procedures.

A credential rotation should consider service dependencies to avoid causing unnecessary outages.

## 28. Movement-Path Analysis

After an assessment or incident, reconstruct:

```text id="c6m8q2"
Initial Host
    ↓
Credential / Access
    ↓
Target
    ↓
Service
    ↓
Authentication
    ↓
Authorization
    ↓
New Host
    ↓
New Credential / Access
    ↓
Next Target
```

Then ask:

> Which control could have interrupted this chain earlier?

This is often more useful than focusing on a single technique.

## 29. Defensive Control Mapping

| Movement Stage      | Prevent / Limit                           | Detect                          | Investigate                   |
| ------------------- | ----------------------------------------- | ------------------------------- | ----------------------------- |
| Credential exposure | Secret management, credential protection  | Credential-access telemetry     | Identify exposed material     |
| Target discovery    | Segmentation, access controls             | Network scanning telemetry      | Identify source and targets   |
| Service access      | Firewall / ACLs                           | Network telemetry               | Source-to-target analysis     |
| Authentication      | Strong authentication, account controls   | Authentication logs             | Account/source analysis       |
| Authorization       | Least privilege                           | Access-denied/success telemetry | Permission analysis           |
| Remote session      | Restrict remote administration            | Endpoint/session logs           | Session reconstruction        |
| New-host access     | Network segmentation                      | Cross-host telemetry            | Movement-chain analysis       |
| Pivoting            | Network boundaries, restricted forwarding | Network/proxy telemetry         | Reconstruct intermediate host |
| Credential reuse    | Unique/managed credentials                | Authentication correlation      | Identify reuse path           |

## 30. Assessment Findings

A lateral-movement finding should describe the actual condition.

Prefer:

> A domain account authenticated from a workstation to a server over a remotely accessible management service and obtained the tested level of access.

Then document:

* source
* target
* account
* service
* authentication method
* authorization
* impact
* evidence
* recommended control

Avoid vague statements such as:

> Lateral movement is possible.

The finding should explain **why** it was possible.

## 31. Defensive Validation Questions

After completing a lateral-movement assessment, ask:

### Identity

> Can defenders identify which account moved?

### Source

> Can defenders identify the originating host?

### Target

> Can defenders identify the destination?

### Service

> Can defenders identify which remote service was used?

### Authentication

> Can defenders determine how authentication occurred?

### Process

> Can defenders identify the process involved?

### Sequence

> Can defenders reconstruct the movement chain?

### Response

> Are there established procedures for containing the affected identity and host?

## 32. Defensive Improvement Loop

Use:

```text id="m8q3v7"
Observed Movement
      ↓
Identify Enabling Condition
      ↓
Identify Missing Control
      ↓
Choose Preventive / Limiting Control
      ↓
Improve Detection
      ↓
Validate Again
```

This creates a feedback loop between offensive assessment and defensive improvement.

## 33. Practical Defensive Checklist

```text id="q4m8v3"
Identity
[ ] Privileged accounts reviewed
[ ] Service accounts reviewed
[ ] Unnecessary access removed
[ ] Credential reuse reduced

Authentication
[ ] Strong authentication controls
[ ] Authentication logging enabled
[ ] Privileged authentication monitored
[ ] Kerberos / NTLM activity understood

Remote Services
[ ] SMB exposure reviewed
[ ] WinRM exposure reviewed
[ ] RDP exposure reviewed
[ ] SSH exposure reviewed
[ ] WMI / RPC exposure reviewed

Network
[ ] Segmentation implemented where appropriate
[ ] Internal traffic monitored
[ ] Administrative paths documented
[ ] Pivot paths restricted

Endpoint
[ ] Process telemetry available
[ ] PowerShell telemetry available
[ ] Remote-management telemetry available
[ ] Endpoint security deployed

Logging
[ ] Centralized collection
[ ] Appropriate retention
[ ] Source and destination identifiable
[ ] Authentication events available
[ ] Process events available

Response
[ ] Account containment process exists
[ ] Credential rotation process exists
[ ] Endpoint isolation process exists
[ ] Movement-chain investigation process exists
```

## 34. What Good Defensive Visibility Looks Like

A mature environment should make it possible to reconstruct something close to:

```text id="x7m3q8"
10:01:12
WORKSTATION-01
     ↓
User: example\user
     ↓
TCP 5985
     ↓
SERVER-01
     ↓
WinRM Authentication
     ↓
Remote Session
     ↓
PowerShell Process
     ↓
10:01:20
Additional Internal Connection
```

The exact telemetry will vary.

The important property is **correlation**.

## 35. Final Defensive Principle

The strongest defensive position is not simply:

> Block every lateral-movement technique.

A practical security model is:

```text id="p5m8q2"
Reduce Unnecessary Access
        ↓
Protect Credentials
        ↓
Restrict Reachability
        ↓
Limit Privileges
        ↓
Monitor Authentication
        ↓
Monitor Endpoint Activity
        ↓
Monitor Network Relationships
        ↓
Correlate Events
        ↓
Respond Quickly
```

Lateral movement is a chain.

Breaking any important link can reduce the ability of a compromised identity or host to move further.

## What Next?

Section 13 — **Detection and Defense** is now complete.

Continue to `References/README.md`.

That will be the final file in the repository and will provide the reference structure for the concepts, protocols, tools, standards, and documentation used throughout the workflow.
