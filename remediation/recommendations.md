# Remediation Recommendations

## Executive Summary

The assessment identified a chain of individually common — and individually fixable — weaknesses spanning endpoint security, credential hygiene, and Active Directory configuration. No single exotic technique or unpatched software vulnerability was required to achieve full domain compromise. The primary defensive themes arising from this assessment are: enforcing endpoint execution and privilege controls, eliminating privileged credential exposure on standard workstations, hardening service and user account configurations against Kerberos-based attacks, and restricting directory replication rights to prevent domain-wide credential extraction.

## Finding-Based Recommendations

### Finding: Untrusted Executable Run by Standard User (Simulated Phishing)

**Description**
A standard domain user executed an untrusted, attacker-supplied executable, establishing a remote command-and-control session, with endpoint antivirus protection disabled at the time of execution.

**Security Impact**
Provides the attacker with an initial foothold and code execution within the security context of a legitimate domain user.

**Recommendation**
Enforce always-on endpoint protection (real-time antivirus/EDR) with Tamper Protection enabled and centrally managed, deploy attack surface reduction/application allowlisting, and conduct regular phishing awareness training.

**Priority**
High

**Validation**
Confirm endpoint protection cannot be disabled by standard users and that test payloads are detected/blocked in a controlled validation exercise.

---

### Finding: AlwaysInstallElevated Enabled

**Description**
The `AlwaysInstallElevated` registry policy was enabled in both `HKLM` and `HKCU` on the workstation, allowing any user to install `.msi` packages with SYSTEM-level privileges.

**Security Impact**
Allows trivial local privilege escalation from a standard user to full SYSTEM control of the endpoint.

**Recommendation**
Disable `AlwaysInstallElevated` via Group Policy across all endpoints; audit periodically to confirm the setting has not been re-enabled.

**Priority**
Critical

**Validation**
Confirm via `reg query` (or equivalent GPO reporting) that the value is disabled on all managed endpoints.

---

### Finding: Privileged Account Credentials Cached on a Standard Workstation

**Description**
A Domain Administrator account was used to log on interactively to a standard employee workstation, leaving its credential material recoverable from LSASS process memory.

**Security Impact**
Converts the compromise of a single low-value endpoint into exposure of the organization's highest-privilege identity.

**Recommendation**
Implement a tiered administration model in which privileged accounts are never used on standard workstations; enforce use of dedicated Privileged Access Workstations (PAWs) for administrative tasks.

**Priority**
Critical

**Validation**
Review authentication logs for privileged account logons to non-administrative endpoint types; confirm none occur outside of designated PAWs.

---

### Finding: LSASS Credential Material Recoverable via Memory Dump

**Description**
Although live in-memory credential extraction was initially blocked by LSA Protection/Credential Guard, credential material remained recoverable via a full memory dump of the LSASS process, parsed offline.

**Security Impact**
Demonstrates that partial mitigations (protections against live tooling) do not fully eliminate the underlying risk if offline extraction paths remain viable.

**Recommendation**
Enforce Credential Guard and LSA Protection consistently and verify their effective status (not just configured status) across the endpoint fleet; monitor for LSASS memory access and dumping behavior via EDR.

**Priority**
Critical

**Validation**
Confirm Credential Guard/LSA Protection report as actively running (not merely configured) via endpoint auditing tooling, and confirm EDR alerts on LSASS memory-dump attempts in a controlled test.

---

### Finding: NTLM Pass-the-Hash Accepted to the Domain Controller

**Description**
A stolen NTLM hash for a privileged account was sufficient to authenticate directly to the Domain Controller and obtain SYSTEM-level command execution, without ever requiring the account's plaintext password.

**Security Impact**
Bypasses password strength/complexity entirely as a protective control; enables rapid lateral movement to the organization's most critical system.

**Recommendation**
Deploy a Local Administrator Password Solution (LAPS) to prevent credential reuse across machines; restrict/monitor NTLM authentication in favor of Kerberos; segment network access to the Domain Controller.

**Priority**
Critical

**Validation**
Confirm unique local administrator credentials per host via LAPS reporting; monitor and alert on NTLM authentication events involving privileged accounts from unexpected source hosts.

---

### Finding: Kerberoastable Service Account with Weak Password

**Description**
The `SQLService` account was registered with a Service Principal Name (SPN) and configured with a weak, crackable password, allowing any authenticated domain user to request and crack a Kerberos service ticket tied to the account.

**Security Impact**
Enables credential theft for a service account using only standard, low-privilege domain access — no special permissions required.

**Recommendation**
Migrate service accounts to Group Managed Service Accounts (gMSA) where feasible, or enforce long, randomly generated passwords; monitor for abnormal Kerberos TGS request volume.

**Priority**
High

