---
name: jive-multi-source-decomposition
description: Use when you have two or more data matrices (e.g., scRNA-seq batches,
  or matched gene expression/methylation/miRNA sources like BRCA TCGA data) sharing
  samples/cells and need to separate shared signal from batch-specific variation.
when_to_use_negative:
- When the data are a single matrix with no multi-source/multi-batch structure — there
  is no joint-vs-individual decomposition to estimate
- When the goal is a benchmarked batch-correction comparison against methods like
  Seurat v5, Harmony, LIGER, or ComBat-seq on metrics such as kBET, ASW, and LISI
  — JIVE is one candidate but the skill alone does not cover those comparators' workflows
- For very large scRNA-seq datasets when restricted to the original r.jive implementation,
  which was designed for bulk sequencing data and is computationally infeasible at
  single-cell scale — use the enhanced scJIVE implementation instead
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: JIVE (scJIVE)
  role: Enhanced JIVE implementation for large-scale scRNA-seq data, installed via
    devtools::install_github from the oconnell-statistics-lab/scJIVE repo
  repo: https://github.com/oconnell-statistics-lab/scJIVE
- name: r.jive
  role: Original R package providing jive(), jive.predict(), jive.perm(), and bic.jive()
    used to estimate the decomposition and select ranks
- name: R
  role: Runtime environment for the r.jive/scJIVE packages
- name: SpatioTemporal
  role: Provides SVDmiss for imputing missing values before decomposition
- name: gplots
  role: Plotting functions supporting showHeatmaps and other visualizations
provenance:
  source_task_ids:
  - task_007
  - task_003
  source_papers:
  - doi: 10.1093/bioadv/vbae134
    title: Batch-effect correction in single-cell RNA sequencing data using JIVE
  source_content_hash: sha256:5395e4fd9c75b60aab9b64025afc307f1e4d2b449634b80cbb9362dd4966bd33
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/oconnell-statistics-lab/scJIVE
  tools:
  - Harmony
  - JIVE (scJIVE)
  - R
  - Seurat v5
  - SpatioTemporal
  - gplots
  - r.jive
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# jive-multi-source-decomposition

## Summary

Decompose multi-source (multi-batch or multi-omics) matrices into a joint structure capturing shared biological variability plus per-source individual structures capturing technical/batch variation, using the JIVE method via the scJIVE R implementation for computationally efficient application to large single-cell data.

## When to use

Use when you have two or more data matrices (e.g., scRNA-seq batches, or matched gene expression/methylation/miRNA sources like BRCA TCGA data) sharing samples/cells and need to separate shared signal from batch-specific variation.

## When NOT to use

- When the data are a single matrix with no multi-source/multi-batch structure — there is no joint-vs-individual decomposition to estimate
- When the goal is a benchmarked batch-correction comparison against methods like Seurat v5, Harmony, LIGER, or ComBat-seq on metrics such as kBET, ASW, and LISI — JIVE is one candidate but the skill alone does not cover those comparators' workflows
- For very large scRNA-seq datasets when restricted to the original r.jive implementation, which was designed for bulk sequencing data and is computationally infeasible at single-cell scale — use the enhanced scJIVE implementation instead

## Inputs

- Two or more data matrices sharing samples/cells across sources (e.g., scRNA-seq batches such as 2000 genes x 1867 cells from Bacher CD4+ T-cell data, or BRCA expression/methylation/miRNA matrices)
- Log-normalized counts with top variable features selected (e.g., top 2000 variable genes)
- New prediction data for out-of-sample decomposition via jive.predict(data.new, jive.output)

## Outputs

- JIVE fit object with a joint low-rank estimate and per-source individual low-rank estimates and residuals
- Per-source variance-proportion breakdown (joint/individual/residual) visualizable via showVarExplained
- Low-rank heatmap and PCA visualizations from showHeatmaps and showPCA
- Predicted decomposition for new data from jive.predict
- Sample-cluster groupings recoverable from the joint structure (e.g., clustering matched three previously identified BRCA clusters)

## How to apply

