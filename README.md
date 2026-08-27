# Active Directory Red Team Lab

## Project Overview

This repository documents an authorized, self-directed red team assessment performed against an intentionally vulnerable Active Directory (AD) environment built and operated entirely within an isolated VirtualBox laboratory. The objective was to construct a realistic small-organization AD environment, deliberately introduce common real-world misconfigurations, and document a complete, evidence-based attack chain from initial access through to domain-level compromise.

The lab is **not** a production environment, is **not** connected to any organization's real infrastructure, and was built solely for authorized educational and portfolio purposes.

## Objectives

- Design and deploy a functioning multi-machine Active Directory environment from scratch.
- Identify and deliberately introduce common AD and endpoint misconfigurations found in real-world environments.
- Execute and document a full attack chain: reconnaissance, initial access, privilege escalation, credential access, lateral movement, Active Directory exploitation, and domain-level compromise.
- Map each finding to defensive remediation guidance.
- Produce professional-grade documentation suitable for a security portfolio, academic submission, and technical interviews.

## Environment

| Component | Details |
|---|---|
| Virtualization Platform | VirtualBox |
| Network | Isolated internal VirtualBox network (no production connectivity) |
| Domain Controller | Windows Server 2019 (Active Directory Domain Services) |
| Workstation | Windows 11 (domain-joined) |
| Attacker Platform | Kali Linux |

All three systems reside on the same isolated internal network segment, simulating a small corporate LAN with no route to external production systems.

## Architecture

A full breakdown of the network topology, machine roles, and trust relationships is documented in [`architecture/lab-architecture.md`](architecture/lab-architecture.md).

[`evidence/network-environment.png`](../evidence/network-environment.png)

## Attack Lifecycle

The assessment followed a standard offensive security kill chain, adapted for an Active Directory environment:

1. **Reconnaissance / Enumeration** — Network and service discovery against the domain environment.
2. **Initial Access** — Foothold established on the Windows 11 workstation.
3. **Local Privilege Escalation** — Escalation from a standard domain user to SYSTEM-level access on the workstation.
4. **Credential Access** — Extraction of cached credential material from the compromised workstation.
5. **Lateral Movement** — Use of extracted credential material to authenticate to the Domain Controller.
6. **Active Directory Exploitation** — Direct exploitation of Kerberos-related AD misconfigurations.
7. **Domain-Level Compromise** — Extraction of domain-wide credential material and forged authentication demonstrating full, persistent domain control.

Detailed, stage-by-stage technical documentation for each phase is available under [`attack-chain/`](attack-chain/).

## Repository Structure

```
├── README.md
├── architecture/
│   └── lab-architecture.md        # Network topology and machine roles
├── infrastructure/
│   └── virtualbox/
│       └── environment.md         # VM build/configuration details
├── reconnaissance/
│   └── reconnaissance-readme.md   # Enumeration methodology and findings
├── attack-chain/
│   ├── 01-initial-access.md
│   ├── 02-privilege-escalation.md
│   ├── 03-credential-access.md
│   ├── 04-lateral-movement.md
│   ├── 05-active-directory.md
│   └── 06-domain-compromise.md
├── docs/
│   ├── attack-chain.md            # High-level visual summary of the full chain
│   └── remediation.md             # Pointer to full remediation guidance
├── remediation/
│   └── recommendations.md         # Finding-by-finding defensive recommendations
├── scripts/                       # [INSERT: automation / payload generation scripts, if published]
└── evidence/                      # [INSERT: screenshots and command output referenced throughout]
```

## Key Findings

The assessment identified multiple independent weaknesses across the endpoint and Active Directory layers, including:

- Execution of an untrusted payload by a standard domain user (simulated phishing).
- A local privilege escalation vector on the workstation permitting standard users to gain SYSTEM-level access.
- Exposure of privileged (Domain Administrator) credential material in workstation memory following an interactive administrative logon.
- Acceptance of NTLM credential material (Pass-the-Hash) for authentication to the Domain Controller.
- A Kerberos-exploitable service account (Kerberoasting).
- A domain user account configured without Kerberos pre-authentication (AS-REP Roasting).
- Excessive directory replication rights enabling full domain credential extraction.

A complete, evidence-mapped findings list is maintained in [`remediation/recommendations.md`](remediation/recommendations.md).

## Remediation

Key defensive themes arising from this assessment include privileged account tiering, credential exposure controls (Credential Guard / LSA Protection), service account hardening (gMSA), Kerberos pre-authentication enforcement, and restriction of directory replication rights. Full recommendations, mapped to each finding, are documented in [`remediation/recommendations.md`](remediation/recommendations.md).

## Formal Report

A complete, narrative-style Red Team Assessment Report (Executive Summary, full attack lifecycle, findings, and remediation) is available at:

[INSERT LINK TO REPORT PDF/DOCX IN REPO]

## Disclaimer

This project was conducted entirely within an isolated, self-owned VirtualBox laboratory environment created exclusively for authorized security research, education, and portfolio purposes. No real organization, network, system, or individual was targeted at any point. All techniques described mirror real-world adversary behavior but were executed only against infrastructure built and controlled by the author.
