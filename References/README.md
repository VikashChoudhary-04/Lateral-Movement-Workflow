# References

This directory contains reference material supporting the concepts, protocols, services, tools, and defensive considerations used throughout the **Lateral Movement Workflow**.

The repository is designed to be a workflow guide rather than a collection of copied commands.

References should therefore be used to:

* verify technical behavior
* understand protocol details
* confirm platform-specific functionality
* investigate defensive telemetry
* deepen knowledge after completing a workflow
* validate commands against current documentation

## Reference Philosophy

Use references in this order:

```text id="m8q3v7"
Understand the Workflow
        ↓
Perform the Relevant Enumeration
        ↓
Interpret the Result
        ↓
Consult Primary Documentation
        ↓
Validate the Next Action
```

Do not use references as a substitute for understanding the decision process.

## Primary Documentation

Prefer official documentation and standards whenever possible.

### Microsoft

Microsoft documentation is useful for Windows administration, authentication, Active Directory, PowerShell, WinRM, SMB, WMI, RPC, RDP, and Windows event logging.

Useful areas include:

* Windows security auditing
* Windows authentication
* Active Directory
* Kerberos
* NTLM
* PowerShell
* WinRM
* SMB
* WMI
* Remote Desktop Services
* Windows Event Log
* Windows Filtering Platform

Start with the official Microsoft documentation:

