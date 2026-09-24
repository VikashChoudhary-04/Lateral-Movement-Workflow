# SOCKS Proxies

A SOCKS proxy provides an application-level way to route supported connections through a pivot host toward internal destinations.

It is useful when the assessment requires access to **multiple internal hosts or services** and creating a separate port forward for every service would be inefficient.

The core workflow is:

```text id="q7m2v9"
Identify Internal Network
        ↓
Identify Pivot Host
        ↓
Confirm Pivot Reachability
        ↓
Determine Required Internal Access
        ↓
Establish SOCKS Proxy
        ↓
Configure Supported Application
        ↓
Validate Proxy Path
        ↓
Validate Target
        ↓
Validate Service
        ↓
Authenticate
        ↓
Validate Authorization
        ↓
Continue Internal Enumeration
```

This workflow is intended for authorized assessments and controlled labs.

## Objective

The objective is to use a pivot host as a controlled proxy for reaching internal services that are not directly accessible from the assessment machine.

The key questions are:

* Why is a proxy required?
* Which internal networks can the pivot reach?
* Which hosts/services are relevant?
* Does the application support SOCKS?
* Is the proxy path functioning?
* Can the target service be reached through the proxy?
* Does authentication succeed?
* What additional internal resources become visible?

## SOCKS vs Port Forwarding

The main distinction is scope.

### Port Forwarding

Usually maps a specific local endpoint to a specific target:

```text id="d5w8q1"
Local Port
    ↓
Pivot
    ↓
Target Host:Port
```

### SOCKS Proxy

Provides a reusable proxy endpoint through which supported applications can request connections to different destinations:

```text id="n3k7v4"
Application
     ↓
SOCKS Proxy
     ↓
Pivot
     ↓
Internal Host A
     ↓
Internal Host B
     ↓
Internal Host C
```

The proxy therefore avoids creating a separate local listener for every destination.

## 1. Identify Why a SOCKS Proxy Is Needed

Before creating the proxy, determine whether it solves the actual problem.

A SOCKS proxy may be appropriate when:

* multiple internal hosts need assessment
* multiple ports need access
* targets are dynamically selected
* applications already support SOCKS
* creating individual forwards would be cumbersome

A single port forward may be better when:

```text id="v6m2r8"
One Host
+
One Port
+
Known Service
```

The rule is:

```text id="p4x9c3"
Choose the smallest mechanism that satisfies the objective.
```

## 2. Identify the Pivot Host

The pivot must have network access to the internal targets.

Determine:

```text id="h7q3m1"
Pivot Host:
Pivot IP:
Interfaces:
Routes:
Internal Networks:
```

Linux:

```bash id="w5k8n2"
ip addr
ip route
```

Windows:

```powershell id="r9v4c6"
ipconfig
Get-NetRoute
```

The critical relationship is:

```text id="y2m6p8"
Pivot
 ↓
Internal Target
```

## 3. Confirm Internal Connectivity From the Pivot

Before troubleshooting SOCKS, verify that the pivot itself can reach the intended target.

For example:

```text id="c8n3v5"
Pivot
 ↓
10.10.20.15:80
```

Test the relevant port from the pivot using an appropriate connectivity check.

The result should be:

```text id="j4q7x2"
Pivot → Target
      ↓
Reachable?
```

If the pivot cannot reach the target, the proxy will not create connectivity that the pivot itself does not possess.

## 4. Understand the Traffic Path

A SOCKS connection can be visualized as:

```text id="s6p2w9"
Application
     ↓
SOCKS Client / Proxy Configuration
     ↓
SOCKS Endpoint
     ↓
Pivot Host
     ↓
Internal Target
```

For example:

```text id="k3v8m5"
Browser
  ↓
SOCKS
  ↓
Pivot WEB01
  ↓
10.10.20.15:8080
```

The target sees traffic arriving through the pivot path rather than through the assessment machine's original network interface.

## 5. Choose the SOCKS Version

SOCKS implementations commonly support:

```text id="x7n2q4"
SOCKS4
SOCKS5
```

SOCKS5 generally provides more capabilities, including authentication support and broader address-handling features.

Use the version supported by the authorized tooling and application.

Do not assume that every application supports every SOCKS feature.

## 6. Establish the Proxy

The exact command depends on the authorized pivoting tool.

The conceptual result should be:

```text id="m9c4w7"
Assessment Machine
       ↓
SOCKS Endpoint
       ↓
Pivot
       ↓
Internal Network
```

After establishment, verify that the SOCKS service is actually listening.

On Linux:

```bash id="v3k8p1"
ss -lnt
```

On Windows:

```powershell id="q5m2x9"
Get-NetTCPConnection -State Listen
```

A listening SOCKS endpoint confirms the local proxy exists.

