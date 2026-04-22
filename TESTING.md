# Testing Guide

Behavioral verification checklist for all six actions. These are manual tests —
run them by invoking the skill in your agent after installation.

Each test lists: the invocation, the expected behavior, and what failure looks like.

---

## Before you start

Install and confirm the skill loads.

**Claude Code:**
```
/plugin marketplace add FavioVazquez/clarity
/plugin install clarity@clarity-marketplace
/reload-plugins
```
Type `/clarity` — Claude Code should show it as an available skill.

**Windsurf:**
```bash
npx skills add FavioVazquez/clarity
```
Type `@clarity` — Windsurf should recognize the skill and show the description.

**curl (any agent):**
```bash
curl -fsSL https://raw.githubusercontent.com/FavioVazquez/clarity/main/install.sh | bash
```

---

## `map` — 6 tests

### T1: Basic invocation — no pre-classification

**Invoke:** `/clarity map` or `@clarity map`

**Expected:**
- Agent scans the codebase and identifies top-level modules
- For each module, asks "Walk me through what X does" before classifying anything
- Waits for each answer before moving to the next
- Does NOT say "I think this is Green" or classify before asking

**Failure:** Agent pre-classifies, or batches all questions at once.

---

### T2: Classification accuracy

**Setup:** For one module, give a complete explanation (what + why). For another,
say "I'm not sure, the agent wrote it."

**Expected:**
- First module: Green
- Second module: Red
- `CLARITY_MAP.md` written to the project root with both entries

**Failure:** Same zone for both, or file not written.

---

### T3: Yellow on surface-only answer

**Setup:** For one module, explain what it does correctly but when asked why it
is built that way, say "I'm not sure about the specific decision."

**Expected:**
- Module classified Yellow (knew the what, not the why)
- Not Green (incomplete), not Red (not a total blank)

**Failure:** Agent classifies it Green because the what-answer was correct.

---

### T4: `--quick` flag

**Setup:** Run `map` once to completion. Then run `map --quick`.

**Expected:**
- Agent skips all Green modules from the prior run
- Only re-evaluates Red, Yellow, and new modules

**Failure:** Agent re-asks about previously Green modules.

---

### T5: `--explain` flag

**Invoke:** `/clarity map --explain` or `@clarity map --explain`

**Expected:**
- Before asking Question A for each module, agent gives a 2-3 sentence reading
  of what it sees in that module
- Then proceeds to ask Question A ("Walk me through what X does")
- The pre-read does not replace the question — the user still classifies

**Failure:** Agent skips the pre-read, or pre-reads and then skips the question.

---

### T6: Graph generation

**After any `map` run:**

**Expected:**
- `clarity-graph.html` exists in the project root
- Opening in a browser shows colored nodes (one per module)
- Clicking a node shows the detail panel with your explanation

**Failure:** File not created, graph is empty, or colors are wrong.

---

## `debt` — 6 tests

### T7: Diff mode (default)

**Setup:** Make at least one meaningful code change and commit it.

**Invoke:** `/clarity debt` or `@clarity debt`

**Expected:**
- Agent reads the actual diff, not a generic question
- Asks 3 questions: one what, one why, one what-if
- Each question references a real function or line from the diff
- Waits for each answer before asking the next

**Failure:** Generic questions, or all three asked at once.

---

### T8: Scan mode — no git required

**Setup:** Start in a project with no git initialized, or delete the `.git` folder.

**Invoke:** `/clarity debt --scan` or `@clarity debt --scan`

**Expected:**
- Agent does NOT try to run git
- Instead reads the codebase and identifies the most complex areas
- Asks 3 questions about actual code from those areas

**Failure:** Agent fails because no git, or asks generic questions not derived from code.

---

### T9: Session mode

**Invoke:** `/clarity debt --session` or `@clarity debt --session`

**Expected:**
- Agent asks you to describe what was built or changed in this session
- Uses your description to form 3 targeted questions about real code
- Does not run git diff

**Failure:** Agent runs a diff anyway, or asks generic questions ignoring your description.

---

### T10: Scoring honesty

**Setup:** Answer one question fully. For another, say "I don't know."

**Expected:**
- Full answer: 80-100
- "I don't know": 0-19
- Session score (average) reflects the split
- Score logged to `CLARITY_MAP.md` under `## Cognitive Debt Log` with mode noted

**Failure:** "I don't know" scores above 20, or log not written.

---

### T11: Alert threshold

**Setup:** Answer all three questions vaguely (target score < 70).

