# Web AppSec Report Templates

Practical, **copy-and-fill** Markdown templates and hands-on command references for authorised Web/API penetration testing and timed written assessments. Examples use **fictional systems and synthetic test evidence**, with concise British English suitable for a client-facing report.

## Open a template

| Written-test task | Document | Includes |
| --- | --- | --- |
| **Executive Summary / Summary of Findings** | [01 — Executive Summary Template](01-executive-summary-template.md) | Eight-fact worksheet, three-paragraph 100–150-word structure, risk/impact language, **12 worked example summaries** |
| **Detailed Finding / Proof of Concept** | [02 — Detailed Finding Template](02-detailed-finding-template.md) | Reproduction steps, expected vs actual behaviour, evidence/screenshot captions, impact, severity rationale, mitigation, fix and retesting; **12 worked finding examples** |
| **Hands-on Web/API practical test** | [03 — Hands-on Technical Cheat Sheet](03-hands-on-web-api-cheat-sheet.md) | Burp Repeater runbook, curl / jq / ffuf commands, safe lab payloads, **14 testing recipes**, evidence worksheet and triage instructions |

## Fast workflow for a hands-on assessment

1. **Test:** Use the [hands-on cheat sheet](03-hands-on-web-api-cheat-sheet.md) to identify an in-scope weakness using authorised test accounts and synthetic data.
2. **Prove:** Keep baseline and modified requests, validate the relevant response or actual state, and note attack prerequisites.
3. **Document:** Copy **Template B** for each confirmed finding. Record evidence and limitations without inventing impact.
4. **Summarise:** Fill the eight facts in **Template A**, then write a short management-facing summary focused on the most significant issue(s).
5. **Review:** Check that confirmed behaviour is distinguished from possible consequences, severity is justified and no sensitive data is disclosed.

## Vulnerabilities covered

BOLA/IDOR · BFLA/refund approval · SQL injection · Stored XSS · SSRF · Session not revoked after logout · Password-reset OTP rate limiting · Mass assignment · CSRF · Unsafe file upload · CORS misconfiguration · Outdated JavaScript dependency.

> **Important:** These are writing references, not ready-made claims about a real environment. Follow the actual assessment instructions and rules of engagement. Never publish credentials, session tokens, client details, personal data, private endpoints or unredacted screenshots. Risk ratings in examples are illustrative.
