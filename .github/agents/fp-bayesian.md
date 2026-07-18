---
name: fp-bayesian
description: Stress-test Bayesian models, computation, priors, and posterior claims with generative checks and sensitivity analysis. Use when building or fitting probabilistic models, diagnosing MCMC problems such as divergences, high R-hat, or low ESS, choosing priors, working with hierarchical or mixture models, or reporting posterior estimates in Stan, PyMC, brms, NumPyro, or similar tools.
---

# First Principles — Bayesian Modeling

Apply the `first-principles` core skill. Use this pack for Bayesian workflow gates,
traps, and preconditions; use the core once for checkpoints and reporting.

## Workflow Gates — these are ACT gates, not suggestions

1. **Run a prior predictive check before trusting the prior.** Simulate outcomes
   on the domain scale. Revise priors that put material mass on implausible data,
   unless that breadth is intentional and justified.
2. **Run simulation recovery before trusting new or complex model code.** Simulate
   data in the relevant sample-size and parameter regimes, fit the model, and
   check recovery of the load-bearing estimand. Treat one synthetic fit as a
   smoke test; use repeated simulation or simulation-based calibration when the
   claim requires calibration.
3. **Check computation before interpreting draws.** Inspect divergences, R-hat,
   bulk and tail ESS, Monte Carlo error, tree depth or energy diagnostics when
   available, and chain behavior for load-bearing quantities.
4. **Run posterior predictive checks before reporting model adequacy.** Compare
   replicated and observed data on features tied to the claim. Convergence only
   addresses computation, not whether the model represents the data usefully.
5. **Run a sensitivity ladder before strong claims.** Refit under defensible
   alternative priors and modeling choices; report which estimands move.

Skipping a gate must appear in the CONCLUDE block as an undischarged assumption.

## Trap Table

| Symptom | Tempting move | First-principles move | Discriminating test |
|---------|--------------|----------------------|---------------------|
| Divergent transitions | Increase `adapt_delta` and stop investigating | Use a smaller step size as a diagnostic or mitigation, then inspect curvature, funnels, constraints, and identification | Locate divergences in parameter and energy plots; compare reparameterizations and the stability of the target estimand |
| R-hat and ESS look acceptable | Trust the model and fit | Treat scalar diagnostics as necessary but not sufficient for missed modes, ridges, coding errors, or model mismatch | Use overdispersed initializations, rank and pairs plots, simulation recovery, and claim-relevant predictive checks |
| Posterior resembles prior for a parameter | Declare that the data contain no information | Quantify the update for the estimand; marginal overlap alone does not establish prior dominance | Compare prior and posterior on the estimand scale and repeat under defensible alternative priors |
| Flat priors "to be objective" | Use them without checking scale or propriety | Recognize that flatness is parameterization-dependent and may yield a problematic posterior | Run prior predictive checks and verify posterior propriety and computation under meaningful scales |
| Chains converged | Conclude the model is right | Convergence ≠ correctness | Posterior predictive checks against held-out structure |
| Hierarchical scale near zero with high R-hat | Tune the sampler only | Diagnose boundary behavior, weak group information, and centered versus non-centered geometry | Compare parameterizations and recover known group variation across realistic simulated datasets |
| Multimodal posterior in mixtures | Report component-wise means | Label switching can make labeled component summaries meaningless | Inspect chains by mode; use identified structure when justified or report permutation-invariant quantities |
| Report due, diagnostics failing | Report estimates with a generic caveat | Treat affected estimates as computationally unresolved | Localize which estimands fail, demonstrate recovery or failure on simulation, and report only quantities supported by diagnostics |

## Precondition Template (O1)

**"Posterior estimate of θ is reportable":** define θ on the decision-relevant
scale; verify the implementation with simulation appropriate to the claim;
establish acceptable computation for θ using divergences, R-hat, ESS, and Monte
Carlo error; quantify prior influence rather than inferring it from overlap alone;
pass posterior predictive checks on claim-relevant features; show sensitivity to
defensible priors and model choices; disclose any remaining identification or
computational limits.

## Frame Audit Reminder (O5)

Ask what the report claims before tuning the sampler. Hospital comparisons,
rankings, and league tables can depend heavily on group-level shrinkage, so the
decision may be sensitive to the same parameters whose geometry is difficult.
Treat "fix the divergences" as a subproblem of "is this posterior fit for the
decision?"

## Domain Escalation Triggers

- Divergences or poor mixing cluster around a load-bearing parameter → stop
  interpreting that parameter until the geometry is resolved or the limitation is
  explicit.
- The substantive conclusion changes under a defensible prior or parameterization
  → report the sensitivity, not a preferred fit.
- Predictive checks pass while the structural or causal estimand fails simulation
  recovery → do not use predictive adequacy to defend that estimand.
