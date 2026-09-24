# SSH Keys

SSH keys are authentication credentials used to establish SSH connections without necessarily requiring a password.

For lateral movement, the important question is not simply:

> "Are there SSH keys on this system?"

The practical question is:

> **Which SSH keys or configuration artifacts exist, which identity do they belong to, which remote systems are associated with them, and can they establish authorized access to another host?**

The workflow is:

```text
SSH Artifacts
      ↓
Identify Key Type
      ↓
Identify Owner
      ↓
Inspect SSH Configuration
      ↓
Identify Candidate Hosts
      ↓
Determine Authentication Compatibility
      ↓
Validate Authorized SSH Access
      ↓
Validate Authorization
      ↓
Record Result
      ↓
Reassess Movement Paths
```

---

## 1. Where This Fits

SSH keys are one branch of credential and access discovery:

```text
Credentials
    ↓
Passwords
    ↓
Hashes
    ↓
Kerberos
    ↓
Tokens and Sessions
    ↓
SSH Keys
    ↓
Existing Access
```

SSH is especially relevant to:

```text
Linux
Unix
Network Devices
Cloud Hosts
Development Infrastructure
Administrative Systems
```

It can also be present on Windows systems when OpenSSH is installed and configured.

---

## 2. Why SSH Keys Matter

A password-based SSH path looks like:

```text
Username
   +
Password
   ↓
SSH
   ↓
Target
```

A key-based path can look like:

```text
Username
   +
Private Key
   ↓
SSH
   ↓
Target
```

The key may therefore provide a movement path without discovering the user's plaintext password.

However:

```text
Private Key Found
      ≠
SSH Access Guaranteed
```

The key may be:

```text
Expired in practice
Revoked
Unused
Associated with another account
Restricted
Protected by a passphrase
Authorized only on specific hosts
```

---

## 3. Public vs Private Keys

SSH normally uses a key pair:

```text
Private Key
     +
Public Key
```

The private key is kept by the client.

The public key is placed on the server in an authorized location.

Conceptually:

```text
Client
Private Key
    │
    │ Authentication
    ▼
Server
Authorized Public Key
```

For lateral movement, the private key is the sensitive authentication material.

---

## 4. Common SSH Key Types

You may encounter:

```text
RSA
ED25519
ECDSA
DSA
```

Modern systems commonly use:

```text
ED25519
RSA
```

The key type itself does not determine whether the key provides useful access.

You still need:

```text
Owner
Target
Username
SSH Configuration
Server Authorization
```

---

## 5. First Identify the Current User

Before searching for SSH artifacts, establish the current identity.

### Linux

```bash id="w8l3b6"
whoami
id
```

### Windows

```cmd id="q4x7m1"
whoami
whoami /user
```

This establishes:

```text
Current User
Account Authority
Home Directory
Security Context
```

---

## 6. Linux: Inspect the Current SSH Directory

For the current user:

```bash id="p6m2r9"
ls -la ~/.ssh
```

Typical files may include:

```text
id_rsa
id_rsa.pub
id_ed25519
id_ed25519.pub
authorized_keys
known_hosts
config
```

Each has a different purpose.

---

## 7. Understand the SSH Files

### Private Key

Examples:

```text id="c7v5n2"
id_rsa
id_ed25519
```

Potential authentication material.

---

### Public Key

Examples:

```text id="m3q8x6"
id_rsa.pub
id_ed25519.pub
```

Public portion of the key pair.

---

### `authorized_keys`

Contains public keys authorized to authenticate to the account.

Conceptually:

```text id="r4k1p7"
Public Key
    ↓
authorized_keys
    ↓
Account
    ↓
SSH Login
```

It does not itself provide the private key.

---

### `known_hosts`

Contains host keys associated with systems previously contacted through SSH.

It can reveal:

```text id="n8x2m5"
Previously Contacted Hosts
Hostnames
IP Addresses
Host Key Information
```

This makes it useful for target discovery.

---

### `config`

SSH client configuration can define:

```text id="v6q9c3"
Host Aliases
Remote Hostnames
Usernames
Identity Files
ProxyJump
Ports
Connection Options
```

This file can be extremely useful for understanding existing SSH relationships.

---

## 8. `known_hosts` as a Target Discovery Source

Suppose:

```bash id="a3r7k2"
cat ~/.ssh/known_hosts
```

reveals:

```text id="h9m4v6"
server01.corp.local
server02.corp.local
10.10.20.15
```

This does not prove current access.

It does show:

