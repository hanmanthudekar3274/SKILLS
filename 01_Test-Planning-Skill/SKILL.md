---
name: test-plan-generator
description: 
Turn a JIRA ticket and all discussions happend in tickets comments into review ready test plan.When user or QA Lead
Says "Write a test plan for ticket ABC-1234" or  "Plan the testing activities for story ABC-1234",
"What I should test here ABC-123?" or "User diractly pastes acceptance criteria and ask to create test plan"

Fetch the ticket descriptions and comments and analyze the gaps and ambiguities and fill the standerd test plan template,
and stop for human review before treated as final.

license: MIT
metadata:
    author: Hanmant Hudekar
    stlc-phase: Test Planning
    version: 1.0.0

---

# Test Plan Generator

You have to produce a **Test plan a human still has to approve** - never a done artifact.
Your job is doing analysis on requirement and disussions happned in comment section + drafting + surfacing what is missing,
No silent completion.

## When to use
- One or more Jira ticket keys or story text provided and user wants to plan testing activities.
- User asks "What are the risks/ edge cases/ gaps in this ticket"
- User askes like "Write a test plan for ABC-123".

### 1. Fetch the ticket (confluence).
- If the JIRA key is given (e.g. AAB-123), Fetch it prefer JIRA MCP tool. If none,
Run `Scripts\fetch_JIRA.sh AAB-123` (needs `JIRA_BASE_URL`,`JIRA_EMAIL` and `JIRA_TOKEN` env vars)
if nether works ask user to paste the ticket body **Do not invent ticket content** 
- Capture: Summary, description, acceptance criteria, components and linked issues, attachments, fix versions and comments.

### 2. Analyse and Find missing pieces.
Run the ticket through `Referances\requirement-checklist.md` for every item mark present/ ambigous/ missing 
Explicitly List:
- Missing or vague acceptance criteria
- Undefined edge cases, error states, empty/limit/boundry conditions.
- Unstated, non functional needs (pef, security, ally, i18n, permisions / roles )
- Missing testdata, environments or dependencies.
- Ambiguous wording that two Engineers reads in two different ways.
### 3. Draft a test plan 
Fill `references/test-plan-template.md` completely. Derive test scenarios from the
acceptance criteria and the gaps you found. Cover positive, negative, boundary,
and cross-role/permission paths. Tag each scenario P0/P1/P2 by risk.

### 4. Wait for Human Review (Mandatory).
End with a **Human Review Gate**:
- Summarize what you assumed and what you could not confirm.
- List the open questions from step 2 that block sign-off.
- Ask the tester to confirm/edit before the plan is considered approved.
- Do **not** proceed to write test cases or automation until a human approves.


## Output shape
```
## Test Plan — <JIRA-KEY>: <title>
1. Scope & Objectives
2. Gaps & Questions for the author   <-- surface missing pieces here
3. Test Scenarios (P0/P1/P2)
4. Test Data & Environment
5. Risks & Assumptions
6. Entry / Exit criteria
--- HUMAN REVIEW GATE ---
Assumptions made / Open questions / "Approve or edit before I continue"
```

## Guardrails
- Never mark the plan "final" — a human owns sign-off.
- Never fabricate acceptance criteria; a missing AC is a finding, not a blank to fill.
- Keep scenarios traceable: each maps to an AC or a gap.

## References
- `references/requirement-checklist.md` — the gap-analysis checklist
- `references/test-plan-template.md` — the plan template to fill
- `scripts/fetch_jira.sh` — pull a ticket over the JIRA REST API
- `copilot/test-plan.prompt.md` — the same skill as a GitHub Copilot prompt file

