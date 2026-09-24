# Linux SSH Movement

This scenario demonstrates an end-to-end lateral-movement workflow from one Linux host to another using SSH.

The scenario focuses on the reasoning around SSH movement rather than treating SSH as simply:

```text
ssh user@target
```

The workflow establishes the current position, identifies reachable Linux targets, understands SSH configuration and authentication material, validates access, and then treats the destination as a new assessment position.

The scenario is intended for authorized penetration tests and controlled labs.

## Scenario Objective

Demonstrate a Linux-to-Linux movement path:

```text
Compromised Linux Host
        ↓
Position Assessment
        ↓
Network Discovery
        ↓
SSH Target Discovery
        ↓
SSH Credential / Key Context
        ↓
Target Reachability
        ↓
SSH Authentication
        ↓
Authorization
        ↓
New Linux Shell
        ↓
Access Validation
        ↓
New-Host Enumeration
        ↓
Credential Reassessment
        ↓
Continue or Stop
```

The central principle is:

> SSH authentication proves that an account accepted the authentication attempt. It does not automatically prove root access or unrestricted control of the target.

## Starting Position

Assume the assessment begins with an authorized foothold on:

```text id="p8m3q7"
Host: LINUX-01
IP: 10.10.10.25
User: analyst
Network: 10.10.10.0/24
```

The exact values are illustrative.

Use values from the authorized assessment environment.

## Scope

Assume the authorized environment includes:

```text id="x5q8m2"
Source:
LINUX-01

Candidate Target:
LINUX-02

Network:
10.10.10.0/24

Primary Remote Service:
SSH
```

Only systems explicitly included in the assessment scope should be tested.

## Phase 1 — Establish the Current Position

Start by confirming the current host.

```bash id="m7q3v9"
hostname
```

Identify the current user:

```bash id="c4n8p2"
whoami
```

Identify account context:

```bash id="v6m3r8"
id
```

Check groups:

```bash id="q9p4m7"
groups
```

The initial position should be documented:

```text id="t8x3m5"
Host:
LINUX-01

User:
analyst

Network:
10.10.10.0/24

Objective:
Validate authorized SSH access to LINUX-02.
```

## Phase 2 — Enumerate the Current Network

Identify interfaces:

```bash id="n5m8q3"
ip addr
```

Identify routes:

```bash id="r7p2v9"
ip route
```

Review DNS context:

```bash id="x3q8m5"
cat /etc/resolv.conf
```

Where applicable:

```bash id="m9p4r2"
resolvectl status
```

Determine:

```text id="v5n8q2"
Interfaces:
IP Addresses:
Routes:
Gateways:
DNS:
```

## Phase 3 — Identify the Target

Assume the authorized assessment scope identifies:

```text id="c6m3x8"
LINUX-02
10.10.10.50
```

Before attempting SSH access, establish:

```text id="q8p3m7"
Can LINUX-01 reach LINUX-02?
        ↓
Is SSH exposed?
        ↓
Which SSH port is used?
        ↓
Which account should authenticate?
        ↓
What authentication material is available?
```

## Phase 4 — Check SSH Reachability

The standard SSH port is TCP 22, but authorized environments may use a different port.

Check the expected port:

```bash id="m4q7n2"
nc -vz 10.10.10.50 22
```

If the target uses another documented port:

```bash id="x8p3m6"
nc -vz 10.10.10.50 <PORT>
```

The result establishes network-level reachability.

It does not prove:

```text id="r5n8q2"
Authentication
```

or:

```text id="p7m3v9"
Authorization
```

## Phase 5 — Identify SSH Configuration

Inspect the current user's SSH configuration:

```bash id="v3m8q5"
ls -la ~/.ssh/
```

If present, review configuration:

```bash id="q6m2r8"
cat ~/.ssh/config
```

Look for information such as:

```text id="n4p7m3"
Host Aliases
Hostnames
Ports
Usernames
Identity Files
ProxyJump / ProxyCommand
```

Configuration may reveal legitimate relationships between the current host and other systems.

Do not expose private key contents unnecessarily.

## Phase 6 — Review Known Hosts

The SSH known-hosts file can provide historical connection context:

```bash id="m8q3p6"
cat ~/.ssh/known_hosts
```

The objective is to identify systems that the current account has previously interacted with.

Treat these entries as discovery evidence.

A hostname appearing in `known_hosts` does not prove that the current credentials still work.

## Phase 7 — Identify Available SSH Keys

List SSH files:

```bash id="x5m8q2"
find ~/.ssh -maxdepth 1 -type f -printf '%f\n'
```

Common key-related filenames may include:

```text id="r7m3v8"
id_rsa
id_ed25519
id_ecdsa
```

The exact files vary by environment.

If a private key is legitimately available within scope, record:

```text id="p4q8n2"
Key:
<KEY IDENTIFIER>

Potential User:
<USERNAME>

Known Host:
<TARGET>

Known Port:
<PORT>
```

Do not place private key material into the repository.

## Phase 8 — Reassess SSH Agent Context

Check whether an SSH agent is available:

```bash id="v8m3q6"
echo "$SSH_AUTH_SOCK"
```

If an agent is present, list loaded public keys:

```bash id="m5q8p2"
ssh-add -L
```

This can establish that authentication material is already loaded into the current session.

Do not assume that every loaded key provides access to every SSH server.

## Phase 9 — Map the SSH Identity

For each candidate key or account, determine:

```text id="c7m3r8"
Potential User
      ↓
SSH Key
      ↓
Known Host
      ↓
SSH Configuration
      ↓
Target
```

For example:

```text id="x8p2m5"
analyst
   ↓
id_ed25519
   ↓
LINUX-02
   ↓
SSH
```

This is a stronger movement hypothesis than randomly trying usernames.

## Phase 10 — Validate SSH Authentication

If an authorized key is associated with the target account:

```bash id="q4m8v3"
ssh -i ~/.ssh/id_ed25519 analyst@10.10.10.50
```

If the target uses a documented nonstandard port:

```bash id="p7n3r8"
ssh -i ~/.ssh/id_ed25519 -p <PORT> analyst@10.10.10.50
```

If SSH configuration already defines a host alias:

```bash id="m5q8x2"
ssh target-alias
```

The objective is to determine:

```text id="v8m3p7"
Authentication:
Successful / Failed
```

Do not treat a successful connection as proof of root access.

## Phase 11 — Validate the New Shell

Immediately confirm the target host.

```bash id="c4q8m2"
hostname
```

Confirm the identity:

```bash id="n7m3v9"
whoami
```

Confirm account details:

```bash id="x5p8q3"
id
```

Check groups:

```bash id="r4m7v2"
groups
```

The resulting position might be:

```text id="m8q3p5"
Host:
LINUX-02

User:
analyst

UID:
1001

Groups:
analyst
developers
```

This is now a validated host-level movement.

## Phase 12 — Determine Privilege Level

Do not assume the SSH account is privileged.

Check:

```bash id="q6m8r3"
id
```

Where appropriate:

```bash id="v3p7n2"
sudo -l
```

The result may show:

```text id="x8m4q5"
Standard User
```

or:

```text id="n7p3m8"
Specific sudo permissions
```

or another authorized privilege context.

Record the actual result.

## Phase 13 — Compare the Two Positions

Create a comparison:

| Property     | LINUX-01      | LINUX-02      |
| ------------ | ------------- | ------------- |
| User         | analyst       | analyst       |
| IP           | 10.10.10.25   | 10.10.10.50   |
| Network      | 10.10.10.0/24 | 10.10.10.0/24 |
| Groups       |               |               |
| Privilege    |               |               |
| Routes       |               |               |
| Services     |               |               |
| New Networks |               |               |
| New Access   |               |               |

The purpose is to determine what changed after movement.

## Phase 14 — Enumerate Network Interfaces

On LINUX-02:

```bash id="m4q8p2"
ip addr
```

Record:

```text id="c7m3x9"
Interface:
IP:
Prefix:
Network:
Purpose:
```

Look for:

* multiple interfaces
* management networks
* internal subnets
* VPN interfaces
* container networks
* virtual interfaces

An additional interface may indicate a different network position.

## Phase 15 — Enumerate Routes

