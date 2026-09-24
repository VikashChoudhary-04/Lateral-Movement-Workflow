# Linux-to-Linux Lateral Movement

Linux-to-Linux lateral movement occurs when an existing foothold, identity, credential, SSH key, agent, or authorized access on one Linux system is used to establish access to another Linux system.

SSH is the primary remote-access service considered in this workflow, but the decision process remains broader than simply trying SSH.

The objective is:

> **Identify a reachable Linux target, determine whether an authorized SSH path exists, validate authentication and authorization, establish access, and reassess the new position.**

---

## Movement Model

The workflow is:

```text
Linux Host A
      ↓
Current Identity
      ↓
Network Context
      ↓
Target Discovery
      ↓
SSH Service
      ↓
Authentication Material
      ↓
Authentication
      ↓
Authorization
      ↓
SSH Access
      ↓
Validate Linux Host B
      ↓
Re-enumerate
      ↓
Continue the Chain
```

The same evidence-driven process applies whether authentication uses a password, SSH key, agent, certificate, or another authorized mechanism.

---

## Prerequisites

Before attempting movement, determine:

| Requirement    | Question                                        |
| -------------- | ----------------------------------------------- |
| Source         | Which Linux host am I currently on?             |
| Identity       | Which account am I using?                       |
| Privileges     | What can this account do?                       |
| Target         | Which Linux host am I investigating?            |
| Reachability   | Can the source reach the target?                |
| SSH            | Is an SSH service available?                    |
| Authentication | What authorized authentication material exists? |
| Authorization  | Can the account log in?                         |
| Objective      | Why does the target matter?                     |

If these questions cannot be answered, return to the earlier discovery sections.

---

## Step 1 — Establish the Starting Position

Identify the current Linux host:

```bash id="5twxgd"
hostname
```

```bash id="y7h4ep"
hostname -f
```

Identify the current user:

```bash id="f4q1q0"
whoami
```

```bash id="l2m0e8"
id
```

Inspect network interfaces:

```bash id="n2h48x"
ip addr
```

Routes:

```bash id="1f4r9u"
ip route
```

DNS:

```bash id="g7v6fy"
cat /etc/resolv.conf
```

Current connections:

```bash id="0s77lm"
ss -tunap
```

Record:

```text id="v36m7d"
Source Host:
User:
Groups:
Privileges:
IP:
Subnet:
Gateway:
Routes:
DNS:
Known Targets:
```

---

## Step 2 — Identify Existing SSH Evidence

SSH configuration can reveal previously used systems.

Inspect the SSH directory:

```bash id="3xkqgr"
ls -la ~/.ssh
```

Configuration:

```bash id="46ctjy"
cat ~/.ssh/config
```

Known hosts:

```bash id="fcr7gf"
cat ~/.ssh/known_hosts
```

SSH agent:

```bash id="f2b5qf"
echo "$SSH_AUTH_SOCK"
```

Available agent identities:

```bash id="k8e6p8"
ssh-add -L
```

These sources can reveal:

* usernames
* target names
* internal IP addresses
* non-standard ports
* identity files
* jump hosts
* previously contacted systems

Treat these as discovery evidence rather than proof of current access.

---

## Step 3 — Identify Authentication Material

Determine which authorized authentication methods are available.

Potential sources include:

* current account credentials
* SSH private keys
* SSH agent identities
* certificates
* documented service credentials
* existing SSH configuration

For a discovered private key, determine:

```text id="slx42j"
Owner
Username
Target
Port
Passphrase Protection
Authentication Context
```

A key found on disk does not automatically mean it can be used.

It may:

* belong to another account
* require a passphrase
* have been revoked
* correspond to another environment
* not be authorized on the target

---

## Step 4 — Discover Linux Targets

Use evidence from:

* current network routes
* DNS
* `known_hosts`
* SSH configuration
* application configuration
* existing connections
* previous host enumeration
* documented infrastructure

Example:

```text id="t6gk6x"
Target: LINUX-SRV02
IP: 10.10.20.25
Role: Application Server
Reason: Referenced by SSH configuration
```

