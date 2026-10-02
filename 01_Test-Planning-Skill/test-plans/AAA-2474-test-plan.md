# Test Plan — AAA-2474: BearQ should be able to author "api" type steps just given a description

> Status: **DRAFT — pending human review.** Not approved until a QA owner signs off.
>
> **Created:** 2026-08-13 | **Priority:** Major | **Type:** Task
> **Project:** Architecture and AI (AAA) | **Status:** To Do (tracking ticket)

---

## Context: Ticket Scope

AAA-2474 is a **tracking ticket** for the BearQ API step authoring feature. Implementation was split into two completed child tickets and two future ones:

| Ticket | Summary | Status |
|--------|---------|--------|
| AAA-2679 | Description-driven API step authoring for API-mode tests | Done |
| AAA-2680 | Enable API step authoring via edit step manually | Done |
| AAA-2876 | Author a test case with API steps on requirement update | To Do |
| AAA-2877 | Refine test steps should author a test case with API steps | To Do |

**This test plan covers what is already implemented (AAA-2679 + AAA-2680).** Separate test plans should be written for AAA-2876 and AAA-2877 when they are ready.

---

## 1. Scope & Objectives

**In scope:**
- Description-driven API step generation: editing a step description to an API-like description clears the old script; on the next run, BearQ resolves it using LLM and generates backing metadata; on success, writes back to DB
- BearQ authoring API-type steps during test creation when requirements explicitly call for an API call
- Mixed UI + API step tests (some steps UI, some API in the same test)
- Three LLM endpoint resolution outcomes: **API + resolved**, **Not API** (fall back to UI path), **Unresolved** (surfaced to user clearly, not silent failure)
- Script lifecycle: description change → script cleared; edited script wins (no regeneration); write-back only on successful run
- endpointId is optional — script stands alone with `method / url / headers / body / statusCodes`; `responseSchemaRef` only when `endpointId` resolves
- API step auth stays with the runner (liveness session + workspace auth script)
- Regression: existing UI-only tests must not be reclassified

**Out of scope:**
- Real-time auto-generation when user edits description (design decision: LLM is invoked only within a task/run session)
- API Flow Test generation (AAA-2409)
- API step authoring via requirement update (AAA-2876 — To Do)
- Refine test steps authoring API steps (AAA-2877 — To Do)
- Onboarding video update (AAA-2871)
- Auth mechanism design (covered in runner; not a test-plan concern here)

**Objective:** Confirm BearQ correctly detects API-intent from step descriptions, generates valid `APIStepContext` backing metadata during run, writes it back on success, handles all three LLM resolution outcomes explicitly, and does not regress existing UI-only tests.

---

## 2. Gaps & Questions for the Author

> ✅ present · ⚠️ ambiguous · ❌ missing. Every ⚠️/❌ is a potential blocker for a scenario's pass/fail definition.

| # | Area | Finding | Question |
|---|------|---------|----------|
| 1 | Formal ACs | ❌ Missing | No acceptance criteria defined in the ticket. All scenarios in this plan are derived from the two use-case examples and design decisions in comments. Please confirm these are complete and accurate, or add/correct them. |
| 2 | "Clearly interpretable as an API request" | ⚠️ Ambiguous | What is the minimum signal needed for a description to be classified as API? (e.g., explicit HTTP verb? URL pattern? or does the LLM decide freely?) What is the expectation for a description like "Submit the form to create a record" — UI or API? |
| 3 | Unresolved outcome UX | ⚠️ Ambiguous | When LLM returns "Unresolved," what exactly does the user see? An error in the test run log? A step-level error? A specific message text? "Convey clearly" is not defined. |
| 4 | Not-API fallback | ⚠️ Ambiguous | When LLM returns "Not API," does the step silently fall back to the UI runner, or is any indication shown to the user that a fallback occurred? |
| 5 | Write-back failure handling | ❌ Missing | If the API step executes successfully but the DB write-back fails, what happens? Step shown as API-type or UI-type on reload? Is the error surfaced? |
| 6 | Description with full endpoint details (non-DB) | ⚠️ Ambiguous | When a description contains a fully specified endpoint not in the DB (e.g., "POST to example.com/v1/tasks with body {…}"), the offline discussion confirmed this should resolve. What is the expected raw data model shape — specifically, is `statusCodes` always `[200]` as default, or derived from description? |
| 7 | Edited script wins — validation errors | ⚠️ Ambiguous | AAA-2680 introduces a script validator. What is the exact behavior when the user saves an invalid script (bad method, malformed URL, unknown endpoint after URL-method match fails)? Is the save blocked, or saved with a warning state? |
| 8 | Headers editability | ⚠️ Ambiguous | The comment thread flagged headers as potentially editable. The current spec marks them as derived/read-only. Is the current state: headers are NOT editable by the user, and only injected by the runner? |
| 9 | Feature flag | ❌ Missing | Is this feature behind a flag? If yes, what is the flag name and what is the default state in staging? |
| 10 | Staging environment | ❌ Missing | Which environment should testing happen in? Does staging have the LLM endpoint resolution service active and accessible? |
| 11 | Test data / known endpoints | ❌ Missing | What pre-existing endpoints must be discoverable in the test account's workspace DB for resolution tests to pass? Is there a seed script or a reference test account? |
| 12 | Performance / LLM latency | ❌ Missing | Is there an acceptable latency SLA for the LLM resolution step within a run? If resolution takes >X seconds, is that a bug? |

