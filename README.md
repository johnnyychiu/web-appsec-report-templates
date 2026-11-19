# Web AppSec Report Templates

Practical, **copy-and-fill** Markdown templates and hands-on command references for authorised Web/API penetration testing and timed written assessments. Examples use **fictional systems and synthetic test evidence**, with concise British English suitable for a client-facing report.

## Interactive report builder (new)

**[Open the report builder (index.html)](index.html)** to choose from **25 Web/API vulnerability types** (including SSTI) and automatically fill **interim mitigation**, **permanent remediation**, **conditional technical impact** and **retesting guidance**. Add up to five findings and generate a draft **Summary of Findings table**, **Executive Summary** or **Detailed Finding**, then copy or download Markdown. **Business impact, likelihood, existing controls and severity justification are separate fields that require your own evidence**; the site never automatically assigns a severity.

**Publish as a GitHub Pages website:** This repository contains a ready-to-run static `index.html`. If GitHub Pages is not enabled, go to **Settings → Pages → Build and deployment → Source: Deploy from a branch → Branch: main → Folder: /(root) → Save**. GitHub will display the published site URL under Pages. Suggested project-site address after activation: `https://johnnyychiu.github.io/web-appsec-report-templates/` (verify Pages has deployed before sharing).

The builder runs entirely in your browser; it does not transmit or store finding contents. It deliberately leaves confirmed evidence blank until you supply it. **Do not enter live client secrets or production personal data.**

## Hands-on Lab Toolkit — Payload Library and Command Builder

- **[Payload Library](payload-library.html)** — 50 context-aware, copy-ready test strings across eight categories: XSS (image, SVG, script and HTML attributes), SQLi (Boolean, error, UNION, timing), Traversal (plain and encoded paths), SSTI (Jinja2, Twig, FreeMarker, EJS/ERB, Liquid, Velocity), NoSQL, Command, XML and SSRF. Each includes what to observe and limitations. **Copy-only, never executes a payload.**
- **[Command Builder](command-builder.html)** — Choose tool/mode, change as few inputs as necessary, then copy a Bash command for **ffuf, Gobuster, sqlmap, Nmap, Hydra, SSH (local forward, SOCKS, ProxyJump) or curl**. Conservative defaults limit request rates, concurrency and scope. **Never runs commands automatically.**

**Advanced flags:** The Command Builder uses minimal **Quick setup** by default and has a collapsible **Advanced flags** section, with about 69 optional settings. These include ffuf request rates, status/size filters, headers, proxy and raw requests; Gobuster extension/status filters; sqlmap level/risk, DBMS and matching; Nmap scan type/timing/output; Hydra delay and service port; SSH keepalives, keys and forwarding; and curl headers, timeouts and proxies. Additional operations include **ffuf Burp request file**, **SSH remote port forwarding (-R)** and **curl PATCH/PUT/DELETE**. The preview updates immediately and rejects invalid input. **Nmap now offers three extra modes:** `Vulnerability checks · safe NSE scripts` (restricts to non-external, non-intrusive safe/vuln categories), `CVE lookup · Vulners` (uses `-sV --script vulners`, sends software/version information to Vulners and supports a minimum CVSS filter), and `Broad NSE vuln category` (can run intrusive scripts; explicit authorisation required). These scans produce leads for investigation, **not automatically confirmed vulnerabilities**. Check available NSE scripts locally with `nmap --script-help vulners` or `nmap --script-help vuln`.

**Suggested 60-minute Lab workflow:** Use [Payload Library](payload-library.html) for a specific manual test; use [Command Builder](command-builder.html) when you need a reproducible tool command; use [CSRF Helper](csrf-helper.html) only for a suitable browser-based request. Verify test evidence before entering [Report Builder](index.html).

These are learning aids for authorised systems. Confirm that tools are installed, check provided wordlists and scope, do not copy real tokens or customer data, and avoid login guessing without express permission.

## CSRF Lab Helper — browser proof-of-concept generator

**[Open CSRF Lab Helper](csrf-helper.html)** for a 5–8-minute authorised CSRF check. This separate page includes Cantonese notes, a Burp raw-request parser, and a **manually triggered GET link or application/x-www-form-urlencoded POST form**. Paste a request, review the target URL and authentication conditions, then explicitly enable the live trigger to open the request in a signed-in test browser. You can also **copy or download a standalone csrf-poc.html**.

