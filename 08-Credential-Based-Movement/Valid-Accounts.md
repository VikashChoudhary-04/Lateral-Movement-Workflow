# Valid Accounts

Valid-account movement occurs when an existing account and usable authentication material provide access to another host or service.

The important question is not simply:

> "Do I have a username and password?"

The real question is:

```text
What identity do I have?
        ↓
Where is that identity valid?
        ↓
What authentication method can use it?
        ↓
Which target exposes a compatible service?
        ↓
Is the target reachable?
        ↓
Does authentication succeed?
        ↓
Is the account authorized for the required action?
        ↓
What does the new position reveal?
```

This workflow applies to both local and domain accounts.

## Objective

Use known valid accounts to establish authorized access to another system while determining:

* account context
* authentication method
* target systems
* compatible services
* authentication result
* authorization level
* resulting host position
* next enumeration opportunities

A valid account does **not** automatically provide administrative access.

## 1. Identify the Account Context

First determine what kind of account you have.

Common contexts include:

| Account Type           | Example             | Typical Scope                  |
| ---------------------- | ------------------- | ------------------------------ |
| Local account          | `HOST01\user`       | Specific host                  |
| Domain account         | `DOMAIN\user`       | Domain environment             |
| Service account        | `DOMAIN\svc_backup` | Application/service-related    |
| Administrative account | `DOMAIN\admin`      | Potentially broader privileges |
| SSH user               | `user@linux-host`   | Linux/Unix system              |

On Windows, identify the current context:

```powershell
whoami
whoami /user
whoami /groups
```

For domain context:

```powershell
$env:USERDOMAIN
$env:USERDNSDOMAIN
```

On Linux:

```bash
whoami
id
groups
```

The purpose is to understand **which security principal you are dealing with**, not merely to collect a username.

## 2. Determine How the Account Authenticates

Identify the authentication material available.

Examples include:

```text
Username + Password
Username + SSH Private Key
Username + Certificate
Domain Identity + Kerberos
Other authorized authentication mechanisms
```

Do not assume that a credential valid for one service is automatically valid everywhere.

For example:

```text
Username + Password
        ↓
SSH
```

may work while:

```text
Username + Password
        ↓
RDP
```

does not.

The difference may be caused by service configuration, account restrictions, authentication policy, or authorization.

## 3. Identify Candidate Targets

Use information already gathered during target discovery.

Useful sources include:

* known hostnames
* IP addresses
* routing information
* DNS
* domain computer information
* previously discovered services
* network shares
* configuration files
* existing connections
* known administrative relationships

Do not immediately test every host.

Instead, build a small target set:

```text
Credential
   ↓
Account Context
   ↓
Known Hosts
   ↓
Compatible Services
   ↓
Relevant Targets
```

## 4. Check Reachability

Before troubleshooting credentials, determine whether the target is reachable.

### Windows

```powershell
Test-NetConnection TARGET -Port 445
```

For WinRM:

```powershell
Test-NetConnection TARGET -Port 5985
Test-NetConnection TARGET -Port 5986
```

For RDP:

```powershell
Test-NetConnection TARGET -Port 3389
```

### Linux

```bash
nc -vz TARGET 22
nc -vz TARGET 445
nc -vz TARGET 3389
```

A failed connection does not prove the credentials are invalid.

Classify the failure first:

```text
Network Failure
    ≠
Authentication Failure
```

## 5. Map the Account to a Service

Choose the service based on evidence.

Common Windows paths include:

```text
SMB  → 445
WinRM → 5985 / 5986
RDP → 3389
RPC/WMI → 135 + related RPC services
```

Linux commonly uses:

```text
SSH → 22 or configured alternative port
```

The basic decision is:

```text
Valid Account
      ↓
Which service can authenticate this identity?
      ↓
Is that service exposed?
      ↓
Is the account authorized to use it?
```

## 6. Validate SMB Access

For an authorized Windows assessment, test whether the account can authenticate to SMB.

From Windows:

```powershell
net use \\TARGET\IPC$ /user:DOMAIN\username
```

Then, if appropriate:

```powershell
net view \\TARGET
```

From Linux:

```bash
smbclient -L //TARGET -U 'DOMAIN\username'
```

Successful authentication does not necessarily mean administrative access.

Determine what the account can actually access.

For example:

```text
SMB Authentication
        ↓
Share Enumeration
        ↓
Readable Share?
        ↓
Writable Share?
        ↓
Administrative Share?
```

Treat each result separately.

## 7. Validate WinRM Access

If WinRM is exposed and the account is authorized for remote PowerShell access:

```powershell
Test-WSMan TARGET
```

Then an authorized session may be established with:

```powershell
Enter-PSSession -ComputerName TARGET -Credential $cred
```

Or:

```powershell
Invoke-Command -ComputerName TARGET -Credential $cred -ScriptBlock {
    hostname
    whoami
}
```

The first objective is **position validation**, not extensive command execution.

Confirm:

```powershell
hostname
whoami
```

Then assess the new host.

## 8. Validate RDP Access

RDP access depends on both authentication and authorization.

First check reachability:

```powershell
Test-NetConnection TARGET -Port 3389
```

Then determine whether the account is permitted to use RDP.

Relevant Windows groups can include:

```text
Remote Desktop Users
Administrators
```

Membership alone should not be interpreted without considering local policy and configuration.

After successful RDP access, validate the new position:

```cmd
whoami
hostname
ipconfig
```

Then continue with host enumeration.

## 9. Validate SSH Access

For Linux or systems exposing OpenSSH:

```bash
ssh username@TARGET
```

For a non-standard port:

