# Feynman Technique — Reference

This file provides context for the agent when explaining the three-zone
classification system used by `@clarity map`.

---

## The technique

Richard Feynman, the physicist, had a simple test for understanding: if you
cannot explain something in plain language, you do not understand it yet.
Not "it's complicated" — you do not understand it. Complexity in the explanation
is a symptom of gaps in the understanding, not a property of the subject.

The technique has four steps:
1. Pick a concept.
2. Explain it as if teaching someone who has never seen it.
3. Identify where your explanation breaks down or where you reach for jargon
   to paper over a gap.
4. Go back to the source and fill the gap. Then try again.

The moment you start hedging — "I think it works like...", "the AI handled
that part" — you have found a gap.

---

## How clarity applies it

`@clarity map` uses a stripped version of this test. For each module, the
agent asks two questions:

- What does it do? (Explain it to someone new.)
- Why is it built this way? (Name the key decision and the alternative you did not choose.)

The classification follows directly from the answers:

**Green:** You answered both questions without significant hedging. You could
defend a change to this module in a code review. You have a mental model that
matches the actual code.

**Yellow:** You answered the first question but not the second, or gave a vague
answer to either. You know what the module does in the abstract but do not have
a complete picture of why it was built that way. This is enough to use it but
not enough to safely modify it.

**Red:** You could not answer the first question, or said the agent wrote it and
it works. This is the zone where silent bugs live. Someone will modify this
module based on a wrong mental model and the error will be invisible until it
propagates.

---

## The difference between recognition and recall

A key insight from cognitive science: recognizing something is not the same as
understanding it. When a developer reads AI-generated code, it often looks
reasonable. The logic follows. The variable names make sense. The test passes.
This creates a feeling of understanding — what researchers call the
"illusion of competence."

The Feynman test breaks this illusion. If you can only recognize the code as
correct but cannot explain why it is correct or what would break it, you have
recognition without recall. That is a Yellow or Red zone.

`@clarity map` forces recall. The agent does not show you the code and ask
if it makes sense. It asks you to explain it first.

---

## Why "not being able to explain it" is sufficient evidence of a gap

A common objection: "I understand it well enough to use it, I just can't
explain it well." This is rarely true for the kind of understanding that matters
in software.

If you cannot explain a module's purpose and the key decision behind it in two
sentences, you probably cannot predict how it will behave when a new requirement
touches it. You cannot write a meaningful test for it. You cannot confidently
approve a change to it in code review.

The articulation is not a separate skill from the understanding. The articulation
is the evidence that the understanding is there.

---

## Primary source

Feynman, R. (1985). *Surely You're Joking, Mr. Feynman!* W. W. Norton & Company.

The technique as a formal learning method was later systematized by others,
but the core principle — explaining as a test of understanding, not a
demonstration of it — comes directly from Feynman's own descriptions of
how he worked and learned.
