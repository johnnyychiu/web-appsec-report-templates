# Web AppSec Report Templates

Practical, **copy-and-fill** Markdown templates and hands-on command references for authorised Web/API penetration testing and timed written assessments. Examples use **fictional systems and synthetic test evidence**, with concise British English suitable for a client-facing report.

## Interactive report builder (new)

**[Open the report builder (index.html)](index.html)** to choose from **25 Web/API vulnerability types** (including SSTI) and automatically fill **interim mitigation**, **permanent remediation**, **conditional technical impact** and **retesting guidance**. Add up to five findings and generate a draft **Executive Summary** or **Detailed Finding**, then copy or download Markdown. **Business impact, likelihood, existing controls and severity justification are separate fields that require your own evidence**; the site never automatically assigns a severity.

**Publish as a GitHub Pages website:** This repository contains a ready-to-run static `index.html`. If GitHub Pages is not enabled, go to **Settings → Pages → Build and deployment → Source: Deploy from a branch → Branch: main → Folder: /(root) → Save**. GitHub will display the published site URL under Pages. Suggested project-site address after activation: `https://johnnyychiu.github.io/web-appsec-report-templates/` (verify Pages has deployed before sharing).

The builder runs entirely in your browser; it does not transmit or store finding contents. It deliberately leaves confirmed evidence blank until you supply it. **Do not enter live client secrets or production personal data.**

## Quick start for a 30-minute written test

The [interactive report builder](index.html) opens in **Exam Mode** with three report outputs:

| Test instruction | Use this output | What it includes |
| --- | --- | --- |
| **Summary of Findings** | **Default tab** | Scope/overall risk, **one table row per finding** (severity, observed evidence, potential impact and fix), prioritised remediation and evidence limitations |
| **Executive Summary** | Second tab | Concise **100–150-word** three-paragraph management overview when the task requests it |
| **Detailed Finding / PoC** | Third tab (switches to Full Report) | Reproduction steps, evidence/screenshots, risk reasoning, remediation and retesting |

**Fast workflow:** Enter the scope, select the main finding and severity, then complete four essential short fields: **confirmed evidence, prerequisite, concrete business impact, and severity justification**. Add limitations when relevant. For secondary findings, select type/severity and optionally enter **one line of observed evidence**. Suggested fixes appear automatically. The Summary of Findings table flags missing evidence rather than presenting an unsupported claim as confirmed.

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