---

## 3. Test Scenarios

> **Priority key:** P0 = must pass before release · P1 = important, cover before sign-off · P2 = lower risk / cover if time permits
> Scenarios marked *(gap #N)* depend on the corresponding gap being resolved for a clear pass/fail.

### Path A — Edit step description → API step generated on run (AAA-2679)

| ID | Priority | Type | Scenario | Maps to |
|----|----------|------|----------|---------|
| TS-01 | P0 | Positive | Edit an existing test step's description to a clear API call (e.g., "Make a POST request to the logout endpoint to end the session"). Save — verify old script is cleared. Run the test. Verify BearQ generates API backing metadata (`method`, `url`, `statusCodes`, `headers`) and step succeeds. | Use case 1 (editing a step) |
| TS-02 | P0 | Positive | After TS-01 completes successfully, reload the page. Verify the step is displayed as an `api`-type step with correct method and URL in the stored metadata. | Use case 1 — write-back |
| TS-03 | P0 | Negative | Edit a step description to one that is clearly a UI action (e.g., "Click the logout button"). Run. Verify LLM returns "Not API," step runs as UI-type, and no API metadata is generated or stored. | Gap #4 — Not API fallback |
| TS-04 | P0 | Negative | Edit a step description to an ambiguous or unresolvable API description (e.g., "Call the internal secrets endpoint"). Run. Verify the "Unresolved" outcome is surfaced visibly to the user and the step does not fail silently. | Gap #3 — Unresolved UX |
| TS-05 | P1 | Positive | Edit a step description to include a fully specified endpoint not in the DB (e.g., "Send a POST to example.com/v1/tasks with body `{name: 'test'}`"). Run. Verify API step is generated with `method=POST`, correct `url`, `body` from description, and no `endpointId`. | Gap #6 — non-DB endpoint |
| TS-06 | P1 | Positive | Edit a step that previously had valid API backing metadata back to a different API description. Verify old script is cleared on save and new API metadata is generated on the next run. | Script lifecycle |
| TS-07 | P1 | Negative | Edit a step that previously had API backing metadata to a UI-action description. Run. Verify old script is cleared and step falls back to UI runner (not stored as API-type). | Script lifecycle — reverse |
| TS-08 | P1 | Boundary | Edit a step description that includes HTTP verb and URL but with an invalid/unreachable endpoint (404 at runtime). Verify step generates the script but fails the run, and no write-back occurs (step NOT stored as API-type after a failed run). | Gap #5 — write-back on failure |

### Path B — BearQ authoring API steps during test creation (AAA-2679)

| ID | Priority | Type | Scenario | Maps to |
|----|----------|------|----------|---------|
| TS-09 | P0 | Positive | Create a new BearQ test with requirements that explicitly include an API call (e.g., "Make a PUT request to change the record"). After creation/refinement, verify the generated test includes an `api`-type step with correct `method` and `url`. | Use case 2 (authoring a test) |
| TS-10 | P0 | Positive | Mixed test: BearQ authors a test with both UI and API steps (login → create record → PUT request → verify → delete). Run it end-to-end. Verify UI steps run as UI-type and API step runs as API-type; all steps pass. | Use case 2 — mixed |
| TS-11 | P1 | Negative | Generate a BearQ test with requirements that are entirely UI actions. Verify BearQ does NOT author any API-type steps (no API metadata generated for UI steps). | Requirement: not default behavior |

### Path C — Script editing and validation (AAA-2680)

| ID | Priority | Type | Scenario | Maps to |
|----|----------|------|----------|---------|
| TS-12 | P0 | Positive | On an existing API-type step, manually edit the raw data model (e.g., change `method` from GET to POST, update `url`). Save. Run. Verify the edited script is used as-is (no LLM regeneration) and step executes with the updated values. | Gap #7 — edited script wins |
| TS-13 | P1 | Negative | Manually enter an invalid method (e.g., `FETCH`) in the raw data model editor. Attempt to save. Verify a validation error is returned and save is blocked (or warning is shown per spec). | Gap #7 — validation |
| TS-14 | P1 | Negative | Manually enter a malformed URL in the raw data model editor. Attempt to save. Verify a validation error is surfaced. | Gap #7 — validation |
| TS-15 | P1 | Positive | Add a new status code that has no response schema in the DB. Save. Run. Verify a warning is shown (reduced coverage) but save is allowed and run proceeds. | Gap #7 — warning path |

### Path D — Regression

| ID | Priority | Type | Scenario | Maps to |
|----|----------|------|----------|---------|
| TS-16 | P0 | Regression | Run a suite of existing UI-only tests. Verify none are reclassified as API-type steps after this feature is deployed. No unexpected API metadata is generated or stored. | Design assumption — UI steps unchanged |
| TS-17 | P1 | Regression | Run an existing API endpoint test (pre-feature). Verify it continues to function as before with no change in behavior or metadata. | Backward compat |

---

## 4. Test Data & Environment

**Test data needed:**

| # | Data | Purpose |
|---|------|---------|
| TD-01 | Existing test with at least one editable UI-type step | TS-01 through TS-08 |
| TD-02 | A known endpoint in the workspace DB (method + URL + schema) | TS-01, TS-05 resolution tests |
| TD-03 | A non-DB endpoint description with full method/URL/body in text | TS-05 |
| TD-04 | Requirements text that explicitly includes an API call (PUT/POST + URL) | TS-09, TS-10 |
| TD-05 | Requirements text that is entirely UI actions | TS-11 |
| TD-06 | An existing API-type step with stored raw data model | TS-12 through TS-15 |
| TD-07 | Existing UI-only test suite for regression | TS-16 |
| TD-08 | Existing API endpoint test | TS-17 |

**Environment / flags:**
- Target environment: Not specified — confirm staging *(Gap #10)*
- LLM resolution service: Must be active and accessible in the test environment *(Gap #10)*
- Feature flag name and default state: Unknown *(Gap #9)*

**Roles / permissions:**
- Standard user with access to BearQ and the workspace (all scenarios)
- No role-specific behavior specified in the ticket — confirm if any permission gates apply

---

## 5. Risks & Assumptions

**Assumptions made:**
- LLM is invoked at run time only (Option 1 confirmed in comment discussion) — not when user saves description.
- "Script cleared on description change" means the backing script is set to `null`/empty in the DB immediately on save, before any run.
- Write-back to DB happens only after a successful run; a failed step does NOT persist API metadata.
- UI steps are completely unchanged by this feature — the per-step routing decision is made only for steps with no backing script.
- `endpointId` is optional; absence does not block a step from running.
- `headers` are currently injected by the runner and not user-editable (per latest comment thread resolution).
- AAA-2679 and AAA-2680 are the only in-scope child tickets; AAA-2876 and AAA-2877 are explicitly excluded.

**Risks:**
- **False-positive API classification (high):** Steps with words like "request," "send," or "submit" in a UI context could be misclassified as API steps, regressing existing tests. TS-16 covers this but the classification threshold (Gap #2) must be defined.
- **Silent failure (medium):** If the "Unresolved" outcome is not surfaced clearly (Gap #3), failing API steps look like passing tests with no error feedback.
- **Write-back failure undetected (medium):** If DB write-back fails after a successful run (Gap #5), the step remains UI-type visually but the run result shows success — a misleading inconsistency.
- **LLM non-determinism (medium):** The same description might yield different resolution outcomes across runs. Scenario reproducibility may require LLM output to be seeded or mocked in testing.
- **Staging LLM unavailability (medium):** If the LLM endpoint is not active in staging, all Path A/B scenarios are untestable.
- **No formal ACs (high):** All scenarios are derived from use cases and comment thread discussions. If the implementation diverges from those discussions, the plan may need significant revision.

---

## 6. Entry / Exit Criteria

**Entry (minimum before test execution begins):**
- AAA-2679 and AAA-2680 are deployed to the test environment
- LLM resolution service is active and reachable in the test environment
- At least one test account exists with pre-discoverable endpoints in its workspace DB (TD-02)
- Gaps #2 (classification signal) and #3 (Unresolved UX) are answered — required to define pass/fail for TS-03 and TS-04
- Feature flag confirmed active (if applicable, Gap #9)

**Exit (sign-off conditions):**
- All P0 scenarios (TS-01, TS-02, TS-03, TS-04, TS-09, TS-10, TS-12, TS-16) pass without defects
- All P1 scenarios pass, or failures are filed as defects and accepted/deferred with documented rationale
- No regressions in existing UI-only test suite (TS-16 passes)
- All remaining open gaps either resolved and re-tested, or formally accepted as out-of-scope for this release

---

## HUMAN REVIEW GATE

- **I assumed:** LLM is only invoked within a run task (not real-time); write-back only on success; `endpointId` is optional; `headers` are runner-injected and not editable; this plan covers AAA-2679 + AAA-2680 only; scenarios derived from use-case examples and comment-thread design decisions since no formal ACs exist.
- **I could not confirm:** Classification threshold for "API intent" (Gap #2), exact Unresolved error UX (Gap #3), Not-API fallback visibility (Gap #4), write-back failure behavior (Gap #5), non-DB endpoint expected output shape (Gap #6), script validation behavior — block vs. warn (Gap #7), headers editability (Gap #8), feature flag name (Gap #9), staging environment with active LLM (Gap #10), test data seed process (Gap #11), latency SLA (Gap #12).
- **Open questions blocking P0 sign-off:** Gap #2 (what counts as API intent) and Gap #3 (Unresolved outcome UX) must be answered before TS-03 and TS-04 have an unambiguous pass/fail definition.

▶ **Approve, or edit, before I write test cases / automation.**
