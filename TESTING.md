# Testing Guide

Behavioral verification checklist for all four actions. These tests confirm the
agent follows the contracts in SKILL.md. They are manual tests — run them by
invoking the skill in your agent after installation.

Each test lists: the invocation, the expected behavior, and what failure looks like.

---

## Before you start

Install the skill and confirm it loads:

**Claude Code:**
```
/plugin marketplace add FavioVazquez/clarity
/plugin install clarity@clarity-marketplace
/reload-plugins
```
Then type `/clarity` — Claude Code should show it as an available skill.

**Windsurf:**
```bash
npx skills add FavioVazquez/clarity
```
Then type `@clarity` in the chat — Windsurf should recognize the skill.

**curl (any agent):**
```bash
curl -fsSL https://raw.githubusercontent.com/FavioVazquez/clarity/main/install.sh | bash
```

---

## `map` — 5 tests

### T1: Basic invocation

**Invoke:** `/clarity map` (Claude Code) or `@clarity map` (Windsurf)

**Expected:**
- Agent scans the codebase and identifies top-level modules or directories
- For each module, asks "Walk me through what X does" before classifying anything
- Waits for your answer before moving to the next module
- Does NOT pre-classify modules or say "I think this is Green" before asking

**Failure:** Agent classifies modules without asking, or batches all questions together.

---

### T2: Classification accuracy

**Setup:** For one module, give a complete answer (what + why). For another,
say "I'm not sure, the agent wrote it."

**Expected:**
- First module classified Green
- Second module classified Red
- `CLARITY_MAP.md` created in the project root with both entries

**Failure:** Both modules get the same zone, or the file is not written.

---

### T3: `--quick` flag

**Setup:** Run `map` once to completion. Then run `/clarity map --quick` (or `@clarity map --quick`).

**Expected:**
- Agent skips all Green modules from the previous run
- Only asks about modules that are Red, Yellow, or new

**Failure:** Agent re-asks about all modules including previously Green ones.

---

### T4: `--module` flag

**Invoke:** `/clarity map --module <name>` where `<name>` is one specific module.

**Expected:**
- Agent only asks about that one module
- Updates only that module's entry in `CLARITY_MAP.md`
- Regenerates `clarity-graph.html`

**Failure:** Agent maps multiple modules or does not update the file.

---

### T5: Graph generation

**After running `map`:**

**Expected:**
- `clarity-graph.html` exists in the project root
- Opening it in a browser shows nodes (one per module) colored by zone
- Clicking a node opens the detail panel with the explanation you gave

**Failure:** File not created, or graph shows no nodes, or wrong colors.

---

## `debt` — 4 tests

### T6: Diff-derived questions

**Setup:** Make at least one meaningful code change and commit it (`git commit`).

**Invoke:** `/clarity debt` or `@clarity debt`

**Expected:**
- Agent reads the actual diff (not a generic question)
- Asks exactly 3 questions: one what, one why, one what-if
- Each question references a real function or line from the diff
- Waits for each answer before asking the next

**Failure:** Questions are generic ("explain your code"), or all three are asked at once.

---

### T7: Scoring honesty

**Setup:** Answer one question fully and correctly. For another, say "I don't know."

**Expected:**
- First answer scores 80-100
- "I don't know" scores 0-19
- Session Comprehension Score (average) reflects the honest split
- Score logged to `CLARITY_MAP.md` under `## Cognitive Debt Log`

**Failure:** "I don't know" scores higher than 20, or log not written.

---

### T8: Alert threshold

**Setup:** Answer all three questions with vague or incorrect responses (aim for score < 70).

**Expected:**
- Agent states the session score
- Flags it as ALERT since it's below the default threshold of 70
- Recommends a specific `map` action on the weakest module

**Failure:** Agent says "good job" or does not flag the low score.

---

### T9: `--history` flag

**Setup:** Run `debt` at least once so the log has an entry.

**Invoke:** `/clarity debt --history` or `@clarity debt --history`

**Expected:**
- Agent reads and displays the full debt log from `CLARITY_MAP.md`
- Does NOT ask new questions
- Shows the running average over the last 5 sessions

