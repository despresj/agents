---
name: fp-engineering
description: Diagnose software failures and derive system designs from evidence, invariants, and explicit requirements. Use when debugging code or system behavior, fixing failing or intermittent tests, investigating performance, explaining an unexpected fix, or designing architecture whose cause, constraints, or tradeoffs are not yet proven.
---

# First Principles — Software Engineering

Apply the `first-principles` core skill. Use this pack for domain traps and
precondition templates; use the core once for checkpoints and reporting.

## Trap Table

| Symptom | Tempting move | First-principles move | Discriminating test |
|---------|--------------|----------------------|---------------------|
| Bug report | Patch where the symptom appears | Reproduce first; trace the failing path end to end; name the first violated invariant | A minimal reproduction fails before the change and passes after, while a neighboring valid case remains unchanged |
| Fix makes the test pass | Move on | Treat a mechanism-free pass as an unresolved diagnosis | Explain the state transition the fix changes; add a test that a coincidental patch would fail |
| Intermittent failure | Add retries/timeouts | Separate containment from diagnosis; inspect race, shared state, ordering, time, and resource pressure | Force or record the relevant interleaving or state until the failure reproduces predictably |
| Library/framework "should" behave per its name or docs | Trust generic documentation | Verify the installed version, configuration, runtime behavior, and source path actually in use | A minimal probe matches the installed implementation and the assumption the change relies on |
| Error message names component X | Fix X | Errors surface where detected, not where caused | Trace backwards to the first point where state diverged from expectation |
| New system design | Add standard layers (cache, queue, service split) | Derive components from measured load, consistency, latency, operability, and failure requirements | Map every component to a forcing requirement; remove one and name the requirement that then fails |
| Performance problem | Optimize suspected code | Measure a representative workload first | A profile identifies the bottleneck, and the same workload shows the change improves the target without shifting the bottleneck harmfully |
| "Works on my machine" | Blame the environment | Enumerate environment differences as testable hypotheses | Diff and bisect versions, configuration, data, locale, time, architecture, and external dependencies |

## Precondition Template (O1)

**"This fix is correct":** reproduce the failure; show it fails before and passes
after; explain the violated invariant and how the change restores it; test the
mechanism rather than only the reported symptom; inspect affected call sites and
compatibility boundaries; cover at least one neighboring or negative case; place
the fix at the layer that owns the invariant, or justify why containment belongs
elsewhere.

**"This design is appropriate":** state workload and failure assumptions with
evidence; define latency, consistency, durability, security, cost, and operability
constraints that matter; compare against the simplest viable design; map every
component to a forcing requirement; record tradeoffs and the conditions that
would require revisiting the design.

## Domain Escalation Triggers

- The fix works and you cannot explain why → call it containment, not a proven
  fix; record the diagnostic gap before shipping or committing it.
- The bug disappears when you add logging/observation → timing-sensitive; the
  cause is still live.
- The same bug has been "fixed" before → the previous fix treated a symptom;
  find the invariant.
- The reproduction depends on production-only data or load → minimize and
  sanitize the case, then test the closest faithful substitute; do not pretend a
  toy case discharged the production assumption.