**Validation**
Attempt to crack extracted service ticket hashes against a standard wordlist post-remediation and confirm failure within a reasonable time budget.

---

### Finding: Domain Account Without Kerberos Pre-Authentication

**Description**
The `j.smith` account was configured with "Do not require Kerberos pre-authentication" enabled, allowing an AS-REP hash to be requested from the Domain Controller with no credentials at all.

**Security Impact**
Enables credential theft attempts against the account by any network-positioned attacker, without any prior access or credentials.

**Recommendation**
Audit all domain accounts for this setting and disable it unless a specific, documented business or application requirement exists.

**Priority**
High

**Validation**
Run a domain-wide audit (e.g., via PowerShell Active Directory cmdlets) confirming no accounts retain this setting without documented justification.

---

### Finding: Excessive Directory Replication Rights (DCSync Exposure)

**Description**
The compromised privileged account context held sufficient directory replication rights to extract password material for all domain accounts, including the `krbtgt` account, via a DCSync-style request.

**Security Impact**
Enables complete, domain-wide credential extraction from a single compromised privileged account, and enabled forgery of persistent Golden Ticket access.

**Recommendation**
Restrict directory replication permissions to Domain Controllers and a minimal, tightly controlled set of Tier-0 administrative accounts; monitor for replication requests originating from non-Domain-Controller hosts.

**Priority**
Critical

**Validation**
Audit `Replicating Directory Changes` and `Replicating Directory Changes All` permissions across the domain and confirm they are held only by authorized principals.

## Active Directory Hardening

- **Privileged account management:** Enforce a tiered administration model; eliminate interactive logons by privileged accounts on standard workstations.
- **Service account security:** Migrate to gMSA or enforce strong, randomly generated passwords for all SPN-registered accounts.
- **Authentication controls:** Enforce Kerberos pre-authentication domain-wide; restrict NTLM usage where feasible.
- **Delegation controls:** [INSERT — document only if delegation configuration was actually assessed/tested]
- **Monitoring:** Deploy AD-aware monitoring capable of detecting Kerberoasting, AS-REP Roasting, and DCSync activity.

## Credential Security

Based on the credential-access findings, priority should be placed on preventing privileged credential exposure at the source (tiered administration, PAWs) and on limiting the usability of any credential material that is nonetheless exposed (Credential Guard, LSA Protection, LAPS).

## Lateral Movement Prevention

The demonstrated Pass-the-Hash path could be substantially disrupted through unique local administrator credentials (LAPS), NTLM restriction, and network segmentation limiting which hosts may initiate administrative connections to the Domain Controller.

## Detection and Monitoring

- Alert on privileged account authentication to non-administrative endpoint types.
- Alert on LSASS memory access/dump behavior from non-security tooling.
- Alert on abnormal Kerberos TGS request volume per user (Kerberoasting indicator) and AS-REP requests for accounts without pre-authentication.
- Alert on directory replication requests originating from non-Domain-Controller hosts (DCSync indicator).
- Monitor `krbtgt` account activity and enforce periodic, planned password rotation as a standing control, not solely an incident-response action.

## Remediation Priority Matrix

| Priority | Finding | Risk | Recommended Action |
|---|---|---|---|
| Critical | AlwaysInstallElevated enabled | Trivial local privilege escalation | Disable via GPO domain-wide |
| Critical | Privileged account cached on standard workstation | Domain Admin credential exposure | Enforce tiered administration / PAWs |
| Critical | LSASS credential material recoverable | Credential theft despite partial mitigations | Enforce and verify Credential Guard / LSA Protection |
| Critical | NTLM Pass-the-Hash accepted | Direct lateral movement to Domain Controller | Deploy LAPS; restrict NTLM; segment DC access |
| Critical | Excessive directory replication rights | Full domain credential extraction (DCSync) | Restrict replication permissions to Tier-0 only |
| High | Untrusted executable executed by standard user | Initial foothold | Enforce EDR, ASR rules, awareness training |
| High | Kerberoastable service account | Service account credential theft | Migrate to gMSA / enforce strong passwords |
| High | Account without Kerberos pre-authentication | Credential-less credential theft | Audit and disable domain-wide |

## Final Defensive Recommendations

In priority order:

1. Restrict directory replication rights to prevent DCSync-based domain-wide credential extraction.
2. Eliminate privileged account logons on standard workstations through a tiered administration model.
3. Enforce and verify Credential Guard / LSA Protection across all endpoints.
4. Deploy LAPS and restrict NTLM authentication to reduce Pass-the-Hash viability.
5. Disable `AlwaysInstallElevated` domain-wide.
6. Harden service accounts (gMSA / strong passwords) and audit Kerberos pre-authentication settings domain-wide.
7. Strengthen endpoint execution controls and user awareness to reduce initial access likelihood.
