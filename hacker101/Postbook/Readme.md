# Postbook — Hacker101 CTF

**Platform:** Hacker101 CTF
**Category:** Web Application Security
**Status:** 🟡 In Progress (5 of 7 flags found)

## Overview

Postbook is a beginner-friendly Hacker101 challenge built around a simple
social posting app with user accounts, posts, and edit/delete functionality.
It contains seven flags, largely centered on broken access control (IDOR)
patterns repeated across different parts of the app, plus one credential
brute-force flag. Below is my methodology per flag — approach and
vulnerability class only, no exact payload-to-flag chains or flag/credential
values.

---

## Flag 1 — IDOR via `user_id` Parameter (Create Post) — ✅ Solved

**Objective:** Manipulate a parameter tied to user identity during post
creation to access another user's content/flag.

**Approach:**
- Created an account and used an intercepting proxy to inspect the raw POST
  request sent when creating a new post.
- Noticed a parameter in the request body representing user identity, with a
  predictable (non-hashed, non-random) value matching the account's own ID.
- Sent the request to a repeater tool, modified that identity parameter to
  reference a lower/different user ID (since new accounts get the highest
  IDs), and resent it.
- Followed the resulting redirect and found the response revealed content
  belonging to another user.

**Key techniques demonstrated:**
- Inspecting POST request bodies (not just URLs) for identity-related
  parameters — IDOR isn't limited to GET/URL parameters.
- Recognizing predictable, sequential-looking values as IDOR candidates
- Using a repeater-style tool to manually modify and resend a captured
  request

**Mitigation:**
- Never trust identity parameters supplied by the client — derive the
  authenticated user's identity server-side from the session, not the
  request body.
- Enforce object-level authorization checks on every write operation.

---

## Flag 2 & 3 — IDOR via Post ID Enumeration — ✅ Solved

**Objective:** Enumerate valid post IDs to access two separate flags hidden
in other users' posts.

**Approach:**
- Noticed a numeric `id` parameter in the URL when viewing an existing post —
  another predictable identifier candidate.
- Used an intercepting proxy's automation feature to sweep a numeric range of
  ID values and compared response sizes, since a distinctly larger response
  reliably indicated a valid post ID versus a "not found" response for
  invalid ones.
- Manually visited each valid ID surfaced this way and found flag content on
  two separate posts.

**Key techniques demonstrated:**
- Numeric ID range enumeration via automated request sweeping
- Using response length/size as a differentiator between valid and invalid
  resource IDs, rather than relying solely on status codes

**Mitigation:**
- Enforce per-resource authorization checks so even a valid ID doesn't return
  data the requester isn't permitted to see.
- Avoid predictable sequential IDs for resources where enumeration is a
  concern; consider UUIDs.
- Rate-limit or detect high-volume sequential requests as potential
  enumeration attempts.

---

## Flag 4 — IDOR via Edit Function — ✅ Solved

**Objective:** Edit another user's post by manipulating the same style of
`id` parameter used elsewhere in the app.

**Approach:**
- Noticed the edit-post feature also uses an `id` parameter in its URL,
  consistent with the pattern seen in flags 2/3.
- Substituted one of the previously identified valid post IDs into the edit
  endpoint and found the app allowed editing content that didn't belong to
  the current account.
- Made and saved an edit to confirm write access, then located the flag in
  the resulting response.

**Key techniques demonstrated:**
- Reusing IDs discovered during read-path enumeration (flags 2/3) against a
  write-path endpoint (edit), since access control is often inconsistent
  between reading and modifying the same resource type

**Mitigation:**
- Apply the same object-level authorization check to write operations
  (edit/delete) as to read operations — access control gaps often appear
  specifically where developers assumed "if they can't see it, they can't
  edit it," without enforcing that assumption server-side.

---

## Flag 5 — MD5-Hashed ID Parameter (Delete Function) — ✅ Solved

**Objective:** Access another resource via the delete function, where the
`id` parameter is obfuscated rather than passed as a plain number.

**Approach:**
- Noticed the delete function's request includes an `id` value in a
  different, longer format than the plain numeric IDs seen elsewhere in the
  app — recognized this format as consistent with a common fast, non-salted
  hashing algorithm based on its length/character set.
