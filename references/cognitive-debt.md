# Cognitive Debt — Research Reference

This file provides context for the agent when explaining the reasoning behind
`@clarity debt` and the 5-7x velocity-comprehension gap.

---

## The core finding

AI coding agents generate code at 140-200 lines per minute. A developer reads
and genuinely understands code at 20-40 lines per minute. The gap is 5-7x and
grows with every build session.

Developers have always written more code than they fully understand. What changed
in 2025-2026 is the rate. A developer using an
AI agent can produce in one hour what previously took a day. The comprehension
rate did not change. The accumulation did.

Sources:
- Byteiota (2026): "AI coding agents create a 5-7x velocity-comprehension gap
  (140-200 lines/min vs 20-40 lines/min)"
- Mrlatte.net, February 2026: "Velocity vs. Comprehension: The Rise of Cognitive
  Debt in AI-Assisted Software Development"
- Mpelembe.net, February 2026: "The velocity metrics look immaculate today.
  This invisible deficit leads to unmaintainable code, superficial reviews."
- Earezki.com, April 2026: "Code may function correctly while the developer
  remains unable to explain or verify the logic with confidence."

---

## What cognitive debt is

Cognitive debt is the gap between what the agent built and what the developer
understands. It is distinct from technical debt in one important way: technical
debt is a property of the code; cognitive debt is a property of the person.

You can have clean, well-tested, well-structured code and still have cognitive
debt. The code passes every check. The developer cannot explain why a key
function is structured the way it is. Six months later, someone modifies that
function based on a wrong mental model. The bug lives silently until something breaks.

---

## Why it does not appear in standard tooling

Linters measure code quality. Test coverage measures which lines execute.
Code review measures whether reviewers catch problems. None of these measure
whether the developer understands what they are reviewing or approving.

Those tools were designed for a different problem. The gap they leave is the one
`clarity` addresses.

---

## The organizational dimension

When a team uses AI agents, each developer accumulates cognitive debt
independently. The team does not have a shared map of what is understood and
what is not. When someone leaves the project or a new person joins, the
knowledge that was in one developer's head (including the gaps) does not transfer.

Atlan (2026): "Agents repeat work because each agent starts with no memory of
what sibling agents have already solved." The same applies to humans on the same
team.

Camunda 2026 State of Agentic Orchestration: only 11% of AI initiatives reached
production. 73% reported a gap between ambitions and reality. Lack of shared
context across team members was cited as a primary factor.

---

## The threshold default: 70

The default Comprehension Score alert threshold of 70 is not a precise scientific
cutoff. It reflects a judgment that a score below 70 means the developer could
not accurately answer at least one of three questions about code they produced
in that session.

At 70 and above, the developer has a workable understanding — enough to catch
a regression, write a meaningful test, or explain the behavior to a colleague.

Below 70, there is a meaningful chance that the next modification to that code
will be based on a wrong mental model. The alert is a prompt to review, not
a hard stop.

Adjust the threshold with `@clarity debt --threshold <n>` based on the risk
profile of the project.

---

## Why not just write better docs?

Documentation describes what the code does. It does not track whether the
author understood it when they wrote it, or whether that understanding has
degraded as the codebase grew.

A CLARITY_MAP.md is different from a README or an architecture doc in one way:
it records the human's state of knowledge at a specific point in time, including
gaps. It ages. It reflects real understanding rather than an idealized description.

The visual graph makes this degradation visible — nodes fade as time passes
without evaluation, showing where cognitive debt has accumulated silently.