The target should have a reason for investigation.

Avoid treating every reachable address as a movement target.

---

## Step 5 — Confirm Reachability

Test the SSH service.

```bash id="rj3v4k"
nc -vz 10.10.20.25 22
```

If ICMP is permitted:

```bash id="7tv5jg"
ping -c 2 10.10.20.25
```

If a non-standard SSH port is known:

```bash id="4j6d9w"
nc -vz 10.10.20.25 2222
```

Interpret the results carefully.

```text id="x1fwjp"
Network Reachability
        ≠
SSH Service
        ≠
Authentication
        ≠
Authorization
        ≠
Shell Access
```

---

## Step 6 — Confirm the SSH Service

If you have authorized access to the target through another path, inspect the service.

On systemd-based systems:

```bash id="t7f14e"
systemctl status ssh
```

or:

```bash id="r2q7qi"
systemctl status sshd
```

Check listeners:

```bash id="qgk2c4"
ss -lntp
```

Look for the configured SSH port.

Commonly:

```text id="g7i8c4"
0.0.0.0:22
```

or:

```text id="xq0y0o"
[::]:22
```

Do not assume the service must use port 22.

---

## Step 7 — Map Identity to Target

Before testing access, build an explicit mapping.

```text id="j6ut1k"
SSH Identity
      ↓
Username
      ↓
Target
      ↓
SSH Port
      ↓
Authentication Method
```

Example:

```text id="eqxq1r"
Key: ~/.ssh/id_ed25519
User: analyst
Target: LINUX-SRV02
Port: 22
Method: Public-key authentication
```

This prevents accidental testing with the wrong identity or username.

---

## Step 8 — Validate Password-Based SSH Access

Where authorized, test the specific account against the identified SSH target.

Example:

```bash id="j4y2w5"
ssh analyst@10.10.20.25
```

The goal is to determine:

```text id="q14d9s"
Target reachable
      ↓
SSH service available
      ↓
Authentication succeeds
      ↓
Account permitted
      ↓
Session established
```

Do not repeatedly guess passwords or broadly test credentials across hosts.

Use credentials only where the assessment scope and evidence support the test.

---

## Step 9 — Validate SSH Key Access

If an authorized key is available:

```bash id="d8p8j7"
ssh -i ~/.ssh/id_ed25519 analyst@10.10.20.25
```

If the SSH server uses a known alternate port:

```bash id="4w4wz7"
ssh -i ~/.ssh/id_ed25519 -p 2222 analyst@10.10.20.25
```

The important result is whether the key is accepted for the specified account on the specified target.

Record:

```text id="2cnpaj"
Target:
Username:
Key:
Port:
Authentication:
Authorization:
Result:
```

Never place real private keys in a Git repository or assessment report.

---

## Step 10 — Validate SSH Agent Access

If an SSH agent exists:

```bash id="z4i4v7"
echo "$SSH_AUTH_SOCK"
```

List available identities:

```bash id="07ib0c"
ssh-add -L
```

If an appropriate identity is available, validate it against the specific authorized target.

The important distinction is:

```text id="p5a9b1"
Agent Identity Available
        ≠
Target Access Guaranteed
```

The target still controls whether that identity is authorized.

---

## Step 11 — Understand SSH Authorization

SSH authorization can be affected by:

* account existence
* account status
* SSH server configuration
* `AllowUsers`
* `AllowGroups`
* `DenyUsers`
* `DenyGroups`
* source-address restrictions
* shell restrictions
* authentication policy
* host-based access controls

Therefore:

```text id="x4ot1n"
Valid Credential
      ↓
Authentication
      ↓
SSH Server Policy
      ↓
Account Authorization
      ↓
Session
```

A credential that authenticates successfully can still be restricted from interactive access.

---

## Step 12 — Validate the New Session

Once SSH access is established, immediately identify the new position.

```bash id="3bqcb3"
whoami
```

```bash id="4u5d7w"
id
```

```bash id="g9z6w8"
hostname
```

