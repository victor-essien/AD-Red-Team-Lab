# Lab Architecture

## Architecture Overview

The laboratory environment consists of three virtual machines, each representing a distinct role within a simulated small-organization network:

- **Kali Linux** — Attacker platform, used for reconnaissance, exploitation, and post-exploitation activity.
- **Windows 11** — Domain-joined workstation, representing a standard employee endpoint.
- **Windows Server 2019** — Domain Controller, providing Active Directory Domain Services (AD DS) and DNS for the environment.

All three machines were deployed and connected within a single isolated VirtualBox network, with no bridging to production or external infrastructure.

## Network Topology

All machines are connected to the same isolated VirtualBox internal network, allowing full communication between the attacker platform, the workstation, and the Domain Controller — mirroring the connectivity a real internal attacker (or a compromised internal endpoint) would have on a typical flat corporate LAN.

[`evidence/network-environment.png`](../evidence/network-environment.png)

## Network Addressing

| System | Operating System | Role | IP Address | Hostname | Notes |
|---|---|---|---|---|---|
| Kali Linux | Kali Linux | Attacker | 192.168.56.103 | kali | Attacker platform |
| Windows 11 | Windows 11 (24H2+, Build 26200) | Workstation | 192.168.156.106 | WIN11-PC | Domain-joined workstation |
| Windows Server | Windows Server 2019 | Domain Controller | 192.168.156.105 | WIN-DC | AD DS / DNS |

## VirtualBox Network Configuration

| Setting | Value |
|---|---|
| Network Type | Host-Only Adapter |
| Network Name | vboxnet0 |
| Subnet | 192.168.56.0/24 |
| DHCP | Disabled — static IP assignment used |
| Host-to-VM Communication | Enabled |
| Internet Access from VMs | Disabled |
| Isolation | Network is isolated from the host's production/external network; no route exists to systems outside the lab. |

## Machine Roles

### Kali Linux

Serves as the attacker platform for the exercise. Used to perform network and service reconnaissance, generate and deliver payloads, catch reverse connections, extract and parse credential material, perform Kerberos-based attacks against the domain, and execute lateral movement and post-exploitation actions against the Domain Controller.

### Windows 11

Represents the initial target — a standard employee workstation joined to the domain. This machine was used to simulate the initial compromise (via a simulated phishing payload), local privilege escalation, and credential exposure resulting from a prior privileged (Domain Administrator) logon.

### Windows Server 2019

Serves as the environment's Domain Controller, hosting Active Directory Domain Services and DNS. This machine represents the ultimate target of the attack chain — the system whose compromise equates to full control over the simulated organization's identity infrastructure.

## Trust and Communication Relationships

- **Kali ↔ Windows 11:** Direct network connectivity was used for payload delivery (HTTP) and command-and-control (Meterpreter reverse TCP sessions).
- **Kali ↔ Windows Server 2019:** Direct network connectivity was used for reconnaissance (port scanning), authenticated and unauthenticated enumeration, Kerberos ticket requests, and post-credential-theft authentication (SMB/Kerberos-based remote execution).
- **Windows 11 ↔ Windows Server 2019:** Standard domain authentication relationship — Windows 11 is domain-joined and authenticates against the Domain Controller via Kerberos/NTLM for logon and directory services.

Network connectivity between machines does not imply unrestricted service accessibility; each stage of the attack chain required a specific service (SMB, Kerberos, HTTP, RPC) to be reachable and, in later stages, valid credential material to be presented before access was granted.

## Security Boundary

The environment is fully isolated from any production network to ensure that all offensive techniques — including payload execution, credential extraction, and domain-wide credential compromise — remain contained entirely within the lab. This isolation is what makes it safe and legal to intentionally introduce and exploit real vulnerabilities without any risk to external systems, individuals, or organizations.
