# Requirement Gap-Analysis Checklist

## Scoring

Score every applicable row:

- ✅ Present — requirement is explicitly defined and testable.
- ⚠️ Ambiguous — requirement exists but interpretation is unclear.
- ❌ Missing — required information is not provided.
- N/A — not applicable to this requirement.

### Important Rules

1. Do NOT require the story to follow a specific user-story format if the
   requirement is understandable and testable.
2. A poorly formatted story can be considered valid if the expected
   behavior is sufficiently clear.
3. Every missing item must become a question or clarification for the
   ticket author.
4. Do NOT invent missing requirements.
5. Do NOT assume industry-standard behavior unless explicitly stated.
6. If information cannot be determined from the requirement, report:
   "Insufficient information to determine."
7. Distinguish between:
   - Requirement-derived information
   - Ambiguous information
   - Missing information
   - QA recommendations / risk-based considerations
8. Generate requirement-based test cases only from verified requirements.
9. Risk-based or exploratory scenarios must be clearly identified as
   recommendations and must not be presented as stated requirements.

---

# 1. Functional

- [ ] Clear user story / goal
      ("As a … I want … so that …")

      NOTE:
      The exact user-story format is preferred but NOT mandatory.
      If the requirement clearly defines the user, goal, and expected
      outcome using another format, mark this as ✅.

- [ ] Acceptance criteria are testable (observable pass/fail)
- [ ] Happy path fully described
- [ ] Negative / error paths described (bad input, failure responses)
- [ ] Boundary & empty states (0, 1, max, empty list, null)
- [ ] State transitions / workflow steps enumerated

---

# 2. Data & Environment

- [ ] Required test data specified or derivable
- [ ] Environment / config / feature flags named
- [ ] External dependencies & integrations listed
- [ ] Preconditions / setup stated

---

# 3. Non-functional