```bash id="v8n3p6"
ip route
```

Look for:

```text id="q5m7r2"
Destination:
Gateway:
Interface:
```

For example:

```text id="x3p8m4"
10.10.10.0/24
10.10.20.0/24
```

If LINUX-02 can reach 10.10.20.0/24 while LINUX-01 could not, the new host may provide additional internal visibility.

## Phase 16 — Enumerate DNS

Review:

```bash id="m7q3v8"
cat /etc/resolv.conf
```

Where available:

```bash id="r4n8p2"
resolvectl status
```

Determine:

```text id="v6m3q9"
DNS Server:
Search Domain:
Internal Resolution:
```

Internal DNS may reveal useful hostnames and service relationships.

## Phase 17 — Enumerate Listening Services

Use:

```bash id="x8q3m5"
ss -lntup
```

Identify:

```text id="p7m4r8"
Port:
Protocol:
Bind Address:
Process:
Likely Service:
```

Examples might include:

```text id="c5n8q2"
22    SSH
80    Web Application
443   HTTPS
3306  Database
```

The actual services depend on the environment.

## Phase 18 — Enumerate Running Services

On systemd-based systems:

```bash id="m3q8v5"
systemctl list-units --type=service --state=running
```

Identify the server's likely role.

For example:

```text id="r7p2m9"
SSH
Nginx
Application Service
Database Service
```

This may indicate:

```text id="x4m8q3"
Web Server
```

or:

```text id="n6p3v7"
Application Server
```

The role should be inferred from multiple pieces of evidence.

## Phase 19 — Review Active Connections

Use:

```bash id="q8m3r5"
ss -tunap
```

Look for relationships with:

* databases
* application servers
* internal APIs
* file services
* management systems
* other internal hosts

A connection provides evidence of communication.

It does not automatically provide:

```text id="v5q8m2"
Credentials
```

or:

```text id="m3p7x9"
Authorization
```

## Phase 20 — Identify the New Host's Role

Combine:

```text id="r8m3q6"
Interfaces
+
Routes
+
Listening Services
+
Running Services
+
Connections
```

Possible roles include:

```text id="p5n7m2"
Web Server
Application Server
Database Server
Jump Host
File Server
Development Host
Monitoring Host
```

Do not rely only on the hostname.

## Phase 21 — Reassess SSH Access

The new host may contain SSH relationships.

Review:

```bash id="x7m4q8"
ls -la ~/.ssh/
```

Where appropriate, inspect:

```bash id="c3p8n5"
cat ~/.ssh/config
```

and:

```bash id="m9q2v7"
cat ~/.ssh/known_hosts
```

Identify:

```text id="r4m8p3"
Known Hosts
Configured Users
Configured Ports
Identity Files
Proxy Relationships
```

This may reveal the next authorized SSH target.

## Phase 22 — Reassess SSH Agent State

Check:

```bash id="q7m3x8"
echo "$SSH_AUTH_SOCK"
```

If applicable:

```bash id="v5n8p2"
ssh-add -L
```

Record only the necessary identity information.

Do not expose private key contents in notes or repository files.

## Phase 23 — Reassess Application Configuration

If LINUX-02 hosts an application, determine how it connects to other systems.

Potential relationships:

```text id="m8q3p6"
Web Application
      ↓
Database

Application
      ↓
Internal API

Application
      ↓
SSH / Remote Service

Application
      ↓
File Storage
```

Relevant configuration may contain:

```text id="x4q7n2"
Hostnames
Ports
Usernames
Credential References
API Endpoints
Database Connections
```

Treat actual secrets as sensitive assessment data.

## Phase 24 — Reassess Environment Variables

Where authorized:

```bash id="p3m8q5"
env
```

or:

```bash id="v7q2n9"
printenv
```

Look for application-specific configuration.

Use placeholders when documenting:

```text id="c8m4r7"
<USERNAME>
<PASSWORD>
<TOKEN>
<API_KEY>
```

Do not commit real secrets to the repository.

## Phase 25 — Identify New Candidate Targets

Suppose LINUX-02 reveals:

