# Internal Network Access

Internal network access begins when an existing foothold provides reachability to systems or services that were not directly accessible from the original assessment position.

The objective is not simply to "get inside."

The objective is to establish exactly:

```text id="q6m3v8"
What became reachable?
        ↓
Which hosts are relevant?
        ↓
Which services are exposed?
        ↓
Which identities can authenticate?
        ↓
What is authorized?
        ↓
What new position or access path exists?
```

The workflow is:

```text id="r8k2p5"
Pivot Established
      ↓
Validate Internal Connectivity
      ↓
Identify Internal Network
      ↓
Discover Relevant Hosts
      ↓
Discover Services
      ↓
Prioritize Targets
      ↓
Map Credentials / Access
      ↓
Validate Authentication
      ↓
Validate Authorization
      ↓
Establish Access
      ↓
Re-enumerate
```

This workflow is intended for authorized assessments and controlled labs.

## Objective

Use an established pivot to systematically assess newly reachable internal resources.

The key goals are:

* verify that the pivot actually works
* determine what internal network is reachable
* identify relevant hosts
* identify exposed services
* distinguish network access from service access
* map available credentials to internal targets
* validate authorized access
* establish the new assessment position
* determine whether another pivot is required

## 1. Confirm the Pivot

Before enumerating the internal network, verify that the pivot path is functioning.

The basic chain is:

```text id="m4v7q1"
Assessment Machine
       ↓
Pivot
       ↓
Internal Target
```

Confirm:

```text id="z8c3n5"
Pivot Reachability
Target Reachability
Target Port
Service Response
```

Do not begin broad discovery until the basic path is known to work.

## 2. Identify the Internal Network

Determine the network that became reachable through the pivot.

Useful information includes:

```text id="w6p2r9"
Network Range:
Gateway:
Pivot Interface:
DNS:
Known Hosts:
Known Services:
```

On the pivot host:

### Linux

```bash id="g3m8v5"
ip addr
ip route
ip neigh
```

### Windows

```powershell id="n5q7c2"
ipconfig
Get-NetIPConfiguration
Get-NetRoute
```

This provides the pivot's view of the internal environment.

## 3. Compare Before and After

One of the most important steps is comparing network visibility.

Before pivoting:

```text id="u4k9m2"
Assessment Machine
      ↓
Networks A
```

After pivoting:

```text id="p7v3x8"
Assessment Machine
      ↓
Pivot
      ↓
Networks A + B
```

The difference represents the newly available assessment surface.

Record:

```text id="c6m2r9"
Previously Reachable:
Newly Reachable:
Potentially Restricted:
```

## 4. Discover Internal Hosts

Use focused host discovery based on the authorized scope.

Potential evidence sources include:

* known DNS records
* routing information
* ARP/neighbour tables
* domain information
* application configuration
* service references
* previously observed connections
* targeted connectivity checks

The workflow is:

```text id="q8n4w6"
Internal Network
      ↓
Potential Hosts
      ↓
Reachability Validation
      ↓
Relevant Hosts
```

Do not assume every address in a subnet represents an active or relevant system.

## 5. Discover Internal Services

Once relevant hosts are identified, determine which services are exposed.

Common examples include:

```text id="v5m8q2"
22    SSH
53    DNS
80    HTTP
443   HTTPS
135   RPC
139   NetBIOS/SMB
445   SMB
389   LDAP
636   LDAPS
3389  RDP
5985  WinRM
5986  WinRM over HTTPS
```

These are examples, not assumptions.

A service may use a non-standard port.

The workflow is:

```text id="f2r7m9"
Host
 ↓
Reachable Port
 ↓
Service Identification
 ↓
Authentication Method
 ↓
Authorization Model
```

## 6. Prioritize Internal Hosts

Do not treat every discovered host equally.

Useful prioritization factors include:

* domain-controller role
* server function
* administrative services
* file-sharing services
* internal applications
* databases
* known account ownership
* previously observed relationships
* assessment objectives

A useful workflow is:

```text id="k9m4x7"
Internal Hosts
      ↓
Service Identification
      ↓
Role Identification
      ↓
Relevant Target
```

## 7. Build an Internal Target Map

Create a simple map:

| Host     | IP          | Service | Role         | Authentication | Status  |
| -------- | ----------- | ------- | ------------ | -------------- | ------- |
| SERVER01 | 10.10.20.10 | SMB     | File Server  | Domain         | Unknown |
| SERVER02 | 10.10.20.20 | HTTP    | Application  | App-specific   | Unknown |
| SERVER03 | 10.10.20.30 | SSH     | Linux Server | SSH            | Unknown |

This helps prevent repeated or random testing.

Update the table as evidence improves.

## 8. Map Credentials to Targets

Return to the credential-and-access workflow.

For each known authentication material, ask:

```text id="x3v7q5"
Which identity?
      ↓
Local or Domain?
      ↓
Which protocol?
      ↓
Which service?
      ↓
Which internal target?
      ↓
Is the target authorized for that identity?
```