```text id="x5p8n1"
This user or system previously established SSH trust with these hosts.
```

Therefore:

```text id="j2q6m9"
known_hosts
   ↓
Candidate Hosts
   ↓
Reachability
   ↓
Authentication
   ↓
Authorization
```

---

## 9. SSH Configuration as a Movement Map

Inspect the configuration when relevant:

```bash id="e8w3q5"
cat ~/.ssh/config
```

You may find:

```text id="k7m1x4"
Host web01
    HostName 10.10.20.10
    User deploy
    IdentityFile ~/.ssh/id_ed25519
```

This is highly useful information.

It maps:

```text id="z4p9c2"
Alias:
web01

Actual Host:
10.10.20.10

Username:
deploy

Identity:
~/.ssh/id_ed25519
```

The configuration itself may reveal an existing intended access path.

---

## 10. Do Not Assume the Configured Key Works

A configuration entry is evidence, not proof.

For example:

```text id="r8n3v6"
IdentityFile ~/.ssh/id_ed25519
```

does not prove:

```text id="u2k7m5"
The key is currently accepted.
```

The target may have:

```text id="c4x9p1"
Removed the public key
Disabled the account
Changed authorization
Changed SSH configuration
Changed the hostname
```

Validate the actual path.

---

## 11. Identify the Key Owner

A key should always be associated with an identity.

For example:

```text id="f6q2n8"
Private Key:
~/.ssh/id_ed25519

Owner:
alice

Candidate User:
alice

Candidate Target:
server01
```

Do not assume:

```text id="x7m4r9"
Current User = Key Owner
```

when examining files belonging to another account.

---

## 12. Other Users' SSH Directories

On an authorized assessment, you may encounter SSH artifacts belonging to other local users.

The important questions are:

```text id="p3v8k1"
Who owns the directory?
What permissions exist?
Which key files exist?
What targets are referenced?
Is the material actually usable?
```

Respect system permissions and assessment scope.

Do not treat every file as automatically accessible.

---

## 13. Linux: Locate Relevant SSH Artifacts

For the current account, start with the known SSH directory:

```bash id="q5n9m2"
ls -la ~/.ssh
```

Then inspect configuration:

```bash id="y3k7p4"
cat ~/.ssh/config
```

and known hosts:

```bash id="n6r1x8"
cat ~/.ssh/known_hosts
```

Use broader filesystem searches only when there is a specific reason.

The workflow should be:

```text id="v2m8c5"
Focused Search
      ↓
Evidence
      ↓
Broader Search if Justified
```

rather than searching the entire filesystem blindly.

---

## 14. Private Key Permissions

SSH private keys commonly have restrictive permissions.

Inspect:

```bash id="h4q9x7"
ls -l ~/.ssh/
```

A private key that is broadly accessible may represent a local security issue.

A private key that is inaccessible to the current user may still be useful as environmental evidence, but it is not automatically usable.

---

## 15. Passphrase-Protected Keys

A private key may itself be protected by a passphrase.

Conceptually:

```text id="k8m3p6"
Private Key
    ↓
Encrypted / Passphrase Protected
    ↓
Additional Authentication Requirement
```

Therefore:

```text id="c5x1r9"
Private Key Found
      ≠
Immediate SSH Authentication
```

The key's protection state is part of your assessment.

---

## 16. SSH Agent

An SSH agent can hold private keys and perform authentication on behalf of a user.

Check whether an agent is available:

```bash id="m7v2n4"
echo "$SSH_AUTH_SOCK"
```

If a socket is configured, an SSH agent may be present.

You can inspect the agent's currently loaded public keys where authorized:

```bash id="p9k5x1"
ssh-add -L
```

If the command returns keys, record:

```text id="q4m8c2"
Key Type
Fingerprint
Current User / Context
```

The important concept is:

```text id="j6n3r7"
SSH Agent
   ↓
Loaded Identity
   ↓
Potential Authentication Context
```

Do not assume that a loaded key is authorized on any particular target.

---

## 17. SSH Key Fingerprints

A fingerprint provides a compact identifier for a key.

For example:

```text id="x2c7m9"
SHA256:xxxxxxxxxxxxxxxx
```

Fingerprints are useful for:

```text id="v5p1k8"
Comparing Keys
Identifying Reuse
Recording Evidence
Correlating Configuration
```

When documenting an assessment, record fingerprints rather than unnecessarily copying private key material.

---

## 18. Identify Candidate Hosts

Combine information from:

