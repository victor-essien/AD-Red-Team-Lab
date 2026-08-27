# Credential Access

## Objective

Extract credential material cached in memory on the compromised Windows 11 workstation, specifically targeting any privileged domain account that had previously logged on interactively to the machine.

## Source of Credential Material

A Domain Administrator account had, prior to this stage, logged on interactively to the Windows 11 workstation (simulating a realistic scenario such as an IT administrator troubleshooting a user's machine). This left the account's authentication material cached in the Local Security Authority Subsystem Service (LSASS) process memory.

## Technique Used

Credential dumping from LSASS process memory (MITRE ATT&CK T1003.001 — OS Credential Dumping: LSASS Memory).

Two approaches were attempted:

1. **Live in-memory extraction** using Mimikatz (via Meterpreter's `kiwi` extension, and subsequently a standalone Mimikatz binary), executed from the SYSTEM-level session obtained during privilege escalation.
2. **Offline extraction**, used after live extraction was blocked by modern Windows credential-protection features (LSA Protection / Credential Guard). A full memory dump of the LSASS process was created using native Windows tooling, transferred to the Kali attacker platform, and parsed offline using `pypykatz`.

## Result

NTLM credential hash material for the Domain Administrator account was successfully recovered via the offline LSASS dump and parsing method.

## Security Significance

This confirms that a single interactive logon by a highly privileged account on an ordinary, lower-trust endpoint is sufficient to expose that account's credential material to any attacker who subsequently gains SYSTEM-level access to that endpoint — regardless of the endpoint's own relative unimportance.

## Evidence

[`evidence/credential-access.png`](../evidence/credential-access.png)

## Defensive Recommendations

- Enforce a strict administrative tiering model (e.g., Microsoft's tiered administration model) so that privileged domain accounts are never used to log on to standard user workstations.
- Enable and enforce Credential Guard and LSA Protection (`RunAsPPL`) across all endpoints to prevent or significantly hinder LSASS credential extraction, including offline dump-based methods where possible.
- Use dedicated, hardened Privileged Access Workstations (PAWs) for any task requiring administrative credentials.
- Monitor for LSASS process access patterns and memory-dump behavior indicative of credential-dumping tooling.

> **Note:** No plaintext passwords, real credential values, or reusable secrets are published in this repository. All credential-related evidence is redacted prior to publication.
