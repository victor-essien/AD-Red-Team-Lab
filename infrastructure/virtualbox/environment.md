# VirtualBox Environment

## Purpose

This document describes how the isolated laboratory environment was actually built and configured in VirtualBox, for authorized security testing and educational purposes.

## Virtualization Platform

| Item | Value |
|---|---|
| VirtualBox Version | [INSERT] |
| Host Operating System | [INSERT IF RELEVANT] |
| VM Storage Location | [NOT INCLUDED — private host path] |
| Network Configuration | Isolated internal VirtualBox network; see [`architecture/lab-architecture.md`](../../architecture/lab-architecture.md) |

## Virtual Machines

### Windows Server 2019

| Item | Value |
|---|---|
| Operating System | Windows Server 2019 |
| Hostname | WIN-DC |
| CPU | [INSERT] |
| RAM | [INSERT] |
| Disk | [INSERT] |
| Network Adapter | [INSERT — adapter type/mode] |
| IP Address | [INSERT WINDOWS SERVER IP] |
| Role | Domain Controller |
| AD DS Configuration | Promoted to Domain Controller; Active Directory Domain Services role installed and configured |
| DNS Configuration | [INSERT] |

### Windows 11

| Item | Value |
|---|---|
| Operating System | Windows 11 (24H2+, Build 26200) |
| Hostname | WIN11-PC |
| CPU | [INSERT] |
| RAM | [INSERT] |
| Disk | [INSERT] |
| Network Adapter | [INSERT] |
| IP Address | [INSERT WINDOWS 11 IP] |
| Domain Membership | Joined to the lab Active Directory domain |

### Kali Linux

| Item | Value |
|---|---|
| Operating System | Kali Linux |
| Hostname | [INSERT KALI HOSTNAME] |
| CPU | [INSERT] |
| RAM | [INSERT] |
| Disk | [INSERT] |
| Network Adapter | [INSERT] |
| IP Address | [INSERT KALI IP] |
| Purpose | Attacker platform — reconnaissance, exploitation, and post-exploitation tooling |

## Virtual Network

All three virtual machines are attached to the same isolated internal VirtualBox network, allowing them to communicate with one another directly, as machines on the same corporate LAN segment would. This network has no bridge or NAT path to the host's production network. See [`architecture/lab-architecture.md`](../../architecture/lab-architecture.md) for full addressing details.

## Domain Configuration

| Item | Value |
|---|---|
| Domain (NetBIOS) Name | LAB |
| Domain FQDN | [INSERT — e.g., LAB.local] |
| Domain Controller Hostname | WIN-DC |
| DNS Configuration | [INSERT] |
| Domain Membership | Windows 11 workstation (WIN11-PC) domain-joined |

## VM Configuration Process

- The Windows Server 2019 VM was manually configured with the Active Directory Domain Services role and promoted to Domain Controller through the standard Windows Server Manager / `dcpromo` workflow.
- Domain user accounts, a service account (used later in the attack chain), and organizational structure were created manually through Active Directory Users and Computers (ADUC) and PowerShell Active Directory cmdlets.
- The Windows 11 VM was manually joined to the domain via standard Windows domain-join settings.
- The Kali Linux VM was configured with standard offensive security tooling (Metasploit Framework, Impacket suite, Hashcat, NetExec) installed via `apt`/`pip`.
- [INSERT: Note here if any part of the configuration was later automated via Ansible/Vagrant/scripts, and reference the relevant file/directory]

## Isolation

The laboratory network has no connectivity to the host's production network or to the internet beyond what is explicitly required for tool installation on Kali. This isolation ensures that all intentionally introduced vulnerabilities and all exploitation activity remain fully contained within the lab, posing no risk to any external system.

## Reproduction Notes

To reproduce this environment, another operator would need to:

1. Install VirtualBox and create three VMs matching the specifications above.
2. Configure an isolated internal network within VirtualBox and attach all three VMs to it.
3. Install Windows Server 2019, promote it to a Domain Controller, and create the domain structure described in this document.
4. Install Windows 11 and join it to the domain.
5. Install Kali Linux and the offensive tooling referenced throughout this repository.
6. [INSERT: reference any published Ansible/Vagrant automation, if applicable, that would allow this environment to be reproduced without manual configuration]

At the time of writing, the environment was built primarily through manual configuration; full one-command reproducibility via Infrastructure-as-Code is [INSERT: CURRENT STATUS — NOT YET IMPLEMENTED / IN PROGRESS / AVAILABLE IN `infrastructure/`].
