# Remediation Overview

This assessment's findings span endpoint security, credential exposure, and Active Directory configuration. The overarching defensive priorities are:

1. Restrict directory replication rights to prevent domain-wide credential extraction (DCSync).
2. Eliminate privileged account exposure on standard workstations through tiered administration.
3. Enforce and verify Credential Guard / LSA Protection across all endpoints.
4. Reduce Pass-the-Hash viability through LAPS and NTLM restriction.
5. Harden local endpoint configuration (`AlwaysInstallElevated`) and Active Directory account configuration (service account passwords, Kerberos pre-authentication).

For the complete, finding-by-finding remediation guidance — including security impact, priority, and validation steps for each individual finding — see:

[`remediation/recommendations.md`](../remediation/recommendations.md)
