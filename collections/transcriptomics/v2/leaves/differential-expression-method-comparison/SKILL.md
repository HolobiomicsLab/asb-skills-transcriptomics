---
name: differential-expression-method-comparison
description: Use when evaluating or selecting a cluster-free differential expression
  method for multi-sample, multi-condition single-cell data where a simulated cohort
  with known ground-truth DE labels is available for validation.
when_to_use_negative:
- The data are bulk RNA-seq or lack a multi-sample, multi-condition design — GEDI's
  benchmark targets cluster-free DE on multisample single-cell cohorts
- No ground truth or simulation is available — the skill's evaluation (median Pearson
  r and AUROC against truth labels) cannot be executed
- The analysis requires non-linear manifold learning — GEDI currently supports only
  linear manifold learning with an optional hyperellipsoid restriction
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: GEDI
  role: Primary method under test; fit with model$setup mode='Bl2' (hyperplane) and
    mode='Bsphere' (hyperellipsoid) for cluster-free DE estimates
  repo: https://github.com/csglab/GEDI/
- name: LEMUR
  role: Competing latent embedding regression method benchmarked on the same simulated
    data
- name: miloDE
  role: Competing differential expression method run on identical simulated counts
- name: DESeq2
  role: Competing pseudobulk differential expression baseline
- name: scuttle
  role: Pre-processing / QC of the simulated SingleCellExperiment before fitting
- name: scran
  role: Pre-processing of counts prior to GEDI fitting
- name: R
  role: Analysis language; notebooks were tested with R 4.0.0
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-024-50963-0
    title: A unified model for interpretable latent embedding of multi-sample, multi-condition
      single-cell data
  source_content_hash: sha256:42dd2f704b406ceaa44886b8ad41d7cd057f2794c8dbde08207ce0a571252193
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/csglab/GEDI
  tools:
  - DESeq2
  - GEDI
  - LEMUR
  - R
  - miloDE
  - scran
  - scuttle
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# differential-expression-method-comparison

## Summary

Benchmark cluster-free differential expression methods (GEDI, LEMUR, miloDE, DESeq2 pseudobulk) on simulated single-cell data with known ground-truth DE vectors, scoring each method by median Pearson correlation of log-fold-changes and AUROC of up/down-regulated gene detection.

## When to use

Use when evaluating or selecting a cluster-free differential expression method for multi-sample, multi-condition single-cell data where a simulated cohort with known ground-truth DE labels is available for validation.

## When NOT to use

- The data are bulk RNA-seq or lack a multi-sample, multi-condition design — GEDI's benchmark targets cluster-free DE on multisample single-cell cohorts
- No ground truth or simulation is available — the skill's evaluation (median Pearson r and AUROC against truth labels) cannot be executed
- The analysis requires non-linear manifold learning — GEDI currently supports only linear manifold learning with an optional hyperellipsoid restriction

## Inputs

- Simulated cohort-level single-cell count matrix (dgCMatrix sparse format, generated from COVID-19 cohort 1 parameters)
- Sample annotation vector for each cell (sample/condition assignment)
- Ground-truth differential expression labels (DE vector per cell, with up- and down-regulated gene sets)

## Outputs

- Per-method median Pearson correlation of estimated log-fold-changes vs. ground truth (e.g., GEDI Bl2 r ≈ 0.4, LEMUR r ≈ 0.14)
- Per-method AUROC of up-regulated and down-regulated gene detection
- Metrics table fig4_de_benchmark_metrics.csv

## How to apply

