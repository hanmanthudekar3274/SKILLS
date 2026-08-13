# Test Plan — AAA-2474: BearQ should be able to author "api" type steps just given a description

> **Status: DRAFT — approved by QA for test case writing. Not final until exit criteria are met.**
>
> **Approved:** 2026-08-13
> **Reporter:** Kyle Sheehan | **Assignee:** Amoli Rajgor | **Priority:** Major

---

## 1. Scope & Objectives

**In scope:**
- BearQ auto-detecting that a step description describes an API call and generating the backing metadata on the next run
- BearQ authoring API-type steps during test refinement when requirements call for it
- Persistence of generated API step metadata after a successful run
- Mixed UI + API step tests authored by BearQ

**Out of scope:**
- BearQ defaulting to API steps when UI steps are applicable
- Authentication/authorization handling within the API step payload itself (unless explicitly confirmed in scope)
- Performance of BearQ's API metadata generation at scale

**Objective:** Confirm that BearQ can detect API-oriented step descriptions and produce correct, stored `api`-type step metadata — during both manual step edits and during test authoring/refinement — without regressing existing UI-only test behavior.

---

## 2. Gaps & Questions for the Author

> These gaps were identified during requirement analysis. Items marked ❌ are missing from the ticket; ⚠️ are ambiguous. Scenarios that depend on unresolved gaps are noted in Section 3.

| # | Area | Finding | Question to Author |
|---|------|---------|-------------------|
| 1 | Detection criteria | ❌ Missing | What exactly makes a step description "clearly interpretable as an API request"? Is it keyword-based (e.g., GET/POST/PUT/DELETE, "endpoint", "request")? Is there a confidence threshold? |
| 2 | Supported HTTP methods | ❌ Missing | Which HTTP methods must be supported: GET, POST, PUT, DELETE, PATCH, HEAD, OPTIONS? |
| 3 | Backing metadata schema | ❌ Missing | What fields constitute the "backing metadata" for an API step? (URL, method, headers, body, expected status?) Is there a schema or data model to reference? |
| 4 | Failure handling | ❌ Missing | If the API call fails at runtime, is metadata still stored? Partially stored? Not stored at all? |
| 5 | Ambiguous descriptions | ❌ Missing | What should BearQ do if a step description could be either UI or API (e.g., "Log out of the app")? Fall back to UI? Ask the user? Skip metadata generation? |
| 6 | Authoring vs. detection code path | ⚠️ Ambiguous | During _refinement_ BearQ _authors_ API steps — is this a separate code path from runtime _detection_, or the same mechanism invoked earlier? |
| 7 | Preference trigger | ⚠️ Ambiguous | What is the exact condition under which BearQ opts to author an API step vs a UI step? Purely instruction-driven, or does BearQ also evaluate feasibility? |
| 8 | Roles & permissions | ❌ Missing | Can all user roles author/run API-type steps, or is there a permission gate? |
| 9 | Feature flag | ❌ Missing | Is this feature behind a flag? If so, what is the flag name and what is the default state? |
| 10 | Regression surface | ⚠️ Ambiguous | Are there existing tests or step types that could be incorrectly re-classified as API steps by this new detection logic? |
| 11 | UI feedback | ❌ Missing | Is there any UI indication to the user that BearQ has detected an API step and is generating metadata (e.g., loading state, label change)? |
| 12 | Reload requirement | ⚠️ Ambiguous | The example says "reload the page" to see the API-type step. Is this a known limitation or expected UX? Will a live update/websocket eventually cover this? |

---

## 3. Test Scenarios

> **Priority key:** P0 = must pass before release · P1 = important, cover before sign-off · P2 = lower risk, cover if time permits
> Scenarios that depend on unresolved gaps are annotated; they may need updating once gaps are closed.