- Confirmed the hash algorithm via a quick lookup, then generated the
  equivalent hash for a target numeric ID already known to be valid.
- Substituted the generated hash into the request's `id` value in place of
  the original, and the app accepted it the same way it accepted the plain
  numeric ID elsewhere, revealing the flag.

**Key techniques demonstrated:**
- Recognizing hash formats by length/character pattern rather than assuming
  all identifiers are opaque or unpredictable
- Understanding that a "hashed" identifier isn't inherently secure if the
  hash algorithm is fast/unsalted and the underlying value space (e.g. small
  integers) is guessable — you can just hash your own guesses and compare

**Mitigation:**
- Don't rely on hashing alone to obscure predictable, low-entropy values
  (like small sequential integers) — this is "security through obscurity"
  and is trivially reversible via precomputation.
- Use proper authorization checks instead of relying on an identifier being
  hard to guess.
- If obfuscated references are required, use a keyed/signed token (e.g. HMAC)
  tied to the object and requesting user, not a bare hash of the ID.

---

## Flag 6 — MD5-Hashed Cookie `id` Value — 🟡 In Progress

**Objective:** Apply the same hashed-ID weakness identified in flag 5, but to
a cookie value instead of a request parameter.

**Approach so far:**
- Noticed that session/request cookies also carry an `id` value in the same
  hashed format identified while solving flag 5, and it appears to match the
  same underlying numeric value already confirmed there.
- Plan: generate the equivalent hash for a different target numeric ID and
  substitute it directly into the cookie value before sending the request,
  the same way it worked for the delete function's parameter.

**Key techniques being applied:**
- Recognizing that a vulnerability pattern identified in one part of an app
  (a hashed, guessable identifier) often reappears elsewhere — here, likely
  the same weakness applied to cookies instead of parameters
- Directly editing cookie values via an intercepting proxy rather than
  relying on the application's normal cookie-setting flow

**Mitigation (general, for this vulnerability class):**
- Same as flag 5 — don't use unsalted hashes of low-entropy values as
  identifiers, whether in parameters or cookies.
- Session/identity cookies specifically should be tied to server-side session
  state (e.g. a random session token mapped server-side to a user), never a
  reversible or guessable encoding of the user's ID.

*Will update this section with the confirmed method once solved.*

---

## Flag 7 — Credential Brute-Force — 🟡 In Progress

**Objective:** Determine the password for a known, generically-named user
account.

**Approach so far:**
- Noticed a user account with a generic, guessable username already visible
  in the app.
- Plan: log out, attempt to log in as that account with an arbitrary
  password, capture the login POST request in an intercepting proxy, then
  send it to an automated attack tool with the password field marked as the
  injection point.
- Intend to try a small, curated common-password wordlist first (rather than
  a large generic list), since the target looks like a
  weak/default-credential scenario, and compare response status codes across
  attempts to spot the successful one.

**Key techniques being applied:**
- Using a small, curated common-password wordlist before jumping to large
  generic lists — often faster for intentionally weak-credential scenarios
- Differentiating successful vs. failed login attempts by comparing HTTP
  status codes across automated attempts

**Mitigation (general, for this vulnerability class):**
- Enforce strong, case-sensitive password policies and reject common/weak
  passwords at account creation.
- Implement account lockout, rate limiting, or CAPTCHA after repeated failed
  login attempts to make automated brute-forcing impractical.
- Avoid returning distinguishable response codes/messages that make it easy
  to automate success/failure detection.

*Will update this section with the confirmed method once solved.*

---

## Overall Lessons Learned (so far)

- This challenge reinforced that a single vulnerability pattern — predictable,
  client-trusted identifiers — repeats across almost every feature of an app
  (create, view, edit, delete) once you find it once.
- A hashed identifier is not automatically a secure identifier — if the
  underlying value space is small and guessable and the hash is unsalted,
  it's just as attackable as a plain integer, just with an extra generation
  step. This technique should transfer directly to the cookie-based flag.
- Automated response-comparison (length for enumeration, status code for
  brute-force) is a recurring technique across very different vulnerability
  classes in this challenge — worth treating as a default first move when
  testing predictable parameters.

## Reference

Challenge: Hacker101 CTF — *Postbook* ([ctf.hacker101.com](https://ctf.hacker101.com/ctf))
