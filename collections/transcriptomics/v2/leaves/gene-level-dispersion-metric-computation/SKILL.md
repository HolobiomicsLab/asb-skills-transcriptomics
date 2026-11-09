---
name: gene-level-dispersion-metric-computation
description: Use when benchmarking imputation methods on scRNA-seq data and you need
  a transcriptome-landscape preservation metric — e.g. comparing median gene-level
  normalized intra dispersion across raw and imputed matrices over multiple datasets.
when_to_use_negative:
- The analysis goal is clustering-quality comparison (Purity Index / ARI against true
  labels via k-means) rather than transcriptome-landscape preservation via dispersion.
- The input matrices have not been pre-filtered (all-zero genes, genes expressed in
  fewer than two cells, or genes with less than one percent non-zero entries) — filter
  first, as the dispersion values are otherwise distorted.
- The data are simulated datasets with different generative parameters than the 25
  real scRNA-seq benchmark datasets; the 3.6e-3 reference value applies to the real-dataset
  benchmark.
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: scISR
  role: Produces the imputed expression matrices whose gene-level normalized intra
    dispersion is computed and compared against raw data and competitor methods.
  repo: https://github.com/duct317/scISR
- name: R stats
  role: R package whose lm function is used in the scISR implementation for the regression-based
    estimation underlying the imputed matrices being evaluated.
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41598-022-06500-4
    title: A novel method for single-cell data imputation using subspace regression
  source_content_hash: sha256:6f7c24b42dc12fa0817e7cfa6330f489141498b4ff9c34a41af1c5f53397973b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/duct317/scISR
  tools:
  - R stats
  - scISR
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# gene-level-dispersion-metric-computation

## Summary

Computes gene-level normalized intra dispersion on raw versus imputed scRNA-seq expression matrices to quantify how much of the transcriptome landscape is preserved after imputation. In the scISR benchmark this yields a median gene-level normalized intra dispersion of 3.6e-3 for scISR, compared against raw data and competitor imputation methods.

## When to use

Use when benchmarking imputation methods on scRNA-seq data and you need a transcriptome-landscape preservation metric — e.g. comparing median gene-level normalized intra dispersion across raw and imputed matrices over multiple datasets.

## When NOT to use

- The analysis goal is clustering-quality comparison (Purity Index / ARI against true labels via k-means) rather than transcriptome-landscape preservation via dispersion.
- The input matrices have not been pre-filtered (all-zero genes, genes expressed in fewer than two cells, or genes with less than one percent non-zero entries) — filter first, as the dispersion values are otherwise distorted.
- The data are simulated datasets with different generative parameters than the 25 real scRNA-seq benchmark datasets; the 3.6e-3 reference value applies to the real-dataset benchmark.

## Inputs

- Raw scRNA-seq expression matrix (genes x cells) with true cell-type labels
- Imputed expression matrix from scISR (e.g. output of scISR(data = raw))
- Imputed expression matrices from competitor methods (scImpute, SAVER, MAGIC, DrImpute, scScope, DCA, DeepImpute, BISCUIT)
- Dataset metadata/accession IDs for the 25 benchmark datasets (GSE53386, GSE52583, GSE36552, E-MTAB-3321, GSE45719, SRP041736, GSE67835, GSE59739, GSE75140, GSE65525, plus the remaining 15)

## Outputs

- Per-gene normalized intra dispersion values for each condition (raw and each imputation method)
- Median gene-level normalized intra dispersion per method (scISR target: 3.6e-3)
- Aggregated summary table written to stability_summary.csv

## How to apply

Load the raw and imputed expression matrices for each dataset (in the scISR benchmark, 25 real scRNA-seq datasets such as Fan GSE53386, Goolam E-MTAB-3321, Deng GSE45719, Pollen SRP041736, Klein GSE65525). Before computing the metric, filter out genes with only zero entries and genes expressed in fewer than two cells or with less than one percent non-zero entries. For each gene, compute a normalized intra dispersion (within-class dispersion normalized against the total), then take the median gene-level value per condition (raw, scISR, and each competitor such as scImpute, SAVER, MAGIC, DrImpute, scScope, DCA, DeepImpute, BISCUIT). Rationale: a higher preserved intra dispersion means the imputation did not over-smoothen the cell-to-cell stochasticity that represents meaningful biological variation; imputation methods that alter all zero-valued entries, including true zeros, risk collapsing this dispersion. Assemble the per-method medians into a summary table (stability_summary.csv in the reference workflow) so raw vs imputed preservation can be compared side by side.

## Related tools

- **scISR** (Produces the imputed expression matrices whose gene-level normalized intra dispersion is computed and compared against raw data and competitor methods.) — https://github.com/duct317/scISR
- **R stats** (R package whose lm function is used in the scISR implementation for the regression-based estimation underlying the imputed matrices being evaluated.)

## Evaluation signals

- Median gene-level normalized intra dispersion for scISR equals 3.6e-3 versus raw data and all competitor imputation methods.
- The summary table (stability_summary.csv) contains one aggregated dispersion value per condition (raw, scISR, and each of the eight competitor methods) across the 25 datasets.
- All-zero genes and genes expressed in less than two cells (or with less than one percent non-zero entries) are absent from the matrices before the dispersion is computed.
- scISR's dispersion value is compared consistently against the same filtered matrices used for the Purity Index comparison (15/25 datasets above raw, Wilcoxon p = 0.007), confirming the same data versions were used.
- Per-gene dispersion values are finite and non-negative; NaN/Inf entries indicate unfiltered genes or misaligned matrices.

## Limitations

- The article defers key robustness evidence to supplementary figures/sections not included (e.g. S19, S21, S24), so the dispersion analysis's full robustness discussion is not directly reproducible from the main text.
- No version is stated for the R stats package or the scISR software, limiting reproducibility of the exact dispersion numbers.
- Imputation methods that alter all zero-valued entries, including true zeros, can artificially depress intra dispersion by over-smoothening meaningful cell-to-cell biological variation — a known failure mode of both model-based and model-free methods described in the article.

## Evidence

- [abstract] scISR consistently improves the quality of cluster analysis regardless of dropout rates, normalization techniques, and quantification schemes: "scISR consistently improves the quality of cluster analysis regardless of dropout rates, normalization techniques, and quantification schemes"
- [results] we use k-means to cluster the cells using the true number of cell types k as the number of clusters.: "we use k-means to cluster the cells using the true number of cell types k as the number of clusters."
- [results] genes expressed in less than two cells; and (ii) genes that have less than one percent of non-zero-valued entries.: "(i) genes expressed in less than two cells; and (ii) genes that have less than one percent of non-zero-valued entries."
- [discussion] In our implementation, we adopt the lm function that is available in the stats R package.: "In our implementation, we adopt the lm function that is available in the stats R package."
- [readme] Filter genes that have only zeros from raw data: "Filter genes that have only zeros from raw data"
