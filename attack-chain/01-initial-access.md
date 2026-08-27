# Initial Access

## Objective

Establish an initial foothold on a domain-joined endpoint by simulating the most common real-world initial access vector: execution of a malicious file by a standard, non-administrative user.

## Target

Windows 11 workstation (WIN11-PC), domain-joined, operated under a standard (non-administrative) domain user account.

## Initial Conditions

- The target user held a standard domain user account with no local administrative rights.
- No prior access to the workstation existed.

## Vulnerability / Weakness

Absence of effective endpoint controls (antivirus real-time protection was disabled for the purposes of this exercise) permitting execution of an untrusted, attacker-supplied executable by a standard user without interception. This simulates a scenario in which an employee executes a malicious email attachment or downloaded file.

## Technique Used

Simulated phishing: delivery and execution of an attacker-generated payload by a logged-in domain user, representing a user opening a malicious attachment (MITRE ATT&CK T1566 / T1204 — User Execution).

## Attack Execution

A payload was generated on the Kali Linux attacker platform and hosted for retrieval. A listener was configured on Kali to receive the resulting connection. The payload was then transferred to and executed on the Windows 11 workstation while logged in as the target standard domain user.

## Result

A remote command-and-control session was established on the Kali attacker platform, running in the security context of the standard domain user who executed the payload. No administrative privileges were held at this stage.

## Evidence

[`evidence/initial-access.png`](../evidence/initial-access.png)
[`evidence/initial-access2.png`](../evidence/intial-access2.png)

## Security Impact

Successful initial access via user execution demonstrates that endpoint compromise does not require any AD misconfiguration — it requires only a single user interaction with an untrusted file. Once a foothold is established, the attacker's next actions are constrained only by the privilege level of the compromised account and the security posture of the endpoint itself, both of which are addressed in subsequent attack-chain stages.

## Remediation

- Enforce endpoint protection (real-time antivirus/EDR) and prevent users from disabling it (e.g., via Tamper Protection and centrally managed policy).
- Deploy application allowlisting or attack surface reduction rules to restrict execution of unsigned/untrusted executables.
- Provide regular phishing/security awareness training paired with a straightforward reporting mechanism for suspicious files.
- Deploy email/web gateway filtering to reduce the likelihood of malicious attachments reaching end users in the first place.
