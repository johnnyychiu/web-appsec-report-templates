# Template B — Detailed Vulnerability Finding / Reproduction & Evidence

> **Purpose:** A reusable, UK-client-friendly finding format for an **authorised** Web/API hands-on exercise. Every worked example below is **fictional**. `example.test` paths, role names, identifiers and responses must be replaced with actual permitted test observations.
>
> **Public repository caution:** Do **not** push real client details, IP addresses, tokens, passwords, cookies, private endpoints, raw production records or unredacted screenshots. Evidence examples below are **descriptions and placeholders**, not real screenshots.

## Quick navigation

1. [When to use this](#when-to-use-this)
2. [Copy-and-fill detailed finding](#copy-and-fill-detailed-finding)
3. [Evidence and screenshot checklist](#evidence-and-screenshot-checklist)
4. [Worked findings by vulnerability](#worked-findings-by-vulnerability)
5. [Pass/fail quality gate](#passfail-quality-gate)

## When to use this

- The written test asks for **reproduction steps, PoC, detailed findings, or a technical report**.
- Create **one detailed finding per confirmed vulnerability**; label hypotheses/observations appropriately when impact isn't confirmed.
- For **Executive Summary / 100–150 words / findings overview**, use [`01-executive-summary-template.md`](01-executive-summary-template.md). Usually you write this **after** collecting detailed evidence.
- Follow the client's scope, testing rules, output format and scoring methodology ahead of this guide.

## Copy-and-fill detailed finding

### Finding [ID] — [SPECIFIC VULNERABILITY NAME]

| Field | Value |
| --- | --- |
| Severity | [HIGH / MEDIUM / LOW — based on evidence and client methodology] |
| Affected asset | [Authorised application, environment, API / component] |
| Endpoint / function | [HTTP METHOD and in-scope route, or UI workflow] |
| Prerequisite | [Role, valid account, victim browser, token possession, etc.] |
| Status | [Confirmed / Observed but not yet validated] |

**Description / root cause**

> The assessment identified **[VULNERABILITY]** in **[ENDPOINT / FUNCTION]**. The application **[MISSING OR DEFECTIVE CONTROL]**, allowing **[ATTACKER ROLE]** to **[SPECIFIC OBSERVED ACTION]**.

**Steps to reproduce (authorised test data only)**

1. **Precondition:** [Create / select test accounts, role assignments and synthetic records].
2. **Baseline:** [Capture a legitimate request and the expected normal response].
3. **Modification:** [Change exactly one relevant input, permission context or workflow step].
4. **Send:** [Submit the request using the stated session / browser and test environment].
5. **Verify:** [Check the relevant response **and actual state**, not just HTTP 200].
6. **Control:** [Compare with the legitimate role/account or a denied request, as relevant].

**Expected:** [The correct secure authorisation / validation / browser behaviour].  
**Actual:** [Exactly what the test demonstrated].

**Evidence / screenshot captions**

- **Figure 1 — Baseline:** [Request and expected normal behaviour; redact secrets].
- **Figure 2 — Controlled change:** [Changed parameter/token/role/input and relevant response].
- **Figure 3 — Impact validation:** [Data field, database state, authorised callback, browser proof or state transition demonstrating outcome].
- **Notes:** [Request IDs, test timestamps, environment, account roles, trace ID / case ID].

**Business impact**

> If exploited, this could result in **[SPECIFIC SECURITY/PROCESS CONSEQUENCE]**, potentially leading to **[DIRECT BUSINESS IMPACT]**. This is **not** evidence that a real breach or loss occurred.

**Likelihood / risk assessment**

> Exploitation requires **[PREREQUISITES]**. Testing established **[EVIDENCE]**, but **[LIMITATIONS / UNKNOWN CONTROLS]**. The finding is assessed as **[RATING]** because **[IMPACT + EXPLOITABILITY + EXISTING CONTROLS]**.

**Immediate mitigation (if feasible)**

> Consider **[NARROW, REVERSIBLE CONTROL]** after assessing business disruption and obtaining the client's approval. Do not rely solely on hiding UI buttons, monitoring or a WAF where the backend remains vulnerable.

**Permanent remediation**

> Implement **[ROOT-CAUSE SECURITY FIX]**. Apply **[DEFENCE-IN-DEPTH MEASURE]** as appropriate.

**Retesting / acceptance criteria**

> Repeat **[ORIGINAL CONTROLLED TEST]** to verify **[DENIED UNAUTHORISED RESULT]**. Also confirm **[LEGITIMATE USE CASE]** remains functional and no comparable routes are affected.

## Evidence and screenshot checklist

| Finding type | Best primary evidence | Screenshot needed? |
| --- | --- | --- |
| BOLA / BFLA / mass assignment | Original vs modified HTTP request **and** actual record/role/state result | Useful; a labelled request/response is often stronger |
| SQLi | Baseline and controlled differential response/trace | Capture only non-sensitive error/differences; never fabricate proof |
| Stored XSS / unsafe upload | Submitted input + rendered browser execution + context | **Yes, browser behaviour is especially useful** |
| Session invalidation | Same issued token before/after logout and timestamps | HTTP evidence usually sufficient |
| OTP limiting | Number/timing of failed attempts and observed absence/presence of controls | Screenshot optional; include clear attempt counts |
| CSRF / CORS | Real browser origin, cookies policy, result and state | Browser/network evidence highly useful |
| SSRF | In-scope request + server-owned callback/test-endpoint log | Callback/trace evidence useful; no internal secrets |
| Old dependency | Exact package name/version, source and applicable advisory | Screenshot optional; avoid claiming a confirmed exploit |

**Figure caption pattern:** `Figure 2 — A Customer Service test account submitted the approval request; the test refund state changed from pending to approved (sensitive headers redacted).`

**Avoid:** screenshots of a `200 OK` alone; production customer records; raw JWTs; copying external target addresses; unsupported conclusions; excessive screenshots that do not establish the security boundary.

## Worked findings by vulnerability

These are **worked writing examples**, not operational instructions to use outside an approved lab. All accounts, routes, amounts, markers and results are fictional.

**Jump straight to a detailed finding:** [BOLA / IDOR](#case-bola) · [BFLA](#case-bfla) · [SQL Injection](#case-sqli) · [Stored XSS](#case-stored-xss) · [SSRF](#case-ssrf) · [Session Token Not Invalidated on Logout](#case-session) · [Password-Reset OTP](#case-otp) · [Mass Assignment](#case-mass-assignment) · [CSRF](#case-csrf) · [Unsafe HTML File Upload Served as Active Content](#case-upload) · [Credentialed CORS Policy Exposes Profile JSON](#case-cors) · [Outdated JavaScript Library](#case-dependency)

<a id="case-bola"></a>

### BOLA / IDOR — Another Customer's Records

**Severity:** High (illustrative; confirm using client methodology)  
**Affected function:** `GET /api/customers/CUST-B`  
**Test identity:** Customer A with an ordinary authenticated account

**Description.** The customer API did not enforce object-level ownership when returning individual customer records. An authenticated user could request a record belonging to another test customer by changing its identifier.

**Steps to reproduce (synthetic authorised lab)**

1. Use authorised test Account A and Account B, each with a distinct synthetic customer record.
2. As A, capture a legitimate request for A's own record using Burp Repeater.
3. Without replacing A's session, change only the record identifier to B's test record ID.
4. Send the changed request and compare the body with B's known synthetic record.
5. Repeat with other relevant endpoints and methods within the approved scope to establish the boundary.

**Expected:** A receives only A's own record; requests for B's record are denied or safely not found.

**Actual:** A received B's test name, email address and postal address in the API response.

**Evidence / screenshots to attach:** Request with A's (redacted) session and B's identifier; matching synthetic response fields; ownership record showing the target belongs to B.

**Business impact:** Unauthorised customer-data disclosure could cause privacy harm and require investigation of potential data breaches.

**Risk reasoning and limits:** A valid customer account is required; no elevated privileges. The test did not establish bulk extraction or any malicious activity.

**Interim measure:** Restrict the affected record-retrieval route or affected data fields after assessing service impact.

**Permanent remediation:** Enforce object-level permissions using the authenticated principal and server-side ownership checks on every read and write, including nested resources.

**Retest / acceptance criteria:** A can access A's records; A cannot access B's records across relevant methods; B still accesses B's records.

---

<a id="case-bfla"></a>

### BFLA — Refund Approval Without Finance Role

**Severity:** High (illustrative)  
**Affected function:** `POST /api/refunds/TEST-2000/approve`  
**Test identity:** Customer Service test account, not a Finance Manager

**Description.** The refund approval endpoint relied on insufficient server-side role enforcement and allowed a lower-privileged user to perform a Finance Manager action.

**Steps to reproduce (synthetic authorised lab)**

1. In the approved test environment, create a £2,000 test refund in `pending` state.
2. Authenticate with a Customer Service role and capture its current session credentials.
3. Submit the refund approval API request using this lower-privileged session.
4. Read the refund state from the authorised test system and verify it changed to `approved`.
5. Confirm no real payout was made and preserve only synthetic identifiers and states.

**Expected:** Customer Service may submit requests but cannot approve them; only an authorised Finance Manager can approve.

**Actual:** The lower-privileged account caused the test refund state to change from `pending` to `approved`.

**Evidence / screenshots to attach:** Redacted role/account evidence, lower-privileged approval request, before-and-after refund state in the test system.

**Business impact:** If present in production, unauthorised approvals could lead to fraudulent refunds, direct financial losses and compromised financial controls.

**Risk reasoning and limits:** Requires valid Customer Service credentials. No real financial loss or wider exploitation was observed.

**Interim measure:** Restrict the approval endpoint to a reliably enforced authorised workflow, or disable API approvals and use a verified manual Finance Manager process.

**Permanent remediation:** Implement server-side function-level authorisation at the controller and business-service layer; enforce maker/checker separation.

**Retest / acceptance criteria:** Customer Service cannot approve even with a crafted request; Finance Manager can approve legitimately; audit events capture approvals.

---

<a id="case-sqli"></a>

### SQL Injection — Report Filter (Evidence-Limited)

**Severity:** High, provisional; validate safely  
**Affected function:** `GET /api/reports?sort=[VALUE]`  
**Test identity:** Authenticated reporting test user

**Description.** Controlled changes to a reporting parameter produced consistent SQL-related errors and differences between two complementary boolean test conditions. This supports a suspected injection weakness but not a demonstrated data extraction.

**Steps to reproduce (synthetic authorised lab)**

1. Capture a normal report request in the authorised lab and record its baseline body, status and timing.
2. Change only the target report-filter parameter using an approved, low-impact probe.
3. Repeat a pair of complementary test conditions while keeping session, route and other inputs constant.
4. Confirm the response differences are reproducible and not due to caching, filtering or unrelated errors.
5. Stop before attempting database extraction, record modification or disruptive queries unless specifically authorised.

**Expected:** Invalid input is handled safely; dynamic query structure cannot be changed by user-controlled data.

**Actual:** SQL-related errors and repeatable differences between controlled conditions were observed; database access scope was not established.

**Evidence / screenshots to attach:** Baseline and controlled requests/responses, timestamps, same account/session, redacted error details; no data dump.

**Business impact:** If the input controls query execution, possible consequences may include unauthorised data access or alteration depending on database privileges.

**Risk reasoning and limits:** Authentication was needed for the reporting endpoint; confidentiality/integrity impact remains unproven.

**Interim measure:** Limit access to the affected report route if a reliable narrow restriction is feasible; minimise database account privileges.

**Permanent remediation:** Use parameterised values; explicitly allowlist SQL identifiers such as `sort` fields and directions; remove unsafe query concatenation.

**Retest / acceptance criteria:** Repeat baseline and safe test conditions: malformed input is rejected safely, valid sorting still works, and no injection indicators remain.

---

<a id="case-stored-xss"></a>

### Stored XSS — Support Ticket Content

**Severity:** Medium (illustrative)  
**Affected function:** `POST /api/tickets` and support-ticket viewer  
**Test identity:** Customer author; support-agent viewer

**Description.** Customer-supplied ticket text was later inserted into a staff page without adequate context-aware output handling, permitting browser-side execution of a harmless test marker.

**Steps to reproduce (synthetic authorised lab)**

1. In the authorised lab, sign in as the customer test user and submit a ticket containing an agreed harmless proof-of-execution marker.
2. Open the saved ticket with a separate support-agent test account.
3. Verify whether the marker executes in the staff browser instead of being shown as inert text.
4. Record the exact stored field, page location and execution context without attempting session theft.
5. Check whether other rendering contexts or staff roles are affected, staying within scope.

**Expected:** The marker is displayed as text or safely sanitised; no arbitrary script runs in a staff page.

**Actual:** The harmless marker executed when the staff test user viewed the ticket.

**Evidence / screenshots to attach:** Submission field, stored ticket identifier, browser execution marker and relevant DOM/response context; sensitive tokens omitted.

**Business impact:** A malicious customer could potentially cause actions in a staff user's browser, with consequences depending on the staff account's permissions and browser protections.

**Risk reasoning and limits:** Requires customer content submission and viewing by another user; no token theft or account compromise was demonstrated.

**Interim measure:** Restrict unsupported rich HTML content or disable unsafe rendering pending a fix.

**Permanent remediation:** Use context-aware output encoding, safe templating, and an appropriate HTML sanitizer only where HTML is required; CSP as secondary defence.

**Retest / acceptance criteria:** Previously stored and newly submitted test markers remain inert for staff and customers; ordinary formatting still works.

---

<a id="case-ssrf"></a>

### SSRF — URL Preview Reaches Internal Test Service

**Severity:** Medium, provisional (illustrative)  
**Affected function:** `POST /api/url-preview`  
**Test identity:** Authenticated user supplying a URL

**Description.** The preview service followed a user-supplied URL to a destination outside the application's intended policy, demonstrating unexpected server-initiated access to an authorised internal test endpoint.

**Steps to reproduce (synthetic authorised lab)**

1. Capture a normal, permitted URL-preview request and response.
2. Within the authorised lab only, substitute an explicitly permitted internal test service URL.
3. Send the request and confirm the service reached that test endpoint using its response or owned request log.
4. Check whether redirects and destination resolution are consistently validated without touching real metadata or production internal hosts.
5. Record exact restrictions and stop without attempting sensitive internal discovery.

**Expected:** The backend rejects untrusted internal or otherwise disallowed destinations before making outbound requests.

**Actual:** An authenticated user induced the preview backend to request a controlled internal test service.

**Evidence / screenshots to attach:** Redacted request URL, application result, owned test-service access log with correlation marker; no sensitive endpoint content.

**Business impact:** Unexpected backend connectivity could expose internal services if additional reachable targets and permissions exist.

**Risk reasoning and limits:** Requires an authenticated account and URL-preview capability; sensitive data access, metadata retrieval and internal compromise were not demonstrated.

**Interim measure:** Disable preview for risky destinations or restrict feature access until outbound controls are deployed.

**Permanent remediation:** Adopt allowlisted destinations and outbound network policy, validate post-redirect and resolved IP destinations, and restrict service permissions.

**Retest / acceptance criteria:** Permitted external previews still work; blocked addresses and redirects to forbidden destinations cannot be reached.

---

<a id="case-session"></a>

### Session Token Not Invalidated on Logout

**Severity:** Medium, provisional (illustrative)  
**Affected function:** `POST /api/logout` and `GET /api/profile`  
**Test identity:** Authenticated user with previously issued token

**Description.** Logging out did not invalidate a previously issued access token, which remained accepted for the observed validity period.

**Steps to reproduce (synthetic authorised lab)**

1. Authenticate as a test user and capture an authorised API request with the issued token.
2. Confirm the request succeeds before logout and record the token's test expiry, without exposing it in the report.
3. Log out using the supported application workflow.
4. Replay the same API request with the previously issued token in the authorised test environment.
5. Record whether it remains valid and the approximate period; compare with a fresh login if needed.

**Expected:** Logout invalidates the session or otherwise prevents reuse of the revoked token as specified by the application security design.

**Actual:** A token used before logout continued to access the API for approximately 30 minutes afterwards.

**Evidence / screenshots to attach:** Pre-logout and post-logout requests using the same redacted token identifier; timestamps and endpoint results.

**Business impact:** An unauthorised holder of an already-issued token could retain API access after the user believes they have logged out.

**Risk reasoning and limits:** Token possession is required; no theft or compromise was found, and expiration behaviour limits exposure.

**Interim measure:** Consider shorter access-token lifetimes and monitoring while evaluating operational implications; not a substitute for revocation.

**Permanent remediation:** Use reliable server-side session invalidation or revocation with well-defined lifecycle across all API entry points.

**Retest / acceptance criteria:** Old tokens are rejected after logout, new tokens work after login, other sessions behave according to documented policy.

---

<a id="case-otp"></a>

### Password-Reset OTP — No Effective Attempt Limiting

**Severity:** Medium, provisional (illustrative)  
**Affected function:** `POST /api/password-reset/verify`  
**Test identity:** Unauthenticated password-reset workflow for a test account

**Description.** The endpoint continued processing repeated incorrect OTP guesses without an observed cooldown or per-reset attempt limit.

**Steps to reproduce (synthetic authorised lab)**

1. Start a reset only for a permitted test account and record its test reset transaction identifier.
2. Submit a small, agreed number of incorrect OTP values, observing status codes and any lockout/cooldown messages.
3. If permitted, continue to the approved test threshold and record how many attempts are processed.
4. Verify expiry and per-account/per-transaction protections without attempting any real user's OTP.
5. Do not claim account takeover unless the reset was actually authorised and proven in the controlled test.

**Expected:** An appropriate threshold or rate control blocks repeated guesses; expiry limits the lifetime of reset codes.

**Actual:** Fifty incorrect OTP submissions were processed without a cooldown; no OTP was guessed.

**Evidence / screenshots to attach:** Series of redacted requests with sequence numbers and timestamps; lack of lockout response; no real reset token published.

**Business impact:** Depending on OTP entropy, expiry and other controls, an attacker might have more opportunities to guess valid codes and reset accounts.

**Risk reasoning and limits:** A valid reset flow is required; probability and real-world impact remain uncertain; no account takeover demonstrated.

**Interim measure:** Temporarily throttle the endpoint with carefully tuned application/gateway controls, monitoring false positives.

**Permanent remediation:** Rate-limit by account and reset transaction, cap attempts, expire OTPs promptly and monitor abuse; avoid reliance solely on IP-based thresholds.

**Retest / acceptance criteria:** Repeated wrong attempts trigger controls, codes expire, and genuine users retain a viable recovery path.

---

<a id="case-mass-assignment"></a>

### Mass Assignment — Subscription Tier Change

**Severity:** Medium (illustrative)  
**Affected function:** `PATCH /api/profile`  
**Test identity:** Authenticated trial customer

**Description.** The profile API bound a client-supplied property that should have been controlled exclusively by a trusted subscription/billing workflow.

**Steps to reproduce (synthetic authorised lab)**

1. Log in as an authorised trial customer and capture a normal profile update.
2. Inspect the API shape against the documented public editable fields.
3. Add the synthetic test property `planType` with a value reserved for an upgraded plan.
4. Submit the update under the ordinary user's session, then query the account's actual entitlement state.
5. Confirm whether the change persists and whether it affects test-only premium functionality.

**Expected:** Only approved profile fields are writable; plan/entitlement changes are rejected outside an authorised billing workflow.

**Actual:** The trial account's plan entitlement changed to premium without completing the normal upgrade process.

**Evidence / screenshots to attach:** Normal vs modified JSON body, unchanged user context, prior/current entitlement response; test account only.

**Business impact:** Unauthorised access to paid features could result in revenue leakage and weakened subscription controls.

**Risk reasoning and limits:** Requires a standard customer session; no real billing change or staff access demonstrated.

**Interim measure:** Reject the affected property at the gateway/service boundary if this can be enforced reliably.

**Permanent remediation:** Use explicit input DTO allowlists, server-side entitlement checks and trusted subscription-change workflows.

**Retest / acceptance criteria:** Untrusted `planType` updates are rejected; legitimate profile edits and authorised upgrades work.

---

<a id="case-csrf"></a>

### CSRF — Notification Email Updated Cross-Site

**Severity:** Medium (illustrative)  
**Affected function:** `POST /account/notification-email`  
**Test identity:** Signed-in victim test user induced by separate test origin

**Description.** The state-changing account route accepted a cross-site browser request without an effective anti-CSRF control.

**Steps to reproduce (synthetic authorised lab)**

1. Record the baseline notification email for an authorised signed-in test account.
2. Use an owned, separate test origin to initiate a browser-based submission to the account-change route.
3. Keep the victim account logged in and perform only the in-scope, benign email change.
4. Confirm whether the browser sends the needed credentials and whether the server accepts the state change.
5. Check the new value in the account interface or response and record cookie/SameSite behaviour.

**Expected:** The application rejects unauthorised cross-site changes; a valid CSRF mechanism or equivalent protects the request.

**Actual:** A cross-site browser request changed the signed-in test account's notification email without further confirmation.

**Evidence / screenshots to attach:** Browser origin, request with sensitive headers redacted, before-and-after account setting; cookie conditions documented.

**Business impact:** An attacker might alter notification routing or trigger unwanted settings changes if a victim visits malicious content while signed in.

**Risk reasoning and limits:** Requires an authenticated victim browser and navigation/interaction. No account takeover demonstrated.

**Interim measure:** Require re-authentication or manual confirmation for the affected action, subject to service feasibility.

**Permanent remediation:** Use robust anti-CSRF tokens and appropriate SameSite cookie policy; confirm sensitive changes server-side.

**Retest / acceptance criteria:** Cross-site requests fail; valid same-origin updates and legitimate workflows still succeed.

---

<a id="case-upload"></a>

### Unsafe HTML File Upload Served as Active Content

**Severity:** Medium (illustrative)  
**Affected function:** `POST /api/tickets/attachments` and attachment download URL  
**Test identity:** Customer uploader; staff reviewer

**Description.** Uploaded HTML was served from the application's origin with content handling that allowed active browser execution when a staff test user opened the attachment.

**Steps to reproduce (synthetic authorised lab)**

1. Authenticate as an authorised customer test user and upload a harmless HTML test attachment through the supported workflow.
2. Capture the upload response, reference URL, declared and detected content types.
3. Open the resulting attachment as a separate staff test user in the controlled lab.
4. Check whether it downloads as an attachment or executes as active HTML under the application origin.
5. Record browser origin and execution indicator; do not attempt session theft or server-side execution.

**Expected:** Untrusted HTML is rejected or downloaded safely, and cannot execute as the application's trusted origin.

**Actual:** The test attachment executed a harmless browser marker from the application's own origin.

**Evidence / screenshots to attach:** Upload request and content type; response link; staff browser origin and harmless marker; real secrets excluded.

**Business impact:** Active content might be used against staff browser sessions if staff open untrusted attachments.

**Risk reasoning and limits:** Requires uploading as a customer and staff opening the attachment; no server-side RCE or data theft confirmed.

**Interim measure:** Block risky content types or force safe download/isolated hosting until storage policy is corrected.

**Permanent remediation:** Allowlist required file formats, verify content types, store on an isolated origin and use safe `Content-Disposition`/MIME handling.

**Retest / acceptance criteria:** Previously risky files are rejected or inert; approved document types remain accessible in intended workflows.

---

<a id="case-cors"></a>

### Credentialed CORS Policy Exposes Profile JSON

**Severity:** High, provisional (illustrative)  
**Affected function:** `GET /api/profile`  
**Test identity:** Authenticated victim test browser visiting separate test origin

**Description.** The API allowed a cross-origin browser script on an untrusted test origin to read the authenticated profile response with credentials.

**Steps to reproduce (synthetic authorised lab)**

1. Sign in to the customer test portal in a controlled browser, and capture the normal profile request.
2. From a separately hosted, authorised test origin, send a credentialed browser request to the profile endpoint.
3. Inspect `Access-Control-Allow-Origin` and `Access-Control-Allow-Credentials` as observed, and confirm the browser permits reading the body.
4. Record only synthetic test profile fields and whether the origin/credentials prerequisites are met.
5. Check whether the behaviour extends to other sensitive routes without collecting unrelated data.

**Expected:** Untrusted origins cannot read credentialed profile responses; only explicitly trusted application origins are allowed.

**Actual:** A controlled script from the test origin read synthetic profile JSON from the signed-in test account.

**Evidence / screenshots to attach:** Test origin, relevant response headers, browser script result, redacted synthetic profile response; not just headers alone.

**Business impact:** A malicious web origin could potentially read private API responses from signed-in users visiting that origin.

**Risk reasoning and limits:** Requires an authenticated browser and malicious-origin visit; affected endpoints and data scope remain partly unknown.

**Interim measure:** Restrict credentialed cross-origin reads on affected sensitive routes while checking legitimate integrations.

**Permanent remediation:** Allowlist explicit trusted origins, set credential policy narrowly and audit shared CORS middleware/endpoint configuration.

**Retest / acceptance criteria:** Trusted app origins still function; untrusted origins cannot read credentialed profile data in a real browser.

---

<a id="case-dependency"></a>

### Outdated JavaScript Library — No Confirmed Exploit

**Severity:** Low, provisional / maintenance observation  
**Affected function:** Client-side library version in web bundle  
**Test identity:** No attacker exploit demonstrated

**Description.** A dependency version was identified that is no longer maintained, but the assessment did not establish an applicable vulnerability in its actual usage.

**Steps to reproduce (synthetic authorised lab)**

1. Identify library name and exact version from an authorised build manifest or client-side bundle.
2. Confirm whether the version is supported and collect the exact relevant vendor/security advisory if one exists.
3. Determine whether the application uses affected functionality, without inferring exploitation from the version alone.
4. Record the dependency location and coverage limits; do not fabricate an exploit proof.
5. If a fix is applied, test application compatibility and inspect the deployed build.

**Expected:** Dependencies are maintained and risk-reviewed, with documented updates when necessary.

**Actual:** An unsupported library version was present; no exploitable application behaviour was demonstrated.

**Evidence / screenshots to attach:** Exact library/package name and version, manifest or bundle reference, applicable advisory and affected function if verified.

**Business impact:** Unsupported software can increase exposure to future or applicable known weaknesses; current exploitability was not established.

**Risk reasoning and limits:** Impact depends on an applicable vulnerability, runtime usage and mitigating controls.

**Interim measure:** Track the dependency review and assess exposure; avoid claiming that a WAF alone resolves a code-level issue.

**Permanent remediation:** Upgrade to a maintained, compatible patched version and establish dependency update monitoring.

**Retest / acceptance criteria:** The deployed version is updated; relevant advisories are addressed and regression checks pass.

---

## Pass/fail quality gate

- [ ] Title describes the actual failure, not a generic "insecure API" label.
- [ ] Scope, account role, prerequisite, method and synthetic test record are unambiguous.
- [ ] Steps can be repeated by a second assessor with a valid test setup.
- [ ] Expected and actual behaviours are distinct; result is verified beyond a status code.
- [ ] Evidence is sufficient, numbered and redacted; screenshot captions explain what each proves.
- [ ] "Confirmed" is reserved for observed behaviour; conditional consequences use "could" / "may".
- [ ] Risk rating is justified using prerequisites, exploitability, impact, other controls and unknowns.
- [ ] A narrow interim measure is distinguished from a root-cause fix and is operationally reviewed.
- [ ] Retest covers both the blocked unauthorised action and legitimate functionality.
- [ ] No confidential client material, production credentials, genuine vulnerabilities or personal data is in this public repository.

**British-English quick fixes:** `server-side`, `object-level authorisation`, `function-level authorisation`, `should be prioritised`, `arrange retesting`, `could result in`, `testing confirmed`, `requires a valid account`.

**Companion:** [`01-executive-summary-template.md`](01-executive-summary-template.md) for the management overview and 12 fully written example executive summaries.