# Skills

A collection of Claude Code custom skills for QA and software engineering workflows.
Each skill lives in its own numbered folder and is invocable from Claude Code.

## Structure

```
Skills/
├── 01_Test-Planning-Skill/   # Generate JIRA-backed test plans for human review
│   ├── SKILL.md              # Skill definition (triggers, steps, guardrails)
│   ├── Assets/               # Supporting project assets
│   ├── Referances/           # Checklist and test-plan template
│   │   ├── requirement-checklist.md
│   │   └── testplan-template.md
│   ├── Scripts/
│   │   └── fetch_JIRA.sh     # Fetch ticket data via JIRA REST API
│   └── test-plans/           # Generated test-plan outputs (per ticket)
└── .env                      # JIRA_BASE_URL, JIRA_EMAIL, JIRA_TOKEN
```

## Skills

### `test-plan-generator` — Test Planning Skill

**Trigger phrases**
- "Write a test plan for ticket ABC-1234"
- "Plan the testing activities for story ABC-1234"
- "What should I test here for ABC-123?"
- Paste acceptance criteria and ask to create a test plan

**What it does**
1. Fetches the JIRA ticket (description, ACs, comments, linked issues) via MCP or `fetch_JIRA.sh`.
2. Runs the ticket through `requirement-checklist.md` to surface gaps, ambiguities, and missing non-functional requirements.
3. Fills `testplan-template.md` with scenarios tagged P0/P1/P2 by risk.
4. Stops at a **Human Review Gate** — never marks the plan final without tester approval.

**Output shape**
```
## Test Plan — <JIRA-KEY>: <title>
1. Scope & Objectives
2. Gaps & Questions for the author
3. Test Scenarios (P0 / P1 / P2)
4. Test Data & Environment
5. Risks & Assumptions
6. Entry / Exit Criteria
--- HUMAN REVIEW GATE ---
```

## Setup

1. Copy `.env.example` to `.env` (or edit `.env`) and fill in your JIRA credentials:
   ```
   JIRA_BASE_URL=https://your-org.atlassian.net
   JIRA_EMAIL=you@example.com
   JIRA_TOKEN=<your-api-token>
   ```
2. Make the fetch script executable:
   ```bash
   chmod +x 01_Test-Planning-Skill/Scripts/fetch_JIRA.sh
   ```

## Adding a New Skill

1. Create a new folder: `NN_Skill-Name/`
2. Add a `SKILL.md` with the standard frontmatter (`name`, `description`, `metadata`).
3. Add any supporting references, scripts, and assets under subfolders.
4. Register the skill in Claude Code settings if needed.

## Author
Hanmant Hudekar — QA Engineering
