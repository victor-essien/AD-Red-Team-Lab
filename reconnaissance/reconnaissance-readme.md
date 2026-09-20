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
| [INSERT — additional enumeration tools actually used, e.g., NetExec/CrackMapExec, Impacket enumeration scripts] 

## Network Discovery

| Host | IP Address | OS | Role | Discovered Services | Notes |
|---|---|---|---|---|---|
| WIN-DC | WINDOWS SERVER IP | Windows Server 2019 | Domain Controller | 53, 88, 135, 389, 445, 3268, per actual nmap output |  |


## Service Enumeration


## Target System Information
* **IP Address:** 192.168.56.105
* **Host Name:** WIN-DC
* **Domain Name:** LAB.local0.
* **Active Directory Site:** Default-First-Site-Name
* **Operating System:** Microsoft Windows


## Confirmed Open Ports and Service Versions

| Port / Protocol | State | Service | Version / Details |
| :--- | :--- | :--- | :--- |
| **53 / tcp** | Open | domain | Simple DNS Plus |
| **88 / tcp** | Open | kerberos-sec | Microsoft Windows Kerberos (Server time: 2026-07-16 13:15:17Z) |
| **135 / tcp** | Open | msrpc | Microsoft Windows RPC |
| **139 / tcp** | Open | netbios-ssn | Microsoft Windows netbios-ssn |
| **389 / tcp** | Open | ldap | Microsoft Windows Active Directory LDAP |
| **445 / tcp** | Open | microsoft-ds? | *Version not explicitly identified* |
| **464 / tcp** | Open | kpasswd5? | *Version not explicitly identified* |
| **593 / tcp** | Open | ncacn_http | Microsoft Windows RPC over HTTP 1.0 |
| **636 / tcp** | Open | ssl/ldap | Microsoft Windows Active Directory LDAP |
| **3268 / tcp** | Open | ldap | Microsoft Windows Active Directory LDAP |
| **3269 / tcp** | Open | ssl/ldap | Microsoft Windows Active Directory LDAP |
| **5985 / tcp** | Open | http | Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP) |

## Scan Summary
* **Filtered Ports:** Filtered TCP ports (no-response)
* **Total Time:** Scanned 1 IP address in 51.41 seconds



## Findings From Reconnaissance

Reconnaissance confirmed the presence and role of the Domain Controller (WIN-DC) within the environment, providing the fingerprint (open Kerberos/LDAP/SMB ports) needed to identify it as the Active Directory server and primary high-value target for the remainder of the assessment. 

## Security Significance

Even before any exploitation occurs, reconnaissance demonstrates that a Domain Controller is often trivially identifiable on an internal network purely from its default port footprint (Kerberos on 88, LDAP on 389, SMB on 445, etc.). This is a normal and largely unavoidable characteristic of how Active Directory functions, which is why internal network segmentation, monitoring for anomalous internal scanning, and hardening of exposed AD services are important compensating controls rather than relying on "security through obscurity" of the Domain Controller's identity.

## Evidence
[`evidence/nmap-dc.png`](../evidence/nmap-dc.png)