**Expected:**
- Agent states the session score
- Flags ALERT (below threshold 70)
- Recommends a specific `map --module` on the weakest area

**Failure:** No alert flagged, or generic "review the code" recommendation.

---

### T12: `--history` flag

**Setup:** Run `debt` at least once so the log has an entry.

**Invoke:** `/clarity debt --history` or `@clarity debt --history`

**Expected:**
- Shows the full debt log from `CLARITY_MAP.md`
- Does NOT ask new questions
- Shows the 5-session running average

**Failure:** Runs a new evaluation instead of showing history.

---

## `review` — 4 tests

### T13: Basic autonomous scan

**Invoke:** `/clarity review` or `@clarity review`

**Expected:**
- Agent reads the codebase without asking any questions
- Produces a module table with agent-estimated zones (Green/Yellow/Red)
- All zones are marked `(agent estimate)`
- Writes `CLARITY_REVIEW.md` to the project root
- Does NOT write to `CLARITY_MAP.md`
- Tells the user how to promote estimates: `map --module <name>`

**Failure:** Agent asks questions, or writes to `CLARITY_MAP.md`, or produces no file.

---

### T14: Risk signals present in output

**Setup:** Use a codebase with at least one complex or multi-concern module.

**Expected:**
- `CLARITY_REVIEW.md` contains a risk signal for the complex module
  (complexity, coupling, opacity, or AI-generation pattern)
- Red-estimated modules have a named primary risk signal, not just "risky"

**Failure:** All modules estimated Green regardless of actual complexity, or
risk signals are generic.

---

### T15: `--deep` flag

**Setup:** Run `review` on a codebase where at least one module is estimated Red.

**Invoke:** `/clarity review --deep` or `@clarity review --deep`

**Expected:**
- For each Red-estimated module, `CLARITY_REVIEW.md` includes a 3-5 sentence
  plain-language description of what the module appears to do and what the primary risk is

**Failure:** No additional descriptions for Red modules, or descriptions are
one-liners.

---

### T16: Anchors to existing map

**Setup:** Run `map` first so `CLARITY_MAP.md` exists. Then run `review`.

**Expected:**
- Agent reads `CLARITY_MAP.md` before scanning
- Does not re-estimate modules already marked Green in the map as Red
- May still flag them for staleness if last evaluated > 14 days ago

**Failure:** Agent ignores the map and produces estimates that contradict
user-verified zones without noting the conflict.

---

## `explain` — 4 tests

### T17: Basic explanation

**Invoke:** `/clarity explain <module>` or `@clarity explain <module>`

**Expected:**
- Agent reads the module and gives a layered explanation in plain language
- Starts with purpose, explains mechanism, names the non-obvious part, names the gotcha
- Pauses after each layer: "Does that make sense?"
- Waits for user questions before continuing

**Failure:** Agent dumps the whole explanation at once without pausing, or
gives a one-line summary.

---

### T18: Stays focused on the module

**Setup:** During `explain`, ask about something in a different module.

**Expected:**
- Agent briefly acknowledges the other module
- Notes it can be covered with `explain <other-module>`
- Stays focused on the current module

**Failure:** Agent goes off on a tangent and loses the thread of the current module.

---

### T19: `--quiz` flag updates map

**Invoke:** `/clarity explain <module> --quiz`

**Expected:**
- After explanation, agent asks 3 comprehension questions (what, why, what-if)
  about this specific module
- Scores answers
- If average >= 70: marks module Green in `CLARITY_MAP.md` (creates file if needed)
- If average < 70: marks module Yellow and notes which question revealed the gap
- Reports the score and what was recorded

**Failure:** No questions asked after explanation, or map not updated.

---

### T20: Without `--quiz` — no map change

**Invoke:** `/clarity explain <module>` (no quiz flag)

**Expected:**
- Agent does NOT update `CLARITY_MAP.md`
- At the end, suggests running `map --module <name>` to record comprehension formally

**Failure:** Agent updates the map without the user being quizzed, or does not
suggest the next step.

---

## `handoff` — 6 tests

### T21: Export with map

**Setup:** Run `map` first so `CLARITY_MAP.md` exists with at least one Red module.

**Invoke:** `/clarity handoff` or `@clarity handoff`

**Expected:**
- `CLARITY_HANDOFF.md` written to project root
- Red zones listed with the user's own words from the map session
- "What to do first" section with 2-3 concrete recommendations including `clarity` commands
- Source noted as "user-verified map"

**Failure:** File not written, Red zones empty, or generic recommendations.

