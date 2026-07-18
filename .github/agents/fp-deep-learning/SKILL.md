---
name: fp-deep-learning
description: Diagnose neural-network training and evaluation claims by checking pipelines, controls, stochastic variation, and contamination before tuning. Use when training or fine-tuning models, debugging loss curves or instability, comparing runs, evaluating architecture or hyperparameter changes, reproducing results, or assessing benchmarks and unexpectedly strong or weak evaluation scores.
---

# First Principles — Deep Learning

Apply the `first-principles` core skill. Use this pack for deep-learning traps and
precondition templates; use the core once for checkpoints and reporting.

## Standing Rule

When debugging training or evaluation, inspect at least eight varied examples end
to end before changing architecture, hyperparameters, or optimizer: raw input,
preprocessing, augmentation, label, mask or weights, model input, and exactly what
the loss and metric consume. Pipeline defects often imitate optimization or
capacity problems.

## Trap Table

| Symptom | Tempting move | First-principles move | Discriminating test |
|---------|--------------|----------------------|---------------------|
| Metric improved after your change | Credit the change | Isolate the intended variable and quantify retrain variance | Use matched seeds when possible and enough independent repeats to resolve the observed effect; treat three seeds as a smoke test, not proof |
| Eval score surprisingly strong | Ship or publish | Test contamination, benchmark reuse, and evaluation-code errors before attributing capability | Audit provenance and train-eval overlap; evaluate on a fresh or independently constructed set; probe with controlled perturbations |
| Loss flat or not decreasing | Use a larger model or broad learning-rate sweep | Separate pipeline, objective, optimization, data, and capacity hypotheses | Overfit a tiny batch with augmentation and regularization disabled; inspect gradients and labels if it fails |
| Validation metric better than training | Call it impossible or celebrate | First check legitimate train/eval asymmetries such as dropout, augmentation, regularization terms, batch normalization, and within-epoch timing | Recompute both datasets at the same checkpoint in evaluation mode with the same metric; audit split difficulty or leakage if the gap remains unexplained |
| Training loss contradicts a claimed theoretical floor | Ignore it or declare leakage immediately | Verify that the floor applies to the exact finite sample, loss, reduction, mask, weighting, and model inputs | Recompute the bound in the same units; run label and feature leakage controls if the contradiction remains |
| NaN/instability | Lower LR globally | Find the specific operation producing the first NaN | Instrument: anomaly detection, per-layer norms, check batch that triggers it |
| "It converges, so it's correct" | Trust the model | Convergence ≠ learning the intended thing | Shuffled-label control (should fail); canary examples; probe shortcuts |
| Improvement in a paper or blog reproduces poorly | Blame the implementation | Reconstruct the baseline, data, compute, tuning budget, checkpoint rule, and evaluation protocol | Match material conditions or ablate each remaining difference; compare distributions, not only best runs |

## Precondition Template (O1)

**"Change X improved the model":** runs differ only in X and unavoidable
interactions are retuned symmetrically; compute and search budgets are comparable;
matched-seed or repeated runs estimate retrain variation with enough precision for
the claimed effect; checkpoint selection and early stopping are identical;
training and evaluation data provenance is audited; preprocessing and metric code
are shared; the result holds on an untouched test set or an explicitly independent
slice; report the distribution and search process, not only the best run.

## Domain Escalation Triggers

- Improvement reverses or disappears on another seed → treat the claim as fragile;
  estimate the effect across enough runs before calling it signal or noise.
- Score approaches or beats published SOTA unexpectedly → assume contamination
  or eval bug until excluded.
- Two nominally identical runs vary more than the documented stochastic process
  predicts → inventory and control randomness, data order, kernels, hardware, and
  checkpoint selection; if variance remains, make it part of the result.
- A tiny-batch overfit or shuffled-label control behaves opposite to expectation →
  stop tuning and inspect the pipeline, objective, and evaluation code.
