
# Network Enumeration with Nmap — HTB Academy

**Platform:** HackTheBox Academy
**Category:** Network Enumeration / Reconnaissance
**Status:** ✅ Completed

## Overview

This module covers host discovery, port/service scanning, output handling, the
Nmap Scripting Engine (NSE), and firewall/IDS-IPS evasion techniques across three
difficulty tiers. Below are my methodology notes per section — approach and
technique only, no target IPs, exact flags, or final submitted answers.

---

## Section 1 — Host Discovery

**Objective:** Identify the target host and infer its operating system from a
discovery-level scan (no port scan yet).

**Approach:**
- Used a ping-sweep style scan with ICMP echo requests, saving output for later
  reference, and disabled ARP-based discovery since the target sits across a
  routed VPN rather than the local subnet.
- Since no ports were scanned at this stage, OS identification had to come from
  something else in the response — the **TTL (time-to-live)** value in the
  ICMP reply.

**Key technique — TTL-based OS fingerprinting:**
| TTL (approx.) | Likely OS |
|---|---|
| 128 | Windows |
| 64 | Linux/Unix |
| 255 | Network device (router/switch) |

**Generic command pattern:**
```
sudo nmap <target> -sn -oA host -PE --packet-trace --disable-arp-ping
```

**Takeaway:** Don't assume OS detection requires `-O` or a full scan — passive
clues like TTL can narrow it down from a lightweight discovery scan alone.

---

## Section 2 — Host and Port Scanning

**Objective:** Enumerate all TCP ports, then identify the target's hostname.

**Approach — full port sweep:**
- Ran a full TCP port scan (`-p-`) rather than the default top-1000 ports, since
  the answer required a total count of discovered ports.

**Approach — hostname enumeration:**
- Initially unclear how to extract a hostname without deeper enumeration.
- Running Nmap's default script set (`-sC`) surfaced this via the
  `smb-os-discovery` script output — a good reminder that default scripts often
  reveal metadata beyond just "port open/closed."

**Generic command pattern:**
```
sudo nmap -p- <target>
sudo nmap -sC <target>
```

**Takeaway:** Default NSE scripts (`-sC`) frequently expose metadata (hostname,
domain, OS build) that a plain SYN scan won't show — always worth running early.

---

## Section 3 — Saving the Results

**Objective:** Run a full TCP scan, export it, and convert it into a readable
report.

**Approach:**
- Exported scan results to XML (`-oX`) rather than plain text, to preserve
  structure for later parsing/reporting.
- Converted the XML report into HTML using Nmap's bundled XSLT stylesheet via
  `xsltproc`.
- Rendered the HTML report directly in-terminal using a text-based browser
  (`lynx`) rather than transferring the file out, useful in restricted/headless
  environments.

**Generic command pattern:**
```
sudo nmap <target> -oX target.xml
xsltproc style.xsl target.xml > output.html
lynx output.html
```

**Takeaway:** XML output isn't just for automation pipelines — converting it to
HTML gives a clean, shareable report format, which matters for real client
deliverables and pentest reporting.

---

## Section 4 — Service Enumeration

**Objective:** Enumerate service versions across a defined set of ports to
locate a flag embedded in one service's banner/response.

**Approach:**
- Rather than rescanning all 65535 ports, targeted the specific ports already
  identified as open from the earlier full sweep, and ran version detection
  (`-sV`) only against that reduced set — faster and quieter.

**Generic command pattern:**
```
sudo nmap -sV -p<comma_separated_ports> <target>
```

**Takeaway:** Once you've done a full port sweep, there's no need to repeat it —
scope follow-up scans down to just the open ports for speed and stealth.

---

## Section 5 — Nmap Scripting Engine (NSE)

**Objective:** Use NSE scripts to uncover a flag hidden within a service.

**Approach:**
- First attempt used an aggressive all-in-one scan (`-A`) against the web port —
  didn't surface anything directly useful.
- Second attempt used the NSE `vuln` script category specifically against the
  web service, which surfaced a reference to a hidden/disallowed path
  (commonly exposed via `robots.txt` on web servers).
- Manually requested that path with `curl` to retrieve the content directly,
  rather than relying on the script output alone.

**Generic command pattern:**
```
sudo nmap <target> -p 80 -A
sudo nmap <target> -p 80 -sV --script vuln
curl http://<target>/robots.txt
```