The helper **does not automatically submit** anything or transmit pasted requests for processing. It omits Cookie, Authorization and detected CSRF/sensitive form or URL parameters. A browser decides whether to include session cookies according to SameSite and other policies. **JSON, XML, multipart, PATCH, PUT and DELETE cannot be reproduced exactly by this basic HTML-form tool**; unsupported requests are flagged, not represented as successful CSRF tests. Check actual HTTP requests in Burp **and verify that the intended test account state changed** before reporting a confirmed finding.

## Quick start for a 30-minute written test

The [interactive report builder](index.html) opens in **Exam Mode** with three report outputs:

| Test instruction | Use this output | What it includes |
| --- | --- | --- |
| **Summary of Findings** | **Default tab** | Scope/overall risk, **one table row per finding** (severity, observed evidence, potential impact and fix), prioritised remediation and evidence limitations |
| **Executive Summary** | Second tab | Concise **100–150-word** three-paragraph management overview when the task requests it |
| **Detailed Finding / PoC** | Third tab (switches to Full Report) | Reproduction steps, evidence/screenshots, risk reasoning, remediation and retesting |

**Fast workflow:** Enter the scope, select the main finding and severity, then complete four essential short fields: **confirmed evidence, prerequisite, concrete business impact, and severity justification**. Add limitations when relevant. **New: each of the five numbered Exam Mode fields displays two different complete model sentences for the chosen vulnerability (250 example choices across 25 vulnerability types). The button label is the exact sentence that will be appended to the field.** No bracket placeholders appear in the suggested sentences. The builder **appends rather than replacing** existing text and prevents duplicate insertions of the same option. You must still **edit and verify** the fictional example against your actual test evidence; the builder warns while an unedited suggestion remains. Secondary findings offer the same sentence-choice buttons for their optional evidence field. For secondary findings, select type/severity and optionally enter **one line of observed evidence**. Suggested fixes appear automatically. The Summary of Findings table flags missing evidence rather than presenting an unsupported claim as confirmed.

**Exam Mode** hides advanced inputs; **Full Report Mode** restores detailed fields, including exposure, controls, likelihood and editable remediation. Mode switching preserves entered facts. The **Summary of Findings** table has **no automatic 150-word cap**; follow the actual test's length and format instructions.

The builder generates a **draft**, not a validated security assessment. Check it against the authorised test evidence before submitting. Never enter real client secrets or personal data in the public repo.

## Open a template

| Written-test task | Document | Includes |
| --- | --- | --- |
| **Executive Summary / Summary of Findings** | [01 — Executive Summary Template](01-executive-summary-template.md) | Eight-fact worksheet, three-paragraph 100–150-word structure, risk/impact language, **12 worked example summaries** |
| **Detailed Finding / Proof of Concept** | [02 — Detailed Finding Template](02-detailed-finding-template.md) | Reproduction steps, expected vs actual behaviour, evidence/screenshot captions, impact, severity rationale, mitigation, fix and retesting; **12 worked finding examples** |
| **Hands-on Web/API practical test** | [03 — Hands-on Technical Cheat Sheet](03-hands-on-web-api-cheat-sheet.md) | Burp Repeater runbook, curl / jq / ffuf commands, safe lab payloads, **14 testing recipes**, evidence worksheet and triage instructions |

## Fast workflow for a hands-on assessment

1. **Test:** Use the [hands-on cheat sheet](03-hands-on-web-api-cheat-sheet.md) to identify an in-scope weakness using authorised test accounts and synthetic data.
2. **Prove:** Keep baseline and modified requests, validate the relevant response or actual state, and note attack prerequisites.
3. **Document:** Copy **Template B** for each confirmed finding. Record evidence, the specific business impact, attacker prerequisites, exposure, existing controls and likelihood reasoning, without inventing consequences.
4. **Summarise:** Fill the eight facts in **Template A**, then write a short management-facing summary focused on the most significant issue(s).
5. **Review:** Check that confirmed behaviour is distinguished from possible consequences, severity is justified and no sensitive data is disclosed.

## Vulnerabilities covered

BOLA/IDOR · BFLA/refund approval · SQL injection · NoSQL injection · SSTI · OS command injection · XXE · path traversal · insecure deserialisation · Stored / Reflected / DOM XSS · SSRF · JWT validation · OAuth/OIDC · session invalidation · password-reset OTP rate limiting · mass assignment · CSRF · unsafe file upload · credentialed CORS · race conditions · business logic abuse · open redirect · outdated JavaScript dependency · verbose error messages.

> **Important:** These are writing references, not ready-made claims about a real environment. Follow the actual assessment instructions and rules of engagement. Never publish credentials, session tokens, client details, personal data, private endpoints or unredacted screenshots. Risk ratings in examples are illustrative.
