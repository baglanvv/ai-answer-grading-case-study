# LLM grading of open-ended answers in B2B EdTech

A product case study. The company isn't named, and there's no code, customer
data or infrastructure detail here.

**Short version:** the platform's whole teaching claim rested on learners
writing answers in their own words, and nothing in the product responded to
what they wrote. I specced and shipped LLM validation across about ten
exercise types, as the only product person on it. It's in production and it
works. I also shipped it without any way to measure whether the grading was
any good, which is the part I think about most.

---

## Context

A B2B platform that trains employees in result-oriented thinking. Companies buy
it for their teams. Learners work through modules that ask them to reformulate
tasks as results, break goals into chains, set priorities, and explain value.

I was the only product owner. The product and AI decisions here were mine.
Engineering was a separate function.

## The problem

Open-ended answers aren't a feature of this product, they're the thing it does.
You can't teach someone to turn a vague task into a concrete result with a
multiple-choice key. The learning happens when they write their own sentence
and something reacts to that sentence.

Nothing reacted to it. A learner wrote their answer, pressed next, and that was
it. No grade, no feedback, no hint about whether they'd understood. The
exercise carrying the entire teaching claim was the one the platform couldn't
close the loop on.

That's a product problem before it's an AI problem. The content was fine. The
mechanism was missing.

## What I decided

LLM validation of free-text answers, across roughly ten exercise types. A
learner submits an answer, the model checks it against that exercise's criteria
and writes back feedback rather than just a verdict: what's missing, what to
rethink, what already works.

Three calls I'd defend:

**Feedback instead of a score.** A number invites an argument and tells the
learner nothing they can use. Prose does two jobs at once: it teaches, and it
makes a model error much cheaper. A wrong score is a verdict someone has to
accept or fight. Slightly-off feedback on a correct answer is just a
conversation that missed. Picking the output format so it could absorb model
error was probably the most useful design decision in the project.

**Separate criteria per exercise type, not one general grader.** Ten exercises
asking for genuinely different thinking don't share a definition of "correct."
"Reformulate this as a result" and "put these in priority order" fail in
different ways, so they got different criteria instead of one prompt with a
mode switch.

**I wrote the criteria myself, and that's the weak link.** It was fast, and
it's the structural mistake I'd now flag in anyone else's work. The person who
decides what "correct" means shouldn't also be the person building the thing
that judges it. There's no independent check anywhere in that setup. If my
criteria were wrong, nothing in the system could have told me. Having a
methodologist write the criteria separately from the implementation fixes it,
and it's cheap compared to finding out later.

## What I didn't do

I shipped LLM grading of people's work with no evaluation harness. No labeled
test set, no baseline to beat, no measurement of how often the grader got it
wrong or which way it leaned. It went to production because the outputs looked
reasonable when I spot-checked them.

I also didn't instrument the outcome, so I can't tell you what it did to
completion rates. I'm not going to dress that up. The feature shipped, it gets
used, and I didn't set up the measurement that would let me prove it was worth
anything or catch it getting worse.

Both gaps come from the same place. Nobody wrote down what "good output" meant
before launch, so there was nothing to measure against afterwards.

## Why that mattered more than I realised

Grading errors in this product aren't symmetrical.

If the model passes a wrong answer, the learner walks away more confident in
something false. Nobody complains, because nobody knows. If the model fails a
correct answer, the learner decides the product is broken, and that's the one
that reaches the person who signed the contract. One is a teaching failure, the
other is a renewal risk.

An accuracy number averages those together and hides which one you've got. So
accuracy was never really the metric I needed, and I didn't work that out until
afterwards.

## What I built after

[churn-reason-eval](https://github.com/baglanvv/churn-reason-eval) is the
harness I think should have existed here from the start. I built it on a
different problem so it could be public: a labeled test set, a cheap baseline
the model has to beat, error analysis weighted by what each mistake actually
costs, and a check on whether the test set is even big enough to support the
decision.

Two things I learned building it that I didn't know while shipping this
product. First, the cost model needs as much scrutiny as the labels. My first
version used save-rate assumptions that were off by roughly 3x, and everything
built on top of them was fiction until I went back and checked. Second, a test
set written by the same person who wrote the rules proves nothing. That's the
same error as the criteria problem above. I've now made it twice in two
different shapes, which is why it's the first thing I look for.

Brought back to this product, the harness I'd want: a labeled set per exercise
type, graded by someone other than me. Error costs weighted so that failing a
correct answer counts as the expensive case. And a queue where low-confidence
grades go to a human, with their correction becoming the next test row, so the
test set grows out of real usage instead of out of my head.

## What I'm least sure about

Whether the format works long-term. Corporate training has a problem that good
content doesn't fix: the learner's employer bought it, not the learner, so
there isn't much intrinsic motivation to work with. AI feedback makes the
exercises genuinely better. A better exercise inside something you were told to
use is still something you were told to use.

I think it's solvable, but not at the level of the exercise. It's a question
about who the product is really for, and what someone gets from finishing it
that they can't get by clicking through. I'd want an answer to that before
putting more into the grading layer.