Examples:

```text id="m8q2c6"
SSH Key
 ↓
Linux Account
 ↓
SSH
 ↓
Internal Linux Server
```

or:

```text id="r4n9v7"
Domain Account
 ↓
Kerberos / NTLM
 ↓
SMB / WinRM / RDP
 ↓
Internal Windows Server
```

## 9. Separate Network Access From Service Access

A target can be reachable without providing useful access.

For example:

```text id="u6k3m8"
Host Reachable
      ↓
Port 445 Reachable
      ↓
SMB Service
      ↓
Authentication Required
```

Similarly:

```text id="w5p8r2"
Host Reachable
      ↓
Port 22 Reachable
      ↓
SSH
      ↓
Credentials Required
```

Always move through the layers sequentially.

## 10. Validate Internal Web Services

For an internal HTTP/HTTPS service:

```text id="q7m4v9"
Host
 ↓
TCP Port
 ↓
HTTP / HTTPS
 ↓
Application
 ↓
Authentication
 ↓
Authorization
```

Once the service is accessible, identify:

* application type
* authentication mechanism
* exposed functionality
* relevant endpoints
* authorized testing scope

Do not treat the existence of an internal web service as evidence of a vulnerability.

## 11. Validate Internal SMB Services

For SMB:

```text id="j2n8x5"
Host
 ↓
445
 ↓
SMB
 ↓
Authentication
 ↓
Share Enumeration
 ↓
Authorization
```

Useful questions include:

* Can the account authenticate?
* Which shares are visible?
* Which shares are readable?
* Which shares are writable?
* Is administrative access present?
* Is remote management available separately?

Keep authentication and authorization distinct.

## 12. Validate Internal SSH Services

For SSH:

```text id="v8p3k6"
Host
 ↓
22 / Configured Port
 ↓
SSH
 ↓
Authentication
 ↓
Shell
 ↓
Authorization
```

If access succeeds, immediately establish the new position:

```bash id="s5m9q2"
whoami
id
hostname
ip addr
ip route
```

Then determine whether this host provides access to another network.

## 13. Validate Windows Management Services

For Windows internal targets, common services include:

```text id="x4q7m2"
SMB
WinRM
RDP
RPC / WMI
```

Each has different authentication and authorization requirements.

For example:

```text id="c9v3n8"
WinRM Reachable
      ↓
WinRM Authentication
      ↓
Remote PowerShell Authorization
```

Do not treat TCP reachability as proof of remote management access.

## 14. Validate Active Directory Services

If the internal network contains Active Directory infrastructure, identify:

* domain controllers
* DNS
* LDAP/LDAPS
* Kerberos
* domain member systems
* administrative services

Useful Windows checks include:

```cmd id="m7r2p5"
whoami
echo %USERDOMAIN%
echo %USERDNSDOMAIN%
klist
nltest /dsgetdc:DOMAIN
```

The purpose is to understand the internal domain position.

## 15. Internal DNS

Internal DNS can reveal useful infrastructure information.

Determine:

```text id="k3v8n6"
Which DNS server is being used?
Can internal hostnames resolve?
Which domain names are visible?
Are service names resolvable?
```

DNS failures should be separated from routing failures.

For example:

```text id="p5m9x2"
Hostname Fails
      ↓
IP Works
```

suggests a naming/DNS problem rather than necessarily a network problem.

## 16. Internal Network Segmentation

The newly accessible network may itself contain multiple segments.

For example:

```text id="f8q3m6"
Pivot
  ↓
Application Network
  ├── App Server
  └── Internal Services
        ↓
     Management Network
        ├── Admin Server
        └── Domain Controller
```

If another host provides access to a network that the current pivot cannot reach, that host may become a second pivot.

The workflow becomes:

```text id="n4v7p1"
Current Pivot
      ↓
Internal Target
      ↓
Does Target Have Additional Network Access?
      ↓
Yes
      ↓
Assess as Potential Second Pivot
```

## 17. Second Pivot Decision

Do not create another pivot simply because another host has multiple interfaces.

First establish:

```text id="z6m2q8"
Current Host
      ↓
Current Reachable Networks
      ↓
Candidate Second Pivot
      ↓
Additional Network
      ↓
Is Additional Network Relevant?
```

Only then determine whether another pivot is necessary.

## 18. Authentication Workflow

Once an internal service is identified:

```text id="r3x8m5"
Target
 ↓
Service
 ↓
Authentication Method
 ↓
Available Credential
 ↓
Authentication
 ↓
Authorization
```

Possible authentication materials include:

* valid accounts
* passwords
* SSH keys
* NTLM hashes
* Kerberos tickets
* application credentials
* other authorized authentication material

The credential type must match the target's authentication mechanism.

## 19. Internal Access Does Not Equal Lateral Movement

This distinction is important.

For example:

```text id="q5v9c3"
SOCKS Proxy
   ↓
Internal HTTP Server
```

means:

```text
Network Access
```

It does not automatically mean:

```text
Host Compromise
```

Likewise:

```text id="m7n2x8"
Internal SMB Share
```

may provide file access without providing a remote shell.

Every resulting access path must be evaluated separately.

## 20. Post-Access Enumeration

When access to an internal host is obtained, restart the position-assessment workflow.

### Windows

```powershell id="c4q8m1"
whoami
hostname
ipconfig
route print
whoami /groups
whoami /priv
```

### Linux

```bash id="r6v3p9"
whoami
id
hostname
ip addr
ip route
ss -tunap
```

Then inspect:

```text id="w2k7m5"
Credentials / Access
Services
Network Context
Trust / Domain Context
Potential Pivot Paths
```

## 21. Internal Network Decision Tree

Use this workflow:

```text id="y8m4q2"
Pivot Established
      ↓
Can Internal Target Be Reached?
      │
 ┌────┴────┐
 No        Yes
 ↓           ↓
Troubleshoot  Identify Service
Pivot Path         ↓
              Authenticate?
                │
           ┌────┴────┐
          No        Yes
          ↓           ↓
      Find / Validate  Check
      Credentials      Authorization
                         │
                    ┌────┴────┐
                   No        Yes
                   ↓           ↓
             Record Limit   Establish Access
                               ↓
                         Validate Position
                               ↓
                         Re-enumerate
                               ↓
                    Additional Network?
                          │
                     ┌────┴────┐
                    No        Yes
                    ↓           ↓
                  Continue    Assess Next
                  Workflow     Pivot
```

## 22. Failure Classification

### Internal Network Cannot Be Reached

Investigate:

* pivot configuration
* routing
* interfaces
* firewall
* segmentation

### Host Reachable but Port Closed

Investigate:

* service state
* host firewall
* correct port
* non-standard service configuration

### Port Open but Service Unknown

Perform service identification before selecting an authentication method.

### Service Reachable but Authentication Fails

Investigate:

* account
* credential
* protocol
* domain context
* authentication policy

### Authentication Succeeds but Authorization Fails

The identity is valid but does not have the required permissions.

### Access Succeeds but No Useful Next Step Exists

Record the access and return to target prioritization.

Not every successful connection produces another movement opportunity.

## 23. Evidence Recording

Maintain an internal-access record:

```text id="p9x3m7"
Pivot Host:
Internal Network:
Target Host:
Target IP:
Target Port:
Service:
Role:
Authentication Method:
Authentication Result:
Authorization Result:
Access Obtained:
New Host:
New Network:
Potential Next Pivot:
Next Target:
```

Example:

```text id="v4m8q2"
Pivot Host: WEB01
Internal Network: 10.10.20.0/24
Target: APP01
Service: HTTPS
Role: Internal Application
Authentication: Successful
Authorization: Application access confirmed
New Host Access: No
New Network: Not identified
Next Step: Enumerate authorized application functionality
```

## 24. Keep the Internal Map Updated

As new evidence is discovered, update the internal target map.

A useful structure is:

```text
Internal Network
│
├── Host A
│   ├── Service
│   └── Access
│
├── Host B
│   ├── Service
│   └── Access
│
└── Host C
    ├── Service
    └── Potential Pivot
```

This prevents losing track of how systems relate to one another.

## 25. Operational Rules

### Validate the Pivot First

Never assume that the forwarding or proxy mechanism works.

### Enumerate From the New Position

The pivot provides a different network perspective.

### Use Focused Discovery

Prioritize systems and services relevant to the authorized assessment.

### Separate Access Layers

Always distinguish:

```text
Network
Service
Authentication
Authorization
Host Access
```

### Document the Path

Record:

```text
Source
 ↓
Pivot
 ↓
Network
 ↓
Target
 ↓
Service
 ↓
Authentication
 ↓
Authorization
```

### Reassess After Every Successful Movement

A new host may provide a new network path or new authentication material.

## What Not to Assume

```text id="h6q2v9"
Internal Network Reachable
   ≠
Every Internal Host Reachable
```

```text id="m3x8p5"
Host Reachable
   ≠
Service Accessible
```

```text id="c7r2n6"
Service Accessible
   ≠
Authentication Success
```

```text id="w9k4m1"
Authentication Success
   ≠
Authorization
```

```text id="p5v8q3"
Internal Access
   ≠
Host Compromise
```

```text id="x2n7m4"
One Pivot
   ≠
Entire Internal Network
```

## Section Completion

This completes the core files in `09-Pivoting-and-Internal-Network-Access`:

* [`Pivoting-Concepts.md`](Pivoting-Concepts.md)
* [`Port-Forwarding.md`](Port-Forwarding.md)
* [`SOCKS-Proxies.md`](SOCKS-Proxies.md)
* `Internal-Network-Access.md`

The next section is `10-Post-Movement-Workflow`.

## What Next?

Continue to `10-Post-Movement-Workflow/README.md`.

This section changes the focus from **how to reach another host** to **what to do immediately after movement succeeds**: validate the new access, reassess the host, discover new credentials and network paths, and decide whether to continue the movement chain.
