# SSH

Secure Shell (SSH) is a network protocol used for secure remote administration and command-line access.

In a lateral movement workflow, SSH is one of the most important remote-access paths for Linux systems and is also available on modern Windows systems through OpenSSH.

The goal is not simply to find TCP 22 open. The goal is to determine:

> **Can an identity or authentication method available from my current position establish authorized SSH access to a useful target?**

---

## Where SSH Fits

The movement chain is:

```text
Current Host
    ↓
Current Identity
    ↓
Target Discovery
    ↓
Network Reachability
    ↓
SSH Service
    ↓
Authentication Method
    ↓
Authorization
    ↓
SSH Access
    ↓
Validate New Position
    ↓
Re-enumerate
    ↓
Continue the Chain
```

SSH is particularly useful when you have:

* valid credentials
* an authorized SSH key
* an existing SSH agent
* known SSH configuration
* an account permitted to use SSH
* a reachable Linux or OpenSSH-enabled Windows target

---

## Requirements

Before testing SSH movement, establish:

| Requirement    | Question                                                 |
| -------------- | -------------------------------------------------------- |
| Target         | Which host am I trying to reach?                         |
| Reachability   | Can I reach the SSH service?                             |
| Service        | Is SSH actually listening?                               |
| Identity       | Which account will authenticate?                         |
| Authentication | Which authentication method is available?                |
| Authorization  | Is that account allowed to log in?                       |
| Access         | What level of access will the resulting session provide? |

A successful connection requires more than an open port.

---

## SSH vs TCP 22

TCP 22 is the standard SSH port, but the port number alone does not prove SSH access.

A useful model is:

```text
TCP 22 open
     ↓
SSH service available
     ↓
Authentication offered
     ↓
Credentials/key accepted
     ↓
Account permitted to log in
     ↓
SSH session established
```

Each stage can fail independently.

SSH can also be configured on a non-standard port, so do not assume that every SSH service uses TCP 22.

---

## Step 1 — Identify the Target

Start with a target that has a reason for investigation.

Sources can include:

* host discovery
* DNS
* previous SSH configuration
* `known_hosts`
* network routes
* existing connections
* server inventories
* application infrastructure
* previous movement

Example:

```text
Target: LINUX-SRV01
IP: 10.10.10.30
Reason: Linux server discovered on reachable internal subnet
Potential service: SSH
```

Target selection should be based on evidence rather than indiscriminate scanning.

---

## Step 2 — Confirm Reachability

From Linux:

```bash
ping -c 2 10.10.10.30
```

Test the SSH port:

```bash
nc -vz 10.10.10.30 22
```

From Windows:

```powershell
Test-NetConnection 10.10.10.30 -Port 22
```

Interpret the result carefully.

### TCP 22 reachable

This establishes network-level connectivity to the port.

It does **not** establish:

* valid authentication
* account authorization
* shell access
* required privileges

### TCP 22 unavailable

Investigate:

* routing
* network segmentation
* firewall
* target availability
* SSH listening port
* SSH service state

---

## Step 3 — Confirm the SSH Service

If you have authorized access to the target through another channel, inspect the SSH service.

On Linux systems using systemd:

```bash
systemctl status ssh
```

Some distributions use:

```bash
systemctl status sshd
```

Check the listening socket:

```bash
ss -lntp
```

Look for an SSH listener such as:

```text
0.0.0.0:22
```

or:

```text
[::]:22
```

The exact configuration may differ.

---

## Step 4 — Identify Your Authentication Material

SSH supports multiple authentication methods.

Common examples include:

* passwords
* private keys
* SSH agents
* certificates
* other environment-specific mechanisms

The important question is:

> **What authentication material do I legitimately possess from the current position?**

Do not assume that a credential discovered for one service will automatically work over SSH.

Create a mapping:

```text
Identity
   ↓
Authentication Material
   ↓
SSH Target
   ↓
Authentication Method
   ↓
Authorization
```

---

## Step 5 — Investigate SSH Keys

Previously discovered SSH artifacts may provide useful evidence.

Common locations on Linux:

```bash
ls -la ~/.ssh
```

Inspect configuration:

```bash
cat ~/.ssh/config
```

