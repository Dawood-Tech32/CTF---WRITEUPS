# Micro-CMS v2 — Hacker101 CTF

**Platform:** Hacker101 CTF
**Category:** Web Application Security
**Status:** 🟡 In Progress (2 of 3 flags found)

## Overview

Micro-CMS v2 is the follow-up to the v1 challenge, with authentication and
access control added on top of the original CMS functionality. It contains
three flags covering SQL injection (authentication bypass), HTTP method /
access control abuse, and credential brute-forcing. Below is my methodology
per vulnerability — approach and vulnerability class only, no exact
payload-to-flag chains or credential/flag values.

---

## Vulnerability 1 — SQL Injection (Authentication Bypass via UNION) — ✅ Solved

**Objective:** Gain admin access despite only having regular-user visibility,
using a hint pointing toward SQL injection.

**Approach:**
- The challenge hints strongly suggested SQL injection, specifically a UNION-based
  attack against the login mechanism.
- Started with basic injection probes (quote characters, common bypass strings)
  against the login form — these triggered errors but didn't leak anything
  directly useful.
- Attempted to reason about the underlying query structure to construct a UNION
  SELECT payload that would return a synthetic row treated as valid credentials
  by the application logic, rather than trying to brute-force real credentials.
- The main obstacle was not knowing the exact table name storing admin
  accounts — this required some additional research beyond blind guessing to
  identify a plausible table name before the UNION payload would work.
- Once the payload correctly matched the query structure and guessed table
  name, the response indicated an authenticated session, granting access to a
  previously restricted page.

**Key techniques demonstrated:**
- Recognizing "union" as a strong hint pointing directly at SQL UNION-based
  injection rather than boolean/blind SQLi
- Constructing a UNION SELECT payload that returns attacker-controlled values
  mapped to expected column names (e.g. a password field)
- Understanding that a UNION attack requires matching the number and type of
  columns in the original query

**Mitigation:**
- Use parameterized queries / prepared statements — UNION-based injection is
  entirely prevented once user input can't alter query structure.
- Apply least-privilege database accounts so even a successful injection
  can't reach sensitive tables (like an admin credentials table) it has no
  business touching.
- Avoid detailed SQL error messages being returned to the client, which can
  otherwise aid an attacker in fingerprinting the query structure.

---

## Vulnerability 2 — Broken Access Control (HTTP Method Bypass) — ✅ Solved

**Objective:** Perform an admin-restricted action (page editing) despite not
being authenticated as admin, based on a hint about "actions a regular user
could previously perform."

**Approach:**
- The hint pointed at comparing what actions were possible in v1 (no auth)
  vs. v2 (auth added) — suggesting the access control added in v2 might not
  be applied consistently across all request types.
- Initially tried manipulating requests through an intercepting proxy using
  the standard request method the UI normally sends — this didn't bypass
  anything.
- Realized the UI likely only exercises one HTTP method against the
  edit-page endpoint, and that the server-side access control might only be
  checked for that specific method — a common gap when authorization logic is
  bolted onto specific routes/handlers rather than applied uniformly.
- Tested the same endpoint using a different HTTP method than the UI normally
  uses, sent directly via command line rather than through the browser/proxy
  flow, and found the restricted action succeeded without authentication.

**Key techniques demonstrated:**
- Treating "what changed between versions" as a direct hint toward what
  might have been incompletely fixed
- Testing the same endpoint with alternate HTTP methods, not just alternate
  parameters — access control is sometimes enforced per-method rather than
  per-route
- Using command-line HTTP clients to send requests outside of what the
  browser/UI normally generates

**Mitigation:**
- Enforce authorization checks at the routing/middleware layer for *all*
  HTTP methods on a given endpoint, not just the one the UI happens to use.
- Apply a default-deny policy for unrecognized or unexpected HTTP methods on
  sensitive routes.
- Test access control with tools that aren't limited to the application's own
  UI, since real attackers won't be either.

---

## Vulnerability 3 — Credential Brute-Force — 🟡 In Progress

**Objective:** Find the username and password remaining after v1/v2's
authentication changes, based on a hint linking "credentials" and "flags"
being similarly secret.

**Approach so far:**
- Interpreting the hint as pointing toward brute-forcing rather than another
  injection/logic bug, since it explicitly references credentials being
  "secret" — implying they exist and need to be guessed rather than bypassed.
- Currently working through a two-stage brute-force: first identifying a
  valid username from a name-based wordlist, then a valid password using the
  same style of wordlist, using an intercepting proxy's automation/attack
  feature to iterate through candidates.
- Watching for response differences (e.g. response length or specific error
  message changes) between "invalid username" and "invalid password" states,
  since that's the key signal that narrows down which value is correct before
  moving to the next stage.
- Considering that a large generic password list may be too slow/broad for
  this challenge, and that a more targeted, smaller wordlist (matching the
  apparent naming pattern of credentials in this challenge) is likely more
  effective than a general-purpose one.

**Key techniques being applied:**
- Two-stage brute-force methodology: isolate the correct username first using
  response differentiation, then attack the password separately, rather than
  brute-forcing both simultaneously
- Reading subtle response differences (length, timing, specific error text)
  as the signal that separates a correct guess from an incorrect one
- Choosing a wordlist scoped to the challenge's apparent theme rather than
  defaulting to large generic lists

**Mitigation (general, for this vulnerability class):**
- Implement account lockout or exponential backoff after repeated failed
  login attempts.
- Use rate limiting / CAPTCHA on authentication endpoints.
- Avoid returning distinguishable responses for "invalid username" vs.
  "invalid password" — use a single generic "invalid credentials" message for
  both cases.
- Enforce strong password policies so common/guessable passwords aren't valid
  in the first place.

*Will update this section with the confirmed method once solved.*

---

## Overall Lessons Learned (so far)

- Challenge hints are often a direct pointer to vulnerability *class*, not
  just flavor text — "union" and "secret credentials" both mapped directly to
  the technique needed.
- Authorization added between application versions is a common place for
  gaps — specifically checking whether new access control logic covers *all*
  routes/methods, not just the ones the UI exercises, is a high-value test.
- Brute-forcing is more efficient as a staged process (username first, then
  password) than a combined attack, when the application's responses allow
  you to distinguish correctness at each stage independently.

## Reference

Challenge: Hacker101 CTF — *Micro-CMS v2* ([ctf.hacker101.com](https://ctf.hacker101.com/ctf))
