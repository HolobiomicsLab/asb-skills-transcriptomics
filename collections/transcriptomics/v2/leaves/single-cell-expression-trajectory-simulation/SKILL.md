---
name: single-cell-expression-trajectory-simulation
description: Use when benchmarking pseudotime inference accuracy on data with a known
  latent trajectory, especially to quantify how robust methods are when the fraction
  of genes with covariate-pseudotime ('pst-beta') interactions varies.
when_to_use_negative:
- The input is real experimental single-cell data with no known latent trajectory,
  so no ground-truth pseudotime exists to score against
- The goal is to detect covariate-pseudotime interactions in existing data rather
  than generate synthetic benchmarks with a controlled interaction fraction
- The analysis requires simulating nonlinear feature evolution along pseudotime, which
  PhenoPath's simulation (like the model) treats as linear
edam_operation: http://edamontology.org/operation_0244
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: PhenoPath
  role: R package providing simulate_phenopath() to generate expression matrices with
    controlled covariate-pseudotime interaction fractions, and phenopath() to fit
    the trajectory with CAVI inference
  repo: https://bioconductor.org/packages/release/bioc/html/phenopath.html
- name: Monocle 2
  role: competitor pseudotime algorithm run on the same simulated matrices to obtain
    comparison trajectories
  repo: https://bioconductor.org/packages/release/bioc/html/monocle.html
- name: Rcpp
  role: underlies the performant co-ordinate ascent variational inference implementation
    of PhenoPath on simulated data
- name: dplyr
  role: aggregation of per-replicate Kendall-tau correlations to medians per method
    per interaction fraction
- name: tidyr
  role: reshaping of simulation-benchmark results into the summary table
- name: ggplot2
  role: visualisation of Kendall-tau accuracy vs covariate-interaction fraction (Fig
    1e-style plots)
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-018-04696-6
    title: Uncovering pseudotemporal trajectories with covariates from single cell
      and bulk expression data
  source_content_hash: sha256:ecf3abe4d75302b8b62db7244342e6a1513ba7ab64c9e7435742bec7d4ccf373
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Monocle 2
  - PhenoPath
  - R
  - Rcpp
  - dplyr
  - ggplot2
  - tidyr
  edam_operation: http://edamontology.org/operation_0244
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5333333333333333
n_claims: 15
---

# single-cell-expression-trajectory-simulation

## Summary

Generates synthetic single-cell expression matrices with known latent pseudotime (z) and controlled fractions of covariate-pseudotime interaction genes using the PhenoPath R package's simulate_phenopath(), enabling quantitative benchmarking of pseudotime algorithms by Kendall-tau correlation against ground truth.

## When to use

Use when benchmarking pseudotime inference accuracy on data with a known latent trajectory, especially to quantify how robust methods are when the fraction of genes with covariate-pseudotime ('pst-beta') interactions varies.

## When NOT to use

- The input is real experimental single-cell data with no known latent trajectory, so no ground-truth pseudotime exists to score against
- The goal is to detect covariate-pseudotime interactions in existing data rather than generate synthetic benchmarks with a controlled interaction fraction
- The analysis requires simulating nonlinear feature evolution along pseudotime, which PhenoPath's simulation (like the model) treats as linear

## Inputs

- simulation parameters: number of cells (e.g. 200, 500), number of genes, gene regime ('de-only', 'pst', 'pst-beta'), fraction of covariate-pseudotime interacting genes (e.g. 5–50%), noise level, replicate index
- sample-specific covariate vector (e.g. binary mutational or stimulant status per cell)

## Outputs

- simulated gene-by-cell expression matrices with associated true latent pseudotime z (sim$z) and per-gene interaction regime labels
- per-replicate Kendall-tau correlations between each method's inferred trajectory and true z
- summary table of median Kendall-tau per method per interaction fraction (e.g. kendall_tau_pseudotime_comparison.csv)

## How to apply

