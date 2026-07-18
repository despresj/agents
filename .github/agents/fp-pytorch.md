---
name: fp-pytorch
description: Debug PyTorch training, inference, and models before tuning hyperparameters or architecture. Use when training curves diverge from expectations, models produce NaN or very large losses, validation metrics plateau or diverge from training, reproducing results fails, or a model behaves differently on different hardware or batch sizes.
---

# First Principles — PyTorch

Apply the `first-principles` core skill. Use this pack for PyTorch-specific traps
and precondition templates; use the core once for checkpoints and reporting.

## Standing Checks — before tuning anything

1. **Verify the training loop is actually training.** Set one batch to a tiny
   learning rate and confirm training loss decreases on that batch alone. If it
   doesn't, the model weights are not being updated — stop and find why.
2. **Inspect one batch end-to-end:** input shape, after embedding/preprocessing,
   after the forward pass, the loss value, and the gradient magnitude for a
   parameter. Batch shape errors and dtype mismatches hide in this path.
3. **Check that validation is in eval mode:** `model.eval()` and `torch.no_grad()`
   matter for BatchNorm, Dropout, and other stateful layers. A forgotten `.eval()`
   makes validation meaningless.
4. **Confirm the learning rate is actually being used.** Print
   `optimizer.param_groups[0]['lr']` and `optimizer.state_dict()` before the
   first step. LR scheduler bugs and `optim.zero_grad()` in the wrong place
   silently break training.

## Trap Table

| Symptom | Tempting move | First-principles move | Discriminating test |
|---------|--------------|----------------------|---------------------|
| Training loss drops, validation plateaus then rises | Add dropout / reduce LR / add weight decay | Separate the signals: is validation data different from training, or is the model overfitting? | Hold-out validation set vs train on validation set — if train metric is also high, the data differs |
| Loss NaN or very large on the first step | Lower LR, clip gradients | Find the first NaN: which operation? Check for division by zero, log of negative, softmax over huge logits | Instrument: print activations/logits per layer; NaN always has a source layer |
| Train and validation loss diverge | The model is overfitting | Separate: is validation measured in `.eval()` mode? Does the model use different code paths (dropout, BN) at train/eval? | Run the same batch in train and eval mode; metrics must match if the model is working correctly |
| Results differ between runs (different seed) | Random initialization or data order | Some variation is normal; instability suggests architecture or data issues | Fix seed + run 3 times; if loss curves diverge wildly, the model is underspecified or the data has hidden dependence |
| Model trains fine locally but fails on multi-GPU or different hardware | Code is hardware-agnostic | Distributed training changes batch-norm statistics, gradient accumulation, synchronization; dtype precision; and initialization order | Run single-GPU vs multi-GPU on identical data and seed; any difference signals sync or dtype problem |
| Validation metric is high but model is overconfident (top class ≠ true class, but softmax max >> 0.9) | The model is well-calibrated | Overconfidence + high accuracy = overconfidence on the right answer, which is fine; overconfidence + high error = a model that's wrong with high certainty | Compare softmax max / entropy between correct and incorrect predictions; if wrong predictions have higher max, the model is learning spurious features |
| Gradient accumulation doesn't improve convergence | Accumulation is working | Gradient accumulation fools the learning rate schedule and batch-norm statistics. A schedule keyed to step (not sample) may no longer fit. | With accumulation, multiply gradient norm and loss smoothness checks by the accumulation steps; check that BN statistics match single-step runs with larger batch |
| Pre-trained model fine-tuning plateaus immediately | Frozen earlier layers or wrong LR | A fine-tuned model often needs different LRs for different layers (lower for pretrained, higher for new heads). A uniform LR meant for training-from-scratch doesn't work. | Inspect which layers are actually being updated (check `param.grad` is not None) and compare LR schedules per layer vs uniform |

## Precondition Template (O1)

**"This model training result is correct":** the one-batch test passes (gradients
flow, loss decreases); forward and backward passes match `.train()` and `.eval()`
intent; validation uses `model.eval()` and `torch.no_grad()`; the learning rate
is logged and inspected, not assumed; seed is fixed and results replicate across
3+ runs; batch-size and data-order changes are understood and their effect
measured; pre-trained layer learning rates differ from new layers if
fine-tuning; multi-GPU and single-GPU training produce identical loss curves on
identical data.

## Domain Escalation Triggers

- Training loss decreases but validation metric doesn't improve → the two are
  measuring different things (data, mode, metric code); audit both before
  concluding overfitting.
- Model performance depends on hardware, batch size, or initialization seed →
  reproducibility is broken; find the source (unsync'd RNG, dtype, batch-norm
  momentum, data order).
- Fine-tuning a pretrained model plateaus → layer-wise learning rates are
  likely needed; uniform LR doesn't work for heterogeneous layer ages.
- Gradient explosion/vanishing with a well-known architecture → check if the
  model code exactly matches the original (skip connections, normalization
  placement) before blaming initialization or learning rate.
