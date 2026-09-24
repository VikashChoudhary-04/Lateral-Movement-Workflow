# Windows-to-Linux Lateral Movement

Windows-to-Linux lateral movement occurs when an existing foothold on a Windows system is used to establish authorized access to a Linux system.

The most common path in this workflow is:

```text id="6q7d9x"
Windows Host
    ↓
Discover Linux Target
    ↓
Identify SSH
    ↓
Map Linux Account
    ↓
Identify Authorized Authentication
    ↓
Validate SSH Access
    ↓
Linux Host
    ↓
Re-enumerate
```

The important point is that the operating-system boundary changes, but the fundamental movement model does not.

The objective is:

> **Use the Windows foothold to identify a reachable Linux target and establish authorized Linux access through a compatible remote service.**

---

## Movement Model

The complete workflow is:

```text id="gq0p7j"
Windows Host A
      ↓
Current Identity
      ↓
Network Context
      ↓
Linux Target Discovery
      ↓
SSH Service Discovery
      ↓
Authentication Material
      ↓
Username Mapping
      ↓
Authentication
      ↓
Authorization
      ↓
Linux Host B
      ↓
Validate Position
      ↓
Re-enumerate
      ↓
Continue the Chain
```

---

## Prerequisites

Before attempting movement, establish:

| Requirement    | Question                                              |
| -------------- | ----------------------------------------------------- |
| Source         | Which Windows host am I on?                           |
| Identity       | Which Windows account/security context do I have?     |
| Network        | Which networks can the Windows host reach?            |
| Target         | Which Linux system am I targeting?                    |
| Service        | Is SSH available?                                     |
| Port           | Which SSH port is configured?                         |
| Identity       | Which Linux account will be used?                     |
| Authentication | What authorized authentication material is available? |
| Authorization  | Is that account permitted to log in?                  |
| Objective      | Why does the Linux target matter?                     |

If these are unknown, return to the relevant discovery section.

---

## Step 1 — Establish the Windows Starting Position

Identify the current host:

```cmd id="h84f1m"
hostname
```

Current identity:

```cmd id="3c0b0p"
whoami
```

Account identifier:

```cmd id="h9h3bf"
whoami /user
```

Groups:

```cmd id="i2y2oh"
whoami /groups
```

Network configuration:

```cmd id="j8r5h6"
ipconfig /all
```

Routes:

```cmd id="q6q9a1"
route print
```

Current connections:

```cmd id="p0y1uw"
netstat -ano
```

PowerShell:

```powershell id="s5v5yw"
Get-NetIPConfiguration
```

```powershell id="m5n9m6"
Get-NetRoute
```

```powershell id="w7k1ct"
Get-NetTCPConnection
```

Record:

```text id="i7e9wo"
Windows Source:
User:
Domain:
Privileges:
IP:
Subnet:
Gateway:
Routes:
Known Networks:
```

---

## Step 2 — Identify Linux Targets

Linux targets may be discovered through:

* DNS
* application configuration
* network enumeration
* existing connections
* SSH configuration
* known infrastructure
* server inventories
* previous movement
* service discovery

A candidate target might be:

```text id="wl9m50"
Target: LINUX-SRV01
IP: 10.10.20.30
Role: Application Server
Reason: Discovered on reachable internal subnet
Potential Service: SSH
```

The target should be relevant to the assessment objective.

---

## Step 3 — Confirm Linux Target Reachability

From Windows:

```powershell id="1t5r0w"
Test-Connection LINUX-SRV01 -Count 2
```

Then test SSH:

```powershell id="b8z1u0"
Test-NetConnection LINUX-SRV01 -Port 22
```

If a non-standard SSH port was discovered:

```powershell id="2k0c90"
Test-NetConnection LINUX-SRV01 -Port 2222
```

Interpretation:

```text id="6pp2p8"
Target reachable
       ≠
SSH reachable
       ≠
Authentication successful
       ≠
SSH authorization granted
```

---

## Step 4 — Confirm SSH

If authorized access to the Linux target is already available through another channel, confirm the SSH service.

Typical Linux commands:

```bash id="u7g4if"
systemctl status ssh
```

or:

```bash id="e9t5hc"
systemctl status sshd
```

Listening sockets:

