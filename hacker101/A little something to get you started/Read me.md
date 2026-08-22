# Hacker101 CTF — "A Little Something to Get You Started"

**Platform:** Hacker101 CTF (HackerOne)
**Difficulty:** Trivial (Level 0)
**Category:** Web / Client-Side Reconnaissance
**Status:** ✅ Completed

## Overview

This is an entry-level room designed as a first introduction to the Hacker101
CTF platform. The challenge contains a single flag and is intentionally simple,
serving as a warm-up for the source-inspection habits used throughout the rest
of the platform's rooms.

## Objective

Locate the flag hidden somewhere within the challenge page.

## Approach

- On loading the room, the page displayed a minimal welcome message with no
  obvious interactive elements — a strong signal that the flag isn't meant to
  be found through normal browsing, but through inspection of the underlying
  page source.
- Opened browser developer tools (F12) to view the raw HTML/CSS rather than
  relying on the rendered page — a habit worth applying to every web-based
  challenge, regardless of how simple the front-end looks.
- While reviewing the `<style>` block, noticed a reference to an image asset
  that wasn't visibly rendered anywhere on the page itself.
- Opened that referenced asset directly in a new browser tab to inspect it on
  its own, rather than as a background element — this is where the flag was
  ultimately located.

## Key Techniques Demonstrated

- Source-first reconnaissance: checking page source before assuming a
  "blank-looking" page has nothing to offer
- Recognizing CSS/style blocks as a common place for developers to
  unintentionally reference sensitive or hidden assets
- Isolating a linked resource (e.g. viewing an image directly) rather than
  only evaluating it in the context it's used

## Root Cause / Why This Matters

This challenge illustrates a very real, low-severity but common issue:
sensitive or unintended content gets referenced in front-end code (CSS, JS,
comments, meta tags) without realizing it's fully accessible to anyone
inspecting page source. In production applications, this same habit —
checking `view-source`, dev tools, and linked assets — is a legitimate and
common first step in web reconnaissance during real bug bounty or pentest
engagements.

## Mitigation (Real-World Application)

- Never reference sensitive files, credentials, or internal-only assets in
  publicly served CSS, JS, or HTML — even indirectly.
- Treat every asset linked in front-end code as fully public; assume it will
  be discovered.
- Run periodic reviews of front-end bundles/stylesheets for accidental
  exposure of internal paths or files before deployment.

## Lessons Learned

- Even "empty-looking" pages deserve a source inspection pass — this is one of
  the fastest, lowest-cost recon steps in web testing.
- Assets referenced but not visibly rendered are a common place developers
  accidentally leave things exposed.
- Good foundational habit-building for later, more complex Hacker101 rooms.

## Reference

Platform: Hacker101 CTF (HackerOne) — Room: *"A little something to get you started"*
