# Template C — Hands-on Web / API Technical Test Cheat Sheet

> **Use case:** Open-book, timed **authorised** Web/API hands-on assessments (for example, your own TryHackMe lab). Optimised for **Burp Suite Community + Repeater**, then `curl` / `jq` / `ffuf` / `sqlmap` if permitted.  
> **Everything below is illustrative.** `lab.example.test`, user IDs, refunds and callback addresses are **fictional**. Use only approved targets, accounts and test records. Never publish live tokens, passwords, private client endpoints, personal records or unredacted screenshots.

## START HERE — 60-second triage

**Memorise only this loop:**

**Normal request → one controlled change → compare the response AND actual application state → record evidence → fix/retest.**

| If you see this feature | Go straight to |
| --- | --- |
| `/api/orders/{id}`, profile IDs, invoices, customer records | [BOLA / IDOR](#1-bola--idor-cross-account-record-access) |
| `/admin`, `/approve`, refund/role-only action | [BFLA / role bypass](#2-bfla--role-bypass) |
| JSON body with `role`, `price`, `isAdmin` | [Mass assignment](#3-mass-assignment--untrusted-json-fields) |
| Login, logout, refresh tokens, session cookies | [Session handling](#4-logout--session-token-replay) / [JWT](#13-jwt--token-inspection) |
| OTP, password reset, login attempts | [Rate limiting](#5-otp--login-rate-limits) |
| Search, filters, ID parameters, database errors | [SQL injection](#6-sqli--manual-differential-probe) · [sqlmap narrow check](#sqlmap-narrow-sqli-verification-from-an-authorised-request) |
| Comments, display names, HTML rendering | [XSS](#7-xss--reflection-and-execution) |
| URL fetch, image importer, PDF preview, webhooks | [SSRF](#8-ssrf--owned-callback-only) |
| Change-email, update profile, transfer forms | [CSRF](#9-csrf--browser-confirmation) |
| `Access-Control-Allow-Origin` headers | [CORS](#10-cors--browser-verified-risk) |
| File uploads, avatars, attachments | [File upload](#11-file-upload--type-validation) |
| Download/export with `file` or `path` parameter | [Path traversal](#12-path-traversal--test-marker) |

**Stop condition:** if you cannot establish the real result with allowed test data, record `not confirmed`. HTTP `200`, `500`, a reflected string, or a DNS callback *alone* is not always proof of a security impact.

## Quick start: tools in under 5 minutes

### Burp Suite — primary workflow (Community Edition is enough)

1. Open **Proxy → Intercept**. Use **Open browser** (Burp's browser); normally leave **Intercept off** while browsing.
2. Perform **one legitimate action** in the application.
3. In **Proxy → HTTP history**, find the request. Right-click → **Send to Repeater**.
4. In **Repeater**, press **Send** without edits: this is your *baseline*.
5. Duplicate the tab, change **one** ID / role / JSON field / token / input, then **Send**.
6. Compare **request, status, response body and actual UI/server-side state**. For access control, verify which session you used.
7. Save **redacted** baseline and altered request/response. Put the finding into [Template B](02-detailed-finding-template.md).

**Practical tip:** Keep separate Repeater tabs named `BASELINE A`, `BASELINE B`, `TEST AS A`. Never accidentally send B's cookie/token while claiming to test as A.

### Shell setup for a *fictional* lab target

```bash
export BASE='https://lab.example.test'   # Replace with authorised THM lab URL
export ID_A='101'                          # Test resource owned by user A
export ID_B='202'                          # Test resource owned by user B

# Optional: enter short-lived lab token without putting it in shell history.
read -rs TOKEN_A; echo; export TOKEN_A
# Paste a valid lab token when prompted; press Enter.
```

| Tool | Copy-ready starter | What it does |
| --- | --- | --- |
| `curl` | `curl -i -sS --max-time 10 "$BASE/"` | HTTP response headers + body, 10 s timeout |
| `curl` + bearer | `curl -i -sS -H "Authorization: Bearer $TOKEN_A" "$BASE/api/me"` | Reissue a request as the supplied **test** account |
| `curl` + JSON | `curl -i -sS -X POST "$BASE/api/example" -H 'Content-Type: application/json' -d '{"test":"marker"}'` | POST a controlled JSON body |
| `jq` | `curl -sS "$BASE/api/public" \| jq .` | Pretty-print a JSON response |
| `curl` URL-encode | `curl -i -sS -G "$BASE/search" --data-urlencode "q=test marker"` | Send special characters safely in query parameters |
| `curl` + Burp | `curl -k -x http://127.0.0.1:8080 -i "$BASE/"` | Proxy through Burp; `-k` only if your lab proxy cert requires it |
| `ffuf` | See next block | Small, **low-rate** content discovery if scope permits |

**Low-impact `ffuf` example** — do **not** brute-force logins or scan unrelated hosts. See [wordlist choice and more ffuf examples](#ffuf-wordlist-selector--choose-by-the-thing-you-are-looking-for):

```bash
printf '%s\n' api login profile orders users > small-paths.txt
ffuf -u "$BASE/FUZZ" -w small-paths.txt -t 1 -rate 3 -maxtime 45 -ac
```

`-u` = URL with `FUZZ` placeholder; `-w` = wordlist; `-t 1` = one concurrent worker; `-rate 3` = at most 3 requests/sec; `-maxtime 45` = stop after 45 seconds; `-ac` = attempt to filter generic responses. **Verify any hit manually** in Repeater. Don’t use automatic enumeration where the test rules prohibit it.

## ffuf wordlist selector — choose by the thing you are looking for

**Don't pick the biggest dictionary by default.** Start with a short, task-specific list, establish the 404/wildcard baseline, and use longer lists only if time, scope and rate limits permit.

### Common dictionaries (SecLists / Kali)

Common Kali location: `/usr/share/seclists/`. Installation varies; check before running. An alternative for basic paths is `/usr/share/wordlists/dirb/common.txt`.

| Task | Suggested wordlist | Example use |
| --- | --- | --- |
| General web paths, fastest first pass | `Discovery/Web-Content/common.txt` | `/admin`, `/api`, `/login`, likely directories/files |
| Directory-focused discovery | `Discovery/Web-Content/raft-small-directories.txt` | Discover routes/subdirectories |
| File-focused discovery | `Discovery/Web-Content/raft-small-files.txt` | Find typical file names; add selected extensions if applicable |
| Word variants / route segments | `Discovery/Web-Content/raft-small-words.txt` | Probe additional path names |
| Hidden GET/POST parameter names | `Discovery/Web-Content/burp-parameter-names.txt` | One benign parameter at a time on read-only endpoint |
| API-specific routes | **Custom 10–30-line list** from observed UI/JS/HTTP history | Avoid irrelevant generic paths |
| File extensions | **Custom list**: `php`, `html`, `json`, `txt`, `js` | Use ffuf `-e` flag rather than huge mixed lists |
| In-scope virtual hosts (only if explicitly allowed) | `Discovery/DNS/subdomains-top1million-5000.txt` | Start with a tiny subset; not for broad public scanning |

**Check availability:**

```bash
export SECLISTS=/usr/share/seclists
ls "$SECLISTS/Discovery/Web-Content/common.txt"
ls "$SECLISTS/Discovery/Web-Content/"raft-small-*.txt
ls "$SECLISTS/Discovery/Web-Content/burp-parameter-names.txt"
# If not installed: check /usr/share/wordlists/dirb/common.txt
```

### Copy-ready ffuf commands (on your authorised lab only)

**A. Small path list — quickest first pass:**

```bash
printf '%s\n' admin api login profile orders assets > paths-small.txt
ffuf -u "$BASE/FUZZ" -w paths-small.txt -ac -t 1 -rate 3 -maxtime 45
```

**B. Common paths from SecLists — consider restricting run time:**

```bash
ffuf -u "$BASE/FUZZ" \
  -w "$SECLISTS/Discovery/Web-Content/common.txt" \
  -ac -t 1 -rate 3 -maxtime 60
```

**C. Files with selected extensions:**

```bash
ffuf -u "$BASE/FUZZ" \
  -w "$SECLISTS/Discovery/Web-Content/raft-small-words.txt" \
  -e '.php,.html,.json,.txt' -ac -t 1 -rate 3 -maxtime 60
```

**D. One benign query-parameter name on a read-only route:**

```bash
# Tiny subset, because the complete parameter list is much longer.
head -n 30 "$SECLISTS/Discovery/Web-Content/burp-parameter-names.txt" > params-small.txt
ffuf -u "$BASE/api/search?FUZZ=harmless-marker" \
  -w params-small.txt -ac -t 1 -rate 3 -maxtime 45
```

**E. Small custom API-route list based on the app you observed:**

```bash
printf '%s\n' api api/v1 api/v2 openapi.json swagger.json health > api-routes.txt
ffuf -u "$BASE/FUZZ" -w api-routes.txt -ac -t 1 -rate 3 -maxtime 45
```

**Optional: only for authorised in-scope virtual-host discovery**, assuming `lab.example.test` resolves to your THM lab and the test rules permit Host header checks:

```bash
head -n 25 "$SECLISTS/Discovery/DNS/subdomains-top1million-5000.txt" > vhosts-small.txt
ffuf -u "$BASE/" -H 'Host: FUZZ.lab.example.test' \
  -w vhosts-small.txt -ac -t 1 -rate 3 -maxtime 45
```

A matching vhost header may not mean the DNS name resolves or that it belongs to your authorised scope. Don't expand scope automatically.

### ffuf flags: what the results actually mean

| Flag | Meaning | Practical advice |
| --- | --- | --- |
| `-w` | Wordlist file | Choose by path, file, parameter, vhost |
| `FUZZ` | Placeholder in URL/header/body | Must appear where the candidate is substituted |
| `-e` | Try suffixes/extensions | Useful for files; avoid unrelated extensions |
| `-ac` | Automatic calibration | Helps remove generic wildcard responses; not always perfect |
| `-mc` | Match HTTP status | Use `-mc all` only with careful filtering |
| `-fc` | Filter HTTP status | E.g. `-fc 404`; beware custom not-found pages |
| `-fs` | Filter body size | E.g. `-fs 1234` after confirming wildcard baseline |
| `-t 1` | One worker | Gentler on lab server |
| `-rate 3` | Maximum ~3 requests/second | Use the actual authorised ceiling |
| `-maxtime 45` | Stop after a limited time | Prevent a long uncontrolled run |

**False-positive check:** first send a random path like `/unlikely-test-5482`. If the app returns `200` for everything, compare **body size/content**, redirects and actual functionality rather than treating every `200` as a discovery. Review a small number of findings manually in Burp Repeater.

### One safe evidence worksheet (copy after each tested issue)

```text
Finding: [title] | Candidate severity: [rating / unconfirmed]
Endpoint + method:
Authorised test identities / privileges:
Expected secure behaviour:
Baseline request + observed response:
One controlled change:
Changed request + observed response:
Confirmed downstream state / data (if any):
Prerequisites, existing controls and limitations:
Impact demonstrated vs impact only possible:
Recommendation / interim control:
Retest expected result:
Figures: [redacted request] [redacted response] [UI/state proof]
```

Screenshots are evidence, not the finding itself. A screenshot must show **what changed**; hide cookies, bearer tokens and personal data. Save only data permitted by the lab or rules of engagement.

---

## TEST RECIPES — choose one feature, then follow the steps

### 1. BOLA / IDOR: cross-account record access

**Use when:** API reads an object using `id` / `orderId` / `customerId`.

**Burp steps**
1. Create or identify test-owned record **A=101** and **B=202**, using two authorised test accounts.
2. With **A's session**, capture the normal request for A's record.
3. Keep **A's session**, change only `101` to `202`, and resend.
4. Verify whether response includes data **belonging to B**; compare to B's own view.

```http
GET /api/orders/202 HTTP/1.1
Host: lab.example.test
Authorization: Bearer TOKEN_FOR_USER_A
Accept: application/json
```

**CLI equivalent:** `curl -i -sS "$BASE/api/orders/$ID_B" -H "Authorization: Bearer $TOKEN_A"`

**Confirm:** A demonstrably accesses B's test record, not merely `200 OK` or a public record. **Expected:** `403` / `404` or no unauthorised data (application-specific).  
**Record:** request identity A, object owner B, minimal proof of returned B data.  
**Fix / retest:** server-side object-level authorisation on every read/write operation; replay with A after patch and expect denial.

### 2. BFLA — role bypass

**Use when:** approval, admin, export or refund operations are intended for a higher role.

1. Confirm a privileged role can perform the action on a **synthetic** pending record.
2. Keep a test record in `pending` state; capture the privileged endpoint and method.
3. Replay using the **lower-privileged** test account's own token (not the manager token).
4. Confirm **actual application state**, not only the response.

```http
POST /api/refunds/789/approve HTTP/1.1
Host: lab.example.test
Authorization: Bearer TOKEN_CUSTOMER_SERVICE
Content-Type: application/json

{"reason":"authorised-lab-test"}
```

**Confirm:** `pending` becomes `approved` under a Customer Service identity without Finance Manager authorisation. **No real payments.**  
**Fix / retest:** enforce server-side role/permission checks for the action; confirm denial for low privilege and success for authorised role.

### 3. Mass assignment — untrusted JSON fields

**Use when:** update endpoints accept a JSON object and map it onto backend model fields.

1. Baseline: edit an allowed field (for example, `displayName`).
2. With a **normal** test account, add one extra field such as `role` or `isAdmin` in a controlled test request.
3. Read back the record or test the permissions, without making destructive changes.

```http
PATCH /api/me HTTP/1.1
Host: lab.example.test
Authorization: Bearer TOKEN_FOR_USER_A
Content-Type: application/json

{"displayName":"Test User","isAdmin":true}
```

**Confirm:** the server actually persists a protected field or changes effective privileges. Merely accepting extra JSON is **not** sufficient.  
**Fix / retest:** allowlist mutable fields, validate ownership/roles on server, protect sensitive model properties.

### 4. Logout / session token replay

**Use when:** users log out but old tokens may remain usable.

1. Login as a **test** user, record a successful API request and the credential used.
2. Log out normally in the browser.
3. In Repeater send **the saved old request**, using the same pre-logout session/token.
4. Confirm whether a **protected** response is still returned after logout.

```http
GET /api/me HTTP/1.1
Host: lab.example.test
Authorization: Bearer OLD_TEST_TOKEN
```

**Confirm:** protected account data still accessible with the old credential. Note whether token is a self-contained JWT, whether logout is intended to invalidate access tokens, and its actual expiry.  
**Prerequisite:** attacker needs an already-valid token; **do not claim token theft occurred**.  
**Fix / retest:** appropriate server-side session invalidation/revocation, expiry and refresh-token rotation; retest old credential after logout.

### 5. OTP / login rate limits

**Use when:** password reset, OTP verify or login can be attempted repeatedly.

1. Create a legitimate **test** reset challenge.
2. Send a **small agreed number** of deliberately wrong codes (for example 5), with the same test identity.
3. Observe whether throttling, `429`, cooldown, attempt lockout or challenge invalidation occurs.
4. Stop at the allowed limit. Do **not** run a large brute-force against a live account.

**Confirm:** missing or ineffective controls for repeated attempts, **not** account takeover unless independently demonstrated.  
**Record:** exact test count, timing, per-account/session behaviour and any controls; don't infer risk from five attempts alone.  
**Fix / retest:** server-side per-challenge and per-account limits, expiry, monitoring, safe recovery flow.

### 6. SQLi — manual differential probe

**Use when:** query/search/filter inputs appear to reach a database.

**Start with a harmless baseline**, then compare one changed input at a time. URL encoding matters.

```bash
# Normal input
curl -i -sS -G "$BASE/search" --data-urlencode "q=sample"

# Basic quote probe (may be escaped/handled safely)
curl -i -sS -G "$BASE/search" --data-urlencode "q=sample'"

# Conditional pair for a compatible quoted-string context ONLY:
curl -i -sS -G "$BASE/search" --data-urlencode "q=sample' AND '1'='1"
curl -i -sS -G "$BASE/search" --data-urlencode "q=sample' AND '1'='2"
```

**Confirm:** consistent query-dependent behaviour, validated against normal input and application logic. A solitary `500` or error string is **not** proof. These fragments will not work in every query/database context.  
**Avoid:** data dumps, destructive queries or heavy time-based tests unless explicitly authorised.  
**Fix / retest:** parameterised queries; least-privileged database account; verify inputs are no longer interpreted as query syntax.

### sqlmap: narrow SQLi verification from an authorised request

**When to use:** Burp manual testing suggests an SQL injection in a **specific parameter**, and the rules of engagement allow automated SQLi testing. This complements — not replaces — evidence-based manual validation.

**A. Simple GET parameter (fictional lab):**

```bash
sqlmap -u "$BASE/products?id=1" -p id \
  --batch --level=1 --risk=1 --technique=BE \
  --threads=1 --delay=0.2 --timeout=10 --retries=0
```

**B. POST / JSON / authenticated API — preferred practical method:**

1. In Burp, intercept a **working authorised lab request** and save its full raw HTTP request to `request.txt` on your own machine.
2. Include correct `Host`, method, content type, request body and valid **test** session/token.
3. Restrict testing to the one parameter you suspect (example `q`).

```bash
chmod 600 request.txt                 # Tokens may be inside. Keep file private.
sqlmap -r request.txt -p q \
  --batch --level=1 --risk=1 --technique=BE \
  --threads=1 --delay=0.2 --timeout=10 --retries=0
```

**How to choose:** `-u` for a simple GET URL; `-r` for a captured authenticated POST / JSON request; `-p` for the specific suspected parameter. `--technique=BE` restricts testing to boolean- and error-based probes. `--level=1 --risk=1` starts with a smaller, lower-risk test set. It still sends **multiple requests** and is not guaranteed harmless, especially on state-changing queries — use only where authorised.

| Option | Purpose |
| --- | --- |
| `-u` / `-r` | Target URL / raw captured HTTP request |
| `-p id` | Test only the named injection parameter |
| `--batch` | Non-interactive prompts (can select defaults; review scope carefully) |
| `--level=1` | Low default breadth; avoid `level 5` for time-limited testing |
| `--risk=1` | Avoid higher-risk test classes |
| `--technique=BE` | Boolean- and error-based; skips time-based and stacked-query tests |
| `--threads=1` | One concurrent request |
| `--delay=0.2` | Pause between requests |
| `--timeout=10 --retries=0` | Bound each request and avoid retries |

**Evidence standard:** A tool's "possible injection" warning is not enough for a client report. Identify the parameter, repeat a consistent true/false or other harmless difference, verify it isn't normal application behaviour, and document exact observed impact. **No `--dump`, `--os-shell`, file reads, stacked commands or disruptive extraction** in this quick-start workflow. Never commit `request.txt`, sqlmap traffic logs, credentials or tokens to GitHub.

### 7. XSS — reflection and execution

**Use when:** a controlled name, comment, search result or message is displayed in HTML.

1. Submit a unique plain marker, e.g. `XSS_PROOF_42`; locate where it appears in the DOM/HTML.
2. Only in your own lab/test record, try a simple **non-exfiltrating** proof of browser JavaScript execution appropriate to the output context.
3. Verify **actual execution** in the browser, not just reflection in response text.

```html
<script>alert('XSS_PROOF_42')</script>
```

Alternative **only if HTML-attribute context requires it**:

```html
"><svg onload=alert('XSS_PROOF_42')>
```

**Confirm:** the application runs test-supplied script in the relevant user's browser context. Reflected text alone is not XSS.  
**Fix / retest:** correct context-sensitive output encoding / safe DOM APIs, sanitisation where HTML is allowed, and CSP as defence in depth. Avoid cookie theft or session exfiltration PoCs.

### 8. SSRF — owned callback only

**Use when:** an API accepts a URL to fetch, import, thumbnail or preview.

1. Find the intended URL-bearing parameter. Verify normal allowed use.
2. Substitute a **controlled callback endpoint you own or the lab provides**.
3. Inspect both API response and controlled callback logs.
4. Do **not** probe cloud metadata, internal networks or other systems outside the agreed scope.

```http
POST /api/preview HTTP/1.1
Host: lab.example.test
Content-Type: application/json

{"url":"https://your-owned-callback.example.test/lab-proof"}
```

**Confirm carefully:** a callback proves a server-side request might occur, but **does not by itself prove SSRF** if arbitrary external URLs are an intentional, authorised feature. Compare against documented/expected destination policy and controls.  
**Fix / retest:** strict destination allowlists, URL/IP validation after redirects and DNS resolution, egress restrictions, network isolation.

### 9. CSRF — browser confirmation

**Use when:** browser sessions use cookies and a state-changing action may lack CSRF protection.

1. Capture a **benign** account update (for example, display name).
2. Check anti-CSRF tokens, Origin/Referer validation and cookie `SameSite` behaviour.
3. In a **lab browser**, attempt the request from a different origin using only a test account and a harmless field change.
4. Confirm state really changed **with victim cookies automatically attached** and without intentional authorisation.

```html
<!-- For an authorised lab that accepts form-urlencoded POST -->
<form action="https://lab.example.test/profile/update" method="POST">
  <input name="displayName" value="csrf-lab-check">
  <button type="submit">Lab check</button>
</form>
```

**Important:** sending a cross-origin-looking request with `curl` or Burp alone doesn't reproduce browser cookie/SameSite restrictions. A sensitive action with strong CSRF validation is not vulnerable.  
**Fix / retest:** CSRF tokens, SameSite cookies, Origin checks as appropriate; verify cross-origin forged form is rejected.

### 10. CORS — browser-verified risk

**Use when:** API reflects origins or allows browser cross-origin reads.

```bash
curl -i -sS "$BASE/api/me" \
  -H 'Origin: https://your-test-origin.example.test' \
  -H "Authorization: Bearer $TOKEN_A"
```

**Look for:** `Access-Control-Allow-Origin`, `Access-Control-Allow-Credentials` and preflight permissions.  
**Confirm:** in the relevant **browser authentication model**, attacker-controlled origin JavaScript can read a sensitive response with victim credentials (for example, cookies actually sent). Header reflection alone is not always exploitable. Explicitly attaching a bearer token with `curl` is **not** proof that a third-party webpage can obtain it.  
**Fix / retest:** precise origin allowlist, no arbitrary reflected origin, correct credential policy.

### 11. File upload — type validation

**Use when:** avatar/document import has an advertised permitted file type.

1. Baseline: upload a valid **small** test file in the allowed format.
2. Create benign `proof.txt`; use a mismatched MIME type in a **lab** request to check whether the backend validates actual content.
3. Check how storage, retrieval, filename and rendering behave. Avoid executable web shells or dangerous file contents.

```bash
printf 'UPLOAD_TEST_MARKER\n' > proof.txt
curl -i -sS "$BASE/api/upload" \
  -H "Authorization: Bearer $TOKEN_A" \
  -F 'file=@proof.txt;type=image/png'
```

**Confirm:** acceptance alone may be intentional; assess whether a specific content restriction/security boundary is actually bypassed and whether dangerous serving or processing is possible.  
**Fix / retest:** signature/content checks, allowlists, safe storage names and paths, non-executable storage, safe download headers.

### 12. Path traversal — test marker

**Use when:** `download?file=`, export or template paths map to files.

1. Find **one test-owned marker file** at a known location in the authorised lab.
2. Try a relative path to that marker from the expected base directory; don't target sensitive OS files.
3. Compare file content with the known marker; confirm request didn't simply match an ordinary intended path.

```bash
curl -i -sS -G "$BASE/download" \
  --data-urlencode "file=../test-owned-marker.txt"
```

**Confirm:** the app serves a known file **outside its allowed directory**. A `404` or error is not proof.  
**Fix / retest:** map opaque IDs to stored files, canonicalise and enforce approved base directory, restrict filesystem permissions.

### 13. JWT — token inspection

**Use when:** `Authorization: Bearer` is JWT-shaped (three dot-separated components).

**Decode header and payload locally; this does NOT verify the signature:**

```bash
# Read a TEST JWT from standard input; do not paste a real credential into a public log.
python3 -c 'import sys,json,base64; t=sys.stdin.read().strip().split("."); assert len(t)==3, "not JWT"; [(print(json.dumps(json.loads(base64.urlsafe_b64decode(p+"="*(-len(p)%4))),indent=2))) for p in t[:2]]'
```

Run this command, paste your **lab-only JWT** into standard input, then send EOF (Ctrl-D on Unix).

**Inspect:** `alg`, `kid`, `iss`, `aud`, `exp`, `sub`, roles; compare authorised normal and lower-privilege tokens.  
**Important:** JWT payloads are readable by design. Editing a claim is **not** a vulnerability unless the server *accepts* a forged/invalidly signed token in the intended authentication flow.  
**Fix / retest:** verify cryptographic signature and approved algorithm, issuer/audience/expiry and server-side authorisation; do not trust client-controlled claims unchecked.

### 14. Other quick checks (only if in scope)

| Symptom | Benign probe / action | Real evidence needed |
| --- | --- | --- |
| Redirect using `next=` or `returnUrl=` | Try a test destination you control, e.g. `https://your-test-origin.example.test/landing` | Browser really redirects to an unintended external origin; assess phishing context |
| Possible server-side template expressions | In a test-only field, compare literal `{{7*7}}` with output | Server evaluates expression (e.g. `49`), not merely echoes text; do **not** escalate to code execution |
| Missing security headers | `curl -I -sS "$BASE/"` | Accurate header inventory and application-specific relevance; missing header ≠ confirmed XSS |
| HTTP methods | In Repeater test permitted methods on the target endpoint | Protected action possible with unexpected method; `OPTIONS` response alone ≠ vulnerability |
| Outdated library | Inspect browser DevTools / asset version | Exact affected version plus applicable advisory and actual use; old version alone may be informational |
| Business logic / price | Change only a synthetic cart/refund value | Server actually accepts an unauthorised state/amount; no real money or service disruption |

---

## What if the test fails? 2-minute troubleshooting

| You see | Check next |
| --- | --- |
| `401` | Token expired or wrong session? Compare baseline; don't assume bypass impossible |
| `403` | Is this intended authorisation? Check a known-valid role to establish control |
| `302` to login | Session/cookie missing, CSRF token expired, or redirect followed unexpectedly |
| `200` but no effect | Response content + actual state. `200` might be error JSON |
| `500` | Could be malformed input, server bug or normal error handling; repeat controlled comparison |
| Two accounts see same data | Is data genuinely owned by B and non-public? |
| Burp vs browser differs | Cookie/SameSite, Origin, TLS/proxy config, redirects, preflight |
| Multipart/body ignored | Correct HTTP method, `Content-Type`, API shape, content length and CSRF requirements |
| No SSRF callback | Check if it was meant to fetch URLs, callback logging, outgoing network policy |
| Time almost up | Stop, save confirmed evidence, write limitations, then draft the report |

## Submission: from hands-on to written test in 10 minutes

**0–3 min — select findings:** separate **confirmed behaviour** from speculation; record each affected endpoint, identity, request/response pair and actual impact.

**3–8 min — per-finding report:** use [02 — Detailed Finding](02-detailed-finding-template.md). Include **reproduction**, **expected vs actual**, minimum redacted **screenshots**, **prerequisites**, **risk**, **fix**, **retest**.

**8–10 min — management summary:** use [01 — Executive Summary](01-executive-summary-template.md): **Scope & findings → strongest evidence & business impact → prioritised action**. Do not dump all HTTP details into it.

**Writing phrases to reuse:**

- **Confirmed:** “Testing confirmed that [role] could [specific action] using [prerequisite].”
- **Potential:** “If exploited in production, this could result in [concrete business consequence].”
- **Limits:** “The test did not establish [unverified impact]; no evidence of live exploitation was identified.”
- **Priority:** “We recommend prioritising [issue] because [demonstrated impact and relevant likelihood factors].”
- **Fix:** “Implement [server-side control] and retest to verify that [previously unauthorised action] is denied.”

## Official documentation (syntax / further reading)

- [PortSwigger — Burp Repeater](https://portswigger.net/burp/documentation/desktop/tools/repeater)
- [PortSwigger — Reissuing requests](https://portswigger.net/burp/documentation/desktop/getting-started/reissuing-http-requests)
- [ffuf — usage, flags and examples](https://github.com/ffuf/ffuf)
- [sqlmap — official usage](https://github.com/sqlmapproject/sqlmap/wiki/usage)
- [SecLists — web-content wordlists](https://github.com/danielmiessler/SecLists/tree/master/Discovery/Web-Content)
- [ffuf — performance / rate controls](https://github.com/ffuf/ffuf/wiki/Performance-and-rate)
- [OWASP Web Security Testing Guide](https://owasp.org/www-project-web-security-testing-guide/)

**Reminder:** This is an assessment aid, not a recipe to run every probe against every site. Respect the actual engagement scope, rules of engagement, test accounts, request-rate limits and permitted evidence handling.
