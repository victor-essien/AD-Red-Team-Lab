# Reconnaissance and Enumeration

## Objective

Before any exploitation was attempted, reconnaissance was performed from the Kali Linux attacker platform to identify live hosts, exposed services, and Active Directory–specific information available to both an unauthenticated and an authenticated attacker. This establishes an honest baseline of what an attacker could realistically observe before taking any exploitative action.

## Scope

Reconnaissance was performed against the following in-scope lab systems:

- Windows Server 2019 Domain Controller (WIN-DC)
- Windows 11 workstation (WIN11-PC)

## Methodology

Reconnaissance was performed in two stages: network/service discovery, and Active Directory enumeration (unauthenticated and authenticated).

| Tool | Purpose | Target | Security Significance |
|---|---|---|---|
| `nmap` | Network and service discovery / port scanning | Windows Server 2019 (Domain Controller) | Confirms live hosts and identifies exposed services (e.g., Kerberos, LDAP, SMB) that fingerprint the host as a Domain Controller and define the attack surface |
| [INSERT — additional enumeration tools actually used, e.g., NetExec/CrackMapExec, Impacket enumeration scripts] | [INSERT] | [INSERT] | [INSERT] |

## Network Discovery

| Host | IP Address | OS | Role | Discovered Services | Notes |
|---|---|---|---|---|---|
| WIN-DC | [INSERT WINDOWS SERVER IP] | Windows Server 2019 | Domain Controller | [INSERT — e.g., 53, 88, 135, 389, 445, 3268, per actual nmap output] | [INSERT] |
| WIN11-PC | [INSERT WINDOWS 11 IP] | Windows 11 | Workstation | [INSERT] | [INSERT] |

## Service Enumeration

[INSERT: document only services actually confirmed via nmap/enumeration output — e.g., confirmed open ports and identified service versions from the Domain Controller scan]

## Active Directory Enumeration

[INSERT: document whether unauthenticated and/or authenticated AD enumeration was performed, and what was actually discovered — domain name, user accounts enumerated, groups, computers, and any notable account configurations identified (e.g., accounts with SPNs, accounts with pre-authentication disabled). Do not include actual passwords or reusable credential material.]

## Findings From Reconnaissance

Reconnaissance confirmed the presence and role of the Domain Controller (WIN-DC) within the environment, providing the fingerprint (open Kerberos/LDAP/SMB ports) needed to identify it as the Active Directory server and primary high-value target for the remainder of the assessment. [INSERT: any additional specific findings that directly informed subsequent attack-chain decisions.]

## Evidence

[INSERT SCREENSHOT]

[INSERT COMMAND OUTPUT]

[INSERT REFERENCE TO EVIDENCE FILE — e.g., `evidence/recon/nmap-dc-scan.txt`]

## Security Significance

Even before any exploitation occurs, reconnaissance demonstrates that a Domain Controller is often trivially identifiable on an internal network purely from its default port footprint (Kerberos on 88, LDAP on 389, SMB on 445, etc.). This is a normal and largely unavoidable characteristic of how Active Directory functions, which is why internal network segmentation, monitoring for anomalous internal scanning, and hardening of exposed AD services are important compensating controls rather than relying on "security through obscurity" of the Domain Controller's identity.