```text id="r8m2q6"
~/.ssh/config
known_hosts
Active SSH Sessions
DNS
Network Connections
Existing Access
```

to create a candidate target list.

Example:

| Source         | Host        | User    | Identity        | Status    |
| -------------- | ----------- | ------- | --------------- | --------- |
| SSH config     | web01       | deploy  | id_ed25519      | Candidate |
| known_hosts    | server02    | Unknown | Unknown         | Candidate |
| Active session | server03    | alice   | Current session | Active    |
| Network data   | 10.10.20.15 | Unknown | Unknown         | Candidate |

This is more useful than treating each artifact independently.

---

## 19. SSH Key-to-Target Mapping

Create a dedicated matrix:

| Key / Identity | User   | Target   | Port | Source     | Reachable | Authentication | Authorization |
| -------------- | ------ | -------- | ---: | ---------- | --------- | -------------- | ------------- |
| id_ed25519     | deploy | web01    |   22 | SSH config | Yes       | Unknown        | Unknown       |
| id_rsa         | alice  | server02 |   22 | SSH config | Yes       | Unknown        | Unknown       |
| Agent key      | alice  | server03 |   22 | SSH agent  | Yes       | Unknown        | Unknown       |

This allows you to choose the next action based on evidence.

---

## 20. SSH and Service Discovery

Before validating an SSH key against a target, establish that SSH is actually available.

For example:

```text id="w3q9n5"
Target:
server01

Port:
22

Service:
SSH

Reachability:
Yes
```

This is stronger evidence than:

```text id="m7x2c8"
known_hosts contains server01
```

The target must currently be reachable and expose an appropriate service.

---

## 21. SSH Username Matters

A key is normally associated with a particular account on the target.

For example:

```text id="f8n4p2"
Key:
id_ed25519

Target:
server01

Username:
deploy
```

This is different from:

```text id="c6m1r9"
Target:
server01

Username:
root
```

Do not assume that a key authorized for one account works for another.

---

## 22. SSH Access Does Not Equal Privilege

Suppose authentication succeeds as:

```text id="k2v7x5"
deploy
```

That proves:

```text id="q9m3n1"
SSH Authentication
```

It does not automatically prove:

```text id="r4c8p6"
Root Access
```

or:

```text id="x1m5v7"
Administrative Access
```

After movement, perform normal position assessment and privilege assessment on the new host.

---

## 23. Validate the Access Path

For an authorized assessment, validate the candidate path using the appropriate SSH client workflow.

The decision sequence is:

```text id="a6p2r8"
Key Found
   ↓
Identify Owner
   ↓
Identify Username
   ↓
Identify Target
   ↓
Check Port / Service
   ↓
Check Reachability
   ↓
Validate Authentication
   ↓
Validate Authorization
```

Record the exact result.

---

## 24. Authentication Success

If SSH authentication succeeds:

```text id="n7c4m2"
Authentication:
Successful
```

record:

```text id="p3x8q6"
Target
Username
Authentication Method
Session Type
Privilege Level
Available Services
```

Then immediately begin post-movement enumeration.

The workflow is:

```text id="w9m1r5"
SSH Access
   ↓
New Host
   ↓
Position Assessment
   ↓
Network Context
   ↓
Credential / Access Discovery
   ↓
Next Movement Path
```

---

## 25. Authentication Failure

If authentication fails, classify the reason where possible.

Potential causes:

```text id="g4k8x2"
Wrong Username
Wrong Key
Key Not Authorized
Passphrase Required
SSH Service Unavailable
Account Disabled
Network Restriction
Authentication Policy
Host Key / Configuration Issue
```

Do not immediately conclude that the key is useless.

Determine whether the failure belongs to:

```text id="s6q2m8"
Key
Identity
Target
Service
Network
Policy
```

---

## 26. Known Hosts vs Authorized Keys

These files are often confused.

### `known_hosts`

Answers:

> Which SSH servers has this client encountered?

```text id="h8m3p7"
Client
 ↓
known_hosts
 ↓
Remote Hosts
```

### `authorized_keys`

Answers:

> Which public keys are authorized to log into this account?

```text id="x5r1n9"
Public Key
 ↓
authorized_keys
 ↓
Local Account
```

The direction is different.

---

## 27. SSH Keys and Lateral Movement Direction

SSH artifacts can reveal movement in either direction.

### Outbound

```text id="q7c2m5"
Current Host
 ↓
Private Key
 ↓
Remote Server
```

### Inbound

```text id="n4v8x1"
Remote User
 ↓
Public Key
 ↓
Current Host
```

