---
name: kendall-tau-rank-correlation-evaluation
description: Use when benchmarking trajectory-inference methods on data with a known
  ground-truth cell ordering (e.g. PhenoPath simulate_phenopath() outputs with latent
  z), especially to quantify how accuracy degrades as the fraction of covariate-pseudotime
  interaction genes grows.
when_to_use_negative:
- Real data with no ground-truth latent ordering exists — Kendall-tau against simulated
  z cannot be computed, so use qualitative validation (e.g. checks that PhenoPath
  model fits are consistent with the data) instead
- Branching or cyclic trajectories where a single one-dimensional latent z is not
  the correct reference ordering
- Evaluating a method specifically designed for covariate-modulated trajectories on
  data with zero interaction genes, where 'de-only' simulation regimes make all methods
  roughly comparable and the benchmark is uninformative
tools:
- name: PhenoPath
  role: Method under evaluation; fitted with phenopath() using ELBO-based convergence
    (elbo_tol, thin) and PC initialisation; its trajectory() extractor supplies the
    inferred pseudotime for correlation against sim$z
  repo: https://bioconductor.org/packages/release/bioc/html/phenopath.html
- name: Monocle 2
  role: Competing pseudotime method run on the same simulated matrices to obtain a
    baseline trajectory for Kendall-tau scoring
  repo: https://bioconductor.org/packages/release/bioc/html/monocle.html
- name: dplyr
  role: Aggregates replicate Kendall-tau values to the median per method per interaction
    fraction
- name: tidyr
  role: Reshapes the per-replicate correlation results into the method-by-fraction
    summary table
- name: ggplot2
  role: Plots median Kendall-tau vs covariate-interaction fraction to reproduce the
    Fig 1e benchmark trend
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-018-04696-6
    title: Uncovering pseudotemporal trajectories with covariates from single cell
      and bulk expression data
  source_content_hash: sha256:b198640ab718ea1662c158f0bf3ba6ce12cbd96b742da6868f99020749509350
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Monocle 2
  - PhenoPath
  - R
  - dplyr
  - ggplot2
  - tidyr
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# kendall-tau-rank-correlation-evaluation

## Summary

Scores inferred single-cell pseudotime trajectories against a known latent ordering (e.g. simulated latent z) using Kendall-tau rank correlation, then aggregates medians per method and per covariate-interaction fraction. This is the accuracy metric used to benchmark PhenoPath against Monocle 2, DPT, Wishbone and TSCAN (Fig 1e of Campbell & Yau 2018).

## When to use

Use when benchmarking trajectory-inference methods on data with a known ground-truth cell ordering (e.g. PhenoPath simulate_phenopath() outputs with latent z), especially to quantify how accuracy degrades as the fraction of covariate-pseudotime interaction genes grows.

## When NOT to use

- Real data with no ground-truth latent ordering exists — Kendall-tau against simulated z cannot be computed, so use qualitative validation (e.g. checks that PhenoPath model fits are consistent with the data) instead
- Branching or cyclic trajectories where a single one-dimensional latent z is not the correct reference ordering
- Evaluating a method specifically designed for covariate-modulated trajectories on data with zero interaction genes, where 'de-only' simulation regimes make all methods roughly comparable and the benchmark is uninformative

## Inputs

- Simulated gene expression matrices from PhenoPath simulate_phenopath() with known latent trajectory z (gene regimes: 'de-only', 'pst', 'pst-beta'; interaction-fraction sweep 5-50%; sample sizes 200 and 500; 40 replicates per setting)
- Inferred pseudotime vectors from each method (phenopath(), Monocle 2, DPT, Wishbone, TSCAN)

## Outputs

- Per-replicate Kendall-tau rank correlation between inferred pseudotime and true latent z for every method
- Median Kendall-tau summary table per method per interaction fraction, written to kendall_tau_pseudotime_comparison.csv
- Optional ggplot2 trend plot reproducing the qualitative shape of Fig 1e

## How to apply

For each simulated expression matrix (generated with simulate_phenopath() sweeping the covariate-interaction gene fraction from 5% to 50% across 'de-only', 'pst' and 'pst-beta' regimes, n=200 or 500 cells, high/low noise, 40 replicates), fit each method: phenopath() with ELBO-based convergence (elbo_tol, thin) and principal-component initialisation, plus the competing pseudotime methods. Extract each inferred trajectory and compute the Kendall-tau rank correlation between inferred pseudotime and the true latent z, analogous to cor(sim$z, trajectory(fit)). Aggregate replicate correlations to the median per method per interaction fraction with dplyr/tidyr and write kendall_tau_pseudotime_comparison.csv. Correct application means PhenoPath's tau is approximately independent of the interaction fraction while Monocle 2, DPT, Wishbone and TSCAN diminish above ~10% interaction genes.

## Related tools

- **PhenoPath** (Method under evaluation; fitted with phenopath() using ELBO-based convergence (elbo_tol, thin) and PC initialisation; its trajectory() extractor supplies the inferred pseudotime for correlation against sim$z) — https://bioconductor.org/packages/release/bioc/html/phenopath.html
- **Monocle 2** (Competing pseudotime method run on the same simulated matrices to obtain a baseline trajectory for Kendall-tau scoring) — https://bioconductor.org/packages/release/bioc/html/monocle.html
- **dplyr** (Aggregates replicate Kendall-tau values to the median per method per interaction fraction)
- **tidyr** (Reshapes the per-replicate correlation results into the method-by-fraction summary table)
- **ggplot2** (Plots median Kendall-tau vs covariate-interaction fraction to reproduce the Fig 1e benchmark trend)

## Examples

```
fit <- phenopath(sim$Y); tau <- cor(sim$z, trajectory(fit), method = "kendall")
```

## Evaluation signals

- The summary table contains exactly one median Kendall-tau row per method per interaction fraction
- PhenoPath's median tau is approximately independent of the interaction fraction, matching the qualitative trend of Fig 1e
- Median tau for Monocle 2, DPT, Wishbone and TSCAN rapidly diminishes once the fraction of covariate-interaction genes exceeds ~10%
- Per-replicate tau values fall in the expected [-1, 1] range, with successful methods approaching high positive correlation with sim$z
- kendall_tau_pseudotime_comparison.csv is written and parses cleanly with dplyr/tidyr

## Limitations

- PhenoPath assumes features evolve linearly with respect to pseudotime; nonlinear effects may occur and are not discounted, so model fits should be checked against the data
- The competing methods (Monocle 2, DPT, Wishbone, TSCAN) do not account for covariate-trajectory interaction effects, so the benchmark is structurally favourable to PhenoPath at high interaction fractions
- Kendall-tau against a single latent z only applies when the truth is one unbranched one-dimensional trajectory

## Evidence

- [readme] PhenoPath learns genomic trajectories (pseudotimes) in the presence of heterogenous environmental and genetic backgrounds encoded as additional covariates: "PhenoPath learns genomic trajectories (pseudotimes) in the presence of heterogenous environmental and genetic backgrounds encoded as additional covariates"
- [methods] We provide an R implementation of our method PhenoPath at https://bioconductor.org/packages/release/bioc/html/phenopath.html.: "We provide an R implementation of our method PhenoPath at https://bioconductor.org/packages/release/bioc/html/phenopath.html."
- [intro] a comprehensive catalogue of single-cell pseudotime algorithms can be obtained from https://github.com/agitter/single-cell-pseudotime: "a comprehensive catalogue of single-cell pseudotime algorithms can be obtained from https://github.com/agitter/single-cell-pseudotime"
