# Clarity Map

**Project:** {{PROJECT_NAME}}
**Last full evaluation:** {{LAST_EVAL_DATE}}
**Cognitive debt threshold:** {{THRESHOLD}} (default: 70)

---

## Knowledge Zones

The three zones reflect your ability to explain the module, not its code quality.

| Module | Zone | Last evaluated | Notes |
|--------|------|---------------|-------|
| {{MODULE_NAME}} | {{ZONE}} | {{DATE}} | {{NOTES}} |

**Zone key:**
- Green: Understood. You explained what it does and why it is built that way.
- Yellow: Partial. You knew the what but not the why, or hedged on the key decision.
- Red: Risk zone. You could not explain it, or said the agent wrote it.

---

## Module Details

<!-- One section per module. Added by clarity map. -->

### {{MODULE_NAME}}

- **Zone:** {{ZONE}}
- **Last evaluated:** {{DATE}}
- **What you said it does:** {{WHAT_SUMMARY}}
- **Key decision you named:** {{WHY_SUMMARY}}
- **Open questions:** {{OPEN_QUESTIONS}}

---

## Cognitive Debt Log

Scores logged by `clarity debt`. Each row is one session.

| Date | Score | Mode | Alert | Notes |
|------|-------|------|-------|-------|
| {{DATE}} | {{SCORE}}/100 | {{MODE}} | {{ALERT}} | {{NOTES}} |

**Running average (last 5 sessions):** {{RUNNING_AVG}}/100

---

## Open Questions

Things captured during map or debt sessions that remain unresolved.
Review these before running `clarity handoff`.

- [ ] {{OPEN_QUESTION}}

---

## Onboarding Sessions

<!-- Added by clarity handoff --import and clarity handoff --sync -->

### Onboarding — {{DATE}}

**New team member:** {{PERSON_OR_CONTEXT}}
**Modules reviewed:** {{MODULES_REVIEWED}}
**New understandings:** {{NEW_UNDERSTANDINGS}}
**Still open:** {{STILL_OPEN}}
