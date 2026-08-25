# Getting Started — HTB Academy

**Platform:** HackTheBox Academy
**Category:** Offensive / Penetration Testing Process
**Status:** ✅ Completed

## Overview

An introductory offensive-security module walking through the full
penetration testing methodology end-to-end on a guided target — from initial
enumeration through exploitation to basic post-exploitation and reporting
mindset. It's the first module that ties together enumeration, exploitation,
and privilege escalation into one connected workflow rather than teaching
each in isolation.

## Topics Covered

### The Penetration Testing Process
- Pre-engagement / scoping (brief overview, not hands-on here)
- Information gathering / enumeration
- Vulnerability identification
- Exploitation
- Post-exploitation
- Reporting

### Enumeration Phase
```
nmap -sV -sC -p- <target>
```
- Identifying open ports, running services, and versions as the foundation
  for everything that follows
- Cross-referencing identified service versions against known vulnerability
  databases

### Exploitation Phase
- Matching an identified vulnerable service to a public exploit or Metasploit
  module
- Verifying exploit/target compatibility before firing (OS, architecture,
  exact version range) — a theme repeated from the Metasploit module
- Confirming shell access and stabilizing it (e.g. upgrading a raw shell to a
  fully interactive TTY)

### Shell Stabilization (generic pattern)
```
python3 -c 'import pty; pty.spawn("/bin/bash")'
# then, in the resulting shell:
export TERM=xterm
```

### Post-Exploitation Basics
- Basic local enumeration once inside a target (`whoami`, `id`, `sudo -l`,
  checking for interesting files/configs)
- Identifying a path to privilege escalation using fundamentals from the
  Linux Fundamentals module (permissions, SUID binaries, sudo
  misconfigurations)

### Reporting Mindset
- Documenting each step as you go (commands run, findings, screenshots)
  rather than reconstructing afterward — directly relevant to how I'm
  building these writeups
- Distinguishing between a finding's technical severity and its actual
  business impact when it comes time to write it up

## Key Takeaways

- This module is really about connecting the dots between everything learned
  individually in earlier modules (Nmap, Metasploit, Linux fundamentals) into
  one realistic, ordered workflow.
- Shell stabilization is a small but critical step that's easy to skip as a
  beginner — an unstable shell makes every subsequent step harder than it
  needs to be.
- Documentation habits built here (logging commands/findings as you go)
  directly carry over into professional pentest reporting, not just personal
  writeups.

## Reference

Official module: HTB Academy — *Getting Started*
Badge : https://academy.hackthebox.com/achievement/2277949/77
