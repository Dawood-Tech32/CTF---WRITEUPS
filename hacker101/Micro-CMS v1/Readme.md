
# Micro-CMS v1 — Hacker101 CTF

**Platform:** Hacker101 CTF
**Category:** Web Application Security
**Status:** ✅ Completed (all flags found)

## Overview

Micro-CMS v1 is a beginner-friendly Hacker101 challenge built around a simple
content management system with page creation/editing functionality. The
challenge contains four distinct flags covering stored XSS, SQL injection, and
broken access control. Below is my methodology per vulnerability — approach and
vulnerability class only, no exact payload-to-flag chains or flag values.

---

## Vulnerability 1 — Stored XSS (Page Title)

**Objective:** Identify and confirm a stored cross-site scripting vulnerability.

**Approach:**
- Started by exploring the app's core functionality — page creation — and
  noticed submitted content was reflected back in the response.
- Reflected input is a strong early indicator of potential XSS, so I tested the
  page title field with a basic script-injection payload to confirm execution.
- Confirmed the injected script executed when navigating back to a page that
  displayed the stored title, proving the input wasn't being sanitized/escaped
  on output.

**Key techniques demonstrated:**
- Recognizing reflected input as a testing priority
- Basic `<script>` tag injection to confirm stored XSS
- Distinguishing *stored* XSS (persists across requests) from reflected XSS

**Mitigation:**
- Encode/escape user input on output (context-aware HTML encoding).
- Apply a Content Security Policy (CSP) to restrict inline script execution.
- Sanitize input server-side using an allow-list approach rather than a
  block-list.

---

## Vulnerability 2 — SQL Injection (Page ID Parameter)

**Objective:** Identify a SQL injection point in a URL parameter.

**Approach:**
- While using the page edit functionality, noticed the page identifier was
  passed directly as a URL parameter — a classic candidate for parameter
  tampering/injection testing.
- Tested the parameter with a single quote character to see if it broke the
  underlying SQL query, which is the standard first check for SQLi.
- The application's error response confirmed the input was being passed
  unsanitized into a database query.

**Key techniques demonstrated:**
- Identifying ID-style parameters as high-value injection targets
- Single-quote probing as the first, lowest-effort SQLi confirmation test
- Reading application error behavior as a signal of unsanitized query
  construction

**Mitigation:**
- Use parameterized queries / prepared statements exclusively — never
  concatenate user input into SQL strings.
- Apply least-privilege database accounts so even a successful injection has
  limited blast radius.
- Implement generic error handling so raw database errors aren't exposed to
  the client.

---

## Vulnerability 3 — Broken Access Control (IDOR)

**Objective:** Access a restricted/unauthorized page by manipulating an ID
value.

**Approach:**
- Noticed that newly created pages were assigned sequential numeric IDs, while
  the two default pages provided by the app used low, separate ID values —
  suggesting a gap of IDs that might correspond to hidden/restricted pages.
- Systematically tested ID values within that gap. Most returned a generic
  "not found" response, but one specific ID returned a "forbidden" response
  instead — a meaningful difference, since it implies the resource *exists*
  but access is being denied, unlike IDs that simply don't exist.
- Found that the same ID parameter was reused in a different application
  context (the edit workflow) which did not enforce the same access
  restriction as the page-view context.
- Substituted the identified ID into that second context and gained access to
  otherwise-restricted content.

**Key techniques demonstrated:**
- Distinguishing "404 Not Found" vs. "403 Forbidden" as different signals
  during ID enumeration (existence vs. authorization)
- Recognizing that the same identifier can be handled inconsistently across
  different application features/endpoints
- Classic IDOR (Insecure Direct Object Reference) methodology — testing
  whether authorization is enforced consistently everywhere an object
  reference appears, not just in the "obvious" place

**Mitigation:**
- Enforce authorization checks consistently across *every* endpoint that
  accepts an object reference, not just the primary access path.
- Avoid predictable, sequential identifiers for sensitive resources where
  feasible; use random/UUID-style identifiers.
- Apply object-level access control checks server-side on every request, not
  just at the UI layer.

---

## Vulnerability 4 — Stored XSS (Filtered Input Bypass)

**Objective:** Bypass a content filter to achieve stored XSS in a field that
blocks direct script tags.

**Approach:**
- Identified that the page body field supported markdown formatting but
  appeared to block `<script>`-style input specifically — meaning the first
  vulnerability's payload wouldn't work here directly.
- Researched alternative XSS vectors that don't rely on `<script>` tags, since
  many filters block that tag specifically while ignoring other
  event-handler-based injection vectors.
- Found and tested an HTML element with an inline event-handler attribute as
  an alternative payload, which executed successfully despite the filter.
- The visual result didn't show anything obviously flag-related, so the next
  step was checking the underlying page source directly rather than relying
  on the rendered output — where the relevant content was actually located.

**Key techniques demonstrated:**
- Filter bypass thinking: identifying *what* a filter blocks (a specific tag)
  vs. what it actually should block (any script-executing vector) — this
  distinguishes surface-level testing from thorough, iterative filter probing.
- Using non-`<script>` XSS vectors, such as event-handler attributes on other
  HTML elements, is one of the most common filter-bypass techniques.
- Checking page/view-source directly instead of assuming rendered output is
  the only place a result would appear

**Mitigation:**
- Never rely on block-listing specific tags (like `<script>`) as a defense —
  use a proper HTML sanitization library with a strict allow-list of tags and
  attributes.
- Strip or neutralize event-handler attributes (`onclick`, `onerror`, etc.)
  regardless of which tag they're attached to.
- Apply CSP as a defense-in-depth layer even if input filtering is in place.

---

## Overall Lessons Learned

- Reflected input anywhere in an app is worth testing early — it's one of the
  fastest ways to find stored/reflected XSS.
- SQL injection testing doesn't require complex payloads to *confirm* — a
  single quote and a broken query response is often enough of a first signal.
- Access control bugs (IDOR) are rarely about a single missing check — they
  usually come from the *same* identifier being handled inconsistently across
  different features of an app.
- Input filters that block specific tags/strings are frequently incomplete —
  always think about the vulnerability class (e.g. "anything that executes
  JS") rather than the specific string the filter is looking for.
- When a payload executes but doesn't produce an obvious visible result,
  check the page source before assuming it failed.

## Reference

Challenge: Hacker101 CTF — *Micro-CMS v1* ([ctf.hacker101.com](https://ctf.hacker101.com/ctf))