Known hosts:

```bash
cat ~/.ssh/known_hosts
```

Check for an SSH agent:

```bash
echo "$SSH_AUTH_SOCK"
```

List identities available through the current agent:

```bash
ssh-add -L
```

On Windows OpenSSH:

```powershell
Get-ChildItem $HOME\.ssh
```

Possible artifacts include:

```text
id_rsa
id_ed25519
id_ecdsa
authorized_keys
known_hosts
config
```

Finding a key file does not automatically prove that it is usable.

---

## Step 6 — Understand the SSH Key Context

For every discovered key, determine:

| Question                    | Why It Matters                                  |
| --------------------------- | ----------------------------------------------- |
| Who owns the key?           | Identifies the associated security context      |
| Which username is expected? | SSH authentication is account-specific          |
| Which target is referenced? | Narrows candidate systems                       |
| Which port is configured?   | May reveal non-standard SSH                     |
| Is a passphrase required?   | Determines whether the key can be used directly |
| Is an SSH agent present?    | The agent may hold usable identities            |
| Is `ProxyJump` configured?  | May reveal an internal path                     |
| Is the key authorized?      | Determines whether authentication can succeed   |

The relationship may look like:

```text
SSH Key
  ↓
User
  ↓
Target
  ↓
SSH Service
  ↓
Authentication
```

---

## Step 7 — Inspect SSH Configuration

SSH configuration can reveal existing relationships between hosts.

```bash
cat ~/.ssh/config
```

Useful entries may include:

```text
Host internal-server
    HostName 10.10.10.30
    User analyst
    Port 22
```

A configuration may also reference:

```text
IdentityFile
ProxyJump
ProxyCommand
Port
User
HostName
```

These entries can reveal:

* usernames
* target names
* internal addresses
* non-standard ports
* jump hosts
* intended authentication keys

Configuration is evidence, not proof of current access.

---

## Step 8 — Inspect Known Hosts

The SSH `known_hosts` file records host-key information for systems previously contacted by the SSH client.

```bash
cat ~/.ssh/known_hosts
```

It may reveal that the current environment has previously communicated with systems such as:

```text
server01
server02
10.10.10.30
10.10.10.40
```

This can provide target-discovery information.

Important distinction:

```text
known_hosts entry
        ≠
current authentication
        ≠
current authorization
```

A host appearing in `known_hosts` only provides evidence of previous SSH host-key interaction.

---

## Step 9 — Check the SSH Agent

An SSH agent can hold private-key identities without exposing the private key directly to the client process.

Check:

```bash
echo "$SSH_AUTH_SOCK"
```

Then:

```bash
ssh-add -L
```

If identities are available, record:

```text
Identity fingerprint
    ↓
Associated user/context
    ↓
Potential SSH targets
```

An agent identity should still be treated as an authentication capability that must be validated against a specific authorized target.

---

## Step 10 — Validate SSH Authentication

When you have an authorized identity and target, perform a controlled SSH connection.

Example:

```bash
ssh analyst@10.10.10.30
```

For a specific authorized key:

```bash
ssh -i ~/.ssh/id_ed25519 analyst@10.10.10.30
```

If a non-standard port is known:

```bash
ssh -p 2222 analyst@10.10.10.30
```

The objective is to determine whether:

```text
Target
    ↓
SSH reachable
    ↓
Authentication succeeds
    ↓
Account allowed to log in
    ↓
Session established
```

Do not treat an authentication prompt as evidence of successful access.

---

## Authentication vs Authorization

SSH makes the distinction especially clear.

### Authentication

The SSH server verifies the identity.

Examples:

```text
Password accepted
Key accepted
Certificate accepted
```

### Authorization

The server determines whether that identity may establish the requested session.

Possible restrictions include:

* account disabled
* SSH access restricted
* `AllowUsers`
* `AllowGroups`
* `DenyUsers`
* `DenyGroups`
* shell restrictions
* source-address restrictions
* policy controls

Therefore:

```text
Valid credentials
        ↓
Authentication
        ↓
Authorization
        ↓
SSH session
```

A valid credential does not necessarily guarantee SSH access.

---

## Step 11 — Validate the New Session