It does not prove that internal targets are reachable.

## 7. Configure the Application

The application used for internal access must send its traffic through the SOCKS proxy.

Potential applications include:

* web browsers
* proxy-aware HTTP clients
* SSH clients
* security-testing tools that support SOCKS
* other TCP applications with proxy support

The configuration should specify:

```text id="a6r3n8"
Proxy Type:
Proxy Address:
Proxy Port:
Optional Authentication:
```

For example:

```text id="u4m7c2"
SOCKS5
127.0.0.1
1080
```

The actual port depends on the chosen setup.

## 8. Validate the Proxy Before Testing Targets

Start with a known internal target.

The validation chain is:

```text id="f8q2v6"
Application
    ↓
SOCKS Proxy
    ↓
Pivot
    ↓
Internal Target
```

Do not immediately perform broad internal discovery.

First prove that:

```text id="r5k9m3"
Proxy Connection ✓
        ↓
Target Connection ✓
        ↓
Service Response ✓
```

## 9. Example: Internal Web Application

Suppose:

```text id="p2x7c4"
Internal Web Server
10.10.20.15:8080
```

The workflow becomes:

```text id="n6v3q8"
Browser
  ↓
SOCKS Proxy
  ↓
Pivot
  ↓
10.10.20.15:8080
  ↓
HTTP Response
```

If the page loads, the proxy path is functioning at the application level.

Then continue with authorized web application enumeration.

## 10. Example: Internal SSH

Suppose:

```text id="m4w8k2"
Internal SSH
10.10.20.30:22
```

The workflow becomes:

```text id="x9q3v6"
SSH Client
   ↓
SOCKS Proxy
   ↓
Pivot
   ↓
10.10.20.30:22
   ↓
SSH Authentication
```

The SOCKS connection provides the network path.

Authentication still requires appropriate credentials or keys.

## 11. Example: Internal SMB

Suppose:

```text id="c7m2p9"
Internal SMB
10.10.20.40:445
```

The workflow is:

```text id="r3v8n5"
SMB Client
   ↓
SOCKS / Proxy Path
   ↓
Pivot
   ↓
10.10.20.40:445
   ↓
SMB Authentication
   ↓
Share Authorization
```

Not every SMB client handles SOCKS directly.

Use a proxy-capable or appropriately configured access method supported by the assessment environment.

## 12. Proxy-Aware vs Non-Proxy-Aware Tools

This is an important distinction.

### Proxy-Aware Application

The application can directly use a SOCKS proxy.

```text id="w6k1q4"
Application
   ↓
SOCKS
```

### Non-Proxy-Aware Application

The application does not understand SOCKS.

A separate proxy-routing mechanism may be required to transparently redirect its traffic.

Conceptually:

```text id="s8n4m2"
Application
    ↓
Traffic Redirection
    ↓
SOCKS Proxy
    ↓
Pivot
```

The exact mechanism depends on the operating system and authorized tooling.

## 13. Validate the Destination

When a connection fails, determine whether the problem is the proxy or the target.

Test in layers:

```text id="j5p8v3"
SOCKS Endpoint
      ↓
Pivot
      ↓
Target IP
      ↓
Target Port
      ↓
Service
```

For example:

```text id="b9m3q7"
SOCKS ✓
Pivot ✓
TCP Target ✗
```

suggests a pivot-to-target or routing problem.

Whereas:

```text id="e4x7n2"
SOCKS ✓
TCP ✓
Service ✗
```

suggests an application/service issue.

## 14. DNS Through SOCKS

DNS can create confusion during proxy-based access.

There are two separate questions:

```text id="r7k2m5"
Can the application resolve the hostname?
```

and:

```text id="v3n8q1"
Can the target connection itself be established through the pivot?
```

In some configurations, DNS resolution occurs locally; in others, the proxy mechanism can support remote name resolution.

If an internal hostname does not resolve, determine whether:

* local DNS lacks the internal record
* the pivot can resolve it
* the proxy configuration supports remote resolution
* the target can be addressed directly by IP for validation

Do not confuse DNS failure with target unreachability.

## 15. Internal Host Discovery Through a Proxy

Once the proxy is validated, focused internal discovery can begin.

The workflow is:

```text id="m8q4w6"
Internal Network
      ↓
Relevant Hosts
      ↓
Relevant Ports
      ↓
Service Identification
      ↓
Target Prioritization
```

Not every discovery tool can operate correctly through SOCKS.

Before relying on a tool, determine whether it supports:

* SOCKS
* proxy-aware TCP connections
* DNS through the proxy
* the required protocol

## 16. Avoid Broad Scanning by Default

A SOCKS proxy makes internal access easier, but that does not justify unrestricted scanning.

Prefer:

