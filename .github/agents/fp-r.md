---
name: fp-r
description: Prevent plausible but silent failures in R data and statistical pipelines with explicit type, cardinality, missingness, and conservation checks. Use when writing or debugging base R, dplyr, or data.table code involving joins, aggregation, factors, dates, time zones, reshaping, or reportable numerical results—especially when code runs without errors but its output is not yet verified.
---

# First Principles — R

Apply the `first-principles` core skill. Use this pack for R-specific silent
failures and preconditions; use the core once for checkpoints and reporting.

## The Governing Fact

Treat successful execution as insufficient evidence. Recycling, coercion, missing
value handling, joins, and reference semantics can produce plausible output
without an error. Assert the properties the result depends on.

## Conservation Checks — after every join/aggregate, before trusting output

```r
stopifnot(anyDuplicated(lookup$key) == 0L)       # expected many-to-one join
stopifnot(nrow(joined) == nrow(left))            # no unexplained fan-out or loss
unmatched_n <- sum(is.na(joined$.lookup_matched)) # add a right-side sentinel first
stopifnot(unmatched_n == expected_unmatched_n)
stopifnot(!anyNA(out$total), !anyNA(input$amount))
stopifnot(isTRUE(all.equal(
  sum(out$total), sum(input$amount),
  tolerance = 1e-8
)))                                               # totals conserved or explained
```

Choose checks that match the intended cardinality. Do not force a row-count or
total invariant when the transformation is meant to change it; state and test the
expected change instead.

## Silent-Failure Catalog

| Trap | What silently happens | Defense |
|------|----------------------|---------|
| Vector recycling | Unequal lengths recycle without error (warning only if non-multiple) | `stopifnot(length(a) == length(b))` where equality is assumed |
| `merge()` defaults | A natural inner join uses every shared column, drops unmatched rows, and fans out duplicate keys | Set `by=` and join type explicitly; assert key cardinality; compare row counts |
| NA in aggregation | `mean()` returns NA; formula methods often omit rows under the configured `na.action` | Count missing values first; set and report the intended policy |
| `NA == x` | Yields NA, not FALSE; filtering behavior then differs by API and may drop or preserve unknown rows unexpectedly | Handle `is.na()` explicitly; use `%in%` only when its never-NA membership semantics are intended |
| `as.numeric(factor)` | Returns internal level codes, not displayed numeric values | Convert through character with an explicit failed-parse check; inspect `class()` and levels at boundaries |
| `ifelse()` | Strips class — Dates/POSIXct become numbers | `dplyr::if_else()` or index assignment |
| `sapply()` | Return type depends on data — list one day, matrix the next | `vapply()` with declared type |
| `apply()` on data.frame | Coerces to matrix; one character column makes everything character | Column-wise `lapply`/`across()` |
| `[ , drop=TRUE]` | Single-column subset silently becomes a vector | `drop = FALSE`, or tibbles |
| `$` partial matching | `df$val` matches `df$value` — typos return data, not errors | `[[ "name" ]]` in code; `options(warnPartialMatchDollar = TRUE)` |
| Function masking | `filter`, `select`, `lag` shadowed after loading packages | Namespace-qualify in scripts: `dplyr::filter` |
| Floating point `==` | Binary representation makes exact decimal equality unreliable | `isTRUE(all.equal(...))` or a scale-appropriate tolerance |
| data.table reference semantics | `:=` modifies the original that other names point to | `copy()` when independence is intended |
| Time-zone defaults | Parsing or conversion can depend on the supplied or system zone; calendar dates can shift across machines | Set and verify the zone at ingestion and conversion boundaries; store instants in UTC when appropriate |

## Precondition Template (O1)

**"This pipeline output is reportable":** explain expected row-count changes at
joins and filters; track missingness in and out; conserve totals or reconcile the
difference; assert intended key cardinality; inspect column classes, factor
levels, encodings, and time zones at boundaries; record package and data versions;
reproduce the result in a clean session such as `R --vanilla`, not only the
current workspace.

## Domain Escalation Triggers

- A number changed when you re-ran unchanged code → hidden state (environment,
  seed, data.table reference, input, locale, or package version). Localize it and
  include the resulting variability or dependency in the claim.
- `summary()`/`str()` shows an unexpected class or NA count → coercion happened
  somewhere upstream; trace it before proceeding.
- Group counts differ between two "equivalent" aggregations → NA-handling or
  factor-level differences are live rivals; reconcile them before reporting.