**Failure:** Agent runs a new evaluation instead of showing history.

---

## `handoff` — 4 tests

### T10: Export (default)

**Setup:** Run `map` first so `CLARITY_MAP.md` exists with at least one Red module.

**Invoke:** `/clarity handoff` or `@clarity handoff`

**Expected:**
- `CLARITY_HANDOFF.md` created in the project root
- Red zones listed with the user's own words from the map session
- Section "What to do first" with 2-3 concrete recommendations
- File does not duplicate what is already in `AGENTS.md`

**Failure:** File not created, or Red zones are empty, or generic "review the code" recommendations.

---

### T11: `--import` mode

**Setup:** `CLARITY_HANDOFF.md` must exist from a previous export.

**Invoke:** `/clarity handoff --import` or `@clarity handoff --import`

**Expected:**
- Agent presents Red zones one by one and explains each
- Asks "Does that make sense?" after each one before continuing
- Presents open questions from the handoff
- At the end, runs a debt evaluation to establish a baseline score

**Failure:** Agent dumps the whole handoff file without guiding through it.

---

### T12: Missing map guard

**Setup:** Delete `CLARITY_MAP.md` (or start in a fresh project with no map).

**Invoke:** `/clarity handoff` or `@clarity handoff`

**Expected:**
- Agent stops and tells the user to run `map` first
- Does NOT generate a blank or empty `CLARITY_HANDOFF.md`

**Failure:** Agent creates an empty handoff, or proceeds without the map.

---

### T13: `--sync` after onboarding

**Setup:** Run `handoff --import`, complete the onboarding. Then run `--sync`.

**Expected:**
- Agent re-evaluates any modules that changed in comprehension during onboarding
- Updates `CLARITY_MAP.md` with new zones where they changed
- Regenerates `clarity-graph.html`

**Failure:** No changes made to the map even after the onboarding revealed new understandings.

---

## `status` — 2 tests

### T14: Normal output

**Setup:** Run `map` and `debt` at least once.

**Invoke:** `/clarity status` or `@clarity status`

**Expected output format:**
```
CLARITY STATUS — <project name>
Last evaluated: <date>

Knowledge zones:
  Green  (understood):  N modules
  Yellow (partial):     N modules
  Red    (risk):        N modules — <names>

Cognitive debt:
  Last session score:      N/100
  5-session average:       N/100
  Sessions below threshold: N

Last handoff: <date or "never">

Recommended action: <specific next step>
```

**Failure:** Status is vague, does not show zone counts, or guesses at unevaluated modules.

---

### T15: Missing map guard

**Setup:** No `CLARITY_MAP.md` in the project.

**Invoke:** `/clarity status` or `@clarity status`

**Expected:**
- Agent says the map does not exist and tells the user to run `map` first
- Does NOT fabricate a status based on reading the codebase

**Failure:** Agent invents zones without having asked the user anything.

---

## Platform-specific checks

### Claude Code invocation

After installing via the marketplace:
- `/clarity map` should trigger the skill (slash prefix)
- `/clarity` with no action should show available actions
- The skill name in `SKILL.md` frontmatter (`name: clarity`) maps directly to `/clarity`

### Windsurf invocation

After installing via `npx skills add`:
- `@clarity map` should trigger the skill (at-mention prefix)
- The skill appears in the Windsurf skills panel
- `.agents/skills/clarity/SKILL.md` should exist in the workspace

### Workspace vs global scope

**Workspace install** (default): skill only available in the current project.
```bash
# Confirm install location
ls .agents/skills/clarity/SKILL.md
```

**Global install**: skill available in all projects.
```bash
# Claude Code global
ls ~/.claude/skills/clarity/SKILL.md

# Windsurf global
ls ~/.codeium/windsurf/skills/clarity/SKILL.md
```

---

## Reporting issues

If a test fails, open an issue at
[github.com/FavioVazquez/clarity/issues](https://github.com/FavioVazquez/clarity/issues)
with:
- Which test number failed
- Which agent and version
- The exact invocation used
- What the agent produced instead of the expected behavior
