# Case study: LLM grading of open-ended answers in B2B EdTech

Product and AI product work on a B2B corporate training platform. The company
is not named, and no code, customer data, or infrastructure detail is included.

**Summary:** the platform's core pedagogical claim depended on learners writing
answers in their own words, and there was no mechanism to respond to what they
wrote. I specified and shipped LLM validation across roughly ten exercise
types, as sole product owner. It works, it is in production, and I shipped it
without an evaluation harness — which is the part of this project I learned the
most from.

---

## Context

A B2B platform that trains employees in result-oriented thinking. Companies buy
it for their teams; learners work through modules that ask them to reformulate
tasks as results, break goals into chains, prioritise, and articulate value.

**My role:** sole product owner. The product and AI feature decisions described
here were mine; engineering was a separate function.

## The problem

Open-ended answers are not a feature of this product, they are the product. You
cannot teach someone to reformulate a vague task into a concrete result with a
multiple-choice key — the learning happens when they write their own sentence
and something responds to *that sentence*.

**And nothing responded to it.** Before the work described here, a learner
wrote their answer and pressed next. No grade, no feedback, no signal about
whether they had understood. The exercise that carried the entire pedagogical
claim was the one exercise the platform could not close the loop on.

That is a product problem before it is an AI problem. The content was good and
the mechanism was missing.

## What I decided

LLM validation of free-text answers across roughly ten exercise types. A
learner submits an answer; the model evaluates it against that exercise's
criteria and returns **feedback, not just a verdict** — what was missing, what
to reconsider, what was already right.

Three decisions worth defending:

**1. Feedback rather than a score.** A number invites argument and tells a
learner nothing they can act on. Prose feedback does two things at once: it
teaches, and it makes the cost of a model error much lower. A wrong score is a
verdict a learner has to accept or fight. Imperfect feedback on a correct
answer is just a conversation that was slightly off — annoying, not damaging.
Choosing the output format to absorb model error was the single most useful
design decision in the project.

**2. Per-exercise criteria rather than one general grader.** Ten exercise types
asking genuinely different cognitive work do not share a definition of
"correct". "Reformulate this task as a result" and "order these by priority"
fail in different ways, so they get different evaluation criteria rather than
one prompt with a mode switch.

**3. Criteria written by me, which is also the project's weakest link.** I
wrote the evaluation criteria for each exercise type myself. That was fast, and
it is exactly the structural mistake I would now flag in anyone else's work:
**the person who defines what "correct" means should not also be the person
building the thing that judges it.** There is no independent check in that
arrangement — if my criteria were wrong, nothing in the system would have told
me. A methodologist writing criteria independently of the implementation is the
fix, and it costs almost nothing compared to discovering the problem later.

## What I did not do, and should have

**I shipped LLM grading of human work with no evaluation harness.** No labeled
test set, no scored baseline, no measurement of how often the grader was wrong
or in which direction. The feature went to production on the strength of
spot-checking outputs that looked reasonable to me.

I also did not instrument the outcome, so I cannot tell you what it did to
completion rates. That is a real gap and I am not going to dress it up: the
honest statement is that the feature shipped, it is used, and I did not set up
the measurement that would let me prove its value or catch its regressions.

The two failures are the same failure. Without a definition of "good output"
written down before launch, there was nothing to measure against afterwards.

## What that cost, specifically

Grading errors in this product are **asymmetric**, and without a harness I had
no visibility into which kind was happening:

| Error | What it costs |
|---|---|
| Passing a wrong answer | The learner is taught something false and leaves more confident in it. Silent — nobody complains. |
| Failing a correct answer | The learner concludes the product is broken. **This is the one the corporate buyer hears about**, and it is a renewal risk, not a learning problem. |

A single accuracy number averages these two together and hides which one you
have. For a product sold to companies, the second is the one that ends
contracts — which means accuracy was never the metric I needed.

## What I built afterwards

[`churn-reason-eval`](https://github.com/baglanvv/churn-reason-eval) is a
worked example of the harness that should have existed here from the start,
built on a different problem so it could be published: a labeled test set, a
cheap baseline the model has to beat, cost-weighted error analysis instead of
raw accuracy, and a power check on whether the test set can support the
decision at all.

Building it taught me two things I did not know while shipping this product:

- **The cost model deserves as much scrutiny as the labels.** My first version
  of that harness used save-rate assumptions that were off by a factor of
  three, and the business case it produced was a fantasy until I checked them.
- **A test set authored by the same person who wrote the rules proves nothing.**
  That is the same structural error as decision 3 above. I found it in my own
  work twice, in two different forms, which is why it is the thing I now look
  for first.

Applied back to this product, the harness I would want is: a labeled set per
exercise type graded by someone other than me, error costs weighted so that
false negatives on correct answers are treated as the expensive case, and a
disagreement queue routing low-confidence grades to a human — whose correction
becomes the next test row, so the test set grows out of production rather than
out of my imagination.

## What I am least sure about

Whether the format is viable in the long run. Corporate training has a
structural problem that no amount of good content fixes: the learner's employer
bought it, not the learner, and intrinsic motivation is thin. AI feedback makes
the exercises genuinely better, but a better exercise inside a product someone
was told to use is still a product someone was told to use.

I think it is solvable, and I do not think it is solvable at the level of the
exercise. It is a question about who the product is actually for and what the
learner gets out of finishing that they cannot get by clicking through — and
that is a product-strategy question I would want to answer before investing
further in the grading layer.