* [Microsoft Learn](https://learn.microsoft.com/)
* [Windows Security Auditing](https://learn.microsoft.com/windows/security/threat-protection/auditing/)
* [PowerShell Documentation](https://learn.microsoft.com/powershell/)
* [Windows Remote Management](https://learn.microsoft.com/windows/win32/winrm/portal)
* [Windows Management Instrumentation](https://learn.microsoft.com/windows/win32/wmisdk/wmi-start-page)
* [Remote Desktop Services](https://learn.microsoft.com/windows-server/remote/remote-desktop-services/)
* [Windows Server Documentation](https://learn.microsoft.com/windows-server/)

## OpenSSH

For SSH behavior, configuration, authentication, keys, and client/server options:

* [OpenSSH](https://www.openssh.com/)
* [OpenSSH Manual Pages](https://man.openbsd.org/ssh)

Useful topics include:

* SSH client configuration
* SSH server configuration
* public-key authentication
* SSH agents
* known hosts
* port forwarding
* SOCKS forwarding

## Samba / SMB

For SMB and Samba behavior:

* [Samba](https://www.samba.org/)
* [Samba Documentation](https://www.samba.org/samba/docs/)

Useful topics include:

* SMB protocol behavior
* shares
* authentication
* access controls
* Samba administration
* SMB client functionality

## Kerberos

Kerberos concepts used throughout the repository include:

* principals
* realms
* TGTs
* service tickets
* SPNs
* authentication
* time synchronization

Primary standards and references include:

* [RFC 4120 — The Kerberos Network Authentication Service](https://www.rfc-editor.org/rfc/rfc4120)
* [MIT Kerberos Documentation](https://web.mit.edu/kerberos/)
* [Microsoft Kerberos Authentication](https://learn.microsoft.com/windows-server/security/kerberos/)

## Active Directory

For Active Directory concepts and administration:

* [Microsoft Active Directory Documentation](https://learn.microsoft.com/windows-server/identity/ad-ds/)
* [Microsoft Active Directory Domain Services](https://learn.microsoft.com/windows-server/identity/ad-ds/)

Relevant topics include:

* domains
* domain controllers
* users
* groups
* trusts
* Group Policy
* Kerberos
* LDAP
* authentication
* authorization

## PowerShell

PowerShell is used throughout the Windows workflow for enumeration and validation.

Primary reference:

* [PowerShell Documentation](https://learn.microsoft.com/powershell/)

Useful topics include:

* `Get-CimInstance`
* `Get-NetTCPConnection`
* `Get-NetIPConfiguration`
* `Get-NetRoute`
* `Test-NetConnection`
* `Test-WSMan`
* remoting
* CIM
* WMI

## Linux Documentation

For Linux networking, processes, SSH, permissions, and system administration:

* [Linux man-pages](https://man7.org/linux/man-pages/)
* [Ubuntu Documentation](https://documentation.ubuntu.com/)
* [Debian Documentation](https://www.debian.org/doc/)

Useful topics include:

* `ip`
* `ss`
* `route`
* SSH
* network configuration
* processes
* services
* permissions
* authentication

## Nmap

Nmap is useful for controlled network and service discovery.

Primary reference:

* [Nmap](https://nmap.org/)
* [Nmap Reference Guide](https://nmap.org/book/man.html)

Use documentation to understand:

* host discovery
* port scanning
* service detection
* scan behavior
* output interpretation

Only scan systems that are authorized and in scope.

## Netcat

Netcat can be useful for basic connectivity validation.

Reference:

* [OpenBSD nc](https://man.openbsd.org/nc)

Typical workflow usage includes validating whether a target port is reachable.

A successful TCP connection establishes network reachability, not authentication or authorization.

## SMB Client Tools

For SMB validation from Linux environments:

* [Samba Documentation](https://www.samba.org/samba/docs/)

Common tooling concepts include:

```text
smbclient
```

Use the documentation to understand authentication, share enumeration, and SMB client behavior.

## Windows Event Logging

Windows event information should be verified against Microsoft documentation.

Useful event categories include:

```text id="q8m3v7"
4624
4625
4648
4672
4688
4768
4769
4771
4776
```

Relevant Microsoft documentation:

* [Windows Security Auditing](https://learn.microsoft.com/windows/security/threat-protection/auditing/)
* [Advanced Security Audit Policy Settings](https://learn.microsoft.com/windows/security/threat-protection/auditing/advanced-security-audit-policy-settings)

Event availability and fields depend on audit configuration and Windows version.

## Sysmon

For detailed Windows endpoint telemetry:

* [Microsoft Sysmon](https://learn.microsoft.com/sysinternals/downloads/sysmon)
* [Sysinternals](https://learn.microsoft.com/sysinternals/)

Useful areas include:

* process creation
* network connections
* DNS activity
* file activity
* registry activity
* process relationships

Sysmon configuration determines what telemetry is available.

## MITRE ATT&CK

MITRE ATT&CK provides a useful taxonomy for understanding adversary behavior and mapping techniques to defensive observations.

* [MITRE ATT&CK](https://attack.mitre.org/)
* [Enterprise ATT&CK](https://attack.mitre.org/matrices/enterprise/)

Relevant areas may include:

* Lateral Movement
* Credential Access
* Discovery
* Command and Control
* Persistence

ATT&CK is a classification and knowledge base.

It should supplement, not replace, the practical workflow in this repository.

## OWASP

For application-layer movement and authentication concepts:

* [OWASP](https://owasp.org/)
* [OWASP Web Security Testing Guide](https://owasp.org/www-project-web-security-testing-guide/)

This can be useful when lateral movement begins through an application or when application credentials provide access to additional infrastructure.

## RFCs and Internet Standards

When protocol behavior matters, use the relevant RFC or standards documentation.

The [RFC Editor](https://www.rfc-editor.org/) provides access to Internet standards and protocol specifications.

Useful topics include:

* TCP/IP
* DNS
* HTTP
* authentication protocols
* network services

## DNS

DNS is important to both lateral movement and Active Directory workflows.

References:

* [ICANN](https://www.icann.org/)
* [IETF RFCs](https://www.rfc-editor.org/)
* [Microsoft DNS Documentation](https://learn.microsoft.com/windows-server/networking/dns/)

Useful topics include:

* name resolution
* internal DNS
* service discovery
* Active Directory DNS
* Kerberos dependencies

## Network Security

For broader network-security concepts:

* [NIST Cybersecurity Framework](https://www.nist.gov/cyberframework)
* [NIST Computer Security Resource Center](https://csrc.nist.gov/)
* [CIS](https://www.cisecurity.org/)

These resources can help connect technical findings to broader security controls.

## Credential Security

Useful defensive references include:

* [Microsoft Security Documentation](https://learn.microsoft.com/security/)
* [NIST Digital Identity Guidelines](https://pages.nist.gov/800-63-3/)
* [MITRE ATT&CK Credential Access](https://attack.mitre.org/tactics/TA0006/)

Topics include:

* credential protection
* authentication
* privileged accounts
* identity security
* credential exposure

## Network Segmentation

For network architecture and defensive segmentation concepts:

* [NIST Cybersecurity Framework](https://www.nist.gov/cyberframework)
* [CISA](https://www.cisa.gov/)
* [CIS](https://www.cisecurity.org/)

Relevant concepts include:

* segmentation
* least privilege
* network access control
* administrative boundaries
* zero-trust principles

## Reference Usage by Repository Section

| Repository Section              | Useful Reference Areas                           |
| ------------------------------- | ------------------------------------------------ |
| Foundations                     | Microsoft, Linux, RFCs                           |
| First 5 Minutes                 | Windows/Linux documentation                      |
| Position Assessment             | Windows, Linux, Active Directory                 |
| Target Discovery                | Nmap, DNS, network documentation                 |
| Credential and Access Discovery | Microsoft, Kerberos, OpenSSH                     |
| Remote Access Services          | SMB, WinRM, RDP, WMI, RPC, SSH                   |
| Lateral Movement Techniques     | Microsoft, OpenSSH, Samba, Kerberos              |
| Credential-Based Movement       | Kerberos, Microsoft authentication documentation |
| Pivoting                        | SSH, networking, proxy/forwarding documentation  |
| Post-Movement Workflow          | Windows/Linux administration documentation       |
| End-to-End Scenarios            | All relevant platform references                 |
| Decision Framework              | Service and authentication documentation         |
| Detection and Defense           | Microsoft Security, Sysmon, MITRE ATT&CK, NIST   |

## Reference Verification Workflow

When a command, protocol behavior, or technical assumption is uncertain:

```text id="m7q3v8"
Identify the Question
        ↓
Find Primary Documentation
        ↓
Verify Current Behavior
        ↓
Test in Authorized Lab
        ↓
Record the Result
        ↓
Update the Workflow if Necessary
```

This keeps the repository maintainable as tools and platforms change.

## Lab Validation

Before relying on a technique in an assessment, validate it in an authorized environment.

Suitable environments include:

* personal virtual machines
* intentionally vulnerable applications
* controlled Active Directory labs
* TryHackMe rooms
* PortSwigger Web Security Academy where relevant
* other explicitly authorized training environments

The purpose of a lab is to understand:

```text
Action
 ↓
Expected Result
 ↓
Actual Result
 ↓
Interpretation
 ↓
Next Decision
```

## Safety and Scope

All techniques in this repository should be used only against systems where the operator has explicit authorization.

Before performing lateral movement, confirm:

```text
[ ] Target is in scope
[ ] Source system is in scope
[ ] Credentials are authorized for testing
[ ] Remote services are authorized for testing
[ ] Pivoting is authorized
[ ] Credential testing limits are understood
[ ] Destructive actions are excluded
[ ] Evidence-handling requirements are understood
```

Avoid unnecessary:

* credential exposure
* password guessing
* account lockouts
* destructive changes
* persistence
* data modification
* service disruption

## Keeping the Repository Current

Tools, operating systems, authentication methods, logging behavior, and documentation change over time.

When updating this repository:

1. Verify the behavior against current documentation.
2. Test important commands in an authorized lab.
3. Remove obsolete commands.
4. Preserve the workflow reasoning.
5. Update affected decision matrices.
6. Update references when official documentation changes.
7. Clearly distinguish platform-specific behavior.
8. Avoid adding commands merely because they are popular.

The workflow should remain more durable than any individual command.

## Reference Quality Hierarchy

When resolving a technical question, prefer:

```text id="x8m3q7"
Official Documentation
        ↓
Standards / RFCs
        ↓
Vendor Documentation
        ↓
Trusted Security Research
        ↓
Community Documentation
```

Community resources can be useful for practical troubleshooting, but important technical claims should be verified against authoritative documentation when possible.

## Final Repository Principle

References support the workflow.

They do not replace it.

The repository's core reasoning remains:

```text id="p5m8q2"
Where Am I?
     ↓
Who Am I?
     ↓
What Can I Reach?
     ↓
What Target Matters?
     ↓
What Service Is Available?
     ↓
What Authentication Material Fits?
     ↓
Can I Authenticate?
     ↓
What Am I Authorized To Do?
     ↓
What Access Did I Gain?
     ↓
What Did the New Position Reveal?
     ↓
What Should I Do Next?
```

> **Don't just run commands. Understand the position, interpret the result, make the next decision, and validate the new position.**

## Repository Complete

The **Lateral-Movement-Workflow** repository now covers the complete workflow:

```text id="q7m3v8"
Foundations
    ↓
First 5 Minutes
    ↓
Position Assessment
    ↓
Target Discovery
    ↓
Credential & Access Discovery
    ↓
Remote Access Services
    ↓
Lateral Movement Techniques
    ↓
Credential-Based Movement
    ↓
Pivoting & Internal Network Access
    ↓
Post-Movement Workflow
    ↓
End-to-End Scenarios
    ↓
Decision Framework
    ↓
Detection & Defense
    ↓
References
```

The repository can now be used as a practical end-to-end guide for reasoning about lateral movement from an initial compromised position through target discovery, authentication, movement, validation, re-enumeration, pivoting, decision-making, and defensive visibility.