- [ ] Performance / load expectations
- [ ] Security / authorization (which roles can/can't)
- [ ] Accessibility (a11y) expectations
- [ ] Internationalization / localization
- [ ] Audit / logging / observability

NOTE:
Mark N/A when the requirement clearly does not involve the relevant
non-functional area.

---

# 4. Cross-cutting

- [ ] Impact on existing features (regression surface)
- [ ] Backward compatibility / migration
- [ ] Mobile / responsive / browser matrix
- [ ] Rollback / feature-flag behavior

---

# 5. AI / ML / GenAI Validation

Complete this section when the requirement contains AI, ML, LLM,
GenAI, RAG, Agent, AI-generated content, AI evaluation, or model
behavior.

## AI Capability

- [ ] AI capability / purpose is explicitly defined
- [ ] Expected AI behavior is defined
- [ ] AI input is defined
- [ ] AI output is defined
- [ ] Expected output structure / format is defined

## AI Quality Expectations

- [ ] Accuracy expectations are defined
- [ ] Relevance expectations are defined
- [ ] Completeness expectations are defined
- [ ] Groundedness / faithfulness expectations are defined
- [ ] Hallucination expectations are defined
- [ ] Consistency / repeatability expectations are defined

## AI Evaluation

- [ ] Evaluation method is defined
- [ ] Evaluation metrics are defined
- [ ] Acceptance thresholds / target values are defined
- [ ] Reference / expected answers are defined where required
- [ ] Evaluation dataset / test dataset is defined where required

## AI Failure Handling

- [ ] AI failure behavior is defined
- [ ] Empty / invalid AI response behavior is defined
- [ ] Model / service unavailable behavior is defined
- [ ] Fallback behavior is defined
- [ ] Timeout behavior is defined

## AI Safety / Security

- [ ] Prompt injection expectations are defined
- [ ] Sensitive data handling is defined
- [ ] Authorization / access control for AI functionality is defined
- [ ] Data isolation requirements are defined
- [ ] Unsafe / prohibited output handling is defined

NOTE:
Do NOT invent AI metrics, thresholds, datasets, or evaluation criteria.
If they are not defined in the requirement, mark them ⚠️ or ❌ as
appropriate and generate a clarification question.

---

# 6. AI / RAG Specific Validation

Complete when the requirement involves RAG, retrieval, knowledge bases,
documents, embeddings, or retrieved context.

- [ ] Knowledge source is defined
- [ ] Retrieval scope is defined
- [ ] Expected retrieved information is defined
- [ ] Retrieval relevance expectations are defined
- [ ] Context requirements are defined
- [ ] Source attribution / citations requirements are defined
- [ ] Out-of-context / unavailable information behavior is defined
- [ ] Hallucination prevention expectations are defined
- [ ] Retrieval evaluation criteria are defined

NOTE:
Do NOT assume metrics such as Context Precision, Context Recall,
Faithfulness, Groundedness, or Answer Relevance unless the requirement
defines them or explicitly requires AI evaluation.

---

# 7. AI Agent / Agentic Workflow

Complete when the requirement involves an AI agent, autonomous task,
tool calling, workflow execution, or multi-step reasoning.

- [ ] Agent objective is defined
- [ ] Agent input is defined
- [ ] Expected agent output is defined
- [ ] Allowed tools / actions are defined
- [ ] Tool permissions are defined
- [ ] Expected workflow / task sequence is defined
- [ ] Success criteria are defined
- [ ] Failure / retry behavior is defined
- [ ] Human approval requirements are defined
- [ ] Agent boundaries / restrictions are defined
- [ ] Unauthorized actions are defined
- [ ] Agent termination / stopping conditions are defined

NOTE:
Do NOT assume an agent is allowed to perform an action simply because
the underlying tool technically supports it.

---

# 8. Clarity

- [ ] No ambiguous wording ("should", "etc.", "handle gracefully")
- [ ] Terms defined consistently
- [ ] Mockups / designs linked and match the text
- [ ] Acronyms / domain-specific terms are defined
- [ ] Expected behavior is distinguishable from implementation detail
- [ ] Requirements do not contain contradictory statements

---

# 9. Requirement Classification

Classify the requirement before generating test cases.

- [ ] Functional
- [ ] UI
- [ ] API
- [ ] Integration
- [ ] Security
- [ ] Performance
- [ ] AI / ML
- [ ] GenAI / LLM
- [ ] RAG
- [ ] Agentic AI
- [ ] Other

Multiple classifications may apply.

---

# 10. Requirement Quality Decision

Based on the checklist, classify the requirement as:

### VALID

Requirement is sufficiently defined to generate requirement-based
test cases.

### PARTIALLY DEFINED

Core behavior is defined, but some areas are missing or ambiguous.

Generate test cases only for verified behavior and report the gaps.

### AMBIGUOUS

Multiple interpretations are possible.

Do not select an interpretation without evidence from the requirement.

### INSUFFICIENT

The requirement does not contain enough information to create reliable
requirement-based test cases.

Do not generate speculative test cases.

---

# 11. Requirement Gaps / Questions

For every ⚠️ or ❌ item, create a clarification question.

Format:

| Area | Status | Gap | Question |
|------|--------|-----|----------|
| Functional | ⚠️ | Error behavior unclear | What should happen when...? |
| AI Validation | ❌ | Evaluation criteria missing | How should AI output quality be evaluated? |
| Security | ❌ | Authorization not defined | Which roles should have access? |

Questions must be derived from the identified gap.

Do NOT create questions based on assumptions.

---

# 12. Test Generation Gate

Before generating test cases:

### If VALID

Generate requirement-based test cases.

### If PARTIALLY DEFINED

Generate test cases for verified requirements.

Also provide requirement gaps.

### If AMBIGUOUS

Do not assume the intended behavior.

Provide clarification questions before generating affected test cases.

### If INSUFFICIENT

Do not generate speculative test cases.

Provide the missing information required to make the requirement
testable.

---

# 13. Anti-Hallucination Validation

Before finalizing the test plan / test cases:

- [ ] Every test case is traceable to a requirement.
- [ ] No unsupported behavior has been introduced.
- [ ] No UI elements have been invented.
- [ ] No API behavior has been invented.
- [ ] No error messages have been invented.
- [ ] No business rules have been invented.
- [ ] No AI metrics have been invented.
- [ ] No AI thresholds have been invented.
- [ ] No expected model behavior has been assumed.
- [ ] No security behavior has been assumed.
- [ ] No implementation details have been treated as requirements.
- [ ] Missing information is explicitly identified.
- [ ] Ambiguous information is explicitly identified.
- [ ] QA recommendations are separated from requirement-derived tests.
- [ ] All assumptions are clearly labeled.
- [ ] "Insufficient information to determine." is used where appropriate.

---

# 14. Final Requirement Validation Summary

Provide the following summary:

Requirement Status:
[VALID / PARTIALLY DEFINED / AMBIGUOUS / INSUFFICIENT]

Requirement Type:
[Functional / UI / API / AI / RAG / Agentic AI / etc.]

Verified Requirements:
[List only explicitly supported requirements]

Requirement Gaps:
[List ⚠️ / ❌ items]

Clarification Questions:
[List questions for ticket author]

AI Validation Required:
[YES / NO]

AI Validation Gaps:
[List missing AI-specific requirements if applicable]

Test Generation Decision:
[Proceed / Proceed with limitations / Block pending clarification]

Anti-Hallucination Check:
[PASS / FAIL]