```text id="m5q8p3"
10.10.20.15 → Application Server
10.10.20.25 → Database
10.10.20.35 → Internal SSH Host
```

Create a candidate table:

| Target   | Network     | Service  | Evidence             | Access Path     |
| -------- | ----------- | -------- | -------------------- | --------------- |
| APP-01   | 10.10.20.15 | HTTPS    | Service / connection | To be validated |
| DB-01    | 10.10.20.25 | Database | Connection observed  | To be validated |
| LINUX-03 | 10.10.20.35 | SSH      | SSH relationship     | To be validated |

Only authorized systems should be tested.

## Phase 26 — Reassess Existing Access

Determine whether the current host already has access to other systems.

Examples:

```text id="r8m3q7"
SSH Agent
Existing SSH Sessions
Mounted Resources
Application Sessions
Database Connections
Service Credentials
```

The workflow is:

```text id="x4p7m2"
Existing Access
      ↓
Identify Identity
      ↓
Identify Target
      ↓
Identify Service
      ↓
Validate Authorization
```

## Phase 27 — Evaluate Credential Reuse

Suppose the new host contains a configured SSH identity associated with:

```text id="q5m8v3"
LINUX-03
```

The correct reasoning is:

```text id="m7p3r9"
Known SSH Relationship
       ↓
Known User
       ↓
Known Target
       ↓
Target Reachability
       ↓
Authorized SSH Validation
```

Do not turn one known relationship into broad credential testing across unrelated systems.

## Phase 28 — Determine Whether a Pivot Is Required

If LINUX-02 can reach a network unavailable from LINUX-01:

```text id="v8m3q5"
LINUX-01
    |
    | Cannot directly reach
    v
10.10.20.0/24
    ^
    |
    | Can reach
    |
LINUX-02
```

LINUX-02 may become a pivot candidate.

Use the pivoting workflow:

[`../09-Pivoting-and-Internal-Network-Access/`](../09-Pivoting-and-Internal-Network-Access/)

The pivot should have a specific purpose.

## Phase 29 — Evaluate the New Position

Compare:

| Property         | LINUX-01      | LINUX-02                      |
| ---------------- | ------------- | ----------------------------- |
| User             | analyst       | analyst                       |
| Network          | 10.10.10.0/24 | 10.10.10.0/24 + 10.10.20.0/24 |
| SSH              | Initial       | Server                        |
| New Services     |               |                               |
| New Credentials  |               |                               |
| New Targets      |               |                               |
| Pivot Capability |               |                               |

The important question is:

> What can LINUX-02 provide that LINUX-01 could not?

## Phase 30 — Decide Whether to Continue

Use:

```text id="p6m3x8"
New Position
      ↓
New Target?
   ┌──┴──┐
  No     Yes
  ↓       ↓
Stop   In Scope?
           │
        ┌──┴──┐
       No     Yes
       ↓       ↓
      Stop   Reachable?
                  │
               ┌──┴──┐
              No     Yes
              ↓       ↓
           Pivot?   SSH Service?
                         │
                    Authentication
                         ↓
                    Authorization
                         ↓
                    New Position
```

## Phase 31 — Failure Handling

### SSH Port Is Closed

```text id="q8m3r2"
Network Reachability
        ✗
```

Check:

* target address
* route
* port
* firewall
* SSH service

### SSH Port Is Open but Authentication Fails

```text id="m4p7v9"
Network ✓
SSH ✓
Authentication ✗
```

Check:

* username
* key
* key format
* SSH configuration
* account status
* target authentication configuration

### Authentication Succeeds but Shell Access Is Restricted

The account may have:

* restricted shell
* command-only access
* service-specific access

Record the actual authorization boundary.

### SSH Works but Root Access Is Unavailable

This is normal for many accounts.

Record:

```text id="x7m3q8"
SSH Authentication:
Successful

Shell:
Available

Root:
Not demonstrated
```

Then determine whether the new user context provides meaningful additional access.

### New Host Reveals No Useful Targets

Compare:

```text id="v5q8m2"
Previous Position
vs.
New Position
```

If no meaningful new information exists, stop rather than continuing automatically.

## Phase 32 — Build the Movement Chain

Example:

```text id="n8m3p7"
P1
LINUX-01
analyst
10.10.10.25
      |
      | SSH
      | Key Authentication
      ↓
P2
LINUX-02
analyst
10.10.10.50
      |
      | New Network Visibility
      ↓
10.10.20.0/24
      |
      ↓
Candidate Linux / Application Targets
```

Each transition must be validated.

## Phase 33 — Evidence Record

Record:

```text id="r4m8q2"
Source Host:
Source User:
Target Host:
Target IP:
SSH Port:
Authentication Method:
Authentication Result:
Authorization Result:
New User:
Privilege Level:
New Networks:
New Credentials / Access:
Next Target:
Next Action:
```

Example:

```text id="m7p3x8"
Source Host: LINUX-01
Source User: analyst
Target Host: LINUX-02
Target Service: SSH
Authentication: SSH key
Authentication Result: Successful
Authorization Result: Interactive shell permitted
New User: analyst
Privilege: Standard user
New Network: 10.10.20.0/24
Next Action: Assess authorized systems on 10.10.20.0/24
```

## Phase 34 — Position Ledger

Maintain:

| Position | Host     | User    | Network                       | Access           | New Visibility   |
| -------- | -------- | ------- | ----------------------------- | ---------------- | ---------------- |
| P1       | LINUX-01 | analyst | 10.10.10.0/24                 | Initial foothold | Baseline         |
| P2       | LINUX-02 | analyst | 10.10.10.0/24 + 10.10.20.0/24 | SSH              | Internal network |

If another host is accessed:

| Position | Host | User | Network | Access | New Visibility |
| -------- | ---- | ---- | ------- | ------ | -------------- |
| P3       |      |      |         |        |                |

## Phase 35 — Complete SSH Movement Workflow

The complete scenario is:

```text id="q8m3p5"
LINUX-01
      ↓
Confirm User
      ↓
Enumerate Network
      ↓
Identify LINUX-02
      ↓
Check SSH
      ↓
Review SSH Configuration
      ↓
Review Known Hosts
      ↓
Review Authorized Key / Agent Context
      ↓
Map Identity to Target
      ↓
Validate SSH Authentication
      ↓
Validate Shell Authorization
      ↓
Confirm New Host
      ↓
Enumerate Interfaces / Routes
      ↓
Enumerate Services
      ↓
Review Connections
      ↓
Reassess SSH / Application Access
      ↓
Identify New Targets
      ↓
Continue?
   ┌──┴──┐
  Yes    No
   ↓      ↓
Next    Document
Step    Chain
```

## What This Scenario Demonstrates

The scenario connects:

```text id="m5q8r3"
Linux Position
      ↓
Network Context
      ↓
SSH Discovery
      ↓
SSH Authentication Material
      ↓
Target Mapping
      ↓
Authentication
      ↓
Authorization
      ↓
New Linux Position
      ↓
Network Reassessment
      ↓
Credential / Access Reassessment
      ↓
Next Movement Decision
```

The most important distinction is:

```text id="x7m3q9"
SSH Authentication
      ≠
Root Access
```

and:

```text id="p4n8m2"
SSH Access
      ≠
Access to Every Host
```

Every new target requires its own validation.

## Safety and Scope

Perform this workflow only against explicitly authorized systems.

Avoid:

* broad password guessing
* indiscriminate credential testing
* unnecessary private-key exposure
* destructive actions
* unauthorized persistence
* access outside the assessment scope

Use known relationships and targeted validation wherever possible.

## Golden Rule

> A successful SSH connection is the beginning of a new assessment position, not the end of the movement workflow.

After every successful movement:

```text id="v8m3q5"
VALIDATE
   ↓
ENUMERATE
   ↓
REASSESS
   ↓
IDENTIFY NEW TARGETS
   ↓
DECIDE
```

That loop turns a single SSH connection into a controlled, evidence-based lateral-movement workflow.

## What Next?

Continue to [`Multi-Host-Chain.md`](Multi-Host-Chain.md).

That scenario will combine multiple hosts and movement methods into one longer chain, including position tracking, credential reassessment, pivoting, and deciding when the chain should stop.