Generate or load a simulated cohort-level single-cell count matrix (built from COVID-19 cohort 1 parameters) with sample annotations and ground-truth differential expression labels, and convert raw counts to sparse dgCMatrix format. Fit GEDI twice via model$setup — once with mode='Bl2' (hyperplane, L2 norm of the entire B matrix fixed) and once with mode='Bsphere' (hyperellipsoid, L2 norms of B columns fixed) — initializing latent variables with randomSeed = 1 and checking convergence with model$plotTracking. Run the competing methods on the same data: LEMUR (latent embedding regression), miloDE, and DESeq2 pseudobulk differential expression. For each method, correlate estimated log-fold-changes with the simulated ground truth, record the median Pearson correlation, and compute AUROC of up-regulated and down-regulated gene detection. Write per-method metrics (Pearson r, AUROC up, AUROC down) to fig4_de_benchmark_metrics.csv; correct reproduction yields GEDI hyperplane median Pearson r ≈ 0.4 versus LEMUR ≈ 0.14, with median AUROC ≈ 0.79 (GEDI Bl2)/0.72 (LEMUR) for up-regulated and ≈ 0.78/0.55 for down-regulated genes. The rationale is that properly modeling sample-to-sample variability is key for unbiased differential expression in single-cell data, and sequential workflows ignore the interplay between inter-sample integration and DGE analysis.

## Related tools

- **GEDI** (Primary method under test; fit with model$setup mode='Bl2' (hyperplane) and mode='Bsphere' (hyperellipsoid) for cluster-free DE estimates) — https://github.com/csglab/GEDI/
- **LEMUR** (Competing latent embedding regression method benchmarked on the same simulated data)
- **miloDE** (Competing differential expression method run on identical simulated counts)
- **DESeq2** (Competing pseudobulk differential expression baseline)
- **scuttle** (Pre-processing / QC of the simulated SingleCellExperiment before fitting)
- **scran** (Pre-processing of counts prior to GEDI fitting)
- **R** (Analysis language; notebooks were tested with R 4.0.0)

## Examples

```
devtools::install_github("csglab/GEDI")  # then follow https://csglab.github.io/GEDI_manuscript/notebooks/CFDE_benchmark.html
```

## Evaluation signals

- Median Pearson correlation between each method's estimated log-fold-changes and the simulated ground-truth DE vector: GEDI hyperplane ≈ 0.4 vs. LEMUR ≈ 0.14
- AUROC of up-regulated gene detection ≈ 0.79 for GEDI Bl2 vs. ≈ 0.72 for LEMUR
- AUROC of down-regulated gene detection ≈ 0.78 (GEDI Bl2) vs. ≈ 0.55 (LEMUR)
- GEDI convergence verified via model$plotTracking after initializing with randomSeed = 1
- Output metrics table fig4_de_benchmark_metrics.csv contains Pearson r, AUROC up, and AUROC down for every method (GEDI Bl2, GEDI Bsphere, LEMUR, miloDE, DESeq2 pseudobulk)

## Limitations

- More extensive analyses are needed to understand how factors such as number of DE genes per cell, DE magnitude, cell-cell/gene-gene correlations, variances, sample/cell counts, and sequencing depth influence the performance of GEDI and other DE methods
- GEDI currently supports linear manifold learning with an optional hyperellipsoid restriction; other parametric manifold approximations are future extensions
- Reproducibility requires specific Zenodo-hosted model/data accessions and the manuscript notebooks; code was tested with R 4.0.0 on CentOS Linux 7 and RcppEigen requires gfortran

## Evidence

- [results] On the simulated dataset, GEDI's single-cell DE estimates correlate strongly with the ground-truth DE vector of each cell, with median Pearson r = 0.4 when modeling the manifold as a hyperplane, while LEMUR showed significantly lower correlation (median r = 0.14).: [evidence span withheld — anchor status: not_found]
- [intro] Sequential workflows ignore the integration-DGE interplay, motivating the unified-model benchmark.: "this sequential approach ignores the interplay between inter-sample integration and DGE analysis"
- [discussion] Sample-to-sample variability must be modeled for unbiased single-cell DE, which is the core rationale for the comparison.: "Previous work has shown that properly modeling sample-to-sample variability is key for unbiased differential expression analysis in single-cell data"
- [readme] The manuscript repository hosts the Fig. 4 benchmark notebook used to reproduce the method comparison.: [evidence span withheld — anchor status: external_doc]
- [methods] The methods section loads the Bioconductor single-cell stack used for pre-processing in the R notebooks.: "library(scuttle)
library(scran)
library(scater)"
