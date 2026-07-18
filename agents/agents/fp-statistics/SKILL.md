---
name: fp-statistics
description: Audit statistical design and inference before reporting p-values, intervals, effects, regressions, power, or subgroup comparisons. Use when comparing groups, analyzing surveys or observational samples, claiming no effect or a driver, handling clustered or missing data, selecting hypotheses, variables, models, or subgroups on the same data, or interpreting a large, tidy result from a small or selected sample.
---

# First Principles — Statistical Inference

Apply the `first-principles` core skill. Use this pack for inference traps and
precondition templates; use the core once for checkpoints and reporting.

## The Governing Facts

1. **A p-value assumes the design.** It quantifies sampling noise under a model
   of how the data arose. Selection, response bias, clustering, peeking, and
   multiple looks can invalidate the nominal reference distribution. Audit the
   design before interpreting the number.
2. **Who is in the sample outranks what the sample says.** Before any test:
   who could have entered the data, who actually did, and who is missing?
   Treat non-random inclusion as an identification problem, not sampling noise.
3. **Selection changes the estimate distribution.** When only significant,
   best-fitting, or otherwise selected results are reported—especially under low
   power—effect magnitudes tend to be exaggerated and signs can be unstable. Do
   not relabel a selected estimate as an "upper bound": quantify the
   selection-conditioned distribution or report that the magnitude is unstable.

## Trap Table

| Symptom | Tempting move | First-principles move | Discriminating test |
|---------|--------------|----------------------|---------------------|
| p < 0.05 | Treat the hypothesis as probably true and report the selected point estimate | Interpret p under the stated null and design; separate statistical compatibility from effect size and practical value | Inventory hypotheses, looks, subgroups, and specifications; use the planned multiplicity or sequential method, or replicate a frozen claim on untouched data |
| p > 0.05 | Report "no effect" | Ask what effect sizes the interval and design can actually rule out | Compare the interval with a prespecified smallest effect of interest; use an equivalence or non-inferiority design when that is the claim |
| "Significant in group A, not in B" | Recommend targeting A | The difference between significant and not significant is not itself evidence of a difference | Estimate and test the interaction or direct contrast on the effect scale with appropriate uncertainty |
| Only k of N invited responded | Analyze respondents as the population | Response can be selective; responders may differ from non-responders on outcome-related factors | Compare respondents to the frame on known covariates; bound the bias; state the population actually described |
| Many predictors, report the significant ones | Call them "drivers" | Account for selection before interpreting coefficient uncertainty or causality | Prespecify, split the data, use valid selective-inference methods, or validate chosen variables on a holdout |
| Small n, big clean effect | Treat the magnitude as precise | Ask how selection and low power distort magnitude and sign | At plausible true effects, simulate or derive the distribution conditional on the selection rule; seek independent replication |
| Clustered or repeated data (users × days, students × schools) | Use iid standard errors | Match the variance model to assignment, sampling, and dependence; effective information depends on cluster count, size, and within-cluster correlation | Recompute with justified cluster-robust, random-effects, randomization-based, or other design-consistent uncertainty |
| Model or covariates chosen by fit on the same data | Report the final model's ordinary p-values | Ordinary inference generally treats the model as fixed before seeing the data | Use prespecification, honest sample splitting, external validation, or a method that accounts for selection |
| Assumption checks demanded ("data aren't normal!") | Transform until a normality test passes | Identify which assumption supports which estimand and whether the procedure is robust at this sample size | Inspect claim-relevant residuals and influence; run a justified robust or randomization-based sensitivity analysis |
| Correlation between measured proxies | Interpret it as the relation between constructs | Model reliability, shared-method bias, range restriction, and construct validity | Use reliability or measurement models, a second measurement mode, and a sensitivity analysis tied to the target construct |
| Missing data | Default to complete cases or claim MCAR is always required | State the estimand and the exact missingness conditions under which the chosen estimator is valid; complete-case validity can depend on the model and target | Model missingness indicators, compare observed cases with the sampling frame, and test conclusions under defensible weighting, imputation, or bounds |
| Effect reported as mean difference | Assume the mean is the story | Heavy tails and outliers make the mean a claim about a few points | Plot the distributions; medians/quantiles as sensitivity; influence check (drop-k) |

## Precondition Template (O1)

**"This inferential claim is reportable":** name the estimand, target population,
sampling frame, and inclusion mechanism; match uncertainty to assignment,
sampling, clustering, repeated measures, and stopping; disclose hypothesis,
subgroup, model, and specification selection; interpret intervals against
decision-relevant effect sizes rather than only a threshold; assess influence and
missingness with methods appropriate to the design; distinguish confirmatory,
exploratory, and externally replicated evidence.

## Domain Escalation Triggers

- The recommendation rests on comparing a significant result to a
  non-significant one → stop; test the difference directly.
- Response or inclusion is differential, poorly understood, or plausibly related
  to the outcome → assess representativeness; response rate alone neither proves
  nor rules out nonresponse bias.
- The selected sample is small and the effect is large and tidy → examine the
  selection-conditioned magnitude and seek independent data before calling it
  stable.
- The same data chose the hypothesis, the variables, or the subgroup and now
  also tests it → label the result exploratory or use inference that accounts for
  selection; prefer fresh or held-out confirmation.
