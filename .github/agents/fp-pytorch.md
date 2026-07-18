---
name: fp-pytorch
description: Diagnose PyTorch-specific silent failures in training loops, autograd, modules, data loading, and mixed precision before tuning hyperparameters or architecture. Use when losses stall, spike, or go NaN; gradients are None or zero; train and eval behavior differ unexpectedly; memory grows across epochs; runs fail to reproduce; or results change across devices, batch sizes, or DataLoader settings.
---

# First Principles — PyTorch

Apply the `first-principles` core skill. Use this pack for framework-level traps
in PyTorch code; use `fp-deep-learning` for experiment methodology (contamination,
run comparison, seed variance) and the core once for checkpoints and reporting.

## The Governing Fact

PyTorch fails silently and plausibly: a loop with a misplaced `zero_grad()`, a
softmax fed into `CrossEntropyLoss`, or a missing `.eval()` still runs, converges,
and produces a number. Successful execution discharges nothing (O2). Verify the
loop's mechanics before interpreting its curves.

## Standing Checks — before tuning anything

1. **Overfit one batch to ~zero loss** (normal LR; augmentation and regularization
   off). A healthy loop memorizes 32 examples in a few hundred steps. If it
   can't, the defect is mechanical — data, objective, or optimizer wiring — and
   no hyperparameter search will find it.
2. **Trace one batch end-to-end:** raw input → transforms → tensors (shape,
   dtype, device) → logits → loss → `loss.backward()` → a sampled `param.grad`
   is non-None and nonzero → `optimizer.step()` changes that weight. Every arrow
   is a place the graph can silently detach.
3. **Verify the mode contract:** `model.train()` for updates; `model.eval()` +
   `torch.no_grad()` for validation; `model.train()` restored afterward. Dropout
   and BatchNorm make train/eval outputs *legitimately* differ — the bug is
   running in the wrong mode, not the difference itself.
4. **Log what the optimizer actually sees:** per-group LR after the scheduler,
   which parameters sit in which group, and the count of `requires_grad`
   parameters. Frozen-by-accident and never-registered parameters (stored in
   plain Python lists instead of `nn.ModuleList`/`nn.Parameter`) train nothing
   and raise nothing.

## Silent-Failure Catalog

| Trap | What silently happens | Discriminating check |
|------|----------------------|---------------------|
| Missing or misplaced `optimizer.zero_grad()` | Gradients accumulate across steps; effective batch size and LR drift from what you think | Compare a sampled `param.grad` before/after `zero_grad()`; assert grads are zero (or None) at loop top |
| Softmax applied before `nn.CrossEntropyLoss` | Double log-softmax: training still "works" but gradients shrink and accuracy caps early | Read the head: `CrossEntropyLoss` takes raw logits; check output layer for `softmax`/`log_softmax` |
| Accumulated loss not divided by accumulation steps | Effective LR multiplied by the accumulation factor | Compare gradient norm at equal effective batch with and without accumulation; they should match |
| Metric accumulation retains the graph (`total += loss`) | Memory grows every step; OOM arrives mid-epoch with a misleading stack trace | Use `loss.item()`/`.detach()`; watch `torch.cuda.memory_allocated()` across steps — it should plateau |
| Graph silently detached (`.item()`, `.numpy()`, `.data`, stray `no_grad`) | `param.grad` is None; those layers never learn | After one backward, assert every trainable parameter has a non-None grad |
| In-place op on a tensor needed for backward | Sometimes a runtime error, sometimes wrong gradients, version-dependent | `torch.autograd.set_detect_anomaly(True)` on a small run |
| `scheduler.step()` called per-batch when tuned per-epoch (or vice versa) | The LR schedule runs orders of magnitude off | Plot the realized LR from `param_groups` per step, not the intended schedule |
| DataLoader workers share augmentation RNG | Identical "random" augmentations across workers or epochs | Log augmentation params from two workers on the same epoch; set `worker_init_fn` seeding |
| AMP: gradient clipping before `scaler.unscale_()` | Clipping applies to scaled grads — the clip threshold is meaningless | Order in code: `unscale_` → clip → `scaler.step()`; check grad norms are in the expected range |
| fp16/bf16 overflow producing NaN | Loss goes NaN at some step with no error | Find the first non-finite tensor: anomaly mode or `torch.isfinite` hooks per layer; NaN always has a source op |
| BatchNorm with tiny per-device batches under DDP | Running stats are noisy and differ from single-GPU; eval degrades though training looks fine | Expected difference — discriminate with `SyncBatchNorm` or larger per-device batch, don't hunt a "sync bug" first |
| Labels out of range / `ignore_index` mismatch | CE loss computed over wrong classes; loss plausible, learning wrong | Assert `labels.min() >= 0` and `labels.max() < num_classes`; count `ignore_index` hits |

## Precondition Template (O1)

**"This training result is mechanically sound":** the one-batch overfit passes;
the gradient path is verified end-to-end (no None grads among trainables); the
mode contract is followed in both loops; the optimizer's realized LR and
parameter groups are logged, not assumed; loss normalization matches the
accumulation scheme; memory plateaus across an epoch; determinism status is
known (seeded, `use_deterministic_algorithms` decision explicit) before
attributing run-to-run differences; AMP ordering (`unscale_` → clip → step) is
respected where used.

## Domain Escalation Triggers

- Some `param.grad` is None after backward → part of the graph is detached or
  parameters aren't registered; nothing downstream is interpretable until found.
- Memory grows monotonically across steps → a tensor with history is being
  retained; find it before any OOM-driven batch-size change.
- Validation diverges from training → audit mode, data parity, and metric code
  first; only then treat it as overfitting (see `fp-deep-learning`).
- Seeded runs still differ → nondeterministic kernels or worker RNG; make the
  nondeterminism explicit and measured rather than tuning through it.
