# JavaScript Deobfuscation — HTB Academy

**Platform:** HackTheBox Academy
**Category:** Defensive / Analysis
**Status:** ✅ Completed

## Overview

A defensive-focused module on reading and de-obfuscating JavaScript code —
relevant both for analyzing client-side attack payloads (e.g. malicious
scripts injected via XSS) and for understanding how web applications hide or
minify logic. Below are my methodology notes per technique area, without
reproducing any specific exercise's final deobfuscated payload.

## Topics Covered

### Why Code Gets Obfuscated
- Legitimate reasons: minification for performance, protecting proprietary
  client-side logic
- Malicious reasons: evading signature-based detection, hiding payload intent
  from a quick manual review

### Common Obfuscation Techniques Encountered
- **Variable/function renaming** to meaningless short names (`a`, `_0x1a2b`)
- **String encoding** — hex, base64, or char-code arrays instead of literal
  strings
- **Control flow flattening** — restructuring logic into harder-to-follow
  switch/loop constructs
- **Dead code injection** — irrelevant code mixed in to increase analysis time
- **Packer-style wrappers** — an outer function that decodes and `eval()`s the
  real payload at runtime

### Manual Analysis Approach
- Start by identifying the outer wrapper/entry point rather than trying to
  read every line linearly.
- Replace `eval()` calls with `console.log()` in a sandboxed environment
  (e.g. browser dev tools or Node) to reveal the decoded output without
  actually executing the payload.
- Incrementally rename variables/functions as their purpose becomes clear,
  rather than trying to understand the whole script at once.

### Tools Used
```
# Browser DevTools — "Pretty print" feature for minified code
# Node.js for sandboxed evaluation of decode routines only (never the full payload)
node -e "console.log(<decode_function_only>)"

# Online/offline beautifiers for readability
js-beautify script.js
```

### Common Encoding/Decoding Patterns
```javascript
// Hex-encoded strings
String.fromCharCode(0x68, 0x65, 0x6c, 0x6c, 0x6f)

// Base64
atob("base64string")

// Char-code arrays
[104,101,108,108,111].map(c => String.fromCharCode(c)).join('')
```

## Key Takeaways

- Never run a fully obfuscated, untrusted script directly — isolate and
  evaluate only the decoding logic in a sandboxed context to reveal intent
  safely.
- Obfuscation is a speed bump, not real security — systematic, layer-by-layer
  analysis (wrapper → decode routine → payload) defeats almost all
  commonly-encountered techniques.
- This skill directly supports SOC/analyst work — being able to quickly
  assess whether an injected or attached script is malicious, and roughly
  what it does, without executing it live.

## Reference

Official module: HTB Academy — *JavaScript Deobfuscation*
Badge : https://academy.hackthebox.com/achievement/2277949/41
