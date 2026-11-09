---
name: stepwise-regression-gene-selection
description: Use when you have a computed GGC matrix from normalized scRNA-seq data
  and need a minimally redundant seed feature set for DUBStepR-style guilt-by-association
  feature expansion.
when_to_use_negative:
- The GGC matrix has not yet been computed — run the correlation-range z-score gene
  filtering (getCorrelationRange/getFilteredData) and getGGC() steps first
- You only need the final optimal feature set rather than the minimally redundant
  seed core — use getOptimalFeatureSet(), which expands the seed via guilt-by-association
- Input is raw counts without QC, normalization (logNormalize), or gene filtering
  — the stepwise regression assumes a filtered, normalized expression matrix
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: DUBStepR
  role: Provides runStepwiseReg() and findElbow() used to perform stepwise regression
    on the GGC matrix and detect the scree-plot elbow defining the seed gene set size
  repo: github.com/prabhakarlab/DUBStepR
- name: R
  role: Runtime environment (R >= 3.5.0) in which the DUBStepR package functions are
    executed
- name: Seurat
  role: Recommended framework for supplying the normalized single-cell data object
    from which the GGC matrix is derived
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-021-26085-2
    title: DUBStepR is a scalable correlation-based feature selection method for accurately
      clustering single-cell data
  source_content_hash: sha256:295758f50c14c2835a0d880ff762cff84f48e65ffb28929d9ba52ff6a7965858
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DUBStepR
  - R
  - Seurat
  - graphics
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# stepwise-regression-gene-selection

## Summary

Runs DUBStepR's stepwise regression on a gene-gene correlation (GGC) matrix to iteratively regress out the gene explaining maximal residual variance, yielding an ordered, minimally redundant seed gene set whose size is set by the elbow point of the stepwise-regression scree plot.

## When to use

Use when you have a computed GGC matrix from normalized scRNA-seq data and need a minimally redundant seed feature set for DUBStepR-style guilt-by-association feature expansion.

## When NOT to use

- The GGC matrix has not yet been computed — run the correlation-range z-score gene filtering (getCorrelationRange/getFilteredData) and getGGC() steps first
- You only need the final optimal feature set rather than the minimally redundant seed core — use getOptimalFeatureSet(), which expands the seed via guilt-by-association
- Input is raw counts without QC, normalization (logNormalize), or gene filtering — the stepwise regression assumes a filtered, normalized expression matrix

## Inputs

- Gene-gene correlation (GGC) matrix computed by getGGC() from a normalized scRNA-seq dataset (e.g., pbmc_norm_small_data shipped with the DUBStepR R package)
- Number of regressed steps (e.g., 30 regressed steps extrapolated to 100)

## Outputs

- Ordered list of regressed-out genes (prioritized seed features)
- Seed gene set membership (genes up to the detected elbow index)
- Scree plot of variance explained per regression step (e.g., scree_plot.png)
- Results table (e.g., stepwise_regression_results.csv)

## How to apply

After filtering mitochondrial, ribosomal, spike-in, and pseudogenes and computing the GGC matrix with getGGC(), run runStepwiseReg() on the GGC matrix: at each step the gene explaining maximal residual variance is regressed out via a single GGC matrix multiplication (30 regressed steps extrapolated to 100). Record the variance explained at each step and plot it as a scree plot. Use findElbow() to detect the elbow point of the scree plot and cut the ordered regressed-out gene list there; genes up to the elbow index form the seed gene set, which DUBStepR subsequently expands by adding correlated features (guilt-by-association). The stepwise approach is key because it identifies an initial core set of genes that most strongly represent coherent expression variation in the dataset.

## Related tools

- **DUBStepR** (Provides runStepwiseReg() and findElbow() used to perform stepwise regression on the GGC matrix and detect the scree-plot elbow defining the seed gene set size) — github.com/prabhakarlab/DUBStepR
- **R** (Runtime environment (R >= 3.5.0) in which the DUBStepR package functions are executed)
- **Seurat** (Recommended framework for supplying the normalized single-cell data object from which the GGC matrix is derived)

## Examples

```
library(DUBStepR); ggc <- getGGC(pbmc_norm_small_data); sw <- runStepwiseReg(ggc); elbow <- findElbow(sw$err, plot = TRUE, returnIndex = TRUE); seed_genes <- sw$err_rank_genes[1:elbow]
```

## Evaluation signals

- The scree plot exhibits a clear elbow with monotonically decreasing variance explained per regression step
- The ordered list contains the expected number of regressed-out genes (e.g., 30 regressed steps extrapolated to 100)
- The seed gene set equals exactly the genes up to the detected elbow index from findElbow()
- stepwise_regression_results.csv contains the ordered gene list and seed membership; scree_plot.png is a valid plot file
- Downstream silhouette index should approach its maximum at a relatively small number (200-500) of feature genes after expansion

## Limitations

- Elbow point computation algorithm is not described in the main text (deferred to Supp. Note), so the exact elbow criterion is opaque from the article alone
- Seed gene set size depends on elbow detection, which may be ambiguous for scree plots without a clear elbow
- Key results on feature-gene counts are deferred to supplementary figures (Supp. Fig. S3, S4) not included in the main text

## Evidence

- [intro] stepwise regression identifies minimally redundant seed features: "DUBStepR then performs stepwise regression on the GGC matrix to identify a minimally redundant subset of seed features"
- [intro] stepwise core set rationale: "A key feature of DUBStepR is the use of a stepwise approach to identify an initial core set of genes that most strongly represent coherent expression variation in the dataset."
- [methods] findElbow function signature: "`findElbow(y, ylab, plot, returnIndex)`"
- [discussion] elbow details deferred to supplementary: "Elbow point computation is described in Supp. Note"
- [discussion] correlation-range filtering precedes GGC exploitation: "After filtering genes based on a correlation range score, DUBStepR exploits structure in the gene–gene correlation matrix to prioritize genes as features for clustering."
- [intro] seed expansion by guilt-by-association: "which are then expanded by adding correlated features (guilt-by-association)"
