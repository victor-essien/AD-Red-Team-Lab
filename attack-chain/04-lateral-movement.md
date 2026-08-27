# Lateral Movement

## Starting Point

Kali Linux attacker platform, in possession of NTLM credential hash material for the Domain Administrator account (obtained during the Credential Access stage).

## Destination

Windows Server 2019 Domain Controller (WIN-DC).

## Credentials / Access Used

Domain Administrator NTLM hash, obtained via LSASS memory extraction on the Windows 11 workstation. No plaintext password for this account was known or required at any point in this stage.

## Technique

Pass-the-Hash authentication (MITRE ATT&CK T1550.002 — Use Alternate Authentication Material: Pass the Hash).

## Preconditions

- Valid NTLM hash for a privileged domain account.
- Network connectivity from the Kali attacker platform to the Domain Controller (SMB/RPC accessible).
- No requirement for NTLM authentication to be restricted or disabled on the target for this technique to succeed.

## Result

The recovered NTLM hash was first validated using a lightweight authenticated SMB check, confirming administrative-level access to the Domain Controller. A full interactive remote command session was then established on the Domain Controller using the same hash, confirmed to be running as `NT AUTHORITY\SYSTEM`.

This stage is distinguished clearly from the previous stage: Credential Access **obtained** the credential material; Lateral Movement **used** that material to authenticate to, and gain command execution on, a separate, higher-value system (the Domain Controller).

## Evidence
[`evidence/confirm-auth.png`](../evidence/confirm-auth.png)

[`evidence/lateral-movement.png`](../evidence/lateral-movement.png)


## Security Impact

This stage represents the point at which the compromise expands from a single endpoint to the organization's central identity infrastructure. Because Pass-the-Hash bypasses the need to know or crack a plaintext password, this technique remains effective even against accounts with strong, complex passwords — meaning password strength alone does not mitigate this risk.

## Remediation

- Deploy a Local Administrator Password Solution (LAPS) to ensure local administrator credentials are unique per machine and cannot be reused laterally across the environment.
- Restrict or disable NTLM authentication where feasible, in favor of Kerberos-only authentication with strong protections.
- Enforce Windows Defender Credential Guard to reduce the exposure of reusable credential material in the first place (see Credential Access remediation).
- Apply network segmentation and access control restricting which endpoints can initiate administrative protocol connections (SMB/RPC) to the Domain Controller.
- Monitor for anomalous authentication patterns consistent with Pass-the-Hash activity (e.g., NTLM logons from unexpected source hosts using privileged accounts).