```bash id="84t8eh"
ss -lntp
```

Look for the configured SSH port.

The important information is:

```text id="rvn8ec"
SSH Service
    ↓
Listening Address
    ↓
Port
    ↓
Network Accessibility
```

---

## Step 5 — Identify the Linux Account

SSH requires an account identity.

The username may come from:

* SSH configuration
* application configuration
* authorized assessment documentation
* previously discovered account information
* known infrastructure
* an authorized credential

Examples:

```text id="4v4e1u"
analyst
deploy
backup
administrator
```

Do not assume the Windows username and Linux username are the same.

For example:

```text id="m1b8px"
Windows: CORP\analyst
Linux: deploy
```

may be a completely different authentication relationship.

---

## Step 6 — Identify Authorized Authentication Material

Potential authentication material includes:

* password
* SSH private key
* SSH agent
* SSH certificate
* other approved authentication mechanisms

Map the material explicitly:

```text id="w5v1o5"
Authentication Material
        ↓
Linux Username
        ↓
Linux Target
        ↓
SSH Port
```

For example:

```text id="7h5tq0"
SSH Key
 ↓
deploy
 ↓
LINUX-SRV01
 ↓
22
```

The presence of a key does not prove that the target authorizes it.

---

## Step 7 — SSH Client Availability on Windows

Modern Windows systems may include an OpenSSH client.

Check:

```powershell id="f8l2ad"
Get-Command ssh
```

If available:

```powershell id="7b9j3r"
ssh
```

This confirms the local client is available.

If the native SSH client is not available, use an authorized assessment environment/tool appropriate to the engagement.

The movement decision remains the same regardless of the client.

---

## Step 8 — Validate Password-Based SSH

Where authorized, test the identified Linux account:

```powershell id="1l1p2s"
ssh analyst@LINUX-SRV01
```

If a non-standard port is known:

```powershell id="e4c8pw"
ssh -p 2222 analyst@LINUX-SRV01
```

The important sequence is:

```text id="95wq5d"
Windows Host
    ↓
Linux Target
    ↓
SSH Reachable
    ↓
Correct Username
    ↓
Authentication
    ↓
Authorization
    ↓
SSH Session
```

Do not repeatedly guess passwords or broadly test accounts across systems.

---

## Step 9 — Validate SSH Key Access

If an authorized private key is available on the Windows host:

```powershell id="8p4v7n"
ssh -i C:\Users\analyst\.ssh\id_ed25519 analyst@LINUX-SRV01
```

Use the actual authorized key path and username identified during assessment.

If a different SSH port is known:

```powershell id="x6g6jp"
ssh -i C:\Users\analyst\.ssh\id_ed25519 -p 2222 analyst@LINUX-SRV01
```

Record:

```text id="g1c1i6"
Target:
Username:
Key:
Port:
Authentication:
Authorization:
Result:
```

Do not commit private keys to the repository or copy real private keys into reports.

---

## Step 10 — Inspect Windows SSH Configuration

If SSH artifacts already exist on the Windows source:

```powershell id="h4s6t8"
Get-ChildItem $HOME\.ssh
```

Inspect configuration:

```powershell id="5c2c8c"
Get-Content $HOME\.ssh\config
```

Known hosts:

```powershell id="x6s0c5"
Get-Content $HOME\.ssh\known_hosts
```

These may reveal:

* Linux hostnames
* usernames
* ports
* identity files
* previously contacted systems
* jump-host configuration

Treat these artifacts as discovery evidence.

---

## Step 11 — Understand `known_hosts`

The Windows OpenSSH `known_hosts` file can contain host-key information for Linux systems previously contacted from the Windows environment.

Conceptually:

```text id="d3l9fz"
known_hosts
     ↓
Previously contacted host
     ↓
Potential target
```

But:

```text id="9r8r4u"
known_hosts entry
        ≠
SSH credential
        ≠
Current access
```

Use it to identify candidate systems, then validate current reachability and authorization.

---

## Step 12 — Check SSH Agent Context

If an OpenSSH agent is being used, identify whether authorized identities are available.

The exact agent configuration varies by Windows version and environment.

Where the OpenSSH agent is configured, PowerShell may expose it through the Windows service model.

For example:

```powershell id="7s0m7y"
Get-Service ssh-agent
```

The important question is:

> **Is an authorized SSH identity already available through the current Windows environment?**

Do not treat the presence of the agent service alone as proof that a usable identity is loaded.

---

## Step 13 — Validate the New Linux Session

After successful SSH access, immediately establish the new position.

```bash id="q7y5i2"
whoami
```

```bash id="0t7q4v"
id
```

```bash id="3j3m7c"
hostname
```

```bash id="b9b0c1"
hostname -f
```

Network:

```bash id="h2m8x9"
ip addr
```

Routes:

```bash id="w4m8up"
ip route
```

Connections:

```bash id="k2r7k5"
ss -tunap
```

Record:

```text id="a2k0ue"
Linux Host:
Linux User:
Groups:
Privileges:
Interfaces:
Routes:
Reachable Networks:
Services:
```

---

## Step 14 — Compare the New Position

Compare Windows Host A and Linux Host B.

| Property    | Windows Source  | Linux Target    |
| ----------- | --------------- | --------------- |
| Host        | `WORKSTATION01` | `LINUX-SRV01`   |
| User        | `CORP\analyst`  | `analyst`       |
| Network     | `10.10.10.0/24` | `10.10.20.0/24` |
| Gateway     | Documented      | Documented      |
| Routes      | Documented      | Documented      |
| Services    | Windows         | Linux           |
| New Targets | Documented      | Documented      |

This comparison shows what the movement actually achieved.

---

## Step 15 — Reassess Linux Network Access

The Linux host may have access to networks unavailable from Windows.

Check:

```bash id="4e0l2m"
ip addr
```

```bash id="g7n1x4"
ip route
```

```bash id="4m5a1d"
ip neigh
```

Determine:

* interfaces
* subnets
* gateways
* routes
* neighboring systems
* additional internal networks

Then return to target discovery.

Do not automatically scan every discovered network.

---

## Step 16 — Reassess Linux Services

Identify the role of the new Linux host.

Listening services:

```bash id="6a4h8w"
ss -lntp
```

Running services:

```bash id="b3g8y0"
systemctl list-units --type=service --state=running
```

Depending on the host, you may discover:

* SSH
* web servers
* databases
* APIs
* file services
* application services
* management interfaces

The role of the target can change the next movement hypothesis.

---

## Step 17 — Reassess SSH Relationships

Inspect:

```bash id="7n9n3p"
ls -la ~/.ssh
```

```bash id="h5d2x6"
cat ~/.ssh/config
```

```bash id="y8f4r2"
cat ~/.ssh/known_hosts
```

If an SSH agent is present:

```bash id="z8j2y7"
echo "$SSH_AUTH_SOCK"
```

```bash id="m4c8h9"
ssh-add -L
```

The new Linux host may expose SSH relationships that were invisible from the Windows host.

---

## Step 18 — Check Existing Linux Sessions

Inspect logged-in users:

```bash id="4z9n4f"
who
```

```bash id="y1p7i9"
w
```

Processes:

```bash id="z4t3u8"
ps aux
```

Network connections:

```bash id="c4r8d5"
ss -tunap
```

Use this information to understand the host's operational context.

Do not assume that another user's session means their credentials are available.

---

## Step 19 — Update the Movement Record

Document the completed movement.

| Field                 | Result                |
| --------------------- | --------------------- |
| Source Host           | `WORKSTATION01`       |
| Source User           | `CORP\analyst`        |
| Target Host           | `LINUX-SRV01`         |
| Target OS             | Linux                 |
| Service               | SSH                   |
| Port                  | 22                    |
| Authentication        | SSH key               |
| Authentication Result | Success               |
| Authorization         | Shell access          |
| New User              | `analyst`             |
| New Privileges        | Documented            |
| New Networks          | Documented            |
| New Services          | Documented            |
| New Targets           | Documented            |
| Next Action           | Position reassessment |

This record makes the cross-platform movement chain clear.

---

## Failure Classification

### Linux target unreachable

Investigate:

```text id="9e6r7n"
Windows routing
Network segmentation
Firewall
Target availability
```

### SSH port unavailable

Investigate:

```text id="0y5m1m"
SSH service
Port
Firewall
Target configuration
```

### Authentication fails