---

### T22: Export without map (agent-estimated)

**Setup:** Delete `CLARITY_MAP.md` or use a fresh project with no map.

**Invoke:** `/clarity handoff` or `@clarity handoff`

**Expected:**
- Agent reads the codebase directly
- `CLARITY_HANDOFF.md` written with all zone assessments marked `(agent estimate)`
- Top of the handoff states that no map session exists and recommends running `map`
- Does NOT stop or refuse

**Failure:** Agent stops and says "run map first," or writes a blank file.

---

### T23: `--cold` flag — fully autonomous

**Invoke:** `/clarity handoff --cold` or `@clarity handoff --cold`

**Expected:**
- Agent reads the codebase with no user input
- Writes a complete `CLARITY_HANDOFF.md` using the same signals as `review`
- All zones marked as agent-estimated
- Does not ask the user any questions

**Failure:** Agent asks questions, or requires a map file.

---

### T24: `--import` with handoff file

**Setup:** `CLARITY_HANDOFF.md` exists from a prior export.

**Invoke:** `/clarity handoff --import` or `@clarity handoff --import`

**Expected:**
- Agent presents Red zones first, explains each from the codebase
- Asks "Does that make sense?" after each before continuing
- Presents Yellow zones with lighter treatment
- Presents open questions
- At the end, offers to run `debt --scan` for a baseline score

**Failure:** Agent dumps the handoff file as-is without guiding through it.

---

### T25: `--import` with no files

**Setup:** Fresh project, no `CLARITY_HANDOFF.md`, no `CLARITY_MAP.md`.

**Invoke:** `/clarity handoff --import` or `@clarity handoff --import`

**Expected:**
- Agent does NOT stop
- Offers to run `review` first to produce a structural overview
- Uses the review as the onboarding base

**Failure:** Agent stops and says a handoff or map is required.

---

### T26: `--sync` after onboarding

**Setup:** Complete a `handoff --import` session. Then run `--sync`.

**Expected:**
- Agent re-evaluates modules that changed in comprehension during onboarding
- Updates `CLARITY_MAP.md` (creates if needed)
- Regenerates `clarity-graph.html`

**Failure:** No changes made even after the onboarding revealed new understandings.

---

## `status` — 3 tests

### T27: Full output with all files

**Setup:** Run `map`, `debt`, `review`, and `handoff` at least once.

**Invoke:** `/clarity status` or `@clarity status`

**Expected output includes:**
- User-verified zones from `CLARITY_MAP.md` (counts per zone, names of Red modules)
- Agent-estimated zones from `CLARITY_REVIEW.md` (count only)
- Debt: last session score with mode, 5-session average, sessions below threshold
- Last handoff date and last review date
- One concrete recommended next step

**Failure:** Missing section, guesses at unevaluated modules, or no recommendation.

---

### T28: Works with no files

**Setup:** Fresh project, no clarity files.

**Invoke:** `/clarity status` or `@clarity status`

**Expected:**
- Agent states no map, review, or handoff exists
- Recommends a starting action (`review` for a quick read, `map` for formal evaluation)
- Does NOT invent zones or scores

**Failure:** Agent fabricates a status or refuses to respond.

---

### T29: Stale map warning

**Setup:** Modify `CLARITY_MAP.md` manually to set the last evaluation date to
more than 14 days ago, and ensure at least one Red module exists.

**Invoke:** `/clarity status` or `@clarity status`

**Expected:**
- Agent notes that the map may be stale
- Recommends running `map --quick` to refresh

**Failure:** No staleness warning even with Red zones and an old evaluation date.

---

## Platform-specific checks

### Claude Code

After marketplace install:
- `/clarity map` triggers the skill
- `/clarity` with no action shows available actions
- `SKILL.md` `name: clarity` maps to `/clarity` automatically

### Windsurf

After `npx skills add`:
- `@clarity map` triggers the skill
- `.windsurf/skills/clarity/SKILL.md` exists (symlinked or copied)

### Workspace vs global scope

```bash
# Workspace install (default)
ls .agents/skills/clarity/SKILL.md

# Global — Claude Code
ls ~/.claude/skills/clarity/SKILL.md

# Global — Windsurf
ls ~/.codeium/windsurf/skills/clarity/SKILL.md
```

---

## Reporting issues

Open an issue at [github.com/FavioVazquez/clarity/issues](https://github.com/FavioVazquez/clarity/issues) with:
- Test number and name
- Agent and version
- Exact invocation
- What the agent produced vs what was expected