Once an SSH session is established, immediately confirm the new position.

```bash
whoami
```

```bash
hostname
```

```bash
id
```

Network context:

```bash
ip addr
```

Routes:

```bash
ip route
```

Current connections:

```bash
ss -tunap
```

Check the domain or host identity where relevant:

```bash
hostname -f
```

The objective is to establish:

```text
New Host
   ↓
New User
   ↓
New Privileges
   ↓
New Network Position
   ↓
New Reachable Targets
```

---

## Post-Movement Enumeration

Do not treat SSH login as the end of the workflow.

After successful movement, return to:

**[`03-Position-Assessment`](../03-Position-Assessment/README.md)**

Then reassess:

* hostname
* current user
* privileges
* interfaces
* routes
* DNS
* domain context
* active sessions
* listening services
* reachable networks
* available credentials/access

A new SSH session can provide a completely different network position.

---

## SSH as a Pivoting Path

SSH can also be relevant when the newly accessed host can reach networks that the original host could not.

Conceptually:

```text
Initial Host
     │
     │ SSH
     ▼
Internal Host
     │
     ├── Internal Network A
     ├── Internal Network B
     └── Additional Targets
```

This does not automatically mean that the host should be used as a pivot.

First establish:

* what networks it can reach
* why those networks matter
* what authorized access exists
* what movement path is appropriate

Pivoting is covered in:

**[`09-Pivoting-and-Internal-Network-Access`](../09-Pivoting-and-Internal-Network-Access/README.md)**

---

## SSH and Windows

Modern Windows systems can also run OpenSSH.

From PowerShell, an SSH client may be available as:

```powershell
ssh user@SERVER01
```

Check whether the client exists:

```powershell
Get-Command ssh
```

If you have authorized access to a Windows target, OpenSSH server configuration can also be investigated.

The same workflow still applies:

```text
Reachability
    ↓
SSH service
    ↓
Authentication
    ↓
Authorization
    ↓
Session
    ↓
Position reassessment
```

---

## SSH Failure Classification

When SSH fails, classify the failure before changing techniques.

| Result                               | Investigation Area                          |
| ------------------------------------ | ------------------------------------------- |
| Host unreachable                     | Routing / segmentation / availability       |
| Port 22 closed                       | SSH service / firewall / port configuration |
| Connection timed out                 | Firewall / filtering / routing              |
| Connection refused                   | No listener / service stopped / host policy |
| Authentication failed                | Credentials / key / username / auth method  |
| Permission denied                    | Authentication or account authorization     |
| Key rejected                         | Wrong key / wrong user / server policy      |
| Password rejected                    | Wrong password / account policy             |
| Login succeeds but access is limited | Account privileges / shell / policy         |
| SSH works but target is unreachable  | New host network restrictions               |

Do not immediately conclude that the credential is invalid when the actual problem may be:

* wrong username
* wrong target
* wrong port
* SSH policy
* source restrictions
* account restrictions

---

## SSH and Usernames

The username matters.

For example:

```text
ssh alice@server01
```

and:

```text
ssh bob@server01
```

may produce completely different results even when both accounts exist.

Always record:

```text
Target
Username
Authentication Method
Result
Authorization
```

This is especially important in environments containing:

* local accounts
* domain accounts
* service accounts
* application accounts
* administrative accounts

---

## SSH and Non-Standard Ports

Do not assume SSH always runs on TCP 22.

If previous enumeration identifies another SSH listener, validate that service specifically.

Example:

```bash
ssh -p 2222 analyst@10.10.10.30
```

The important workflow is:

```text
Identified SSH service
        ↓
Identified port
        ↓
Reachability
        ↓
Authentication
        ↓
Authorization
```

not:

```text
Always test TCP 22
```

---

## SSH Target Mapping

Build a simple mapping when several keys, users, and hosts are present.

| Identity  | Auth Material | Target     | Port | Result     | Authorization |
| --------- | ------------- | ---------- | ---: | ---------- | ------------- |
| `analyst` | Password      | `server01` |   22 | Success    | Shell         |
| `backup`  | SSH key       | `server02` |   22 | Failed     | Unknown       |
| `deploy`  | Agent key     | `server03` | 2222 | Success    | Restricted    |
| `admin`   | Key           | `server04` |   22 | Not tested | Unknown       |