**Takeaway:** NSE script categories (`vuln`, `default`, `discovery`, etc.) behave
very differently — if a broad `-A` scan doesn't surface anything, try a targeted
script category before assuming the port is a dead end. Also: always manually
check paths NSE references (like `robots.txt`) rather than trusting script
output alone.

---

## Section 6 — Firewall and IDS/IPS Evasion (Easy Lab)

**Objective:** Identify the target OS despite scan attempts being blocked.

**Approach:**
- A direct OS-detection scan (`-O`) returned nothing — a strong signal that a
  firewall was silently dropping probes rather than the host being offline.
- Fell back to the TTL-based technique from Section 1, which succeeded quietly
  (0 alerts triggered) and confirmed the host was Linux-based.
- To narrow down the specific distribution, ran a version-detection scan against
  a known set of ports while disabling ARP ping and treating the host as
  "up" by default (`-Pn`), since standard host-discovery probes were being
  filtered.

**Generic command pattern:**
```
sudo nmap -O --disable-arp-ping -Pn <target>
sudo nmap -sn -PE --packet-trace --disable-arp-ping <target>
sudo nmap -sV -p<ports> --disable-arp-ping -Pn <target>
```

**Takeaway:** A scan returning nothing is data too — it usually means "blocked,"
not "closed." Falling back to lower-noise techniques (TTL inference, `-Pn`) is
often more effective against filtered targets than repeating the same
aggressive scan.

---

## Section 7 — Firewall and IDS/IPS Evasion (Medium Lab)

**Objective:** Determine a specific service version despite the standard port
appearing closed.

**Approach:**
- Direct TCP scan against the expected service port returned a closed-port
  response (RST+ACK) rather than a filtered/no-response result — meaning the
  port was reachable, just not on TCP.
- Recognized this service is commonly UDP-based, and re-ran the scan using
  Nmap's combined TCP+UDP version-detection flag against that port over UDP
  specifically.

**Generic command pattern:**
```
sudo nmap -sV -p<port> --disable-arp-ping -Pn <target>
sudo nmap -sUV -p<port> --disable-arp-ping -n -Pn <target>
```

**Takeaway:** A "closed" TCP result doesn't rule out the service — always
consider whether the target service normally runs over UDP before assuming a
dead end.

---

## Section 8 — Firewall and IDS/IPS Evasion (Hard Lab)

**Objective:** Identify the version of a newly added, non-standard service.

**Approach:**
- Ran a version-detection scan across all responsive ports (with discovery
  probes disabled) to spot anything new compared to earlier scans — found an
  unusual high-numbered port.
- Attempted a raw connection using `netcat`/`ncat` with a spoofed source port
  (mimicking a trusted port like DNS/53) to see if that would get past
  filtering — this didn't work from a local machine, but succeeded once run
  from the provided cloud-based lab environment (Pwnbox), after resolving a
  permissions issue by escalating privileges.

**Generic command pattern:**
```
sudo nmap -sV -Pn --disable-arp-ping <target>
sudo ncat -nv --source-port <trusted_port> <target> <discovered_port>
```

**Takeaway:** Source-port spoofing can bypass naive firewall rules that trust
traffic from certain "well-known" ports — but execution environment matters;
local network restrictions (e.g. needing raw socket privileges) can silently
break a technique that works fine from a properly configured lab host.

---

## Overall Lessons Learned

- TTL-based OS fingerprinting is a lightweight, low-noise technique worth trying
  before jumping to `-O` or full aggressive scans.
- Default NSE scripts (`-sC`) and category-specific scripts (`--script vuln`)
  each surface different information — running only one type risks missing
  key findings.
- A blocked scan and a closed port look different and mean different things —
  learning to read *why* a scan came back empty is as important as running it.
- TCP and UDP are separate attack surfaces; don't assume a "closed" TCP port
  means the service isn't reachable another way.
- Evasion techniques (source-port spoofing, `-Pn`, disabling ARP discovery) are
  situational tools, not silver bullets — validate results and be ready to
  switch environments (e.g. local machine vs. provided Pwnbox) if privilege or
  network constraints get in the way.

## Reference

Official module: HTB Academy — *Network Enumeration with Nmap*
Badge Link : https://academy.hackthebox.com/achievement/badge/d0872cc9-2e6c-11f1-9254-bea50ffe6cb4