Simulate expression matrices with PhenoPath's simulate_phenopath(), sweeping the fraction of covariate-interacting genes (e.g. 5–50%) across the 'de-only', 'pst' and 'pst-beta' gene regimes, with sample sizes (e.g. 200 and 500 cells) and noise levels as factors, replicated tens of times per setting. Each simulation returns the expression matrix plus the true latent trajectory z (accessed e.g. as sim$z), which serves as the ranking ground truth. Fit the method under test (e.g. phenopath() with ELBO-based convergence via elbo_tol and thin, and principal-component initialisation of the latent trajectory) and competitors (Monocle 2, DPT, Wishbone, TSCAN) on identical matrices. Score each method per replicate with Kendall-tau rank correlation between inferred pseudotime and true z, then aggregate to medians per method per interaction fraction for comparison against the paper's Fig 1e benchmark.

## Related tools

- **PhenoPath** (R package providing simulate_phenopath() to generate expression matrices with controlled covariate-pseudotime interaction fractions, and phenopath() to fit the trajectory with CAVI inference) — https://bioconductor.org/packages/release/bioc/html/phenopath.html
- **Monocle 2** (competitor pseudotime algorithm run on the same simulated matrices to obtain comparison trajectories) — https://bioconductor.org/packages/release/bioc/html/monocle.html
- **Rcpp** (underlies the performant co-ordinate ascent variational inference implementation of PhenoPath on simulated data)
- **dplyr** (aggregation of per-replicate Kendall-tau correlations to medians per method per interaction fraction)
- **tidyr** (reshaping of simulation-benchmark results into the summary table)
- **ggplot2** (visualisation of Kendall-tau accuracy vs covariate-interaction fraction (Fig 1e-style plots))

## Examples

```
sim <- simulate_phenopath(n_cells = 200, n_genes = 1000, de_only = FALSE); fit <- phenopath(sim$y, elbo_tol = 1e-6, thin = 10)
```

## Evaluation signals

- The summary table contains exactly one median Kendall-tau row per method per covariate-interaction fraction across the sweep (e.g. 5% to 50%)
- PhenoPath's Kendall-tau accuracy is approximately independent of the fraction of covariate-pseudotime interacting genes, matching the qualitative trend of Fig 1e
- Monocle 2, DPT, Wishbone and TSCAN accuracies diminish once the interacting-gene fraction exceeds roughly 10%
- Per-replicate correlations are computed against the true latent z returned by the simulation (analogous to cor(sim$z, trajectory(fit)))
- Each simulation setting is replicated (e.g. 40 times) so medians are stable rather than single-run artefacts

## Limitations

- PhenoPath assumes features evolve linearly with respect to pseudotime; nonlinear effects may occur and are not discounted by the simulation model
- Simulations only validate fits on synthetic ground truth; checks should also be conducted to verify PhenoPath model fits are consistent with real data
- The paper's simulation framework is built around covariate-pseudotime interactions and does not cover complex mixtures of continuous and discrete covariates in high-dimensional settings, which is deferred to future work

## Evidence

- [readme] PhenoPath learns genomic trajectories (pseudotimes) in the presence of heterogenous environmental and genetic backgrounds encoded as additional covariates: "PhenoPath learns genomic trajectories (pseudotimes) in the presence of heterogenous environmental and genetic backgrounds encoded as additional covariates"
- [readme] Inference is performed using co-ordinate ascent variational inference (CAVI) implemented using the Rcpp library for increased performance.: [evidence span withheld — anchor status: external_doc]
- [methods] We perform co-ordinate ascent mean field variational inference (see ref. 45) with an approximating distribution of the form: "We perform co-ordinate ascent mean field variational inference (see ref. 45) with an approximating distribution of the form"
- [discussion] nonlinear effects may occur and checks should be conducted to verify that PhenoPath model fits are consistent with the data: "However, it cannot be discounted that nonlinear effects may occur and checks should be conducted to verify that PhenoPath model fits are consistent with the data."
- [discussion] Future work will expand the ability of PhenoPath to handle complex mixtures of continuous and discrete covariates in high-dimensional settings.: "Future work will expand the ability of PhenoPath to handle complex mixtures of continuous and discrete covariates in high-dimensional settings."
