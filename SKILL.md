---
name: clarity
description: >
  A knowledge mapping skill for projects built with AI agents. Use when you want
  to map what you actually understand about your own codebase, measure cognitive
  debt from AI-assisted sessions, or generate a handoff that captures
  comprehension gaps — not just code state. Invoke with @clarity followed by
  one of: map, debt, handoff, or status.
license: MIT
compatibility: Works with any AgentSkills-compatible agent — Claude Code, Windsurf, Cursor, GitHub Copilot, Gemini CLI, Amp, Warp, Cline, Codex, and more.
metadata:
  author: favio-vazquez
  version: "1.0"
---

# clarity

A knowledge mapping skill that tracks the gap between what your agent built and
what you understand. Where `learnship` manages the agent's memory,
`clarity` manages yours.

**Core principle:** "The code works" and "I understand this code" are not the same
statement. This skill makes the difference visible before it becomes a problem.

Based on research cited in [references/cognitive-debt.md](references/cognitive-debt.md)
and [references/feynman-technique.md](references/feynman-technique.md).

---

## Actions

### `map` — Knowledge map

**Trigger:** `@clarity map` (optionally: `@clarity map --module <name>` or `@clarity map --quick`)

**What to do:**

1. **Scan the codebase.** Identify the top-level modules, components, or areas of the
   project. If `AGENTS.md` exists, read it to understand decisions already documented.
   If `CLARITY_MAP.md` already exists, read it before updating — preserve prior scores
   and dates for unchanged modules.

2. **For each module or area (skip modules with no changes since last evaluation
   if `--quick` is set)**, ask the user two questions in sequence:

   Question A (what): "Walk me through what `<module>` does — as if explaining it to
   someone joining the project today."

   Question B (why): "What was the key decision that shaped how this was built? Why
   that approach and not the obvious alternative?"

   Wait for each answer before moving to the next module. Do not batch questions.

3. **Classify** each module into one of three zones based on the answers:
   - **Green (Understood):** The user explained what it does and why it is built that
     way, with no significant gaps or "I think" hedges.
   - **Yellow (Partial):** The user understood the what but was vague on the why, or
     could not articulate the key decision.
   - **Red (Risk zone):** The user could not explain what the module does, said
     "I'm not sure," or described it as "the AI wrote it and it works."

4. **Write `CLARITY_MAP.md`** to the project root using the template at
   [templates/CLARITY_MAP.md](templates/CLARITY_MAP.md). Preserve all prior entries.
   Update only modules that were evaluated in this session.