Investigate:

```text id="p7m1o8"
Username
Password
SSH key
Authentication method
Account status
```

### Key rejected

Investigate:

```text id="x6r7q9"
Wrong key
Wrong username
Key not authorized
Server policy
Key restrictions
```

### Authentication succeeds but login is denied

Investigate:

```text id="r8j2y6"
SSH authorization
Account restrictions
Allow/Deny rules
Source restrictions
Shell restrictions
```

### SSH succeeds but expected network is unavailable

Investigate:

```text id="y4d5g2"
Linux routing
Firewall
Network segmentation
Host-specific access
```

Classify the failure before changing movement paths.

---

## Windows-to-Linux Target Matrix

When several Linux targets exist:

| Target    | SSH Port | User      | Auth Material | Reachable | Auth       | Authorization | Result        |
| --------- | -------: | --------- | ------------- | --------- | ---------- | ------------- | ------------- |
| `linux01` |       22 | `analyst` | Key           | Yes       | Success    | Shell         | Access        |
| `linux02` |       22 | `deploy`  | Key           | Yes       | Failed     | Unknown       | Investigate   |
| `linux03` |     2222 | `backup`  | Password      | Yes       | Not tested | Unknown       | Investigate   |
| `linux04` |       22 | `analyst` | Key           | No        | N/A        | Unknown       | Network issue |

The table records evidence; it does not replace target-specific validation.

---

## Example Movement Chain

A cross-platform movement chain could look like:

```text id="j6r7w1"
WINDOWS-WORKSTATION
       │
       │ SSH authentication
       ▼
LINUX-APP01
       │
       ├── New subnet
       ├── Internal services
       ├── Additional SSH relationships
       │
       │ SSH
       ▼
LINUX-DB01
```

The workflow remains:

```text id="7v4j2g"
Windows foothold
      ↓
Linux target discovery
      ↓
SSH validation
      ↓
Linux access
      ↓
Position reassessment
      ↓
New target discovery
      ↓
Next movement
```

---

## Decision Tree

```text id="k9v1y5"
Start on Windows Host
        │
        ↓
Identify identity and network
        │
        ↓
Discover Linux target
        │
        ↓
Is SSH reachable?
        │
   ┌────┴────┐
   No       Yes
   │          │
Investigate   ↓
network      Do I have an
or service   authorized Linux identity?
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
         username/key/ Is login authorized?
         authentication       │
                              │
                         ┌────┴────┐
                        No        Yes
                        │           │
                   Investigate      ↓
                     policy    Establish SSH
                                   │
                                   ↓
                             Validate Linux host
                                   │
                                   ↓
                             Re-enumerate
                                   │
                                   ↓
                             Continue chain
```

---

## Common Mistakes

### Assuming Windows credentials automatically work on Linux

Linux accounts and authentication systems may be completely separate.

### Assuming the Windows username is the Linux username

Always identify the target account explicitly.

### Treating `known_hosts` as credentials

It is host-key evidence, not authentication material.

### Treating an SSH key as guaranteed access

The target must authorize that key for the selected account.

### Ignoring non-standard SSH ports

Use the port discovered during enumeration.

### Forgetting Windows OpenSSH support

Modern Windows environments may already contain an SSH client.

### Ignoring the Linux target's role

A Linux application server, database server, and jump host can provide very different movement opportunities.

### Stopping after SSH access

The Linux host becomes the new position.

### Forgetting cross-platform network changes

The Linux system may expose networks unavailable from the original Windows system.

### Testing credentials broadly

Use evidence-based, target-specific validation.

---

## Completion Criteria

Windows-to-Linux movement is complete when you can document:

* Windows source host
* source identity
* source privileges
* Linux target
* target role
* target reachability
* SSH service and port
* Linux username
* authentication material
* authentication result
* authorization result
* resulting SSH access
* new Linux identity
* new privileges
* new network position
* new services
* newly discovered targets
* next movement path

If movement fails, classify the failure and update the next hypothesis.

---

## What Next?

Continue with:

**[`Linux-to-Windows.md`](Linux-to-Windows.md)**

The next workflow covers movement from a Linux foothold into a Windows target using Windows remote-access services such as SMB, WinRM, RDP, and RPC/WMI.
