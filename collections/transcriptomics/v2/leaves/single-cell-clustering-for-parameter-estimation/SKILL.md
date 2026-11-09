---
name: single-cell-clustering-for-parameter-estimation
description: Use when fitting bursting kinetic parameters (kon, koff, s) from scRNA-seq
  counts with an MCMC/Gibbs sampler that assumes all cells share the same kinetics,
  and cells must first be divided by clustering.
when_to_use_negative:
- When the goal is to fit kinetics on a heterogeneous cell population without subdividing
  — the Gibbs sampler assumes all cells within the fitted set share the same kinetic
  parameters.
- When the dataset is small and non-UMI (scVI is only applicable to large datasets;
  non-UMI data such as the Th17 dataset were imputed with MAGIC only).
- When the task is clustering quality evaluation itself (e.g. benchmarking PCA/scVI/SIMLR
  clustering) rather than kinetic parameter estimation.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: SymSim
  role: Provides the UMI cortex count matrix (extdata/cortex_counts.txt) and consumes
    the estimated (kon, koff, s) distributions for simulation
  repo: https://github.com/YosefLab/SymSim
- name: RJAGS
  role: Implements the Gibbs sampler used to fit the two-state kinetic model (kon,
    koff, s) per cluster, with 2000 MCMC iterations
- name: scVI
  role: Imputation of UMI-based counts prior to kinetic parameter fitting (applicable
    to large datasets)
- name: MAGIC
  role: Imputation of UMI-based counts prior to kinetic parameter fitting
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-019-10500-w
    title: Simulating multiple faceted variability in single cell RNA sequencing
  source_content_hash: sha256:dd46b5b54d7bd519d32edbfda7712a467173f7e558e9beb33719426b1ee01cf0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YosefLab/SymSim
  tools:
  - MAGIC
  - RJAGS
  - SymSim
  - scVI
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# single-cell-clustering-for-parameter-estimation

## Summary

Cluster single-cell RNA-seq data into subpopulations prior to fitting two-state transcription kinetic models, so that the shared-kinetics assumption underlying per-gene (kon, koff, s) estimation is satisfied. Applied to the Zeisel UMI cortex dataset to derive the kinetic parameter distributions used in SymSim simulations.

## When to use

Use when fitting bursting kinetic parameters (kon, koff, s) from scRNA-seq counts with an MCMC/Gibbs sampler that assumes all cells share the same kinetics, and cells must first be divided by clustering.

## When NOT to use

- When the goal is to fit kinetics on a heterogeneous cell population without subdividing — the Gibbs sampler assumes all cells within the fitted set share the same kinetic parameters.
- When the dataset is small and non-UMI (scVI is only applicable to large datasets; non-UMI data such as the Th17 dataset were imputed with MAGIC only).
- When the task is clustering quality evaluation itself (e.g. benchmarking PCA/scVI/SIMLR clustering) rather than kinetic parameter estimation.

## Inputs

- UMI-based scRNA-seq count matrix (e.g. SymSim extdata/cortex_counts.txt, Zeisel et al. 3005 cortex cells)
- Cell clustering / subpopulation labels (e.g. oligodendrocytes, pyramidal CA1, pyramidal S1)

## Outputs

- Per-cluster, per-imputation-method kinetic parameter estimates (kon, koff, s), e.g. kinetic_params_zeisel_umi.csv
- Aggregated posterior distributions of kon, koff, and s usable as SymSim input (param_realdata; cf. Fig. 2b)

## How to apply

Load the count matrix (e.g. extdata/cortex_counts.txt from the SymSim package via read.table(header=FALSE)) and apply the same gene filtering used for parameter fitting. Cluster the cells so each cluster plausibly satisfies the assumption that all cells share the same kinetic parameters, keeping large clusters for per-cluster fitting. For the Zeisel UMI cortex dataset (3005 cells), the subpopulations were oligodendrocytes, pyramidal CA1, and pyramidal S1. Impute counts per cluster with scVI and MAGIC (both applicable to large UMI datasets), since imputed rather than raw read counts are used for fitting. Then fit the two-state kinetic model (kon, koff, s) per cluster and per imputation method with the Gibbs sampler in RJAGS (2000 iterations), aggregating posterior distributions into a summary table mirroring Supplementary Tables 2-5.

## Related tools

- **SymSim** (Provides the UMI cortex count matrix (extdata/cortex_counts.txt) and consumes the estimated (kon, koff, s) distributions for simulation) — https://github.com/YosefLab/SymSim
- **RJAGS** (Implements the Gibbs sampler used to fit the two-state kinetic model (kon, koff, s) per cluster, with 2000 MCMC iterations)
- **scVI** (Imputation of UMI-based counts prior to kinetic parameter fitting (applicable to large datasets))
- **MAGIC** (Imputation of UMI-based counts prior to kinetic parameter fitting)

## Examples

```
cortex_counts <- read.table(system.file("extdata", "cortex_counts.txt", package = "SymSim"), header = FALSE); fit_kinetics(cluster, imputed_counts, n_iter = 2000)
```

## Evaluation signals

- kinetic_params_zeisel_umi.csv exists and contains kon/koff/s estimates for every cluster-by-imputation-method combination
- Estimated per-cluster parameter distributions reproduce the patterns reported in Fig. 2b
- Fit parameters are consistent with the reported values in Supplementary Tables 2-5
- Each fitted cluster is large enough to satisfy the shared-kinetics assumption (cells divided by clustering before fitting)
- Fitting runs for the specified 2000 MCMC iterations without chain failure per cluster/imputation pairing

## Limitations

- The kinetic parameter fitting assumes all cells in a cluster share the same kinetic parameters; clustering errors violate this assumption.
- Supplementary Tables 2-5 with the reported fit parameters are referenced but not present in the provided text, limiting exact validation.
- Only two imputation methods (scVI, MAGIC) were used, and results depend on the imputation choice; scVI is not applicable to small non-UMI datasets.

## Evidence

- [discussion] To meet the assumption that all cells share the same kinetic parameters we divide cells by clustering: "To meet the assumption that all cells share the same kinetic parameters we divide cells by clustering"
- [discussion] RJAGS Gibbs sampler 2000 iterations: "We fit this model to the experimental data using the Gibbs sampler implemented in RJAGS."
- [discussion] imputed counts used for fitting: "We use imputed read counts, rather than the raw read counts."
- [discussion] scVI and MAGIC imputation of UMI data: "the UMI-based data were imputed using scVI4 and MAGIC18, and the non-UMI data were imputed using MAGIC18 (as scVI is only applicable to large datasets)"
- [methods] loading cortex counts from SymSim: "cortex_counts <- read.table(system.file("extdata", "cortex_counts.txt", package = "SymSim"), header = F"
