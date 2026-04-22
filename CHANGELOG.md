# Changelog

## 1.1.2 — 2026-04-22

### Fixes
- `references/cognitive-debt.md`: removed em-dashes, fixed "invisible until it isn't" negation pattern, rephrased "not a failure of those tools" opener.
- `references/feynman-technique.md`: removed em-dash in primary source section.
- `SKILL.md`: unified all `--module <name>` references to `--module <module>` for consistency with README. Fixed "what you don't" phrasing. Fixed "If you can't" to "When you cannot".
- `README.md`: fixed "what you don't" and "If you can't" phrasing in *How it relates to learnship* and *The science* sections.
- `install.sh`: post-install usage message now lists all six actions (`map, debt, review, explain, handoff, status`). Previously only listed four.
- `.claude-plugin/plugin.json` and `.claude-plugin/marketplace.json`: version synced to match SKILL.md and README (was `1.0.0`).

## 1.1.1 — 2026-04-22

### Fixes
- `templates/CLARITY_MAP.md`: bare `clarity debt` and `clarity handoff` references now show both `/clarity X` (Claude Code) and `@clarity X` (Windsurf) so users reading the generated file know exactly what to type.
- `templates/CLARITY_HANDOFF.md`: footer updated to show both invocation forms.
- `templates/CLARITY_REVIEW.md`: blockquote and how-to-use section updated to show both invocation forms throughout.

## 1.1.0 — 2026-04-22

### New actions
- `review` — autonomous agent codebase scan with no user input required. Produces `CLARITY_REVIEW.md` with agent-estimated zones and risk signals. `--deep` flag adds plain-language descriptions of Red-estimated modules.
- `explain` — agent teaches a specific module interactively using layered plain-language explanation. `--quiz` flag runs a post-explanation comprehension check and updates `CLARITY_MAP.md`.

### Expanded `debt`
- Three modes: diff (git), scan (codebase read), session (user description). Auto-selects based on context.
- `--scan` flag: forces codebase read mode — no git required.
- `--session` flag: forces session description mode — useful mid-session before committing.
- Debt log now records the mode used alongside the score.
- `CLARITY_MAP.md` created automatically if it does not exist (minimal stub with debt log only).

### Expanded `map`
- `--explain` flag: agent gives a 2-3 sentence reading of each module before asking the user to explain it. Lowers the blank-page problem on unfamiliar code.
- Yellow zone now also assigned when the user's explanation matches surface behavior but misses the mechanism.

### Expanded `handoff`
- No longer requires `CLARITY_MAP.md`. When no map exists, reads the codebase directly and produces an agent-estimated handoff, clearly marked.
- `--cold` flag: fully autonomous handoff with no user input — reads codebase, applies review signals, writes `CLARITY_HANDOFF.md`. For when there is no time for a map session.
- `--import` no longer stops if no handoff file exists — falls back to map, then offers to run `review`.

### Expanded `status`
- Shows agent-estimated zones from `CLARITY_REVIEW.md` alongside user-verified zones from `CLARITY_MAP.md`.
- Shows last review date.
- Works with no files at all — recommends a starting action rather than stopping.

### Files added
- `CLARITY_REVIEW.md` — new file written by `review` and `handoff --cold`. Kept separate from `CLARITY_MAP.md` to distinguish agent estimates from user-verified understanding.

## 1.0.0 — 2026-04-22

Initial release.

### Actions
- `@clarity map` — three-zone knowledge classification with `CLARITY_MAP.md` and `clarity-graph.html` output
- `@clarity debt` — session Comprehension Score from diff-derived questions
- `@clarity handoff` — context transfer document with `--import` and `--sync` modes
- `@clarity status` — five-line project knowledge snapshot

### Visual layer
- `clarity-graph.html` — D3.js v7 force-directed graph with zone coloring, risk-edge highlighting, stale node fading, click-to-detail panel, and Comprehension Score timeline

### References
- `references/cognitive-debt.md` — research on the velocity-comprehension gap
- `references/feynman-technique.md` — theoretical basis for the three-zone classification
