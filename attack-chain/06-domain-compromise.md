# Domain-Level Compromise

## Objective

Demonstrate full, persistent domain-level compromise by extracting the domain's core Kerberos trust material and using it to forge authentication that grants Domain Administrator–equivalent access independent of any single account's actual password.

## Preconditions

- Administrative-level access to the Domain Controller (established during Lateral Movement).
- Directory replication rights sufficient to perform a DCSync-style credential extraction, available to the compromised privileged account context in use at this stage.

## Attack Path

Using the administrative session established on the Domain Controller, a directory replication request was performed to extract password material for domain accounts, including the `krbtgt` account — the account whose key material underpins the validity of all Kerberos tickets issued within the domain (MITRE ATT&CK T1003.006 — OS Credential Dumping: DCSync).

The recovered `krbtgt` key material was then used to forge a Kerberos Ticket Granting Ticket ("Golden Ticket") for a domain account, granting Domain Administrator–equivalent access on demand (MITRE ATT&CK T1558.001 — Steal or Forge Kerberos Tickets: Golden Ticket). Note: an initial attempt to forge a ticket for a non-existent account failed due to PAC (Privilege Attribute Certificate) validation performed by the Domain Controller against real AD objects; the ticket was subsequently forged referencing a real, existing domain account and its correct Relative Identifier (RID), which succeeded.

The forged ticket was used to authenticate to the Domain Controller with full administrative rights, and was confirmed to remain valid even after the referenced account's actual password was reset — demonstrating that access no longer depended on any single account's live credentials.

Full chain summary:

```
Initial Access
  → Local Privilege Escalation
    → Credential Access
      → Lateral Movement
        → Active Directory Exploitation
          → Domain-Level Privileged Access (this stage)
```

## Evidence

[INSERT SCREENSHOT — DCSync output confirming krbtgt extraction]

[INSERT SCREENSHOT — forged ticket authentication confirmation]

[INSERT SCREENSHOT — access confirmed valid after password reset]

## Impact

Domain-level compromise of this kind represents total loss of control over the organization's identity infrastructure. An attacker at this stage can impersonate any user, access any resource protected by domain authentication, extract any account's credential material at will, and — critically — retain this level of access even after standard incident-response actions such as password resets, unless the `krbtgt` account itself is specifically and correctly remediated (see below).

## Attack Chain Summary

| Stage | Target | Technique | Result |
|---|---|---|---|
| Initial Access | Windows 11 workstation | Simulated phishing / user execution | Standard domain user session |
| Privilege Escalation | Windows 11 workstation | `AlwaysInstallElevated` misconfiguration | SYSTEM-level session |
| Credential Access | Windows 11 workstation | LSASS memory extraction (offline, via native dump + `pypykatz`) | Domain Administrator NTLM hash |
| Lateral Movement | Domain Controller | Pass-the-Hash | SYSTEM-level session on Domain Controller |
| AD Exploitation | Domain Controller | Kerberoasting, AS-REP Roasting | Plaintext passwords for `SQLService` and `j.smith` |
| Domain Compromise | Domain Controller | DCSync + forged Golden Ticket | Persistent Domain Administrator–equivalent access |

## Defensive Significance

Compromise of a privileged domain identity — and particularly of the `krbtgt` account's key material — is among the most severe outcomes possible in an Active Directory environment, because it undermines the fundamental trust mechanism the entire domain relies on. Remediation at this stage requires more than a routine password reset: Microsoft-recommended guidance requires resetting the `krbtgt` account's password **twice in sequence** to fully invalidate any forged tickets, alongside a full review of all privileged account activity and directory replication permissions. This stage exists in the report specifically to demonstrate why every preceding finding — even ones that appear individually minor — must be treated as a priority, since together they form a complete path to this outcome.