5. **Generate or update `clarity-graph.html`** in the project root using the template
   at [templates/clarity-graph.html](templates/clarity-graph.html).
   Inject the current module data as JSON into the `CLARITY_DATA` variable at the top
   of the script block. Each module entry must include:
   - `id` (string, module name slug)
   - `label` (string, display name)
   - `zone` ("green" | "yellow" | "red")
   - `lines` (approximate line count — use `wc -l` on the directory if available,
     otherwise estimate from file count)
   - `last_evaluated` (ISO date string)
   - `what` (one sentence summary from the user's answer, or empty string if not asked)
   - `why` (one sentence on the key decision, or empty string if not asked)
   - `dependencies` (array of module id strings this module imports from or calls)

6. **Report** the full classification summary:
   - List all modules with their zone
   - Highlight any new Red zones since the last evaluation
   - Tell the user to open `clarity-graph.html` to see the visual map

**Flags:**
- `--quick`: Skip modules whose zone has not changed since the last evaluation.
  Only re-evaluate modules added, removed, or marked Red/Yellow.
- `--module <name>`: Evaluate only the named module. Update its entry in
  `CLARITY_MAP.md` and regenerate the graph.

**Never** pre-classify modules before asking the user. The point is to surface
what the user actually knows, not what the agent thinks they know.

---

### `debt` — Cognitive debt measurement

**Trigger:** `@clarity debt` (optionally: `@clarity debt --history` or
`@clarity debt --threshold <0-100>`)

**What to do:**

1. **Read the recent diff.** Use `git diff HEAD~1 HEAD` if available. If git is not
   initialized, ask the user to describe what was built in this session. If
   `CLARITY_MAP.md` already exists, read it to understand which modules are already
   Red or Yellow.

2. **Select three areas** from the diff that represent meaningful logic — not
   boilerplate, config, or import changes. Prioritize:
   - New functions or methods
   - Branching logic (conditionals, error handling)
   - Integrations between modules

3. **Ask three questions** derived from the actual code, one at a time:

   **What question:** Point to a specific function or block. "What does
   `<function_name>` do? Walk me through it." Do not summarize it first.

   **Why question:** Point to a specific decision in the diff. "Here you're using
   `<approach>` rather than `<obvious_alternative>`. Why?" Choose a real decision,
   not a trivial one.

   **What-if question:** Point to a specific line or branch. "What happens to
   `<system_or_data>` if `<condition>` is true here?" Pick a failure mode or edge
   case that is non-obvious.

   Wait for each answer before asking the next.

4. **Score each answer** from 0 to 100:
   - 80-100: Complete, accurate, no significant hedges
   - 50-79: Partially correct or missing the key mechanism
   - 20-49: Vague, incorrect, or "I think the AI handled that"
   - 0-19: "I don't know" or no answer

5. **Calculate the session Comprehension Score**: average of the three question scores.

6. **Update `CLARITY_MAP.md`** under the `## Cognitive Debt Log` section:
   ```
   | <ISO date> | <score>/100 | <brief note on what was covered> |
   ```
   Mark the session as ALERT if the score is below the threshold (default: 70).

7. **Update `clarity-graph.html`** if it exists: inject the new debt log data into
   the `CLARITY_DEBT_LOG` variable so the graph's debt timeline panel reflects the
   new entry.

8. **Report** the session score, the running average over the last 5 sessions,
   and any recommendation:
   - Score >= 70: Acknowledge and move on.
   - Score 50-69: Name the specific gaps and suggest one `@clarity map --module`
     command to address the weakest area.
   - Score < 50: State plainly that this session added significant cognitive debt.
     Recommend a review session before continuing to build. Suggest which modules
     to address first using `@clarity map`.

**Flags:**
- `--history`: Show the full debt log from `CLARITY_MAP.md` without running a
  new evaluation.
- `--threshold <n>`: Set the alert threshold for this session. Default is 70.
  The threshold is not persisted — set it in `CLARITY_MAP.md` manually if you
  want a project-level default.

**Do not** be lenient in scoring. An answer that is technically correct but could
not be reproduced under pressure is a 50, not an 80.

---

### `handoff` — Context transfer

**Trigger:** `@clarity handoff` (optionally: `@clarity handoff --import` or
`@clarity handoff --sync`)

**What to do (default — export):**

1. **Read `CLARITY_MAP.md`** in full. If it does not exist, run `@clarity map` first
   and tell the user.

2. **Read `AGENTS.md`** if it exists. Do not duplicate anything already there.
   The handoff captures the human knowledge layer — not the technical project state.

3. **Generate `CLARITY_HANDOFF.md`** using the template at
   [templates/CLARITY_HANDOFF.md](templates/CLARITY_HANDOFF.md). Fill in:
   - **Project snapshot date**: today's ISO date
   - **Red zones**: every module classified Red, with the user's own words from the
     map session (or "not evaluated" if the module was never assessed)
   - **Yellow zones**: every module classified Yellow, with the partial explanation
     captured during the map session
   - **Cognitive debt summary**: the running average score and how many sessions are
     below threshold
   - **Open questions**: things the current owner marked as "I'm not sure" or
     "I don't know why this is here" during any session
   - **What the next person should do first**: based on the Red zones, write 2-3
     concrete recommendations. Be specific about which modules to evaluate first
     and which `@clarity` commands to run.

4. **Tell the user** the file was written and where it is. Suggest committing it
   alongside `CLARITY_MAP.md`.

**What to do (`--import` — new team member or new session):**

1. **Check for `CLARITY_HANDOFF.md`** in the project root. If not found, tell the
   user and stop.

2. **Read it fully**, then guide the new user through a structured onboarding:
   - Present the Red zones first: "These are the areas the previous owner flagged as
     not fully understood. I'll walk you through each one."
   - For each Red zone, give a brief explanation of what it does based on the codebase,
     then ask: "Does that make sense? Any questions before we move on?"
   - Present the open questions: "These are the things the previous owner was unsure
     about. Some of these may still be unresolved."
   - At the end, run `@clarity debt` on a small recent diff to establish a baseline
     Comprehension Score for the new session.

3. **Write a session entry** to `CLARITY_MAP.md` under a new `## Onboarding Session`
   section, noting the date, any new understandings, and any questions that remain open.

**What to do (`--sync`):**

After an onboarding or pairing session, `--sync` re-evaluates any modules that
changed status: modules the new person understood better than the handoff suggested
(upgrade Yellow or Red to Green) or modules that revealed new gaps (downgrade to Red).
Run `@clarity map --quick` internally and update `CLARITY_MAP.md` and the graph.

---

### `status` — Project knowledge snapshot

**Trigger:** `@clarity status`

**What to do:**

1. Read `CLARITY_MAP.md`. If it does not exist, tell the user to run `@clarity map`
   first.

2. Report a concise summary:

```
CLARITY STATUS — <project name or directory>
Last evaluated: <date of most recent map session>

Knowledge zones:
  Green  (understood):  <n> modules
  Yellow (partial):     <n> modules
  Red    (risk):        <n> modules — <list their names>

Cognitive debt:
  Last session score:      <score>/100
  5-session average:       <avg>/100
  Sessions below threshold: <n>

Last handoff: <date, or "never">

Recommended action: <one concrete next step based on current state>
```

3. If there are Red zones and the last map session was more than 14 days ago, note
   that the map may be stale.

4. Do not speculate about zones for modules that have not been evaluated. Mark them
   as "not evaluated" rather than guessing.

---

## Files written to the project

All files are written to the project root. `clarity` never writes to the skill
directory itself.

```
CLARITY_MAP.md          knowledge map — zones, debt log, open questions
CLARITY_HANDOFF.md      handoff snapshot — written only when you run @clarity handoff
clarity-graph.html      interactive visual map — opened in the browser
```

These files are yours. Commit them, share them, review them in retrospectives.
The graph is a single static HTML file — no server, no build step.

---

## How clarity relates to learnship

`learnship` manages the agent's memory: persistent context, structured phases,
workflow state across sessions. `clarity` manages the developer's memory: what
you understand, what you don't, and how to transfer that to someone else.

They are complementary but independent. `clarity` reads `AGENTS.md` if it exists
to avoid duplicating what `learnship` already tracks, but does not require it.

If you use both, a natural rhythm is: build with `learnship`, then run
`@clarity debt` at the end of each phase. Run `@clarity map` before any phase
where you are entering unfamiliar territory.

---

## When to use each action

| Situation | Action |
|-----------|--------|
| Starting a new phase or returning after a break | `@clarity map` |
| End of a long AI-assisted build session | `@clarity debt` |
| Before a code review, you want to see the risk zones | `@clarity status` |
| Someone new is joining the project | `@clarity handoff` |
| You are the person joining an existing project | `@clarity handoff --import` |
| After an onboarding session | `@clarity handoff --sync` |

---

## What clarity is not for

- **Brainstorming or new ideas.** Use `@agentic-learning brainstorm` or `/deliberate`.
- **Learning a new concept.** Use `@agentic-learning learn`.
- **Deciding between two approaches.** Use `/deliberate`.
- **Tracking code quality.** Use your linter and test suite.

`clarity` has one job: make the human's understanding of the project a
first-class artifact, as persistent and inspectable as the code itself.