Treat each batch or data source as one matrix input to the JIVE decomposition, calling jive() (default estimator) from r.jive or the enhanced scJIVE implementation, which scales each matrix by its total variation and enforces row-orthogonality between the joint and individual estimates. Select ranks via either the permutation-based approach (jive.perm with nperms and alpha) or the BIC-motivated approach (bic.jive with n, d, conv, maxiter parameters), optionally using SVDmiss for missing-value imputation. Inspect the fitted object with showVarExplained (barchart of variation explained), showHeatmaps (low-rank matrix approximations), and showPCA (point-cloud view). Apply to single-cell data by letting the joint structure capture biological variability and the individual structures capture technical batch effects; on matched TCGA BRCA expression/methylation/miRNA data the expected fit was a final joint rank of 2 with individual ranks of 27, 26, and 25. The enhanced scJIVE implementation produces near-identical decompositions to the original while running ~8-10x faster (~4.1 s vs ~35.8 s mean on 200x1000 simulated data over 20 runs).

## Related tools

- **JIVE (scJIVE)** (Enhanced JIVE implementation for large-scale scRNA-seq data, installed via devtools::install_github from the oconnell-statistics-lab/scJIVE repo) — https://github.com/oconnell-statistics-lab/scJIVE
- **r.jive** (Original R package providing jive(), jive.predict(), jive.perm(), and bic.jive() used to estimate the decomposition and select ranks)
- **R** (Runtime environment for the r.jive/scJIVE packages)
- **SpatioTemporal** (Provides SVDmiss for imputing missing values before decomposition)
- **gplots** (Plotting functions supporting showHeatmaps and other visualizations)

## Examples

```
devtools::install_github("oconnell-statistics-lab/scJIVE"); result <- jive(list(batch1 = X1, batch2 = X2)); showVarExplained(result)
```

## Evaluation signals

- Row-orthogonality holds between the joint and individual estimates and among the individual estimates themselves
- The estimated joint structure reproduces known sample groupings (e.g., matches the three previously identified BRCA sample clusters; clusts vector from the lock2013bcc analysis)
- Variance-proportion summary (showVarExplained) attributes variation to joint, individual, and residual components with individual structures concentrated in technical batch variation
- The enhanced scJIVE implementation gives near-identical results to the original r.jive (e.g., close-to-identical joint/individual/residual variance proportions on 200x1000 rank-1 simulated data) at an ~8-10x runtime ratio (~4.1 s vs ~35.8 s mean over 20 runs)
- If permutation rank selection is used, the chosen joint/individual ranks pass the permutation test at the specified alpha; BIC rank selection minimizes the bic.jive criterion over candidate ranks

## Limitations

- The original JIVE implementation was designed for bulk sequencing data and is computationally infeasible for large-scale single-cell datasets; use scJIVE for that setting
- JIVE performed best in scenarios where batch sizes are balanced; performance may degrade with imbalanced batch sizes
- Missing values require imputation (via SVDmiss from SpatioTemporal) before decomposition
- Two rank-selection routes (permutation-based and BIC-motivated) may yield different rank choices; the choice must be made explicitly

## Evidence

- [intro] scJIVE aims to decompose scRNA-seq datasets into a joint structure capturing the true biological variability and individual structures which capture technical variability within each batch.: "scJIVE aims to decompose scRNA-seq datasets into a joint structure capturing the true biological variability and individual structures which capture technical variability within each batch."
- [methods] The default function to estimate the JIVE decomposition is `jive`: "The default function to estimate the JIVE decomposition is `jive`"
- [methods] Final joint rank: 2 , final individual ranks: 27 26 25: "Final joint rank: 2 , final individual ranks: 27 26 25"
- [methods] One is a permutation-based approach described in @Lock, the other is a BIC-motivated approach.: "One is a permutation-based approach described in @Lock, the other is a BIC-motivated approach."
- [methods] The package requires two dependencies: **SpatioTemporal** for inputing missing values, and **gplots** for some plotting functions.: "The package requires two dependencies: **SpatioTemporal** for inputing missing values, and **gplots** for some plotting functions."
- [methods] We see that the estimated joint corresponds well to the three previously identified clusters: "We see that the estimated joint corresponds well to the three previously identified clusters"
- [readme] You can install the development version of scJIVE from GitHub: "You can install the development version of scJIVE from GitHub with:

``` r
# install.packages("devtools")
devtools::install_github("oconnell-statistics-lab/scJIVE")
```"
