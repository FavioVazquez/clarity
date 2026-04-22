# Contributing

Contributions are welcome. This document describes how to improve existing
actions, propose new ones, and what correct behavior looks like for this skill.

---

## What "correct behavior" means for clarity

Each action has a clear behavioral contract. A contribution that changes agent
behavior must preserve these:

**`@clarity map`**
- Never pre-classifies a module without asking the user first
- Always waits for the user's answer before moving to the next module
- Produces both `CLARITY_MAP.md` and `clarity-graph.html` in the project root
- Preserves prior entries — does not overwrite evaluations from previous sessions

**`@clarity debt`**
- Questions must be derived from the actual diff, not generic
- Each question targets a different dimension: what, why, what-if
- Scores honestly — a vague or hedged answer is not an 80
- Does not block the user from continuing; the alert is informational

**`@clarity handoff`**
- In export mode: captures the human knowledge layer, not what is in AGENTS.md
- In import mode: guides the new person through Red zones and open questions
  before evaluating their comprehension
- Does not invent explanations for modules that were never evaluated

**`@clarity status`**
- Concise — no more than 10 lines
- Does not guess zones for unevaluated modules

---

## How to propose a new action

Open an issue describing:
1. The problem the action solves (one paragraph)
2. The trigger (invocation syntax)
3. What the agent does, step by step
4. What files it writes to the project, if any
5. What it should not do (the constraint)

---

## How to improve the visual graph

The graph template is at `templates/clarity-graph.html`. It is plain HTML,
CSS, and JavaScript — no build step, no framework, no bundler.

When changing the graph:
- Keep it self-contained (one file, CDN-loaded D3)
- Do not add a server requirement
- Test with the demo data structure in the CLARITY_DATA block
- Keep the CLARITY_DATA schema backward-compatible or document the migration

---

## How to improve reference documents

The files in `references/` provide the agent with context on the reasoning
behind each action. They should:
- Cite primary sources where claims are empirical
- Not describe what the agent should do (that belongs in SKILL.md)
- Be readable by a human, not just an agent

---

## Code of conduct

Be direct and constructive. Disagree on specifics, not on people.
