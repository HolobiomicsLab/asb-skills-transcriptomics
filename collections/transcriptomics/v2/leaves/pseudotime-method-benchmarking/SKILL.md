---
name: pseudotime-method-benchmarking
description: Use when comparing how pseudotime algorithms degrade as the fraction
  of genes with covariate-pseudotime interactions increases, or when validating a
  new trajectory method against standard ones on simulated data with a known latent
  time.
when_to_use_negative:
- The dataset lacks a ground-truth or simulated latent time, so no rank correlation
  against truth can be computed
- The goal is biological trajectory discovery on real data rather than quantitative
  method comparison
- Genes are expected to evolve nonlinearly with pseudotime, since PhenoPath (used
  as the reference implementation) assumes features evolve linearly with respect to
  pseudotime
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2269
tools:
- name: PhenoPath
  role: Generates the benchmark simulations (simulate_phenopath) and provides the
    covariate-aware trajectory fit being benchmarked
  repo: https://bioconductor.org/packages/release/bioc/html/phenopath.html
- name: Monocle 2
  role: Competing standard pseudotime method run on the same simulated matrices as
    the baseline
  repo: https://bioconductor.org/packages/release/bioc/html/monocle.html
- name: R
  role: Implementation language for the simulation, fitting, and aggregation pipeline
  repo: https://bioconductor.org/packages/release/bioc/html/phenopath.html
- name: dplyr
  role: Aggregates per-replicate Kendall-tau correlations to medians per method per
    interaction fraction
- name: tidyr
  role: Reshapes the summary table for export and plotting
- name: ggplot2
  role: Plots median Kendall-tau vs covariate-interaction fraction per method
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-018-04696-6
    title: Uncovering pseudotemporal trajectories with covariates from single cell
      and bulk expression data
  source_content_hash: sha256:72c7d3bac42a385d227fdd3ab507c5dc5f7513d8c9602765a8f023c29e214395
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
  edam_operation: http://edamontology.org/operation_2428
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5294117647058824
n_claims: 17
---

# pseudotime-method-benchmarking

## Summary

Benchmark pseudotime inference methods (PhenoPath, Monocle 2, DPT, Wishbone, TSCAN) on simulated single-cell expression data with varying fractions of covariate-trajectory interaction genes, scoring accuracy by Kendall-tau rank correlation against the true latent trajectory as in Campbell & Yau (Fig 1e).

## When to use

Use when comparing how pseudotime algorithms degrade as the fraction of genes with covariate-pseudotime interactions increases, or when validating a new trajectory method against standard ones on simulated data with a known latent time.

## When NOT to use

- The dataset lacks a ground-truth or simulated latent time, so no rank correlation against truth can be computed
- The goal is biological trajectory discovery on real data rather than quantitative method comparison
- Genes are expected to evolve nonlinearly with pseudotime, since PhenoPath (used as the reference implementation) assumes features evolve linearly with respect to pseudotime

## Inputs

- Simulated gene-by-cell expression matrices from simulate_phenopath() with known latent pseudotime z
- Simulation settings: interaction-gene fraction 5-50%, gene regimes (de-only, pst, pst-beta), n = 200 or 500 cells, high/low noise, 40 replicates per setting
- Optional: real single-cell expression matrices with a reference trajectory for applying the same scoring scheme

## Outputs

- Per-replicate Kendall-tau rank correlations between inferred pseudotime and true latent z for each method
- Median Kendall-tau summary table (one row per method per interaction fraction), e.g. kendall_tau_pseudotime_comparison.csv
- Comparison plot of accuracy vs covariate-interaction fraction (reproducing Fig 1e-style trends)

## How to apply

Generate simulated expression matrices with the R PhenoPath package's simulate_phenopath(), sweeping the fraction of genes simulated with covariate-pseudotime interactions from 5% to 50% across the 'de-only', 'pst' and 'pst-beta' gene regimes, with sample sizes of 200 and 500, high/low noise levels, and 40 replicates per setting. Fit PhenoPath with phenopath() using ELBO-based convergence control (elbo_tol, thin) and principal-component initialisation of the latent trajectory. Run Monocle 2, DPT, Wishbone and TSCAN on the same matrices to obtain competing inferred trajectories. Score every method per replicate with the Kendall-tau rank correlation between inferred pseudotime and the true latent z (analogous to cor(sim$z, trajectory(fit))), then aggregate to median per method per interaction fraction with dplyr/tidyr and export e.g. kendall_tau_pseudotime_comparison.csv. The key rationale is that standard methods implicitly assume homogeneous genetic/phenotypic backgrounds, so their accuracy should collapse above ~10% interaction genes while PhenoPath's stays approximately flat.

## Related tools

- **PhenoPath** (Generates the benchmark simulations (simulate_phenopath) and provides the covariate-aware trajectory fit being benchmarked) — https://bioconductor.org/packages/release/bioc/html/phenopath.html
- **Monocle 2** (Competing standard pseudotime method run on the same simulated matrices as the baseline) — https://bioconductor.org/packages/release/bioc/html/monocle.html
- **R** (Implementation language for the simulation, fitting, and aggregation pipeline) — https://bioconductor.org/packages/release/bioc/html/phenopath.html
- **dplyr** (Aggregates per-replicate Kendall-tau correlations to medians per method per interaction fraction)
- **tidyr** (Reshapes the summary table for export and plotting)
- **ggplot2** (Plots median Kendall-tau vs covariate-interaction fraction per method)

## Examples

```
sim <- simulate_phenopath(n = 200, G = 1000, pct_interactions = 0.3); fit <- phenopath(sim$y, sim$x, elbo_tol = 1e-5, thin = 20); tau <- cor(sim$z, trajectory(fit), method = 'kendall')
```

## Evaluation signals

- The summary table contains exactly one median Kendall-tau row per method per interaction fraction
- PhenoPath's median Kendall-tau is approximately independent of the covariate-interaction fraction, matching the qualitative trend of Fig 1e
- Monocle 2, DPT, Wishbone and TSCAN accuracies drop sharply once the interaction-gene fraction exceeds ~10%
- Each simulation setting has 40 replicates before aggregation; per-replicate correlations fall in [-1, 1]

## Limitations

- PhenoPath assumes features evolve linearly with respect to pseudotime; nonlinear effects may occur and are not discounted
- Benchmark conclusions apply to the linear covariate-interaction model; future work is needed for complex mixtures of continuous and discrete covariates in high-dimensional settings
- Results depend on simulation regimes (de-only, pst, pst-beta), sample size, and noise level; 40 replicates per setting are needed for stable medians

## Evidence

- [results] Standard methods do not account for covariate interactions: [evidence span withheld — anchor status: not_found]
- [readme] Convergence control by ELBO: [evidence span withheld — anchor status: external_doc]
- [abstract] Standard methods assume homogeneous backgrounds: "methods to date typically implicitly assume homogeneous genetic, phenotypic or environmental backgrounds, which becomes limiting as data sets grow in size and complexity"
- [discussion] Linearity assumption caveat: "An assumption made by PhenoPath is that features evolve linearly with respect to pseudotime."
