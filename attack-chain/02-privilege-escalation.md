# Local Privilege Escalation

## Starting Privileges

Standard, non-administrative domain user session on the Windows 11 workstation (WIN11-PC), obtained during the Initial Access stage.

## Vulnerability / Misconfiguration

The `AlwaysInstallElevated` policy was enabled in both the `HKEY_LOCAL_MACHINE` and `HKEY_CURRENT_USER` registry hives on the Windows 11 workstation. This configuration instructs the Windows Installer service to install `.msi` packages with SYSTEM-level privileges regardless of the invoking user's actual privilege level.

## Technique

Local privilege escalation via `AlwaysInstallElevated` misconfiguration (MITRE ATT&CK T1548.002 — Abuse Elevation Control Mechanism: Bypass User Account Control).

## Execution Summary

Using the low-privilege session obtained during Initial Access, a malicious Windows Installer (`.msi`) package was generated on the Kali attacker platform and transferred to the workstation. The package was executed via the standard Windows Installer command-line utility (`msiexec`). Because `AlwaysInstallElevated` was enabled, the installer executed with SYSTEM-level privileges rather than the invoking user's standard privileges.

## Result

A new session was established running as `NT AUTHORITY\SYSTEM` on the Windows 11 workstation — the highest privilege level available on the endpoint — escalated directly from the original standard domain user session.

## Evidence

[`evidence/initial-access2.png`](../evidence/intial-access2.png)

[`evidence/privilege-escalation.png`](../evidence/privilege-escalation.png)



## Security Impact

SYSTEM-level access on the endpoint removes the operating system's normal protections around credential storage (LSASS memory), security tooling, and administrative functions. This privilege level was a required precondition for the credential access stage that followed, since standard users are blocked from reading protected credential material by design.

## Remediation

- Disable `AlwaysInstallElevated` in both `HKLM` and `HKCU` via Group Policy across all endpoints.
- Ensure standard users do not hold local administrator rights by default.
- Periodically audit endpoints for common local privilege escalation misconfigurations (unquoted service paths, weak service ACLs, `AlwaysInstallElevated`, etc.) using enumeration tooling as part of routine security assessments.
- Deploy endpoint detection and response (EDR) tooling capable of flagging anomalous SYSTEM-level process creation following installer execution by a standard user.
