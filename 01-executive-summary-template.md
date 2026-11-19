# Template A — Executive Summary / Summary of Findings

> **Purpose:** Fast, client-facing British-English writing for a **hypothetical, authorised** Web/API penetration test. **All examples and endpoints are fictional.** These are examples of how to communicate *evidence*, not claims about any real organisation.
>
> **Public-repository safety:** Never commit live access tokens, credentials, customer records, private client URLs, report attachments or unredacted screenshots. Use synthetic data only.

## Fast navigation

1. [Which output does the test ask for?](#which-output-does-the-test-ask-for)
2. [Fill these eight facts first](#fill-these-eight-facts-first)
3. [Universal 100–150-word template](#universal-100150-word-template)
4. [One-finding / uncertain-impact alternatives](#one-finding--uncertain-impact-alternatives)
5. [Quick impact and remediation lookup](#quick-impact-and-remediation-lookup)
6. [Complete worked answers by vulnerability](#complete-worked-answers-by-vulnerability)
7. [Five-minute final review](#five-minute-final-review)

## Which output does the test ask for?

| Wording in test | What to deliver | What **not** to include |
| --- | --- | --- |
| **Executive Summary** | 3 short paragraphs: scope/findings → demonstrated risk → priorities/actions | Raw HTTP traffic, detailed exploitation steps, screenshots |
| **Summary of Findings** | At least a findings table; add short overall risk paragraph if requested | Full reproduction for every issue unless requested |
| **Detailed Finding / PoC** | Use **Template B — Detailed Finding** with steps, expected/actual, evidence and retesting | Unverified allegations or unsupported severity |
| **Full report** | Executive Summary + summary table + one Template B per confirmed finding | Credentials and private customer data |

**Default length for a timed executive-summary exercise:** 100–150 words unless the instructions specify otherwise. Risk ratings are **illustrative**, not automatic CVSS scores or substitutes for the client's methodology.

## Fill these eight facts first

| # | Required field | Fill with factual evidence, not a guess |
| --- | --- | --- |
| 1 | **Scope** | Application and API actually tested; boundary of assessment |
| 2 | **All findings** | Exact names, confirmed vs observations, ratings if justified |
| 3 | **Main demonstrated behaviour** | Who did what to which resource and what changed |
| 4 | **Prerequisite** | Account, role, victim interaction, token possession, network reachability |
| 5 | **Business consequence** | Specific exposure, unauthorised transaction, account misuse, disruption |
| 6 | **Risk justification** | Impact + ease/prerequisites + controls + evidence uncertainty |
| 7 | **Interim control** | Narrowest feasible mitigation, subject to operational review |
| 8 | **Permanent fix + retest** | Root-cause control and evidence required to close the finding |

**Critical rule:** "Testing confirmed..." describes tested behaviour. "Could result in..." describes an unproven consequence. **A test-only £2,000 refund is not a real loss.** Publicly accessible does not equal unauthenticated exploitation; an account prerequisite does not automatically make the risk Medium.

## Universal 100–150-word template

**Paragraph 1 — Scope / Findings**

> The penetration test of **[APPLICATION AND API]** identified **[NUMBER]** security findings, including **[FINDING A] ([RATING])**, **[FINDING B] ([RATING])** and **[FINDING C] ([RATING])**. The most significant weakness affects **[DATA / TRANSACTION / BUSINESS PROCESS]**.

**Paragraph 2 — Confirmed behaviour / business risk**

> Testing confirmed that **[ROLE]** could **[PRECISE UNAUTHORISED ACTION]**. Although exploitation requires **[REAL PREREQUISITE]**, **[OBSERVED RESULT OR LIMIT OF CONTROL]**. If exploited in production, this could result in **[SPECIFIC CONSEQUENCE]**, potentially leading to **[DIRECT BUSINESS IMPACT]**. **[IMPORTANT LIMITATION: NO REAL LOSS / NO EVIDENCE OF ATTACK / IMPACT NOT ESTABLISHED]**.

**Paragraph 3 — Decision / action**

> We recommend prioritising **[MAIN FINDING]** **[BASED ON EVIDENCE AND RISK]**. As an interim measure, **[NARROW MITIGATION]** should be considered after assessing its operational effect. The permanent fix should involve **[ROOT CAUSE]**, followed by retesting. The remaining findings should be addressed according to their assessed risk.

**Use:** Replace every bracketed field, remove unsupported claims, and count the final words. If there are fewer than three findings, adapt the first and final sentence; never invent findings to fill slots.

## One-finding / uncertain-impact alternatives

### Only one confirmed finding

> The assessment of **[SCOPE]** identified **[VULNERABILITY]** affecting **[FUNCTION]**. Testing confirmed **[SPECIFIC RESULT]** under **[PREREQUISITE]**. This could result in **[BUSINESS CONSEQUENCE]**, although **[LIMITATION]**. Given **[IMPACT]** and **[EXPLOITABILITY]**, we recommend **[JUSTIFIED PRIORITY]**. Implement **[ROOT-CAUSE FIX]**, consider **[SAFE INTERIM CONTROL]**, and retest **[EXPECTED SECURE RESULT]**.

### Evidence does not prove an exploit or impact

> Testing identified **[OBSERVATION]** on **[FUNCTION]**. We observed **[FACTUAL SYMPTOM]**, but did not establish **[UNPROVEN OUTCOME]**. The potential impact depends on **[MISSING CONDITIONS OR CONTROLS]**. We recommend **[VALIDATION OR HARDENING ACTION]** and reassessing the rating after **[SPECIFIC VERIFICATION]**.

**Do not** write "Attackers can steal data" when the only evidence is an error message, missing rate limit or old version number.

## Quick impact and remediation lookup

| Finding | Business consequence to consider (**conditional**) | Typical root-cause fix |
| --- | --- | --- |
| BOLA / IDOR | Exposure or modification of other customers' data | Check object ownership/authorisation server-side on every request |
| BFLA / privilege boundary | Unauthorised approval, refund or administrator action | Enforce function-level permissions at API/business layer |
| SQL injection | Data disclosure/modification depending on DB rights | Parameterised queries; least privilege; validate query construction |
| Stored XSS | Actions in victim browser, potential session impact | Context-aware encoding and safe rendering; CSP as defence in depth |
| SSRF | Unintended service access via server | Strict outbound destination policy; redirect/DNS/IP controls |
| Session not invalidated | Continued API use with a previously issued token | Reliable server-side invalidation/revocation; retest logout |
| Password-reset OTP rate limiting | Increased opportunity for automated guessing | Per-account/reset rate limits, expiry and monitoring |
| Mass assignment | Unauthorised change to entitlements or profile control fields | Explicitly allowlisted input binding and server-side business rules |
| CSRF | Unintended state change on victim's behalf | Anti-CSRF tokens, SameSite and sensitive-action confirmation |
| Unsafe uploads | Untrusted active content executed in browser | Type restrictions, separate origin, safe download handling |
| Credentialed CORS error | Sensitive API reads from untrusted web origin | Explicit origin allowlist and credential policy |
| Outdated library | Uncertain risk until version/usage/advisory validated | Supported version and dependency management |

## Complete worked answers by vulnerability

Every answer below is a **fictional standalone 3-paragraph executive summary**, not a statement that a real client was compromised. Use them as patterns; replace all evidence before including anything in a client report.

**Jump straight to a worked answer:** [Broken Object-Level Authorisation (BOLA / IDOR)](#case-bola) · [Broken Function-Level Authorisation (Refund Approval)](#case-bfla) · [SQL Injection in Reporting API](#case-sqli) · [Stored Cross-Site Scripting (XSS)](#case-stored-xss) · [Server-Side Request Forgery (SSRF)](#case-ssrf) · [Session Token Remains Valid After Logout](#case-session) · [Insufficient Password-Reset OTP Rate Limiting](#case-otp) · [Mass Assignment / Excessive Property Binding](#case-mass-assignment) · [Cross-Site Request Forgery (CSRF) on Profile Change](#case-csrf) · [Unsafe File Upload / Active Content](#case-upload) · [Cross-Origin Resource Sharing (CORS) Misconfiguration](#case-cors) · [Outdated JavaScript Dependency (Evidence-Limited)](#case-dependency)

<a id="case-bola"></a>

### Broken Object-Level Authorisation (BOLA / IDOR)

**Practice facts (fictional):** Scenario: Fictional customer portal. Authenticated Account A retrieved Account B's name, email and address. Valid account required; no mass extraction or attacker activity demonstrated. Two other observations: password-reset OTP rate limiting and an outdated JavaScript library.

The penetration test of the internet-facing customer portal and associated APIs identified three findings: broken object-level authorisation (High), insufficient password-reset OTP rate limiting (Medium), and an outdated JavaScript library (Low). The access-control finding presents the clearest demonstrated risk to customer information.

Testing confirmed that an authenticated customer could retrieve another customer's name, email address and postal address without authorisation. Exploitation requires a valid customer account, but no elevated privileges were needed. This could expose personal information and lead to privacy concerns and regulatory scrutiny. The potential scale of exposure and any malicious exploitation remain unconfirmed.

We recommend prioritising server-side object-level authorisation checks and retesting the affected endpoints. Pending remediation, consider restricting the affected data-retrieval functionality after reviewing operational impact. Address the other findings through a risk-based plan.

<a id="case-bfla"></a>

### Broken Function-Level Authorisation (Refund Approval)

**Practice facts (fictional):** Scenario: Fictional e-commerce platform. Customer Service role approved a £2,000 TEST refund through API without Finance Manager authorisation; no production transfer or actual loss. Additional observations: weak OTP limiting, outdated library.

The assessment of the e-commerce platform and its APIs identified broken function-level authorisation (High), insufficient password-reset OTP rate limiting (Medium), and an outdated JavaScript library (Low). The most significant issue affects the refund approval process and its financial control boundary.

In a controlled test, a Customer Service account approved a £2,000 refund and changed its status from pending to approved without Finance Manager authorisation. Valid Customer Service credentials were required, but no additional role privileges were needed. If this behaviour were exploited in production, unauthorised refunds could result in financial loss and weaken internal financial controls. No real payment or malicious exploitation was demonstrated.

Prioritise server-side role-based checks on the approval endpoint and verify the fix through retesting. Until then, restrict API approval to a reliably enforced authorised workflow or disable that action while retaining a reviewed manual approval route.

<a id="case-sqli"></a>

### SQL Injection in Reporting API

**Practice facts (fictional):** Scenario: Fictional authenticated reporting API. Changing the `sort` input caused repeatable database errors; two controlled boolean conditions produced different result sets. No database records were extracted. Other findings: missing rate limiting, an outdated dependency.

The penetration test of the reporting application and API identified a suspected SQL injection weakness in a report-filtering parameter, alongside insufficient login rate limiting and an outdated dependency. The injection issue warrants priority investigation because it may affect the confidentiality or integrity of reporting data.

Controlled input changes produced consistent database error behaviour and different outcomes for two complementary test conditions, supporting the presence of injection. Exploitation required an authenticated reporting account; the assessment did not extract data or demonstrate modification of records. Potential consequences include unauthorised database queries, but the extent of accessible data remains unknown. The risk is provisionally High pending safe confirmation of impact and database permissions.

We recommend implementing parameterised database access and reviewing dynamic query construction. Restrict access to the affected reporting function if a safe interim control is available, then retest both malicious and legitimate inputs.

<a id="case-stored-xss"></a>

### Stored Cross-Site Scripting (XSS)

**Practice facts (fictional):** Scenario: Fictional support-ticket web app. Customer submitted a harmless proof-of-execution marker, then support user opening the ticket executed it in browser. No cookie theft or account takeover shown. Other issues: missing headers and outdated library.

The assessment of the support-ticket application identified stored cross-site scripting (Medium), missing recommended response security headers (Low), and an outdated JavaScript dependency (Low). The main concern is unsafe rendering of customer-supplied ticket content in the staff interface.

Testing confirmed that a harmless script marker submitted through a customer ticket executed when a support user viewed that ticket. Exploitation requires the content to be stored and opened by another user. This could permit unauthorised actions within the victim's browser, depending on its session privileges and other protections. The assessment did not demonstrate credential theft, privilege escalation or compromise outside the test accounts. The rating should be reviewed against the actual viewing audience and controls.

Prioritise context-appropriate output encoding and safe HTML rendering. Consider limiting rich content until the fix is deployed, review Content Security Policy as defence in depth, and retest ticket submission and display.

<a id="case-ssrf"></a>

### Server-Side Request Forgery (SSRF)

**Practice facts (fictional):** Scenario: Fictional URL-preview API. Authenticated user could instruct preview service to request an authorised internal lab endpoint; response demonstrated unexpected internal connectivity. No cloud metadata, credentials or sensitive system data accessed. Other findings: weak OTP rate limits, outdated library.

The test of the URL-preview feature and associated API identified a server-side request forgery weakness (Medium), insufficient password-reset rate limiting (Medium), and an outdated dependency (Low). The most significant uncertainty is which network destinations the preview service can reach.

With a valid user account, testing induced the preview service to contact an internal test endpoint outside its intended destination policy. This demonstrates unexpected server-side network access, but not exposure of internal credentials or sensitive information. Depending on reachable systems, an attacker might use this path to interact with internal services. The potential impact and overall severity remain conditional on network segmentation and service permissions.

We recommend restricting outbound destinations with a deny-by-default policy, revalidating redirects and resolved addresses, and retesting. Temporarily limit the preview feature if the restriction cannot be enforced reliably; review operational impact first.

<a id="case-session"></a>

### Session Token Remains Valid After Logout

**Practice facts (fictional):** Scenario: Fictional SaaS portal. Previously issued access token remained accepted for 30 minutes after logout. Token had to be possessed already; no theft observed. Other findings: missing rate limiting and security headers.

The penetration test of the customer portal and REST API identified continued token validity after logout (Medium, provisional), inadequate login rate limiting (Medium), and missing security headers (Low). The session-management issue affects the user's ability to end access to their account.

Testing confirmed that a token issued before logout continued to authorise API requests for 30 minutes afterwards. An attacker would first need to obtain a valid token; the assessment found no evidence of token theft or account compromise. If a token were stolen or otherwise exposed, this behaviour could allow continued account access after the user logs out. The appropriate severity depends on token lifetime, sensitive actions and other safeguards.

We recommend reliable server-side session or token invalidation, with retesting of both API and browser logout. Shorter token lifetimes may reduce exposure temporarily but do not replace proper invalidation.

<a id="case-otp"></a>

### Insufficient Password-Reset OTP Rate Limiting

**Practice facts (fictional):** Scenario: Fictional consumer app. OTP verify endpoint processed 50 incorrect attempts without cooldown; no successful guess, no account takeover. Other findings: outdated library and missing security headers.

The assessment of the consumer application's password-reset flow identified insufficient OTP verification rate limiting (Medium, provisional), an outdated JavaScript library (Low), and missing recommended security headers (Low). The primary concern is exposure of the account-recovery process to automated guessing.

The test account's reset endpoint processed 50 consecutive incorrect OTP submissions without a cooldown or account-level lockout. No correct OTP was guessed and no unauthorised account reset was demonstrated. Exploitation would additionally depend on OTP entropy, expiry, valid reset state, and any controls applied across sessions or IP addresses. If those controls are weak, repeated attempts could increase the chance of account compromise.

We recommend applying server-side attempt limits per reset transaction and account, sensible expiry and monitoring, then retesting legitimate and abusive flows. Review the dependency and header observations in the normal remediation backlog.

<a id="case-mass-assignment"></a>

### Mass Assignment / Excessive Property Binding

**Practice facts (fictional):** Scenario: Fictional subscription app. Standard customer updated `planType` from trial to premium via undocumented JSON field; feature entitlement changed. No billing charges or access to staff resources demonstrated. Other findings: missing rate limiting and outdated library.

The test of the subscription application's account API identified a mass-assignment weakness (Medium), insufficient login rate limiting (Medium), and an outdated JavaScript library (Low). The most material weakness concerns the separation between customer-editable profile data and subscription entitlements.

Testing confirmed that an authenticated trial user could include an undocumented property in a profile update and change their entitlement to a premium plan. This required a standard customer account and did not demonstrate access to other customers' information. If the behaviour were reproducible in production, it could enable unauthorised use of paid features and undermine billing controls. No production billing effect or financial loss was demonstrated.

We recommend accepting only explicitly permitted fields and enforcing entitlement changes in a dedicated server-side business workflow. Consider disabling direct changes to affected fields temporarily, and retest normal profile updates and privilege boundaries.

<a id="case-csrf"></a>

### Cross-Site Request Forgery (CSRF) on Profile Change

**Practice facts (fictional):** Scenario: Fictional account portal. In a controlled browser test, cross-site form submission changed test account notification email while authenticated, and no anti-CSRF control blocked request. No takeover observed. Other findings: outdated library and missing headers.

The assessment of the account-management portal identified a cross-site request forgery weakness in the notification-email change function (Medium), an outdated dependency (Low), and missing recommended security headers (Low). The primary finding affects a state-changing action expected to require the account holder's intention.

In a controlled browser test, a request initiated from a separate test origin changed the signed-in test user's notification email without an additional authorisation check. Exploitation would require the victim to remain authenticated and visit attacker-controlled content. This could redirect account notifications and contribute to further abuse, although no account takeover or sensitive data disclosure was demonstrated. The overall severity depends on the role of that email address and existing confirmation controls.

We recommend anti-CSRF protection and appropriate SameSite cookie settings, with server-side confirmation for sensitive changes. Retest both legitimate profile updates and cross-origin submissions.

<a id="case-upload"></a>

### Unsafe File Upload / Active Content

**Practice facts (fictional):** Scenario: Fictional ticketing system. Authenticated customer uploaded HTML attachment; browser rendered it as active content on application's own origin when staff clicked link. No server-side code execution or session theft demonstrated. Other findings: low risk headers and dependency.

The penetration test of the ticketing application identified unsafe handling of uploaded HTML attachments (Medium), an outdated JavaScript library (Low), and missing response security headers (Low). The upload issue affects how untrusted customer files are served to staff users.

Testing confirmed that an authenticated customer could upload an HTML file and have it rendered as active content under the application's origin when a staff member opened the attachment. This could expose staff browser sessions to malicious content, depending on browser isolation and session protections. The assessment did not demonstrate server-side code execution, credential theft or broader compromise. A valid customer account and staff interaction were necessary in the tested workflow.

We recommend restricting permitted file types, storing untrusted uploads outside the application's origin, and serving downloads with safe content handling. Retest uploads, downloads and normal staff workflows.

<a id="case-cors"></a>

### Cross-Origin Resource Sharing (CORS) Misconfiguration

**Practice facts (fictional):** Scenario: Fictional portal API. In browser with authorised victim test account, arbitrary origin was echoed alongside credentialed CORS support and JavaScript on separate origin read account profile JSON. No real victims impacted. Other findings: outdated library and missing security header.

The assessment of the customer portal API identified a credentialed CORS misconfiguration (High, provisional), an outdated JavaScript library (Low), and a missing recommended security header (Low). The most significant issue affects the separation between trusted application origins and untrusted websites.

In a controlled browser test, JavaScript running on a separate test origin successfully read the signed-in test customer's profile response using existing browser credentials. Exploitation would require a customer to be authenticated and visit a malicious origin; no real user data was accessed. Depending on which endpoints share this policy, the weakness could expose personal information to an unauthorised website. The full scope of accessible data was not established.

We recommend replacing dynamic origin reflection with an explicit allowlist, reviewing credential handling and retesting cross-origin access. Restrict affected API responses temporarily if reliable enforcement can be applied without disrupting customers.

<a id="case-dependency"></a>

### Outdated JavaScript Dependency (Evidence-Limited)

**Practice facts (fictional):** Scenario: Fictional client portal. Library version discovered in bundle; currently unsupported but no applicable vulnerability or exploitable path demonstrated. Other findings: missing headers and harmless error disclosure.

The security review of the client portal identified an outdated JavaScript dependency, missing recommended response security headers, and limited non-sensitive error-detail disclosure. None of these observations was demonstrated to provide a practical attack path during the assessment. The dependency issue has provisionally been classified as Low pending further review.

The installed library version was identified from the application bundle and is no longer maintained. However, no applicable vulnerability was validated in the way the application uses the component. The business risk is therefore uncertain: unsupported software may increase exposure to future or existing security issues, but the test does not establish data compromise or service disruption. Priority should follow a review of known advisories and actual usage.

We recommend identifying the supported replacement version, testing compatibility and planning a controlled update. Confirm the deployed version and relevant security fixes during retesting, while addressing the other observations through the maintenance backlog.

## Five-minute final review

- [ ] **Evidence:** Is each claim supported by the tests? Distinguish confirmed behaviour from possible impact.
- [ ] **Prerequisites:** Valid account, session, role, victim interaction and environment correctly stated?
- [ ] **Impact:** Specific business consequence stated; no automatic "reputational damage" filler?
- [ ] **Severity:** Justified with impact, exploitability, controls and uncertainties rather than the vulnerability name alone?
- [ ] **Mitigation:** Interim control achievable, narrowly scoped and compatible with business operations?
- [ ] **Remediation:** Root-cause fix (not only WAF, monitoring or hiding a UI button) + retesting?
- [ ] **Clarity:** English spelling `authorisation`, `prioritised`, `server-side`, `object-level`? No `allows ... can`, `do a retesting` or comma splices?
- [ ] **Safety:** Synthetic examples only in a public repo; no live evidence, tokens, secrets or client data.

**Useful British English micro-patterns**

| Avoid | Prefer |
| --- | --- |
| "We found out ..." | "The assessment identified ..." |
| "allows the user can access ..." | "allows the user to access ..." |
| "This will lead to financial loss" | "This could result in financial loss" |
| "The risk is High because it is public" | "The finding is rated High because [demonstrated impact + realistic exploitation conditions]" |
| "should prioritise first" | "should be prioritised" |
| "service-side" | "server-side" |
| "do a retesting" | "retest the affected functionality" |

**Companion file:** `02-detailed-finding-template.md` covers reproduction, expected vs actual result, HTTP evidence, screenshot captions, impact, risk and retesting.