| ID | Priority | Type | Scenario | Maps to |
|----|----------|------|----------|---------|
| TS-01 | P0 | Positive | Edit an existing step's description to a clear API call (e.g., "Make a POST request to `/api/logout` to end the session"). Save changes (script is cleared). Run the test. Verify BearQ generates API backing metadata and the step is stored as an api-type step after the run. | Use case 1 — editing a step |
| TS-02 | P0 | Positive | After TS-01 run completes, reload the page and verify the step is displayed as an api-type step with the correct HTTP method and endpoint stored in metadata. | Use case 1 — persistence |
| TS-03 | P0 | Positive | Author a new BearQ test with requirements that explicitly include an API call (e.g., "Make a PUT request to change the record"). After refinement, verify the generated test includes an api-type step with the correct HTTP method and URL. | Use case 2 — authoring a test |
| TS-04 | P0 | Positive | Mixed test: BearQ authors a test containing both UI steps and API steps (matching use case 2 example: login → create record → PUT request → verify → delete). Verify all UI steps remain ui-type and all API steps are api-type; run the test end-to-end successfully. | Use case 2 — mixed steps |
| TS-05 | P1 | Positive | Repeat TS-01 for each HTTP method described in natural language: GET, POST, PUT, DELETE, PATCH. Verify each is correctly classified. *(Blocked on Gap #2 — confirm method list.)* | Gap #2 |
| TS-06 | P1 | Positive | Refine an existing test where one step describes an API call. Verify BearQ authors it as an api-type step without altering adjacent UI steps. | Use case 2 — refinement path |
| TS-07 | P1 | Negative | BearQ should NOT auto-generate API metadata for a step that is clearly UI (e.g., "Click the logout button"). Run the test and verify the step remains ui-type with no API metadata added. | Gap #7 — not default behavior |
| TS-08 | P1 | Negative | Edit a step that previously had API backing metadata back to a UI description (e.g., change "POST to `/api/logout`" → "Click the logout button"). Run/refine. Verify step is re-classified as ui-type and prior API metadata is cleared or replaced. | State transition — reverse path |
| TS-09 | P1 | Negative | Provide a step description that could be interpreted as either UI or API (ambiguous, e.g., "Submit the form to create the record"). Verify BearQ's behavior matches the agreed spec. *(Blocked on Gap #5 — confirm expected behavior.)* | Gap #5 |
| TS-10 | P1 | Negative | Step description references an unreachable API endpoint during run. Verify the test fails gracefully and no partial or corrupt metadata is stored. *(Blocked on Gap #4 — confirm failure handling.)* | Gap #4 |
| TS-11 | P1 | Boundary | A test with 3+ consecutive API steps. Verify all api-type steps are generated and stored correctly with no metadata collision. | Boundary — multi-step |
| TS-12 | P1 | Boundary | Step description includes request body/payload details (e.g., `PUT /api/records with body {"firstName": "Jane"}`). Verify BearQ captures body in metadata. *(Blocked on Gap #3 — confirm schema.)* | Use case 2 example + Gap #3 |
| TS-13 | P2 | Positive | As a user with a non-admin role, verify API-type step authoring and detection behaves as expected (or is blocked per spec). *(Blocked on Gap #8 — confirm permissions.)* | Gap #8 |
| TS-14 | P2 | Negative | Run a test where an API step returns a non-2xx response. Verify no metadata is stored, or behavior matches spec. *(Blocked on Gap #4.)* | Gap #4 |
| TS-15 | P2 | Regression | Run a suite of existing UI-only tests. Verify none are incorrectly reclassified as API steps after this change is deployed. | Gap #10 — regression surface |
| TS-16 | P2 | Positive | If feature-flagged: verify the feature is off by default; enable the flag and verify it activates correctly. *(Blocked on Gap #9 — confirm flag name.)* | Gap #9 |

---

## 4. Test Data & Environment

**Data needed:**
- An existing test account with at least one saved test containing editable steps
- A reachable API endpoint (staging/sandbox) that accepts GET, POST, PUT, DELETE for API-step scenarios
- Test record data for creation/update/deletion (use case 2 example)
- An existing UI-only test suite for regression run (TS-15)

**Environment / flags:**
- Environment: Staging assumed — needs confirmation
- Feature flag name and default state: Unknown — see Gap #9
- BearQ must be enabled and accessible in the test environment

**Roles / permissions:**
- Admin/owner account for primary test runs
- At least one non-admin role to cover TS-13 (pending Gap #8 answer)

---

## 5. Risks & Assumptions

**Assumptions made:**
- "Backing metadata" means at minimum: HTTP method, URL, and optionally headers/body/expected status
- "Clearly interpretable as an API request" implies the description contains an explicit HTTP verb (GET/POST/PUT/etc.) and/or signal words like "endpoint" or "request"
- Metadata is stored at the step level, tied to the test
- The page-reload requirement in use case 1 is a current known UX limitation, not the intended final behavior
- All HTTP methods — GET, POST, PUT, DELETE, PATCH — are in scope

**Risks:**
- **Detection false-positives (high):** Existing UI steps mentioning words like "request", "submit", or "send" could be misclassified as API steps — primary regression risk
- **Undefined ACs:** The plan is derived entirely from two use-case examples; additional scenarios may emerge once formal ACs are written
- **Untestable scenarios:** Gaps #1, #3, #5, #6, and #7 must be resolved before several P0/P1 scenarios have a clear pass/fail definition
- **Latency risk:** If BearQ's metadata generation is synchronous, it may introduce noticeable delay on test run start

---

## 6. Entry / Exit Criteria

**Entry (minimum before test execution begins):**
- AAA-2474 is implemented and deployed to the test environment
- Gaps #1, #2, #3, and #6 are answered (required to execute P0 scenarios with a clear pass/fail)
- A reachable API sandbox endpoint is available for test runs
- Feature flag (if applicable) is confirmed and active in the test environment

**Exit (sign-off conditions):**
- All P0 scenarios pass without defects
- All P1 scenarios pass, or any failures are filed as defects and accepted/deferred with documented rationale
- No regressions found in existing UI-only tests (TS-15)
- Remaining open gaps either answered and re-tested, or formally accepted as out-of-scope for this release

---

## Appendix: Requirement Checklist (Gap Analysis)

| Category | Item | Status |
|----------|------|--------|
| Functional | Clear user story / goal | ✅ |
| Functional | Acceptance criteria are testable | ❌ None defined — use cases used as proxy |
| Functional | Happy path fully described | ✅ (two use cases) |
| Functional | Negative / error paths | ❌ Missing |
| Functional | Boundary & empty states | ❌ Missing |
| Functional | State transitions enumerated | ⚠️ Partially described |
| Data & environment | Required test data | ❌ Not specified |
| Data & environment | Environment / feature flags | ❌ Not named |
| Data & environment | External dependencies | ⚠️ API endpoint implied, not specified |
| Data & environment | Preconditions / setup | ⚠️ Partially described |
| Non-functional | Performance / load expectations | ❌ Missing |
| Non-functional | Security / authorization (roles) | ❌ Missing |
| Non-functional | Accessibility | ❌ Missing |
| Non-functional | i18n / l10n | ❌ Missing |
| Non-functional | Audit / logging / observability | ❌ Missing |
| Cross-cutting | Impact on existing features | ⚠️ Not assessed |
| Cross-cutting | Backward compatibility | ❌ Missing |
| Cross-cutting | Mobile / browser matrix | ❌ Missing |
| Cross-cutting | Rollback / feature-flag behavior | ❌ Missing |
| Clarity | No ambiguous wording | ⚠️ "clearly interpretable" is undefined |
| Clarity | Terms defined consistently | ⚠️ "backing metadata" not defined |
| Clarity | Mockups / designs linked | ❌ None linked |
