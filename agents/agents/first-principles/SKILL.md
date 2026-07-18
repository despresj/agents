---
name: first-principles
description: Apply rigorous first-principles reasoning to substantive analysis, debugging, investigation, design, and evidence-backed conclusions. Use when an agent must verify a diagnosis, compare rival explanations, audit a problem frame, plan work under uncertainty, or report a result that is surprising, unusually strong, quickly inferred, or not yet mechanistically explained.
---

# First-Principles Reasoning Engine

## Overview

Produce an answer that survives attack. Apply the six proof obligations to every
load-bearing claim, diagnosis, or decision: discharge each obligation, or record
it as undischarged with its consequence. Treat a claim as load-bearing when
changing it would change the plan, action, or conclusion. Do not burden routine
status updates or mechanical steps with ceremony.

Prevent fluent, confident error. Prefer a rough analysis of the right question to
a polished analysis of the wrong one.

## The Six Obligations

| # | Obligation | Discharged when |
|---|-----------|-----------------|
| O1 | **Preconditions** | Enumerate the material conditions that must all hold for the plan or conclusion to be valid |
| O2 | **Provenance** | Cite the source actually inspected for every load-bearing fact: file and line, query, command output, dataset, or source link; distinguish observation from inference |
| O3 | **Steelman** | State the strongest plausible rival explanation and the evidence that separates it from the current explanation; if none does, do not present either as established |
| O4 | **Kill test** | Name an observation that would falsify the current belief and inspect it when feasible; otherwise label the missing check and its risk |
| O5 | **Frame audit** | Restate the decision or question the work must resolve and verify that the requested proxy matches the real goal (for example, causal vs predictive or metric vs outcome) |
| O6 | **DoF ledger** | List reasonable analytical or design choices that could have differed and identify which could change the conclusion |

## Three Checkpoints

Maintain these audit blocks at each transition for substantive tasks. Emit them as
concise working updates when the interface supports updates; otherwise use them
as an internal checklist and surface unresolved limits in the final response.
These blocks record decisions and evidence, not private chain-of-thought. Hard
cap: **120 words per block.** Make every line reviewable. Explain empty fields
instead of omitting them.

**PLAN — before the first action:**
```
## PLAN
Decision/question served: <what must this work resolve? (O5)>
Valid only if all true: <numbered material preconditions (O1)>
Least-known step: <the step I least know how to do>
First check: <the cheapest check aimed at that uncertainty>
```
Investigate the least-known consequential step first. If the plan begins with
familiar work while a cheaper high-risk uncertainty remains, reorder it.

**ACT — before any code, model, or data change:**
```
## ACT
Observed facts: <each with the source inspected (O2)>
Current explanation: <what best explains the observations so far>
Rival: <strongest competing explanation (O3)>
Decisive test: <cheapest observation that separates them>
```
Run the decisive test before implementing the favored explanation when it is
safe and in scope. Choose by expected information per unit cost. If the test
cannot be run, state why and narrow the claim.

**CONCLUDE — before reporting findings:**
```
## CONCLUDE
Finding/status: <what the evidence supports now>
Kill test: <what would falsify this and where I checked (O4)>
Sensitive to: <choices that could change the conclusion (O6)>
Undischarged: <unverified assumptions and consequence, or "none">
```
Re-read the PLAN block. Do not promote an unverified assumption to fact. Label it
as `assumes X (unverified)` and state how that limits the result.

## Escalation Triggers — stop and re-run the frame audit

- The result is unusually good (metric far above history, huge effect, tiny p).
- The result exactly matches a salient expectation or round target.
- A fix works and you cannot explain the mechanism.
- Two views of the same data disagree.
- The evidence came only from checks the favored explanation was expected to pass.
- You are about to write "clearly," "obviously," or "as expected" without a
  cited discriminator.

**No mystery results:** treat an unexplained success as unresolved risk, not as a
proven fix or improvement. Give pleasant surprises the same scrutiny as failures.
A temporary mitigation may still be appropriate, but label it as containment and
record the missing diagnosis.

## Rationalizations

| Excuse | Reality |
|--------|---------|
| "The result is consistent with my hypothesis" | Consistency is not discrimination. What does the rival predict here? |
| "The fix works — moving on" | Works-for-unknown-reasons is an undiagnosed bug with good timing |
| "Verifying would take too long" | Choose the cheapest discriminator and compare its cost with the cost of acting on the wrong explanation |
| "The stakeholder already framed the question" | The frame is data about their beliefs, not ground truth about the problem |
| "I established that earlier" | Cite the earlier evidence or re-check it; otherwise it remains an assumption |
| "This is a standard situation" | Standard patterns are exactly how novel problems disguise themselves |

## Red Flags — the answer is driving the evidence

- The conclusion arrived before the first verification.
- Every check could confirm the favored explanation; none could refute it.
- No concrete kill test is named.
- The plan starts with the easiest work while a cheaper consequential uncertainty
  remains.

## Domain Packs

Load the relevant pack alongside this skill: `fp-data-science`,
`fp-deep-learning`, `fp-bayesian`, `fp-engineering`, `fp-git`, `fp-r`,
`fp-python`, `fp-statistics`, `fp-analytics-frontend`, or `fp-client-outcomes`.
Load more than one only
when the task genuinely crosses domains. Packs add domain traps and precondition
templates; do not duplicate the core checkpoints for each pack.
