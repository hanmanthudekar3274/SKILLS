# Test Plan — LOGIN: Login Page Functional Requirements

> Status: **APPROVED — ready for test case writing and automation.**
>
> **Created:** 2026-08-13 | **Approved:** 2026-08-13
> **Priority:** Critical

---

## 1. Scope & Objectives

**In scope:**
- Login page UI elements: Email field, Password field, Sign In button, Forgot Password link
- Successful login with valid credentials (AC-01)
- Error handling for invalid credentials, unregistered email, invalid email format (AC-02)
- Validation for empty required fields (AC-03)
- Password masking behavior (AC-04)
- Forgot Password flow initiation (AC-05)
- Boundary conditions: field length limits, whitespace handling, special characters

**Out of scope:**
- Post-login application functionality and navigation
- Password recovery flow itself (only the initiation is in scope per AC-05)
- Account creation / registration
- Session management, token expiry, logout
- OAuth / SSO / third-party sign-in (not mentioned in requirements)

**Objective:** Confirm the login page renders correctly, authenticates valid users, rejects invalid credentials with appropriate messages, masks passwords, and routes users to password recovery — covering all five ACs and key edge cases.

---

## 2. Gaps & Questions for the Author

> ✅ present · ⚠️ ambiguous · ❌ missing. Every ⚠️/❌ is a question blocking sign-off.

| # | Area | Finding | Question to Author |
|---|------|---------|--------------------|
| 1 | Error messages | ⚠️ Ambiguous | What is the exact error message text for each failure case? (Invalid credentials, unregistered email, invalid format, empty fields.) Should the error distinguish "wrong password" from "email not found", or show a generic "Invalid email or password" for security? |
| 2 | Email format validation | ❌ Missing | Is format validation client-side (on blur/submit) or server-side only? What formats are considered invalid — is `user@domain` without TLD valid? |
| 3 | Field character limits | ❌ Missing | What are the max character lengths for email and password fields? Is there a minimum password length enforced at login? |
| 4 | Whitespace handling | ❌ Missing | Should leading/trailing whitespace in the email field be trimmed automatically? What about whitespace in the password field? |
| 5 | "Remember me" / session persistence | ❌ Missing | Is there a "Remember Me" checkbox or session persistence option? If so, it is not in the requirements. |
| 6 | Rate limiting / account lockout | ❌ Missing | Is there a lockout policy after N failed login attempts? If so, what is the threshold and lockout duration, and what message is shown? |
| 7 | Forgot Password destination | ⚠️ Ambiguous | AC-05 says "password recovery page/flow is opened" — is this a new page, a modal, or a redirect? Is there a specific URL or component name? |
| 8 | Accessibility (a11y) | ❌ Missing | Are there ARIA labels, keyboard navigation requirements, or screen-reader expectations for the login form? |
| 9 | Browser / device matrix | ❌ Missing | Which browsers and OS/device combinations must be supported? Mobile-responsive layout required? |
| 10 | Security requirements | ❌ Missing | Is the login form required to be served over HTTPS only? Are there CSP, autocomplete="off", or other security header requirements on the form? |
| 11 | Performance expectation | ❌ Missing | Is there a maximum response time SLA for the login action (e.g., must respond within 3 s under normal load)? |
| 12 | Post-login redirect | ⚠️ Ambiguous | Where is the user redirected after a successful login? Is there a deep-link / return-URL parameter to honor? |

---

## 3. Test Scenarios

> **Priority key:** P0 = must pass before release · P1 = important, cover before sign-off · P2 = lower risk / cover if time permits
> Scenarios marked *(gap)* may need updating once the corresponding gap is resolved.

