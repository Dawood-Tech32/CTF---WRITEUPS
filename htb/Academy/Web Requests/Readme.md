# Web Requests — HTB Academy

**Platform:** HackTheBox Academy
**Category:** General / Web Fundamentals
**Status:** ✅ Completed

## Overview

Covers how HTTP actually works under the hood — request/response structure,
methods, headers, status codes — and how to interact with web servers
directly from the command line and via Python, rather than only through a
browser. This module underpins every later web application security module.

## Topics Covered

### HTTP Request/Response Structure
- Request line (method, path, HTTP version), headers, and optional body
- Response line (status code, reason phrase), headers, and body
- Common headers: `Host`, `User-Agent`, `Content-Type`, `Cookie`,
  `Authorization`

### HTTP Methods
- `GET` — retrieve data
- `POST` — submit data
- `PUT`/`DELETE` — update/remove resources
- `HEAD`/`OPTIONS` — metadata-only requests, useful for enumeration

### Status Codes
| Range | Meaning |
|---|---|
| 2xx | Success |
| 3xx | Redirection |
| 4xx | Client error (e.g. 401 Unauthorized, 403 Forbidden, 404 Not Found) |
| 5xx | Server error |

### Interacting via curl
```
curl -s http://<target>/
curl -I http://<target>/                 # headers only
curl -X POST -d "param=value" http://<target>/endpoint
curl -H "Header-Name: value" http://<target>/
curl -b "cookie=value" http://<target>/  # send cookies
curl -c cookies.txt http://<target>/     # save cookies
```

### Interacting via Python (requests library)
```python
import requests

r = requests.get("http://<target>/")
r = requests.post("http://<target>/endpoint", data={"param": "value"})
print(r.status_code)
print(r.headers)
print(r.text)
```

### Basic Web Enumeration
```
curl -s http://<target>/robots.txt
curl -s http://<target>/sitemap.xml
```

## Key Takeaways

- Being comfortable crafting raw requests via `curl`/Python matters — browsers
  hide a lot of what's actually happening on the wire, and many later web
  security techniques require manually crafted requests.
- Status codes are a first-class signal during enumeration (e.g.
  distinguishing 403 from 404 during ID/path testing) — this habit shows up
  repeatedly in real assessments.
- `robots.txt` and similar well-known paths are a cheap, non-intrusive first
  check on any web target.

## Reference

Official module: HTB Academy — *Web Requests*
Badge : https://academy.hackthebox.com/achievement/2277949/35