For lateral movement, outbound authentication is usually the more direct movement question.

Inbound authorization is still useful because it reveals who or what may already have access to the current host.

---

## 28. Authorized Keys as Environmental Evidence

Suppose:

```text id="k9p3r6"
authorized_keys
```

contains a key associated with:

```text id="w2m7x5"
automation
```

This may reveal:

```text id="b8n4c1"
Automation Account
Deployment System
Administrative Workflow
Remote Management
```

The key itself may not be available locally.

Therefore:

```text id="v6q1m8"
authorized_keys
      ↓
Access Relationship
```

not:

```text id="p3r7x2"
authorized_keys
      ↓
Private Credential
```

---

## 29. SSH Config and ProxyJump

SSH configuration may contain proxying relationships such as:

```text id="z4m8c2"
ProxyJump
```

Conceptually:

```text id="q7n1v5"
Current Host
      ↓
Jump Host
      ↓
Internal Target
```

This can reveal an existing path into an otherwise less-visible network.

It is therefore relevant to:

```text id="j3r6m9"
Pivoting
Internal Network Access
```

which is covered later in:

```text id="c8p2x7"
09-Pivoting-and-Internal-Network-Access/
```

---

## 30. SSH Configuration and Alternate Ports

The SSH configuration may specify:

```text id="w5m9q3"
Port
```

rather than the default:

```text id="x2k7r4"
22
```

Therefore, when analyzing a configured host, record:

```text id="n6p1v8"
Host
Hostname
Port
Username
IdentityFile
ProxyJump
Other Relevant Options
```

This prevents false assumptions about the target.

---

## 31. SSH Key Reuse

A key may be configured for multiple hosts.

For example:

```text id="v8c3m5"
id_ed25519
   ├── web01
   ├── web02
   └── backup01
```

This creates multiple candidate paths.

Do not assume the key is actually authorized everywhere.

Validate each target based on scope and evidence.

---

## 32. SSH Key Rotation

A discovered key may be old.

Ask:

```text id="m4x8q1"
When was the key created?
Is it referenced by current configuration?
Does the target still authorize it?
Is the associated account active?
```

Old keys are useful evidence but should not automatically be treated as valid credentials.

---

## 33. Cloud and Development Hosts

SSH keys are frequently used in:

```text id="r7n2c5"
Cloud Servers
Development Systems
CI/CD Hosts
Git Servers
Backup Systems
Administrative Jump Hosts
```

Therefore, an SSH key or configuration artifact may reveal infrastructure beyond the local subnet.

The workflow remains:

```text id="a5k9m3"
Artifact
 ↓
Identity
 ↓
Target
 ↓
Reachability
 ↓
Authentication
 ↓
Authorization
```

---

## 34. Git and SSH Configuration

SSH configuration can sometimes reference Git infrastructure.

For example:

```text id="j8v3p6"
Host git.internal
    HostName git.internal.corp.local
    User git
```

This reveals:

```text id="f2m7r4"
Internal Git Host
SSH Relationship
Potential Infrastructure Dependency
```

It does not automatically mean:

```text id="q6x1n8"
Repository Access
```

Validate the actual authorized access.

---

## 35. Do Not Publish Private Keys

Never place real private keys in:

```text id="w4c8m2"
GitHub
README Files
Public Reports
Screenshots
Issue Trackers
Shared Notes
```

For documentation, use:

```text id="n7p3x5"
Key Fingerprint
Key Type
File Path
Owner
Target
```

instead.

The workflow repository itself should contain only sanitized examples.

---

## 36. SSH Key Investigation Record

Use this structure:

```text id="m2r8v4"
Key / Artifact:
____________________

Key Type:
____________________

Fingerprint:
____________________

Owner:
____________________

Username:
____________________

Source:
____________________

Target:
____________________

Port:
____________________

Identity File:
____________________

Authentication State:
____________________

Authorization State:
____________________

Passphrase Protected:
____________________

Evidence:
____________________

Movement Relevance:
____________________

Next Action:
____________________
```

---

## 37. Decision Tree

