---
name: mcmc-gibbs-sampling-inference
description: Use when you need per-gene or per-cluster kinetic parameter distributions
  (kon, koff, s) estimated from real scRNA-seq count data to parameterize a two-state
  transcription bursting model, e.g. for SymSim simulation inputs.
when_to_use_negative:
- Raw (non-imputed) counts are used for fitting — the paper explicitly uses imputed
  read counts rather than raw read counts
- The cell population is heterogeneous and cannot be partitioned into clusters sharing
  the same kinetic parameters
- The dataset is small non-UMI data where scVI is inapplicable — restrict imputation
  to MAGIC in that case
edam_operation: http://edamontology.org/operation_3658
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3518
tools:
- name: RJAGS
  role: Provides the Gibbs sampler used to fit the two-state kinetic model (kon, koff,
    s) via MCMC with 2000 iterations
- name: scVI
  role: Imputes UMI-based counts prior to kinetic parameter fitting
- name: MAGIC
  role: Imputes counts prior to kinetic parameter fitting (used for both UMI and non-UMI
    data)
- name: SymSim
  role: Supplies the Zeisel cortex count matrix (extdata/cortex_counts.txt) and consumes
    the estimated kon/koff/s distributions for simulation
  repo: https://github.com/YosefLab/SymSim
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-019-10500-w
    title: Simulating multiple faceted variability in single cell RNA sequencing
  source_content_hash: sha256:b93b4bc3a25520b5d3cf16d3d5c943ebd2826c4c351b0ef3a9deac6137dbb090
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YosefLab/SymSim
  tools:
  - MAGIC
  - RJAGS
  - SymSim
  - scVI
  edam_operation: http://edamontology.org/operation_3658
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3518
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# mcmc-gibbs-sampling-inference

## Summary

Fits the two-state transcription kinetic model (kon, koff, s) to single-cell RNA-seq counts via MCMC with the Gibbs sampler implemented in RJAGS, using imputed counts from cell clusters to obtain posterior parameter distributions for SymSim simulations.

## When to use

Use when you need per-gene or per-cluster kinetic parameter distributions (kon, koff, s) estimated from real scRNA-seq count data to parameterize a two-state transcription bursting model, e.g. for SymSim simulation inputs.

## When NOT to use

- Raw (non-imputed) counts are used for fitting — the paper explicitly uses imputed read counts rather than raw read counts
- The cell population is heterogeneous and cannot be partitioned into clusters sharing the same kinetic parameters
- The dataset is small non-UMI data where scVI is inapplicable — restrict imputation to MAGIC in that case

## Inputs

- Raw scRNA-seq UMI count matrix (e.g. SymSim extdata/cortex_counts.txt, Zeisel et al. 3005-cell cortex dataset)
- Cell cluster assignments satisfying the shared-kinetics assumption (e.g. oligodendrocytes, pyramidal CA1, pyramidal S1)
- Imputed count matrices from scVI and MAGIC

## Outputs

- Posterior distributions of kon, koff, and s per cluster per imputation method (e.g. kinetic_params_zeisel_umi.csv)
- Summary fit table mirroring Supplementary Tables 2-5
- Aggregated kinetic parameter distributions for SymSim (Fig. 2b)

## How to apply

Load the raw count matrix (e.g. cortex_counts.txt from SymSim extdata via read.table(header=FALSE)) and cluster the cells first, because the fitting assumes all cells in a cluster share the same kinetic parameters — the Zeisel UMI cortex dataset (3005 cells) was split into subpopulations such as oligodendrocytes, pyramidal CA1, and pyramidal S1. Impute the counts before fitting (scVI and MAGIC for large UMI datasets; MAGIC only for non-UMI data, since scVI is only applicable to large datasets). For each cluster and imputed dataset, fit the two-state kinetic model with the Gibbs sampler implemented in RJAGS, running the MCMC chain for 2000 iterations. Aggregate the posterior distributions of kon, koff, and s per cluster and imputation method into a summary table mirroring the reported fit parameters (Supplementary Tables 2-5) to derive the distributions shown in SymSim Fig. 2b.

## Related tools

- **RJAGS** (Provides the Gibbs sampler used to fit the two-state kinetic model (kon, koff, s) via MCMC with 2000 iterations)
- **scVI** (Imputes UMI-based counts prior to kinetic parameter fitting)
- **MAGIC** (Imputes counts prior to kinetic parameter fitting (used for both UMI and non-UMI data))
- **SymSim** (Supplies the Zeisel cortex count matrix (extdata/cortex_counts.txt) and consumes the estimated kon/koff/s distributions for simulation) — https://github.com/YosefLab/SymSim

## Examples

```
cortex_counts <- read.table(system.file("extdata", "cortex_counts.txt", package = "SymSim"), header = F); # fit (kon, koff, s) per cluster with RJAGS Gibbs sampler, n.iter = 2000
```

## Evaluation signals

- kinetic_params_zeisel_umi.csv exists and contains kon/koff/s estimates for every cluster-by-imputation-method combination
- Posterior chains ran the full 2000 iterations of the RJAGS Gibbs sampler without premature termination
- Estimated per-cluster parameter distributions reproduce the patterns reported in Fig. 2b
- Fit parameters are consistent with Supplementary Tables 2-5 (e.g. relative capture efficiency ordering across protocols)

## Limitations

- The fitting assumes all cells in a cluster share the same kinetic parameters, so clustering quality directly constrains validity
- Supplementary Tables 2-5 containing the reported fit parameters are referenced but not present in the provided text
- For non-UMI data scVI is only applicable to large datasets, limiting imputation choices to MAGIC

## Evidence

- [discussion] We fit this model to the experimental data using the Gibbs sampler implemented in RJAGS.: "We fit this model to the experimental data using the Gibbs sampler implemented in RJAGS."
- [discussion] We estimated kinetic parameters from experimental data using an MCMC approach... The number of iterations is set to 2000.: "The number of iterations is set to 2000."
- [discussion] We use scVI4 and MAGIC18 for the imputation.: "We use scVI4 and MAGIC18 for the imputation."
- [discussion] To meet the assumption that all cells share the same kinetic parameters we divide cells by clustering: "To meet the assumption that all cells share the same kinetic parameters we divide cells by clustering"
- [methods] cortex_counts <- read.table(system.file("extdata", "cortex_counts.txt", package = "SymSim"), header = F: "cortex_counts <- read.table(system.file("extdata", "cortex_counts.txt", package = "SymSim"), header = F"
