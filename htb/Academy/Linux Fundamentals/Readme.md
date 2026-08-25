# Linux Fundamentals — HTB Academy

**Platform:** HackTheBox Academy
**Category:** General / Foundational Skills
**Status:** ✅ Completed

## Overview

A foundational module covering Linux command-line basics needed for every
later offensive module — navigation, file management, permissions, package
management, service/process handling, and basic networking commands. Below are
my notes per topic area — general syntax and concepts, not tied to any
specific exercise answer.

## Topics Covered

### Navigation & File Management
```
pwd
ls -la
cd <path>
find / -name "<filename>" 2>/dev/null
locate <filename>
cp / mv / rm
```

### File Permissions
```
ls -l
chmod <mode> <file>
chown <user>:<group> <file>
```
- Understanding the `rwx` permission triplet (owner/group/other) and numeric
  (octal) notation (e.g. `755`, `644`).
- SUID/SGID/sticky bit as special permission flags worth recognizing during
  later privilege escalation work.

### Users & Groups
```
whoami
id
sudo -l
cat /etc/passwd
cat /etc/group
```

### Process & Service Management
```
ps aux
top / htop
systemctl status <service>
systemctl start/stop/restart <service>
kill / killall
```

### Package Management (Debian/Ubuntu-based)
```
apt update && apt upgrade
apt install <package>
dpkg -l
```

### Networking Basics
```
ip a
ifconfig
netstat -tulnp
ss -tulnp
```

### Text Processing & Piping
```
cat / less / head / tail
grep <pattern> <file>
awk '{print $1}'
sed 's/old/new/g'
| (pipe) to chain commands together
```

### Redirection & File Descriptors
```
command > file      # stdout, overwrite
command >> file     # stdout, append
command 2> file     # stderr
command &> file     # both
```

## Key Takeaways

- Piping and redirection are the backbone of nearly every later enumeration
  technique — comfort with `grep`/`awk`/`sed` pays off constantly.
- Understanding permission bits (including SUID) here directly sets up later
  privilege escalation modules.
- `sudo -l` early in any engagement is a habit worth building now — it
  frequently reveals privilege escalation paths later.

## Reference

Official module: HTB Academy — *Linux Fundamentals*
Badge : https://academy.hackthebox.com/achievement/2277949/18
