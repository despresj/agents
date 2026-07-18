---
name: fp-python
description: Prevent plausible but silent failures in Python and pandas pipelines with explicit cardinality, alignment, missingness, type, environment, and conservation checks. Use when writing or debugging Python data work involving merges, groupby, reshaping, NumPy or pandas alignment, datetimes, mutable state, notebooks, imports, environments, or numerical results that will be reported.
---

# First Principles — Python

Apply the `first-principles` core skill. Use this pack for Python- and
pandas-specific silent failures and preconditions; use the core once for
checkpoints and reporting.

## The Governing Fact

Treat successful execution as insufficient evidence. Joins can fan out, groupby
can exclude missing keys, labels can align unexpectedly, and mutable state can
survive between notebook runs. Check the properties the result depends on.

## Conservation Checks — after every merge/groupby/reshape

```python
import numpy as np

merged = left.merge(
    lookup,
    on="key",
    how="left",
    validate="m:1",
    indicator=True,
)
if len(merged) != len(left):
    raise ValueError("unexpected row-count change")
unmatched_n = merged["_merge"].eq("left_only").sum()
if merged["revenue"].isna().sum() != left["revenue"].isna().sum():
    raise ValueError("revenue missingness changed")
if not np.isclose(merged["revenue"].sum(), left["revenue"].sum()):
    raise ValueError("revenue total changed")
```

Use the validation relationship that matches the intended cardinality. A declared
many-to-many merge does not check multiplicity, so measure and bound its fan-out
explicitly. Use durable runtime checks for production invariants: Python removes
plain `assert` statements when invoked with `-O`.

## Silent-Failure Catalog

| Trap | What silently happens | Defense |
|------|----------------------|---------|
| Merge fan-out | Duplicate right-side keys duplicate left rows — sums inflate | Declare the intended `validate=` relationship; check row-count change explicitly |
| Inner-join default | Unmatched rows vanish; totals shrink | `how="left"` deliberately + `indicator=True`; account for `left_only` |
| Null merge keys | pandas matches null keys to each other, unlike typical SQL joins | Decide whether null-to-null is valid; reject, sentinelize, or isolate null keys before merging |
| `groupby` drops NaN keys | Rows with NaN group silently excluded — segments don't sum to total | `dropna=False`, or fill a sentinel first |
| Index alignment | Arithmetic between Series aligns by label, not position; mismatched labels can introduce missing values or move values to unintended rows | Assert index equality; use `.to_numpy()` only when positional semantics are explicit |
| Chained assignment | Behavior varies on older pandas; under pandas 3 Copy-on-Write it cannot update the original | Assign in one operation with `.loc` or `.iloc`; test supported pandas versions and treat chained-assignment warnings as failures |
| Mutable default args | `def f(x, acc=[])` shares one list across calls | Default `None`, create inside |
| Nested mutable objects | Assignment and shallow copies share objects; even pandas `deep=True` does not recursively copy Python objects stored in `object` columns | Normalize nested data or use `copy.deepcopy()` deliberately; test mutation isolation |
| Late-binding closures | Lambdas in a loop all capture the final loop value | Bind at definition: `lambda i=i: ...` |
| `is` vs `==` | `is` "works" for small ints/interned strings, then breaks | `==` for values; `is` only for `None`/sentinels |
| Naive datetimes | A timestamp without a zone cannot identify a unique instant; libraries may reject or implicitly localize mixed operations | Attach the source zone at ingestion, convert instants to UTC, and retain a zone only when calendar semantics require it |
| Float equality / money | `0.1 + 0.2 != 0.3`; cents drift | `math.isclose`/`np.isclose`; `Decimal` or integer cents for money |
| `object` dtype creep | Mixed Python types can change comparison, sorting, missingness, and vectorization behavior | Check dtypes at boundaries; coerce explicitly and inspect values that become missing |
| Bare `except:` | Swallows the real error, including `KeyboardInterrupt` | Catch the narrowest exception; log, don't pass |
| Exhausted iterators | Generators yield nothing the second time — empty loop, no error | Materialize with `list()` when reused |
| Wrong environment | `pip` installs to a different interpreter than the one running | `python -m pip`; verify `sys.executable` |

## Precondition Template (O1)

**"This analysis output is reportable":** every merge declares and verifies its
intended cardinality; row-count changes are reconciled; totals and missingness are
conserved or the difference is itemized; null-key behavior is intentional; index
semantics and dtypes are checked at boundaries; input, package, locale, and time
zone versions are recorded; the result reproduces from top to bottom in a fresh
interpreter rather than depending on notebook execution order or process state.

## Domain Escalation Triggers

- Two computations of the same quantity disagree → alignment, NaN handling, or
  dtype is a live rival; reconcile it before reporting.
- A number changed on re-run of unchanged code → hidden state (notebook order,
  mutation, seed, input, environment, or library version). Localize it and include
  any remaining variability in the claim.
- Segment/group sums ≠ overall total → rows are being dropped (NaN keys,
  filtering, or inner join) or counted more than once; reconcile membership before
  transforming the result.