```text id="f6v2k9"
Known Network
      ↓
Assessment Scope
      ↓
Relevant Hosts
      ↓
Relevant Ports
```

rather than:

```text id="q3m8x1"
Entire Internal Network
      ↓
All Ports
      ↓
All Hosts
```

Focused discovery produces clearer evidence and reduces unnecessary traffic.

## 17. Authentication Through SOCKS

A proxy only transports traffic.

It does not automatically provide authentication.

For example:

```text id="h4p9v2"
SOCKS Access
    ↓
Internal SSH
    ↓
Need SSH Credential
```

or:

```text id="c6n3m8"
SOCKS Access
    ↓
Internal SMB
    ↓
Need SMB Authentication
```

Keep these layers separate:

```text id="w2r7q5"
Network Access
      ↓
Service Access
      ↓
Authentication
      ↓
Authorization
```

## 18. Failure Classification

### SOCKS Endpoint Unreachable

```text id="m5q8v3"
Application
    X
SOCKS
```

Investigate:

* proxy listener
* address
* port
* local firewall
* proxy configuration

### SOCKS Works but Target Is Unreachable

```text id="r2n7k4"
SOCKS ✓
   ↓
Pivot ✓
   ↓
Target ✗
```

Investigate:

* pivot routing
* target address
* internal firewall
* target availability

### Target Port Is Reachable but Service Fails

```text id="x8v3p6"
TCP ✓
Service ✗
```

Investigate the target application.

### Service Works but Authentication Fails

```text id="k4m9q2"
Network ✓
Service ✓
Authentication ✗
```

Return to credential and access discovery.

### Authentication Works but Authorization Fails

```text id="n6w2c8"
Authentication ✓
Authorization ✗
```

The account lacks the required access.

## 19. SOCKS and Port Forwarding Decision

Use this simple rule:

```text id="p7v4m1"
Do I need one known service?
        │
       Yes
        ↓
Port Forward
```

Otherwise:

```text id="c3x8q5"
Do I need multiple internal destinations?
        │
       Yes
        ↓
Consider SOCKS
```

And:

```text id="w9k2n6"
Do I need broad network-level access?
        │
       Yes
        ↓
Consider a routing-based pivot
```

The final mechanism depends on the assessment environment.

## 20. Multi-Hop SOCKS

In more complex environments:

```text id="f5m8r3"
Assessment Machine
       ↓
Pivot A
       ↓
Pivot B
       ↓
Internal Network
       ↓
Target
```

The important principle is to validate each hop.

```text id="q7n2v9"
Assessment → Pivot A
       ↓
Pivot A → Pivot B
       ↓
Pivot B → Target
```

A failure at one hop should not be diagnosed as a failure of the entire chain.

## 21. Evidence Recording

Record the proxy path:

```text id="s4m8x2"
Source:
SOCKS Version:
Local Proxy Address:
Local Proxy Port:
Pivot Host:
Pivot Network:
Target:
Target Port:
Protocol:
Application:
Connectivity:
Service:
Authentication:
Authorization:
Next Step:
```

Example:

```text id="a9v3k7"
Source: Assessment Machine
Proxy: SOCKS5 127.0.0.1:1080
Pivot: WEB01
Target: 10.10.20.15:8080
Protocol: HTTP
Application: Browser
Connectivity: Successful
Service: HTTP response received
Authentication: Not required
Next Step: Enumerate authorized internal application
```

## 22. Operational Rules

### Validate the Proxy First

Do not troubleshoot the target before confirming the proxy itself is functioning.

### Confirm Pivot-to-Target Connectivity

The pivot must have a route and usable path to the target.

### Use Proxy-Aware Applications

Verify that the selected application actually sends traffic through the proxy.

### Separate Network and Authentication

SOCKS provides a traffic path. It does not provide credentials.

### Minimize Internal Scanning

Use focused discovery based on scope and evidence.

### Keep the Proxy Controlled

Use appropriate local binding and authentication settings where supported.

### Remove Temporary Access

Close the proxy when it is no longer required.

## What Not to Assume

```text id="u6q3m8"
SOCKS Listener
   ≠
Internal Network Access
```

```text id="b4n7p2"
SOCKS Connection
   ≠
Target Service Access
```

```text id="k8m3v5"
Target Service Access
   ≠
Authentication
```

```text id="r2x9c6"
Authentication
   ≠
Authorization
```

```text id="f7w4q1"
SOCKS Proxy
   ≠
Interactive Shell
```

The proxy is a transport mechanism. The rest of the assessment still follows the normal target, service, authentication, and authorization workflow.

## What Next?

Continue to [`Internal-Network-Access.md`](Internal-Network-Access.md).

That file brings the pivoting workflow together: once internal connectivity has been established, it explains how to validate and systematically enumerate the newly accessible network from the new position.
