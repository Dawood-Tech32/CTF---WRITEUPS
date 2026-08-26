# Index Slip

**Category:** Forensics
**Difficulty:** Easy
**Points:** 50
**Platform:** FlagYard

## Challenge Description

> I remember opening a file containing all of my important stuff, now I don't even remember its name.
> Flag format: `FlagY{md5(filename_without_extension)}`

We're given a `Downloads/[root]` folder containing an extracted Windows filesystem tree (`ProgramData`, `Program Files`, `Program Files (x86)`, `Users`, `Windows`) — not a live system or raw disk image.

## Steps

**1. Explore the extracted filesystem**

```bash
ls "/././Downloads/[root]"
```

Standard Windows C:\ layout. No raw image, so classic disk-imaging tools (mmls/fls against a `.dd`/`.E01`) don't apply here — this is a straight file export.

**2. Identify the real user profile**

```bash
find / -iname '$I30' 2>/dev/null
```

The extraction preserved raw NTFS `$I30` index attributes as standalone files (X-Ways-style forensic export). Filtering the results down, only one profile has real user activity: `Users\Administrator` (Default/Public are just OS scaffolding).

**3. Check the Windows Recent folder**

Instead of diving straight into `$I30` slack parsing, check the simplest artifact for "recently opened files" first — Windows auto-creates a `.lnk` shortcut every time a document is opened:

```bash
ls -la "/././Downloads/[root]/Users/Administrator/AppData/Roaming/Microsoft/Windows/Recent"
```

Output:
```
$I30   AutomaticDestinations   CustomDestinations   desktop.ini   MY_SUP3R_S3CR3T_D4T4.txt.lnk
```

One `.lnk` file stands out immediately.

**4. Read the LNK file**

```bash
cat "MY_SUP3R_S3CR3T_D4T4.txt.lnk"
```

The binary LNK data contains the full original path in plaintext:

```
C:\Users\Administrator\Desktop\MY_SUP3R_S3CR3T_D4T4.txt
```

**5. Compute the MD5 of the filename (without extension)**

```bash
echo -n "MY_SUP3R_S3CR3T_D4T4" | md5sum
```

Output:
```
7ec0*******************
```

## Flag
Submit FLag 
```
FlagY{7ec0*******************}
```