```bash id="6v9o1k"
hostname -f
```

Network interfaces:

```bash id="0x0p6u"
ip addr
```

Routes:

```bash id="x3j8c1"
ip route
```

Current connections:

```bash id="5a7skc"
ss -tunap
```

The resulting position should be recorded as:

```text id="h8y44h"
New Host:
New User:
Groups:
Privileges:
Interfaces:
Routes:
Reachable Networks:
Services:
```

---

## Step 13 — Reassess the Network

The new Linux host may have network access unavailable from the original host.

For example:

```text id="s6u6jq"
Host A
10.10.10.15
    │
    │ SSH
    ▼
Host B
10.10.20.25
    │
    ├── 10.10.30.0/24
    └── 10.10.40.0/24
```

Inspect:

```bash id="fgr4qg"
ip addr
```

```bash id="eq4a2e"
ip route
```

```bash id="m0e9fc"
ip neigh
```

Then determine whether the new networks contain relevant targets.

Do not automatically scan every discovered network.

Return to the target-discovery workflow and investigate systems that are relevant and within scope.

---

## Step 14 — Reassess SSH Configuration

On the new host:

```bash id="24knsx"
ls -la ~/.ssh
```

```bash id="ngt5a9"
cat ~/.ssh/config
```

```bash id="gj3w2s"
cat ~/.ssh/known_hosts
```

If an SSH agent is present:

```bash id="v4a0t5"
echo "$SSH_AUTH_SOCK"
```

```bash id="k3p4mc"
ssh-add -L
```

The new host may contain different SSH relationships than the original host.

This can reveal:

* additional target names
* internal servers
* jump hosts
* different usernames
* additional network paths

Treat all findings as evidence that must be validated.

---

## Step 15 — Reassess Local Services

After movement, inspect services that may explain the role of the new system.

For example:

```bash id="r1d0j8"
ss -lntp
```

On systemd-based systems:

```bash id="z7y6my"
systemctl list-units --type=service --state=running
```

The goal is to identify:

* SSH
* web services
* databases
* application services
* file-sharing services
* management services
* custom services

A server's role may reveal additional movement opportunities.

---

## Step 16 — Check Existing Sessions

Inspect current user activity where authorized:

```bash id="4w3wfj"
who
```

```bash id="s7l8j8"
w
```

```bash id="f5z4s7"
users
```

Process context:

```bash id="s8u8y4"
ps aux
```

Network connections:

```bash id="4q0z9v"
ss -tunap
```

The purpose is to understand the current system's operational context.

Do not automatically interpret another logged-in user as possession of their credentials.

---

## Step 17 — Update the Movement Record

Record the movement explicitly.

| Field                 | Result          |
| --------------------- | --------------- |
| Source Host           | `LINUX-SRV01`   |
| Source User           | `analyst`       |
| Target Host           | `LINUX-SRV02`   |
| Service               | SSH             |
| Port                  | 22              |
| Authentication        | SSH key         |
| Authentication Result | Success         |
| Authorization         | Shell access    |
| New User              | `analyst`       |
| New Privileges        | Documented      |
| New Networks          | `10.10.30.0/24` |
| New Services          | Documented      |
| New Targets           | Documented      |
| Next Action           | Re-enumerate    |

This becomes important when the assessment contains multiple Linux systems.

---

## Failure Classification

Classify the failure before changing techniques.

### Target unreachable

Investigate:

```text id="03v0h5"
Routing
Segmentation
Target availability
Firewall
```

### SSH port unavailable

Investigate:

```text id="i9k3e4"
SSH service
Port configuration
Firewall
Target state
```

### Connection refused

Possible causes:

* no SSH listener
* service stopped
* wrong port
* local firewall behavior

### Authentication failure

Investigate:

```text id="0l6nq8"
Username
Password
SSH key
Authentication method
Account status
```

### Key rejected

Investigate:

```text id="3q2zj8"
Wrong key
Wrong user
Key not authorized
Server policy
Key restrictions
```

### Authentication succeeds but login is denied

Investigate:

```text id="v8r7v1"
Account restrictions
SSH policy
Allow/Deny rules
Source restrictions
Shell restrictions
```

### Login succeeds but expected network is unreachable

Investigate:

```text id="w9p0jy"
Routing
Firewall
Segmentation
Host-specific network access
```

The failure should tell you which layer requires further investigation.

---

## SSH Target Matrix

When multiple Linux systems are available, maintain a simple matrix.

| Target  | SSH Port | Username  | Auth Material | Reachable | Auth       | Authorization | Result        |
| ------- | -------: | --------- | ------------- | --------- | ---------- | ------------- | ------------- |
| `srv01` |       22 | `analyst` | Key           | Yes       | Success    | Shell         | Access        |
| `srv02` |       22 | `analyst` | Key           | Yes       | Failed     | Unknown       | Investigate   |
| `srv03` |     2222 | `deploy`  | Password      | Yes       | Not tested | Unknown       | Investigate   |
| `srv04` |       22 | `backup`  | Agent         | No        | N/A        | Unknown       | Network issue |

This prevents repeated testing and preserves the movement history.

---

## Example Movement Chain

A Linux movement chain may look like:

```text id="z3m5dx"
LINUX-SRV01
    │
    │ SSH key
    ▼
LINUX-SRV02
    │
    ├── New subnet
    ├── New SSH configuration
    ├── New application servers
    │
    │ SSH
    ▼
LINUX-SRV03
```

The correct workflow is:

```text id="d5st4m"
Move
 ↓
Validate
 ↓
Re-enumerate
 ↓
Discover
 ↓
Map access
 ↓
Move again
```

Not:

```text id="bq6kjo"
Move → Move → Move
```

without reassessment.

---

## Decision Tree

```text id="g3c1q6"
Start on Linux Host A
        │
        ↓
Identify user, privileges and network
        │
        ↓
Identify Linux targets
        │
        ↓
Is SSH reachable?
        │
   ┌────┴────┐
   No       Yes
   │          │
Investigate   ↓
network/    Do I have an
service     authorized identity?
              │
         ┌────┴────┐
        No        Yes
        │           │
Credential          ↓
Discovery       Does authentication
                succeed?
                    │
               ┌────┴────┐
              No        Yes
              │           │
         Investigate      ↓
         identity/key/ Is the account
         auth policy   authorized?
                           │
                      ┌────┴────┐
                     No        Yes
                     │           │
                Investigate      ↓
                  account    Establish SSH
                  policy         │
                                 ↓
                           Validate Host B
                                 │
                                 ↓
                           Re-enumerate
                                 │
                                 ↓
                           Discover next path
```

---

## Common Mistakes

### Assuming SSH means port 22

SSH can run on another configured port.

### Treating `known_hosts` as credentials

A known-host entry records host-key information; it does not provide authentication.

### Treating a private key as guaranteed access

The key must correspond to an authorized account and target.

### Ignoring the username

Different usernames can produce completely different authorization results.

### Forgetting SSH agents

An agent may contain identities even when the corresponding private-key file is not directly available.

### Testing credentials indiscriminately

Use evidence to select specific targets and accounts.

### Ignoring server-side restrictions

Authentication can succeed while account authorization still prevents the intended session.

### Stopping after SSH login

The new host must become the starting point for another position assessment.

### Forgetting network changes

The new host may expose networks that were unavailable from the original host.

### Losing movement history

Record every meaningful movement attempt and result.

---

## Completion Criteria

Linux-to-Linux movement is complete when you can document:

* source host
* source user
* source privileges
* target host
* target role
* target reachability
* SSH service and port
* authentication material
* authentication result
* authorization result
* resulting SSH access
* new host identity
* new user
* new privileges
* new network position
* new targets
* next movement path

If movement fails, classify the failure and update the next hypothesis.

---

## What Next?

Continue with:

**[`Windows-to-Linux.md`](Windows-to-Linux.md)**

The next workflow covers movement from a Windows foothold into a Linux target, primarily by combining Windows-side discovery with Linux SSH access.
