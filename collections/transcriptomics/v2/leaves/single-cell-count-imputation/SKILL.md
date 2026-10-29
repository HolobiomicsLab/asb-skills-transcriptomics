---
name: single-cell-count-imputation
description: Use when raw UMI-based scRNA-seq counts must be denoised/imputed prior
  to kinetic parameter estimation (kon, koff, s) with the RJAGS Gibbs sampler, or
  when scVI/MAGIC imputation is needed to meet modeling assumptions.
when_to_use_negative:
- 'The dataset is small (<~hundreds of cells): the paper notes scVI is only applicable
  to large datasets, so for the 130-cell non-UMI Th17 dataset MAGIC alone is appropriate'
- The downstream task does not require imputed values (e.g., analyses explicitly designed
  for raw counts), since imputation alters count distributions before fitting
- Cells have not been clustered, violating the assumption that all cells share the
  same kinetic parameters for per-cluster fitting
edam_operation: http://edamontology.org/operation_3557
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scVI
  role: Deep generative model used to impute UMI-based count data before kinetic parameter
    fitting
  repo: https://github.com/YosefLab/SymSim
- name: MAGIC
  role: Imputation method applied to both UMI-based (with scVI) and non-UMI datasets
    (alone) prior to fitting kon/koff/s
- name: RJAGS
  role: Provides the Gibbs sampler MCMC that consumes the imputed counts to estimate
    kinetic parameters (2000 iterations)
- name: SymSim
  role: R package supplying the cortex count data (extdata/cortex_counts.txt) and
    consuming the imputed-data-derived kinetic parameter distributions for simulation
  repo: https://github.com/YosefLab/SymSim
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-019-10500-w
    title: Simulating multiple faceted variability in single cell RNA sequencing
  source_content_hash: sha256:cd94370bf1239907ade2433c31c84130d20854414f56ebc6fd1bef5b08acb789
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YosefLab/SymSim
  tools:
  - MAGIC
  - RJAGS
  - SymSim
  - scVI
  edam_operation: http://edamontology.org/operation_3557
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6428571428571429
n_claims: 14
---

# single-cell-count-imputation

## Summary

Imputes raw single-cell RNA-seq count matrices (e.g., the Zeisel UMI cortex dataset of 3005 cells) with scVI and MAGIC before fitting two-state kinetic parameters (kon, koff, s) used by SymSim, since raw counts are unsuitable for MCMC-based kinetic fitting.

## When to use

Use when raw UMI-based scRNA-seq counts must be denoised/imputed prior to kinetic parameter estimation (kon, koff, s) with the RJAGS Gibbs sampler, or when scVI/MAGIC imputation is needed to meet modeling assumptions.

## When NOT to use

- The dataset is small (<~hundreds of cells): the paper notes scVI is only applicable to large datasets, so for the 130-cell non-UMI Th17 dataset MAGIC alone is appropriate
- The downstream task does not require imputed values (e.g., analyses explicitly designed for raw counts), since imputation alters count distributions before fitting
- Cells have not been clustered, violating the assumption that all cells share the same kinetic parameters for per-cluster fitting

## Inputs

- Raw scRNA-seq count matrix (e.g., UMI cortex_counts.txt from SymSim extdata, 3005 Zeisel cortex cells)
- Cell cluster assignments satisfying the shared-kinetics assumption (e.g., oligodendrocytes, pyramidal CA1, pyramidal S1)
- Gene filtering configuration used for parameter fitting

## Outputs

- scVI-imputed count matrix
- MAGIC-imputed count matrix
- Per-cluster, per-imputation-method kinetic parameter estimates (kon, koff, s) suitable for SymSim simulation input (e.g., kinetic_params_zeisel_umi.csv mirroring Supplementary Tables 2-5)

## How to apply

Load the raw count matrix (e.g., extdata/cortex_counts.txt from the SymSim package via read.table with header=FALSE) and apply the same gene filtering used for parameter fitting. Impute counts using two complementary methods: scVI and MAGIC, both of which the authors applied to the large UMI-based cortex dataset of 3005 cells. Note the applicability constraint from the paper: scVI is only applicable to large datasets, so for small datasets (e.g., the 130-cell non-UMI Th17 dataset from Gaublomme et al.) MAGIC alone should be used. Use the imputed read counts, rather than raw counts, as input for downstream kinetic fitting — the authors fit the two-state transcription model per cluster of cells sharing the same kinetic parameters, running the MCMC (RJAGS Gibbs sampler) for 2000 iterations. Aggregate kon, koff, and s posteriors per cluster and per imputation method to form distributions usable in SymSim simulations (e.g., the parameter pool behind Fig. 2b, built from oligodendrocyte, pyramidal CA1, and pyramidal S1 subpopulations).

## Related tools

- **scVI** (Deep generative model used to impute UMI-based count data before kinetic parameter fitting) — https://github.com/YosefLab/SymSim
- **MAGIC** (Imputation method applied to both UMI-based (with scVI) and non-UMI datasets (alone) prior to fitting kon/koff/s)
- **RJAGS** (Provides the Gibbs sampler MCMC that consumes the imputed counts to estimate kinetic parameters (2000 iterations))
- **SymSim** (R package supplying the cortex count data (extdata/cortex_counts.txt) and consuming the imputed-data-derived kinetic parameter distributions for simulation) — https://github.com/YosefLab/SymSim

## Examples

```
cortex_counts <- read.table(system.file("extdata", "cortex_counts.txt", package = "SymSim"), header = FALSE)
```

## Evaluation signals

- The imputed matrices exist for both methods (scVI and MAGIC) over the filtered cortex_counts.txt matrix
- kinetic_params_zeisel_umi.csv contains kon/koff/s estimates for every cluster-by-imputation-method combination (oligodendrocytes, pyramidal CA1, pyramidal S1 × scVI/MAGIC)
- The estimated kinetic parameter distributions reproduce the per-cluster parameter patterns reported in Fig. 2b and Supplementary Tables 2-5
- For the non-UMI Th17 dataset, only MAGIC imputation is produced (scVI absent), matching the stated applicability constraint
- MCMC chains completed the specified 2000 iterations before posterior aggregation

## Limitations

- scVI is only applicable to large datasets, restricting its use on small non-UMI datasets like the 130-cell Th17 data
- Imputed counts, not raw counts, are required for fitting, so raw-count-only workflows cannot reuse this skill's outputs directly
- Kinetic parameter fitting assumes all cells in a cluster share the same kinetic parameters, so clustering quality directly limits fit validity
- Supplementary Tables 2-5 (fit parameters) are referenced but not present in the provided text, limiting direct numeric validation

## Evidence

- [discussion] scVI and MAGIC imputation on UMI data: "We use scVI4 and MAGIC18 for the imputation."
- [discussion] scVI dataset-size constraint: "the UMI-based data were imputed using scVI4 and MAGIC18, and the non-UMI data were imputed using MAGIC18 (as scVI is only applicable to large datasets)"
- [discussion] Use of imputed counts for fitting: "We use imputed read counts, rather than the raw read counts."
- [discussion] Clustering to meet shared-kinetics assumption: "To meet the assumption that all cells share the same kinetic parameters we divide cells by clustering"
- [discussion] MCMC iterations for kinetic fitting: [evidence span withheld — anchor status: not_found]
- [methods] Loading the cortex count matrix: "cortex_counts <- read.table(system.file("extdata", "cortex_counts.txt", package = "SymSim"), header = F"
- [discussion] Cortex dataset size: "we used a UMI-based dataset of 3005 cortex cells by Zeisel et al.38"
