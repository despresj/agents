---
name: fp-data-science
description: Audit causal, experimental, predictive, and metric-based claims before they drive decisions. Use when analyzing experiments or A/B tests, estimating effects or lift, investigating metric movements, building models that choose actions, or evaluating claims about churn, conversion, revenue, campaigns, and other business outcomes.
---

# First Principles — Data Science & Causal Analysis

Apply the `first-principles` core skill. Use this pack for data-science traps and
precondition templates; use the core once for checkpoints and reporting.

## Trap Table

| Symptom | Tempting move | First-principles move | Discriminating test |
|---------|--------------|----------------------|---------------------|
| Metric moved after a launch | Credit the launch | Define the estimand, denominator, counterfactual, and concurrent changes | Compare with a randomized or credible concurrent holdout; recompute on a fixed eligible cohort |
| "Model will improve decisions" | Optimize AUC or accuracy | Specify the action and whether it needs risk, response, or incremental treatment effect | On randomized or defensibly identified data, estimate treatment-control differences within pre-treatment score strata and evaluate policy value |
| Effect far above historical results | Celebrate | Twyman's law: hunt leakage and selection before believing | Ablate feature groups; audit feature timestamps vs label time |
| Metric conditioned on post-treatment behavior (per-purchaser, per-active-user) | Treat as the outcome | Post-treatment conditioning bias: treatment changes who is in the denominator | Intent-to-treat on assigned users, zeros included |
| Arms or cohorts cover different time windows | Pool and compare | Time is a confounder (seasonality, promos, staggered launch) | Restrict to concurrent windows; check daily effect series |
| Aggregate trend is clear | Conclude from the aggregate | Mix shift / Simpson's reversal | Segment decomposition; check sign within major segments |
| Improvement after intervening on extreme performers | Credit the intervention | Regression to the mean | Control group selected at the same threshold |
| Result found after many analysis variants | Report the variant that worked | Treat undisclosed forking paths as selection on noise | Re-run a frozen specification on untouched data; show a specification curve and disclose the search |
| Experiment "significant," small p | Ship | Recognize that p-values address sampling variation under assumptions, not design or telemetry bias | Check assignment and exposure integrity, SRM, attrition, logging, analysis unit, and concurrent interventions |
| Dashboard crossed a threshold after repeated peeking | Stop and ship | Account for optional stopping and repeated looks | Reconstruct the stopping rule; use a fixed horizon or a valid sequential design |

## Precondition Templates (O1)

**"X causes Y":** define the treatment, outcome, population, horizon, and estimand;
state the counterfactual; verify assignment, exposure, sample ratio, telemetry,
attrition, and analysis unit; avoid post-treatment conditioning; address
interference and concurrent changes; use uncertainty that matches clustering,
repeated measures, and stopping; distinguish prespecified analysis from disclosed
sensitivity checks; state external-validity limits.

**"This offline model will improve decision D":** serving features computed
identically to training (inspect both paths); label and eligibility mean the same
thing at decision time; no feature leaks the outcome or future information; the
score ranks what D needs (risk, response, and persuadability are different);
offline evaluation represents deployment; calibration, capacity, costs, and the
decision threshold are included; policy value is validated when actions affect
outcomes.

## Domain Escalation Triggers

- You cannot state the estimand and counterfactual in plain language → stop; the
  causal question is not yet well posed.
- Effect size several times larger than any historical result → assume artifact
  until leakage/selection ruled out.
- Conclusion flips across reasonable analysis choices → report the sensitivity,
  not the favorite spec.
- The intervention changes eligibility, measurement, or the metric denominator →
  report intent-to-treat first and make conditional analyses explicitly
  secondary.
- An observational causal claim depends on untestable exchangeability or exclusion
  assumptions → label those assumptions and add a negative control, sensitivity
  analysis, or design improvement where possible.
