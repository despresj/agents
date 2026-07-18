# reason-from-first-principles

An agent skill kit that helps capable models reason through unfamiliar data and
engineering problems before pattern-matching to a familiar solution.

## Why

On problems with no established playbook, capable models can fail by **confident
premature convergence**: a polished analysis of the wrong question. The kit
counters this with six proof obligations and three short checkpoints—PLAN, ACT,
and CONCLUDE—that keep assumptions, evidence, rival explanations, and sensitivity
visible through long agentic sessions.

## Contents

```
skills/
  first-principles/   # the engine: six obligations, three checkpoints, escalation triggers
  fp-data-science/    # causal/analytics traps: leakage, uplift vs propensity, forking paths
  fp-deep-learning/   # training/eval traps: contamination, seed variance, pipeline-bug-first
  fp-bayesian/        # workflow gates: predictive checks, simulation recovery, diagnostics
  fp-engineering/     # debugging/design traps: reproduce first, invariants, no mystery fixes
  fp-git/             # git invariants: reflog recovery, conflicts as intents, secrets = rotate first
  fp-r/               # R silent-failure catalog: recycling, NA, factors, merge fan-out
  fp-python/          # Python/pandas silent-failure catalog: fan-out, NaN groups, alignment
  fp-statistics/      # inference traps: selection, multiplicity, dependence, missingness
  fp-client-outcomes/ # business/sales: outcomes over deliverables, value in client units
  fp-analytics-frontend/ # data interfaces: score semantics, uncertainty, decision-first design
```

The core skill defines the method once. Domain packs add only their trap tables,
workflow gates, and claim-specific preconditions. Each folder also includes
`agents/openai.yaml` metadata for compatible skill UIs.

## Install

Install the core and every domain pack you intend to use; a pack depends on
`first-principles`.

**GitHub Copilot (project):** copy the folders under `skills/` into
`.github/skills/`, `.agents/skills/`, or `.claude/skills/`.

**GitHub Copilot CLI (personal):** copy them into `~/.copilot/skills/` or
`~/.agents/skills/`.

**Claude Code:** copy them into `.claude/skills/` for a project or
`~/.claude/skills/` for personal use.

If the reasoning discipline should be mandatory rather than automatically invoked
only when descriptions match, add this to the repository's `AGENTS.md` or
`CLAUDE.md`:

```markdown
For substantive analysis, debugging, investigation, design, or evidence-backed
conclusions, apply `first-principles` and the relevant `fp-*` domain pack.
```

## How it works

Every load-bearing claim carries six obligations: material preconditions,
provenance, a steelmanned rival, a kill test, a frame audit, and a
degrees-of-freedom ledger. Short, hard-capped PLAN / ACT / CONCLUDE records keep
the process reviewable without asking for private chain-of-thought. Anything still
unverified stays labeled in the final report instead of quietly becoming fact.

[Read the design rationale](docs/superpowers/specs/2026-07-18-first-principles-skill-kit-design.md).