This prevents repeated or unnecessary testing.

---

## Practical Linux Workflow

A practical authorized assessment sequence:

```bash
# 1. Identify current identity
whoami
id

# 2. Review SSH configuration
ls -la ~/.ssh
cat ~/.ssh/config
cat ~/.ssh/known_hosts

# 3. Check for an SSH agent
echo "$SSH_AUTH_SOCK"
ssh-add -L

# 4. Test target reachability
nc -vz 10.10.10.30 22

# 5. Establish authorized SSH access
ssh analyst@10.10.10.30

# 6. Validate the new position
whoami
hostname
id
ip addr
ip route
```

Interpret each result before proceeding.

---

## Practical Windows Workflow

From a Windows host with OpenSSH available:

```powershell
# 1. Identify current identity
whoami
whoami /user
whoami /groups

# 2. Confirm SSH client availability
Get-Command ssh

# 3. Test SSH reachability
Test-NetConnection SERVER01 -Port 22

# 4. Establish authorized SSH access
ssh analyst@SERVER01
```

After successful access, validate the resulting session using the target's native commands.

---

## Decision Tree

```text
Do I have a specific SSH target?
        │
       No
        ↓
Return to Target Discovery
        │
       Yes
        ↓
Can I reach the SSH port?
        │
   ┌────┴────┐
   No       Yes
   │          │
Investigate   ↓
network      Is SSH actually listening?
              │
         ┌────┴────┐
        No        Yes
        │           │
   Investigate      ↓
   service/       Do I have an
   port           authorized identity?
                       │
                  ┌────┴────┐
                 No        Yes
                 │           │
          Return to          ↓
          Credential      Does authentication
          Discovery       succeed?
                              │
                         ┌────┴────┐
                        No        Yes
                        │           │
                 Investigate        ↓
                 identity/key/   Is the account
                 auth policy     authorized?
                                    │
                               ┌────┴────┐
                              No        Yes
                              │           │
                         Investigate      ↓
                           account    Validate session
                           policy          │
                                          ↓
                                  Re-enumerate host
                                          │
                                          ↓
                                  Continue the chain
```

---

## Common Mistakes

### Assuming TCP 22 means SSH access

The port only establishes network-level reachability.

### Assuming a known host means valid access

`known_hosts` is evidence of previous host-key interaction, not proof of current authentication.

### Assuming a private key guarantees access

The key may:

* belong to another user
* require a passphrase
* be revoked
* no longer be authorized
* target a different system

### Ignoring the username

SSH authentication is account-specific.

### Ignoring SSH configuration

`config` can reveal:

* usernames
* internal targets
* ports
* identity files
* jump hosts

### Forgetting SSH agents

An agent may contain usable identities even when private-key files are not visible.

### Treating authentication as authorization

A successfully authenticated account can still be restricted by server policy.

### Stopping after login

SSH access creates a new position.

Immediately reassess:

```text
Host
User
Privileges
Network
Routes
Services
Sessions
Credentials
Targets
```

### Using credentials indiscriminately

Validate credentials against specific authorized targets based on evidence rather than broadly testing them across an environment.

---

## Completion Criteria

SSH investigation is complete when you can answer:

* What target am I testing?
* Can I reach its SSH service?
* Which port is SSH using?
* Which account am I testing?
* What authentication material is available?
* Is there an SSH key or agent involved?
* Did authentication succeed?
* Is the account authorized to log in?
* What access level did the SSH session provide?
* What is the new host and network position?
* What new targets or credentials became visible?
* What should I investigate next?

If the SSH path is not usable, classify the failure and return to another movement path.

---

## What Next?

The remote-access service section is now complete.

Continue with:

**[`07-Lateral-Movement-Techniques/README.md`](../07-Lateral-Movement-Techniques/README.md)**

This section moves from individual remote services to the actual **cross-host movement workflow**, covering:

* Windows-to-Windows movement
* Linux-to-Linux movement
* Windows-to-Linux movement
* Linux-to-Windows movement
* Active Directory lateral movement
