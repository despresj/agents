---
name: fp-analytics-frontend
description: Design and audit analytical interfaces so visual encodings, score semantics, uncertainty, missingness, freshness, and actions remain faithful to the evidence. Use when building or reviewing dashboards, BI views, charts, model-output UIs, or data apps in React, Streamlit, Dash, Shiny, or mockups—especially when users ask to show everything or the display will drive decisions, targeting, ranking, or spend.
---

# First Principles — Front Ends for Analytical Products

Apply the `first-principles` core skill. Use this pack for analytical-interface
traps and preconditions; use the core once for checkpoints and reporting.

## The Governing Facts

1. **Every pixel is a claim.** A two-decimal score claims that precision exists.
   A red badge claims a threshold is meaningful. A default sort claims a
   ranking is trustworthy. Match each encoding to evidence the analysis supports.
2. **The viewer's decision is the unit of design.** Not the dataset. Start every
   surface from "what will this person decide differently after looking?" A
   surface whose decision cannot be named is not yet scoped (O5).
3. **The UI inherits every unresolved modeling question.** Uncalibrated scores,
   predictive-not-causal attributions, silently dropped rows — the front end is
   where these stop being footnotes and start changing behavior.

## Trap Table

| Symptom | Tempting move | First-principles move | Discriminating test |
|---------|--------------|----------------------|---------------------|
| "Show everything" request | Build the kitchen sink | Name each audience's decision; one surface per decision | For each widget: which decision consumes it? None → cut |
| Model scores displayed | Show `0.91` with a red badge | Define whether the score is a probability, rank, distance, category, or decision rule and verify its stability | Check calibration and ranking stability on deployment-like data; show only precision and semantics those checks support |
| Feature attributions such as SHAP shown to decision-makers | Label a "top driver" and recommend action | Keep predictive attribution distinct from causal effect | If no experiment or defensible causal design identifies intervention effects, label the feature as associated and remove causal action language |
| Targeting or spend list ranked by risk | Sort by risk × value | Risk ranking is not incremental treatment-effect ranking | On randomized or defensibly identified data, estimate treatment-control differences within pre-treatment score strata and evaluate the proposed policy |
| Missing data in a chart | Encode missing as zero, an ordinary gap, or a valid category | Distinguish absent measurement from measured absence | Show missingness explicitly and report its rate; verify that filtering, scales, and summaries do not silently discard it |
| Point estimates on display | Show clean single numbers | Display uncertainty when it could change the decision; avoid unsupported digits | Test whether plausible intervals cross an action threshold or reorder candidates; encode the consequential uncertainty |
| Aggregate KPI tiles | Show the mean | Aggregation hides distribution; means hide the tail that matters | Would the decision change seeing the distribution? Show it |
| Interactive filters on everything | Maximize flexibility | Treat free slicing as an exploratory analysis surface with multiplicity and sparse-slice risk | Show filtered n and coverage; define evidence-based suppression or warning rules; keep confirmatory claims tied to a frozen view |
| Two metrics side by side | Assume comparability | Reconcile denominators, windows, units, eligibility, and time zones | Display accessible definitions in context and compute both metrics on a shared test fixture before shipping |
| Data freshness | Assume current | Make staleness part of the decision state | Show the relevant as-of time and visibly flag or block actions when freshness exceeds a justified threshold |
| Truncated bars, dual axes, or unordered color scales | Choose the most dramatic encoding | Preserve the comparison the mark implies | Start bar charts at zero; use a scale matched to ordered or diverging data; avoid dual axes unless the mapping remains unambiguous under inspection |
| Meaning conveyed only by color, hover, or a graphic | Add a legend and move on | Provide redundant labels, keyboard and touch access, contrast, and a text or tabular equivalent | Complete the decision task without color perception, a mouse, or access to the graphic |
| Demo with clean data | Ship it | Empty, loading, partial, and error states ARE the product for data apps | Render with: zero rows, one row, nulls, 10× volume, failed fetch |

## Precondition Template (O1)

**"This dashboard is ready to drive decisions":** name the decision and audience
for each surface; trace each number to its definition and source; verify score,
threshold, ranking, and attribution semantics; show consequential uncertainty;
keep nulls distinct from zeros; expose sample size, coverage, and freshness where
they affect interpretation; make definitions keyboard- and touch-accessible; do
not rely on color alone; provide text or tabular access to important chart data;
test empty, partial, stale, error, one-row, and high-volume states.

## Domain Escalation Triggers

- You are about to attach an action recommendation to a predictive attribution
  → stop; identify the causal evidence or remove the recommendation (O3).
- A stakeholder wants a threshold colored red/green → ask what decision flips at
  that threshold, how it was chosen, and who validated it.
- A filter changes the population, denominator, or missingness rate → update the
  semantics and uncertainty, not only the marks.
- The mockup only works with the happy-path fixture → the real product is the
  degraded states; design them now.
