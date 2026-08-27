# Active Directory Exploitation

## Objective

Demonstrate that the Active Directory environment itself contained exploitable configuration weaknesses, independent of the endpoint compromise and credential theft documented in the preceding stages.

## Domain Environment

LAB Active Directory domain, hosted on the Windows Server 2019 Domain Controller (WIN-DC).

## Attack Technique

Two distinct Kerberos-based attack techniques were implemented against the domain:

1. **Kerberoasting** (MITRE ATT&CK T1558.003) — A domain service account (`SQLService`) was registered with a Service Principal Name (SPN) and configured with a weak, crackable password. Using only a standard, already-authenticated domain user account, a Kerberos service ticket (TGS) tied to `SQLService` was requested from the Domain Controller. This is normal, permitted Kerberos behavior for any authenticated user and did not require any special access. The ticket was taken offline and cracked, recovering the service account's plaintext password.

2. **AS-REP Roasting** (MITRE ATT&CK T1558.004) — A separate domain user account (`j.smith`) was configured with the "Do not require Kerberos pre-authentication" setting enabled. This allowed a Kerberos AS-REP response for that account to be requested directly from the Domain Controller **with no credentials of any kind**. The response was taken offline and cracked, recovering the account's plaintext password.

## Attack Path

Both techniques exploit legitimate, by-design Kerberos protocol behavior combined with a specific account misconfiguration (a weak password on an SPN-registered account; a missing pre-authentication requirement on a user account) rather than a software vulnerability. Both were performed directly against the Domain Controller from the Kali attacker platform.

## Result

- Kerberoasting: plaintext password recovered for the `SQLService` account.
- AS-REP Roasting: plaintext password recovered for the `j.smith` account, obtained without presenting any credentials.

## Evidence

[INSERT SCREENSHOT — cracked Kerberoasting hash]

[INSERT SCREENSHOT — cracked AS-REP hash]

## Security Impact

This stage demonstrates that Active Directory misconfigurations represent an independent compromise path that does not depend on the phishing/endpoint compromise chain documented in Stages 1–4. An organization that remediated only the endpoint-based attack path (Stages 1–4) would remain fully exposed via this stage alone, underscoring the importance of addressing Active Directory configuration weaknesses as a distinct risk category.

## Remediation

- Enforce long, high-entropy, randomly generated passwords for all service accounts, or migrate service accounts to Group Managed Service Accounts (gMSA), which rotate credentials automatically and eliminate human-memorable passwords.
- Audit all domain user accounts for "Do not require Kerberos pre-authentication" and disable it unless explicitly and documentedly required.
- Monitor for abnormal volumes of Kerberos TGS requests for SPN-registered accounts (a Kerberoasting indicator) and AS-REP requests for accounts without pre-authentication.
- Regularly audit Service Principal Name (SPN) assignments and remove unnecessary ones.
