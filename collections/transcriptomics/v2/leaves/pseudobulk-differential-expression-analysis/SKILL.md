---
name: pseudobulk-differential-expression-analysis
description: Use when a cohort-level single-cell count matrix with sample annotations
  and known or simulated differential expression ground truth must be analyzed cluster-free,
  or when comparing DE methods' accuracy per cell.
when_to_use_negative:
- The analysis goal is standard per-cell-type pseudo-bulk DGE from a sequential workflow
  (normalization, integration, cluster identification), not cluster-free DE along
  a cell-state continuum
- The dataset is single-sample or lacks sample-level replication, so sample-to-sample
  variability cannot be modeled
- 'A nonlinear manifold is required: GEDI currently supports only linear manifold
  learning with optional hyperellipsoid restriction'
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: GEDI
  role: Generative Bayesian model fit with mode='Bl2' or 'Bsphere' to produce cluster-free
    single-cell DE estimates correlated against ground truth
  repo: https://github.com/csglab/GEDI
- name: LEMUR
  role: Latent embedding regression comparator method for cluster-free DE
- name: miloDE
  role: Comparator method for cluster-free differential expression in the benchmark
- name: DESeq2
  role: Pseudobulk differential expression comparator run on the same data
- name: scuttle
  role: Used for basic QC and preprocessing of the single-cell counts before modeling
- name: scran
  role: R/Bioconductor dependency used in the DE analysis workflow notebooks
  repo: https://bioconductor.org/packages/release/bioc/html/scran.html
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-024-50963-0
    title: A unified model for interpretable latent embedding of multi-sample, multi-condition
      single-cell data
  source_content_hash: sha256:f6245211a5b085b2bdab3d92936f0d92c78c50facafbd55199a8ee3350f11494
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/csglab/GEDI
  tools:
  - BEDTools
  - DESeq2
  - GEDI
  - LEMUR
  - R
  - Samtools
  - SoloTE
  - edgeR
  - miloDE
  - scTE
  - scran
  - scuttle
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9333333333333333
n_claims: 15
---

# pseudobulk-differential-expression-analysis

## Summary

Benchmark and perform cluster-free differential expression on multi-sample, multi-condition single-cell data by comparing GEDI's transcriptomic vector field estimates against LEMUR, miloDE, and DESeq2 pseudobulk, using median Pearson correlation to simulated ground truth and AUROC of up/down-regulated gene detection as metrics.

## When to use

Use when a cohort-level single-cell count matrix with sample annotations and known or simulated differential expression ground truth must be analyzed cluster-free, or when comparing DE methods' accuracy per cell.

## When NOT to use

- The analysis goal is standard per-cell-type pseudo-bulk DGE from a sequential workflow (normalization, integration, cluster identification), not cluster-free DE along a cell-state continuum
- The dataset is single-sample or lacks sample-level replication, so sample-to-sample variability cannot be modeled
- A nonlinear manifold is required: GEDI currently supports only linear manifold learning with optional hyperellipsoid restriction

## Inputs

- Simulated cohort-level single-cell count matrix (generated from COVID-19 cohort 1 parameters)
- Sample annotation vector
- Ground-truth differential expression labels per gene per cell

## Outputs

- fig4_de_benchmark_metrics.csv with per-method Pearson r, AUROC up, AUROC down
- Per-method single-cell log-fold-change estimates (GEDI Bl2, GEDI Bsphere, LEMUR, miloDE, DESeq2 pseudobulk)

## How to apply

Convert the raw single-cell count matrix to sparse dgCMatrix format and fit GEDI twice via model$setup: once with mode='Bl2' (hyperplane; L2 norm of the entire B matrix fixed) and once with mode='Bsphere' (hyperellipsoid; L2 norms of B columns fixed), initializing latent variables with randomSeed=1 and checking convergence with model$plotTracking. Run the competing methods on the same data — LEMUR (latent embedding regression), miloDE, and DESeq2 pseudobulk differential expression. For each method, correlate estimated log-fold-changes with the ground-truth DE vector per cell and record the median Pearson correlation, then compute AUROC of up-regulated and down-regulated gene detection per method. Write all per-method metrics (Pearson r, AUROC up, AUROC down) to fig4_de_benchmark_metrics.csv.

## Related tools

- **GEDI** (Generative Bayesian model fit with mode='Bl2' or 'Bsphere' to produce cluster-free single-cell DE estimates correlated against ground truth) — https://github.com/csglab/GEDI
- **LEMUR** (Latent embedding regression comparator method for cluster-free DE)
- **miloDE** (Comparator method for cluster-free differential expression in the benchmark)
- **DESeq2** (Pseudobulk differential expression comparator run on the same data)
- **scuttle** (Used for basic QC and preprocessing of the single-cell counts before modeling)
- **scran** (R/Bioconductor dependency used in the DE analysis workflow notebooks) — https://bioconductor.org/packages/release/bioc/html/scran.html

## Examples

```
devtools::install_github("csglab/GEDI"); model <- GEDI::model$setup(counts, C, mode='Bl2', randomSeed=1)
```

## Evaluation signals

- GEDI hyperplane (mode='Bl2') median Pearson r ≈ 0.4 between estimated and ground-truth single-cell DE vectors, versus ≈ 0.14 for LEMUR
- Median AUROC ≈ 0.79 (GEDI Bl2) versus ≈ 0.72 (LEMUR) for up-regulated gene detection
- Median AUROC ≈ 0.78 (GEDI Bl2) versus ≈ 0.55 (LEMUR) for down-regulated gene detection
- model$plotTracking shows convergence of the GEDI fits for both mode='Bl2' and mode='Bsphere'
- All per-method metrics (Pearson r, AUROC up, AUROC down) present in fig4_de_benchmark_metrics.csv

## Limitations

- GEDI currently supports only linear manifold learning, with the option to further restrict the manifold to a hyperellipsoid; other parametric manifold approximations are future extensions
- More extensive analyses are needed to understand how the number of DE genes per cell, DE magnitude, cell-cell/gene-gene correlations, variances, sample/cell counts, and sequencing depth influence DE performance
- Properly modeling sample-to-sample variability is key for unbiased differential expression analysis, so unreplicated designs are unsuitable

## Evidence

- [other] On the simulated dataset, GEDI's single-cell DE estimates correlate strongly with the ground-truth DE vector of each cell, with median Pearson r = 0.4: [evidence span withheld — anchor status: not_found]
- [other] once with mode='Bl2' (hyperplane, L2 norm of the entire B matrix fixed) and once with mode='Bsphere': [evidence span withheld — anchor status: not_found]
- [discussion] Previous work has shown that properly modeling sample-to-sample variability is key for unbiased differential expression analysis in single-cell data: "Previous work has shown that properly modeling sample-to-sample variability is key for unbiased differential expression analysis in single-cell data5,10"
- [discussion] GEDI currently supports linear manifold learning, with the option to further restrict the manifold to a hyperellipsoid: "GEDI currently supports linear manifold learning, with the option to further restrict the manifold to a hyperellipsoid"
- [readme] Fig. 4: Comparison of cluster free differential expression methods: [evidence span withheld — anchor status: external_doc]