```bash
ssh -p PORT username@TARGET
```

For a private key:

```bash
ssh -i key username@TARGET
```

After successful authentication:

```bash
whoami
id
hostname
ip addr
ip route
```

The objective is to determine:

* which user was obtained
* which host was reached
* what networks are visible
* what privileges are available
* what new credentials or targets may exist

## 10. Authentication vs Authorization

Always separate these results.

### Authentication succeeds

```text
Username + Credential
        ↓
Service accepts identity
```

This proves the credential works for that authentication path.

It does **not** prove:

```text
Administrative Access
```

### Authorization succeeds

```text
Authenticated Identity
        ↓
Required Permission Granted
```

Examples:

* SMB share can be read
* WinRM session can be established
* RDP login is permitted
* SSH session is permitted
* required administrative operation is allowed

Record these separately.

## 11. Local vs Domain Accounts

This distinction is especially important on Windows.

Consider:

```text
TARGET\administrator
```

versus:

```text
DOMAIN\administrator
```

The names may look similar, but the accounts belong to different security authorities.

When testing a credential, explicitly identify the intended context.

For example:

```text
DOMAIN\username
```

or:

```text
TARGET\username
```

A credential failing in one context does not necessarily mean the underlying password is incorrect.

## 12. Credential Reuse

If an account works on one system, determine whether there is evidence that the same identity is used elsewhere.

Useful evidence may include:

* administrator documentation
* configuration files
* service configurations
* known account ownership
* previously identified access relationships
* domain membership
* authorized credential-management information

Do not blindly test the same credential across an entire network.

Instead:

```text
Known Successful Account
        ↓
Evidence of Reuse
        ↓
Relevant Target
        ↓
Reachability
        ↓
Compatible Service
        ↓
Authorized Validation
```

## 13. Failure Handling

When authentication fails, classify the problem.

### Target Cannot Be Reached

```text
Credential
   ↓
Network
   X
```

Investigate:

* routing
* firewall
* segmentation
* VPN
* incorrect address

### Service Not Available

```text
Target Reachable
       ↓
Required Service
       X
```

Investigate:

* listening port
* service state
* non-standard port
* host firewall

### Authentication Rejected

```text
Service Reachable
       ↓
Credential
       X
```

Investigate:

* username
* password
* account context
* authentication method
* account status
* authentication restrictions

### Authentication Succeeds but Access Is Denied

```text
Authentication
      ✓
      ↓
Authorization
      X
```

This means the account may be valid but lacks the required permission.

Do not unnecessarily change credentials when the evidence indicates an authorization problem.

## 14. Validate the New Position

Once movement succeeds, immediately establish where you landed.

### Windows

```powershell
whoami
hostname
ipconfig
route print
```

For domain context:

```powershell
whoami /groups
whoami /priv
```

### Linux

```bash
whoami
id
hostname
ip addr
ip route
ss -tunap
```

The goal is to answer:

```text
Who am I?
Where am I?
What network can I see?
What services are available?
What privileges do I have?
What is different from the previous host?
```

## 15. Re-enumerate From the New Host

Do not simply repeat the exact previous actions.

The new host may have:

* different network interfaces
* access to additional subnets
* different DNS visibility
* different credentials
* different services
* different domain relationships
* different administrative privileges
* different trust relationships

The workflow becomes:

```text
Host A
  ↓
Valid Account
  ↓
Host B
  ↓
Re-enumerate Host B
  ↓
New Position
  ↓
New Credentials / Targets
  ↓
Host C
```

This is where valid-account movement becomes a chain rather than a single login.

## 16. Evidence Recording

For every successful validation, record:

```text
Source Host:
Source User:
Account Type:
Target Host:
Target IP:
Target Service:
Authentication Method:
Authentication Result:
Authorization Result:
New User:
New Host:
New Network Visibility:
New Discovery:
Next Target:
```

Example:

```text
Source Host: WORKSTATION-01
Source User: DOMAIN\user
Account Type: Domain Account
Target Host: SERVER-01
Target Service: WinRM
Authentication: Successful
Authorization: Remote PowerShell permitted
New User: DOMAIN\user
New Host: SERVER-01
Next Step: Re-enumerate SERVER-01
```

## 17. Decision Workflow

Use this workflow whenever a valid account is available:

```text
Valid Account
      ↓
Identify Local / Domain Context
      ↓
Identify Authentication Material
      ↓
Identify Candidate Targets
      ↓
Check Reachability
      ↓
Identify Compatible Service
      ↓
Authenticate
      │
      ├── Failed
      │     ↓
      │  Classify Failure
      │
      └── Successful
            ↓
       Validate Authorization
            │
            ├── Denied
            │     ↓
            │  Record Limitation
            │
            └── Allowed
                  ↓
              Establish Access
                  ↓
              Validate Position
                  ↓
              Re-enumerate
                  ↓
              Continue the Chain
```

## What to Remember

The key lessons are:

1. A username alone is not access.
2. A valid password does not guarantee authorization.
3. Local and domain accounts are different security contexts.
4. A reachable target does not mean its service is usable.
5. An exposed service does not mean the account is authorized.
6. Authentication failure and authorization failure are different problems.
7. Avoid indiscriminate credential testing.
8. Validate the new host immediately after movement.
9. Re-enumerate from every newly obtained position.
10. Treat credentials and authentication material as sensitive evidence.

## What Next?

Continue to [`Pass-the-Hash.md`](Pass-the-Hash.md).

That workflow covers situations where NTLM hash material is available and the assessment requires understanding whether it can be used as authentication material for authorized Windows lateral movement.