| ID | Priority | Type | Scenario | Maps to |
|----|----------|------|----------|---------|
| TS-01 | P0 | Positive | Navigate to the login page. Verify Email field, Password field, Sign In button, and Forgot Password link are all visible and rendered correctly. | FR — UI elements |
| TS-02 | P0 | Positive | Enter a valid registered email and valid password. Click Sign In. Verify user is successfully logged in and redirected to the expected destination. | AC-01 |
| TS-03 | P0 | Negative | Enter a valid registered email and an incorrect password. Click Sign In. Verify an appropriate error message is displayed and the user is not logged in. | AC-02, Test data row 2 |
| TS-04 | P0 | Negative | Enter an unregistered email and any password. Click Sign In. Verify an appropriate error message is displayed. | AC-02, Test data row 3 |
| TS-05 | P0 | Negative | Enter a string that is not a valid email format (e.g., `notanemail`). Click Sign In. Verify an appropriate format validation error is displayed. | AC-02, Test data row 4 |
| TS-06 | P0 | Negative | Leave both Email and Password fields empty. Click Sign In. Verify validation errors are shown for both required fields. | AC-03, Test data row 7 |
| TS-07 | P0 | Negative | Leave only the Email field empty, enter a password. Click Sign In. Verify a validation error is shown for the Email field only. | AC-03, Test data row 5 |
| TS-08 | P0 | Negative | Enter an email, leave the Password field empty. Click Sign In. Verify a validation error is shown for the Password field only. | AC-03, Test data row 6 |
| TS-09 | P0 | Positive | Click in the Password field and type any characters. Verify each character is masked (displayed as • or *) and not readable as plain text. | AC-04 |
| TS-10 | P0 | Positive | Click the Forgot Password link. Verify the user is taken to the password recovery page/flow (new page or modal). | AC-05 |
| TS-11 | P1 | Boundary | Enter an email with leading/trailing whitespace (e.g., ` user@example.com `). Click Sign In with valid password. Verify behavior — trimmed and logged in, or error shown. *(Gap #4)* | Gap #4 |
| TS-12 | P1 | Boundary | Enter an email at the maximum allowed character length. Verify the field accepts it and login proceeds normally. *(Gap #3)* | Gap #3 |
| TS-13 | P1 | Boundary | Enter a password at the maximum allowed character length. Verify the field accepts it and login proceeds normally. *(Gap #3)* | Gap #3 |
| TS-14 | P1 | Boundary | Attempt to enter more characters than the allowed max in each field. Verify input is truncated or rejected. *(Gap #3)* | Gap #3 |
| TS-15 | P1 | Negative | Enter email with a valid format but missing TLD (e.g., `user@domain`). Verify whether this is accepted or rejected as per the agreed validation rule. *(Gap #2)* | Gap #2 |
| TS-16 | P1 | Security | Verify that the password field has `type="password"` in HTML so the value is not exposed in the DOM or browser history. | AC-04, Gap #10 |
| TS-17 | P1 | Negative | After N consecutive failed login attempts, verify the lockout or rate-limiting behavior and message shown. *(Gap #6)* | Gap #6 |
| TS-18 | P1 | Positive | Verify the Sign In button is accessible via keyboard (Tab to focus, Enter to submit) and the Forgot Password link is reachable by keyboard. *(Gap #8)* | Gap #8 |
| TS-19 | P2 | Positive | Verify the login page renders correctly on mobile viewport (e.g., 375px wide). *(Gap #9)* | Gap #9 |
| TS-20 | P2 | Positive | Verify the login page renders and functions on each required browser (Chrome, Firefox, Safari, Edge). *(Gap #9)* | Gap #9 |
| TS-21 | P2 | Performance | Submit valid credentials and measure response time. Verify login completes within the agreed SLA. *(Gap #11)* | Gap #11 |
| TS-22 | P2 | Security | Verify the login page is only accessible via HTTPS; HTTP redirects to HTTPS. *(Gap #10)* | Gap #10 |
| TS-23 | P2 | Positive | Verify that copying from the password field is not possible (or that the clipboard is cleared), if this is a stated requirement. | AC-04, Gap #10 |

---

## 4. Test Data & Environment

**Data (as provided in requirements):**

| # | Email | Password | Purpose |
|---|-------|----------|---------|
| TD-01 | Valid registered email | Valid password | Successful login (TS-02) |
| TD-02 | Valid registered email | Invalid/wrong password | Wrong password error (TS-03) |
| TD-03 | Unregistered email | Any password | Unregistered email error (TS-04) |
| TD-04 | Invalid format (e.g., `notanemail`, `user@`) | Any | Format validation error (TS-05, TS-15) |
| TD-05 | Empty | Any password | Empty email validation (TS-07) |
| TD-06 | Valid email | Empty | Empty password validation (TS-08) |
| TD-07 | Empty | Empty | Both fields empty validation (TS-06) |
| TD-08 | ` user@example.com ` (with spaces) | Valid | Whitespace trimming behavior (TS-11) |
| TD-09 | 255-char email string | Valid | Max length boundary (TS-12) |
| TD-10 | Valid email | Max-length password | Max length boundary (TS-13) |

**Environment / flags:**
- Target environment: Not specified — confirm staging or production-like env
- Feature flags: None mentioned — confirm none gate this feature
- HTTPS required: Assumed — confirm with team

**Roles / permissions:**
- Standard registered user (all P0/P1 scenarios)
- Unregistered / non-existent user account (TD-03)
- Locked-out user (TS-17, pending Gap #6)

---

## 5. Risks & Assumptions

**Assumptions made:**
- The login form submits credentials to a backend API; error messages originate server-side for credential failures and client-side for format/empty validation.
- "Appropriate error message" (AC-02, AC-03) means a visible, user-readable message on the same page — not a redirect or alert dialog.
- Password masking (AC-04) means `<input type="password">` behavior — bullets/dots replacing characters in real time.
- Forgot Password (AC-05) is a link that navigates away or opens a modal; the recovery flow itself is out of scope.
- No "Remember Me" or OAuth options exist (not in requirements).
- Test environment has at least one pre-registered test account available.

**Risks:**
- **Security message leakage (high):** If error messages distinguish "email not found" from "wrong password," this leaks account existence. Gap #1 must be resolved before TS-03/TS-04 have a clear pass/fail.
- **Whitespace edge case (medium):** Unspecified whitespace trimming could cause false login failures in real use.
- **Rate limiting unknown (medium):** Without Gap #6 answered, aggressive test runs (TS-17) may accidentally trigger lockouts that affect other test scenarios.
- **Browser/device coverage unknown (low–medium):** Gap #9 is unresolved; cross-browser/mobile scenarios (TS-19, TS-20) cannot be scoped.

---

## 6. Entry / Exit Criteria

**Entry (minimum before test execution begins):**
- Login page is deployed to the test environment and accessible
- At least one valid registered test account is available (TD-01)
- Gaps #1 and #2 are answered (required to confirm pass/fail for all P0 error scenarios)
- Gap #6 is answered or rate limiting is disabled in the test environment to prevent lockouts during testing

**Exit (sign-off conditions):**
- All P0 scenarios (TS-01 through TS-10) pass without defects
- All P1 scenarios pass, or failures are filed as defects and accepted/deferred with documented rationale
- No regressions introduced in any adjacent flows (if regression suite exists)
- All remaining open gaps either resolved and re-tested, or formally accepted as out-of-scope for this release

---

## HUMAN REVIEW GATE

- **I assumed:** Error messages are visible inline on the same page; password masking is standard HTML `type="password"`; Forgot Password is a navigation action; no SSO/OAuth in scope.
- **I could not confirm:** Exact error message copy (Gap #1), email format validation rules (Gap #2), field length limits (Gap #3), whitespace trimming policy (Gap #4), lockout policy (Gap #6), Forgot Password destination (Gap #7), a11y requirements (Gap #8), browser/device matrix (Gap #9), security headers (Gap #10), performance SLA (Gap #11), post-login redirect target (Gap #12).
- **Open questions blocking P0 sign-off:** Gap #1 (error message text/distinction) and Gap #2 (format validation rules) must be answered before TS-03, TS-04, and TS-05 have a clear, unambiguous pass/fail definition.

▶ **Approve, or edit, before I write test cases / automation.**