```text id="c5n8q2"
SSH Artifact Found
        │
        ▼
Identify Artifact Type
        │
        ├── Private Key
        ├── Public Key
        ├── known_hosts
        ├── authorized_keys
        └── SSH Config
        │
        ▼
Identify Owner / Context
        │
        ▼
Identify Candidate Target
        │
        ▼
Identify Username
        │
        ▼
Check SSH Service
        │
        ▼
Check Reachability
        │
        ▼
Is Authentication Plausible?
       │          │
      No         Yes
       │          │
       ▼          ▼
Record /      Validate
Reassess      Authentication
                  │
                  ▼
             Validate Authorization
                  │
             ┌────┴────┐
             ▼         ▼
          Useful     Not Useful
             │         │
             ▼         ▼
       New Host /   Record Result
       New Path          │
             │           │
             └─────┬─────┘
                   ▼
            Update Movement Map
```

---

## 38. Practical Linux Workflow

From a Linux foothold:

```text id="u9p4m7"
1. whoami
2. id
3. ls -la ~/.ssh
4. Inspect SSH configuration
5. Inspect known_hosts
6. Check active SSH sessions
7. Check SSH agent context where relevant
8. Identify candidate users
9. Identify candidate targets
10. Check SSH service and reachability
11. Validate authorized authentication
12. Validate authorization
13. Record result
14. Reassess movement paths
```

The key reasoning chain is:

```text id="x6m2c8"
SSH Artifact
 ↓
Owner
 ↓
Username
 ↓
Target
 ↓
SSH Service
 ↓
Authentication
 ↓
Authorization
```

---

## 39. Practical Windows Workflow

If OpenSSH is installed on Windows, inspect the current user's SSH environment.

For example:

```powershell id="q3n8v1"
Get-ChildItem $HOME\.ssh
```

You may encounter:

```text id="f7m4x9"
config
known_hosts
id_ed25519
id_ed25519.pub
id_rsa
id_rsa.pub
```

The same reasoning applies:

```text id="a8p2k5"
Artifact
 ↓
Owner
 ↓
Target
 ↓
Username
 ↓
SSH Service
 ↓
Authentication
 ↓
Authorization
```

---

## 40. Common Mistakes

### Mistake 1: Treating `known_hosts` as Credentials

`known_hosts` contains host verification information, not private authentication keys.

---

### Mistake 2: Treating a Public Key as a Private Key

A public key alone does not provide the corresponding private authentication capability.

---

### Mistake 3: Assuming a Private Key Works Everywhere

A private key must correspond to an authorized public key on the target account.

---

### Mistake 4: Ignoring the Username

The same key may be authorized for one account but not another.

---

### Mistake 5: Ignoring SSH Configuration

`config` can reveal targets, usernames, ports, identity files, and jump hosts.

---

### Mistake 6: Ignoring Key Passphrases

A private key may require additional authentication.

---

### Mistake 7: Treating Historical Hosts as Current Targets

`known_hosts` shows previous SSH relationships, not necessarily current availability.

---

### Mistake 8: Storing Real Private Keys in Notes

Record metadata and fingerprints instead.

---

## 41. Completion Criteria

SSH-key investigation is sufficiently complete for the current position when you can answer:

```text id="e8q4m7"
[ ] Are SSH artifacts present?
[ ] What type of artifacts exist?
[ ] Who owns them?
[ ] Which private keys exist?
[ ] Are keys passphrase protected?
[ ] Is an SSH agent present?
[ ] Which hosts appear in SSH configuration?
[ ] Which hosts appear in known_hosts?
[ ] Which users are associated with candidate connections?
[ ] Which SSH services are reachable?
[ ] Which authentication paths are plausible?
[ ] Has authentication been validated?
[ ] Has authorization been validated?
[ ] Are there jump-host or proxy relationships?
[ ] Has the movement map been updated?
[ ] What is the next action?
```

The objective is not:

```text id="r5m1x8"
Collect every SSH key.
```

It is:

```text id="c7q3n9"
Determine whether existing SSH authentication artifacts
create a realistic and authorized path to another host.
```

---

## 42. What Next?

The final branch of this credential-and-access-discovery section is broader than any single credential type:

> **"What remote access do I already have, even if I have not yet identified a reusable password, hash, ticket, token, or SSH key?"**

Continue with:

```text id="v2m8q5"
05-Credential-and-Access-Discovery/
└── Existing-Access.md
```

The workflow becomes:

```text id="n4p7x1"
SSH Keys
      ↓
Existing Access
      ↓
Identify Already-Available Access
      ↓
Map Access to Target
      ↓
Validate Authentication
      ↓
Validate Authorization
      ↓
Determine Movement Value
      ↓
Move or Continue Discovery
```

The core principle is:

> **An SSH artifact is valuable because of the access relationship it represents. Identify the owner, target, username, service, authentication state, and authorization before treating it as a lateral-movement path.**
