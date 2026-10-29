---
name: transcriptional-bursting-parameter-interpretation
description: Use when raw single-cell count matrix needs gene-level kon/koff/s kinetic
  parameter estimates (e.g., to parameterize SymSim's Beta-Poisson model, or to characterize
  transcriptional bursting per cell cluster).
when_to_use_negative:
- The goal is simulation parameter fitting by matching summary statistics rather than
  gene-level kinetic parameter estimation (use SymSim's BestMatchParams grid-search
  workflow instead)
- The dataset is a small non-UMI dataset where scVI is not applicable (as with the
  130-cell Gaublomme Th17 dataset, where only MAGIC was used for imputation)
- Cells are not clustered, violating the assumption that all cells in the fitted group
  share the same kinetic parameters
edam_operation: http://edamontology.org/operation_3658
edam_topics:
- http://edamontology.org/topic_0204
- http://edamontology.org/topic_3308
tools:
- name: RJAGS
  role: Provides the Gibbs sampler used to fit the two-state kinetic model (kon, koff,
    s) via MCMC with 2000 iterations
- name: SymSim
  role: Supplies the cortex_counts.txt test dataset, the kinetic model framework,
    and consumes the estimated parameter distributions for simulation
  repo: https://github.com/YosefLab/SymSim
- name: scVI
  role: Imputes UMI count data prior to kinetic parameter fitting
- name: MAGIC
  role: Second imputation method applied to UMI cortex counts before fitting
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-019-10500-w
    title: Simulating multiple faceted variability in single cell RNA sequencing
  source_content_hash: sha256:1a780c59f191d72432a5ef6befbafd548d7eed93f06948005ef6976f9a1772bf
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
  - http://edamontology.org/topic_0204
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# transcriptional-bursting-parameter-interpretation

## Summary

Estimates two-state transcription kinetic parameters (kon, koff, s) from real scRNA-seq count data via MCMC (Gibbs sampling in RJAGS), and interprets the resulting bursting parameter distributions for reuse in SymSim simulations. Grounded in the fitting of the Zeisel UMI cortex dataset after scVI/MAGIC imputation.

## When to use

Use when raw single-cell count matrix needs gene-level kon/koff/s kinetic parameter estimates (e.g., to parameterize SymSim's Beta-Poisson model, or to characterize transcriptional bursting per cell cluster).

## When NOT to use

- The goal is simulation parameter fitting by matching summary statistics rather than gene-level kinetic parameter estimation (use SymSim's BestMatchParams grid-search workflow instead)
- The dataset is a small non-UMI dataset where scVI is not applicable (as with the 130-cell Gaublomme Th17 dataset, where only MAGIC was used for imputation)
- Cells are not clustered, violating the assumption that all cells in the fitted group share the same kinetic parameters

## Inputs

- Raw gene-by-cell UMI count matrix (e.g., cortex_counts.txt from SymSim extdata, 3005 Zeisel cortex cells)
- Cell cluster assignments (e.g., oligodendrocytes, pyramidal CA1, pyramidal S1 subpopulations)
- Imputed count matrices from scVI and MAGIC

## Outputs

- Per-gene posterior distributions of kon, koff, and s per cluster and per imputation method
- Summary table of fitted kinetic parameters mirroring Supplementary Tables 2-5 (e.g., kinetic_params_zeisel_umi.csv)
- Kinetic parameter distributions for input into SymSim simulations (Fig. 2b-style distributions)

## How to apply

Load the count matrix (e.g., SymSim's extdata/cortex_counts.txt via read.table(header=FALSE)) and cluster cells to satisfy the shared-kinetics assumption, keeping large clusters (e.g., oligodendrocytes, pyramidal CA1, pyramidal S1) for per-cluster fitting. Impute counts before fitting — for UMI data use both scVI and MAGIC, since imputed rather than raw read counts are used for kinetic parameter estimation. For each cluster-by-imputation combination, fit the two-state kinetic model with the Gibbs sampler implemented in RJAGS, running the MCMC chain for 2000 iterations. Aggregate posterior distributions of kon, koff, and s per gene per cluster and per imputation method into a summary table that can be fed to SymSim (e.g., param_realdata.zeisel.imputed) to reproduce the parameter distributions reported in Fig. 2b and Supplementary Tables 2-5.

## Related tools

- **RJAGS** (Provides the Gibbs sampler used to fit the two-state kinetic model (kon, koff, s) via MCMC with 2000 iterations)
- **SymSim** (Supplies the cortex_counts.txt test dataset, the kinetic model framework, and consumes the estimated parameter distributions for simulation) — https://github.com/YosefLab/SymSim
- **scVI** (Imputes UMI count data prior to kinetic parameter fitting)
- **MAGIC** (Second imputation method applied to UMI cortex counts before fitting)

## Examples

```
cortex_counts <- read.table(system.file("extdata", "cortex_counts.txt", package = "SymSim"), header = F); # then fit kon/koff/s per cluster with RJAGS (2000 iterations) on scVI- and MAGIC-imputed counts
```

## Evaluation signals

- kinetic_params_zeisel_umi.csv exists and contains kon/koff/s estimates for every cluster-by-imputation-method combination
- Estimated per-cluster parameter distributions reproduce the patterns shown in Fig. 2b of the SymSim paper
- Fitted parameters are consistent with the values reported in Supplementary Tables 2-5 (e.g., higher inferred capture efficiency for one protocol vs the other)
- MCMC chains ran for the specified 2000 iterations and converged to stable posteriors for kon, koff, and s
- Imputed (scVI and MAGIC) count matrices, not raw counts, were the fitting input as required by the workflow

## Limitations

- Kinetic parameter fitting assumes all cells within a cluster share the same kinetic parameters, so clustering quality directly constrains the fit
- Supplementary Tables 2-5 with the reported fit parameters are referenced but not included in the accessible text
- Parameters estimated from imputed data may not fully distinguish kinetic rates from capture-efficiency effects between protocols

## Evidence

- [discussion] We fit this model to the experimental data using the Gibbs sampler implemented in RJAGS.: "We fit this model to the experimental data using the Gibbs sampler implemented in RJAGS."
- [discussion] Imputation before fitting: "We use imputed read counts, rather than the raw read counts."
- [discussion] MCMC iterations: [evidence span withheld — anchor status: not_found]
- [discussion] Shared kinetics assumption: "To meet the assumption that all cells share the same kinetic parameters we divide cells by clustering"
- [discussion] Imputation tool selection: "the UMI-based data were imputed using scVI4 and MAGIC18, and the non-UMI data were imputed using MAGIC18 (as scVI is only applicable to large datasets)"
- [methods] Cortex dataset loading: "cortex_counts <- read.table(system.file("extdata", "cortex_counts.txt", package = "SymSim"), header = F"
- [readme] Install command: "devtools::install_github("YosefLab/SymSim")"
