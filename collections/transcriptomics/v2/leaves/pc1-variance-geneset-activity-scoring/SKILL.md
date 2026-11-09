---
name: pc1-variance-geneset-activity-scoring
description: Use when you have a normalized gene-by-sample expression matrix and gene
  sets (e.g., MSigDB GMT) and need sample-wise module activity scores with statistical
  significance distinguishing shifted vs overdispersed gene sets.
when_to_use_negative:
- The expression matrix is not normalized (rRoma works with normalized data — apply
  cpm/edgeR, vst/DESeq2, or another normalization first)
- The goal is gene-level differential expression rather than coordinated module activity
  scoring
- The dataset contains many missing values that have not been imputed — rROMA recommends
  using datasets for which missing values have already been imputed (mice imputation
  is a fallback)
edam_operation: http://edamontology.org/operation_0571
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0085
- http://edamontology.org/topic_3518
tools:
- name: rROMA
  role: R package implementing PC1-based module activity scoring with L1/Median Exp
    statistics and random-gene-set significance testing
  repo: https://github.com/sysbio-curie/rROMA
- name: irlba
  role: Computes only the first principal component (first column of V, first value
    of S) via truncated SVD for efficiency
- name: mice
  role: Imputes missing values in the global expression matrix via iterative PCA before
    activity scoring
- name: edgeR
  role: Provides cpm normalization of RNA-seq counts prior to rRoma analysis
- name: DESeq2
  role: Alternative normalization (vst) accepted as input to rRoma
- name: Roma
  role: Java implementation of the original ROMA algorithm by Zinovyev and co-authors
  repo: https://github.com/sysbio-curie/Roma
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41540-024-00331-x
    title: Representation and quantification of module activity from omics data with
      rROMA
  source_content_hash: sha256:40418eea6d51a3f3d9b5b1b286eb94c4c4b93b159d984da993ecc6d4dad49deb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sysbio-curie/rROMA
  tools:
  - DESeq2
  - R
  - Roma
  - edgeR
  - irlba
  - mice
  - rROMA
  edam_operation: http://edamontology.org/operation_0571
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0085
  - http://edamontology.org/topic_3518
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6470588235294118
n_claims: 17
---

# pc1-variance-geneset-activity-scoring

## Summary

Quantify gene set (module) activity from a normalized omics expression matrix by computing PC1 via SVD (irlba) for each gene set, deriving the L1 statistic (% variance explained by PC1) and Median Exp (median sample projection on PC1), and testing significance against null distributions from random gene sets of matched size.

## When to use

Use when you have a normalized gene-by-sample expression matrix and gene sets (e.g., MSigDB GMT) and need sample-wise module activity scores with statistical significance distinguishing shifted vs overdispersed gene sets.

## When NOT to use

- The expression matrix is not normalized (rRoma works with normalized data — apply cpm/edgeR, vst/DESeq2, or another normalization first)
- The goal is gene-level differential expression rather than coordinated module activity scoring
- The dataset contains many missing values that have not been imputed — rROMA recommends using datasets for which missing values have already been imputed (mice imputation is a fallback)

## Inputs

- Normalized gene expression matrix (genes as rows, samples as columns; e.g., cpm from edgeR or vst from DESeq2), with sample column names and gene row names
- Module/gene set list loaded from MSigDB (e.g., SelectFromMSIGdb('HALLMARK') on MsigDB v6.2) or from a GMT file via ReadGMTFile
- Optional sample labels (e.g., sample_labels with CF vs non-CF Type annotations)
- PCADims = 2, nSamples = 100 random gene sets, ApproxSample percentage for sharing null background sets, fixed random seed

## Outputs

- ModuleMatrix table with per-module L1, Median Exp, ppv L1, ppv Median Exp, and BH-adjusted q L1 / q Median Exp
- Shifted/overdispersed classification per gene set (threshold 0.05)
- Sample-wise PC1 activity scores and PC1 gene weights per module (explorable via GetTopContrib, PlotOutliers, SelectGeneSets)

## How to apply

Center the global expression matrix so every gene has mean zero across samples (required before rROMA's PCA interpretation), imputing any missing values with the mice package's low-rank iterative PCA. For each gene set, extract compatible genes and compute PC1 via SVD using the R irlba package (PCADims = 2, computing only the first column of V and first value of S), then derive L1 (percentage of variance explained by PC1) and Median Exp (median of sample projections on PC1). Build null distributions by sampling nSamples = 100 random gene sets matched in size, sharing background sets across modules whose sizes differ by less than the ApproxSamples percentage; null sets are selected on a log-spaced grid of gene set sizes between the minimum and maximum module size (ApproxSample parameter). Compute empirical ppv p-values by comparing each module's observed L1 (overdispersion) and Median Exp (shift) against its null distribution, then BH-correct for multiple testing (q L1, q Median Exp). Classify a module as shifted if 'ppv Median Exp' <= 0.05 and as overdispersed if 'ppv L1' <= 0.05, and assemble all statistics in the ModuleMatrix table. Resolve PC1 sign ambiguity (which gene direction defines 'high activity') using gene signs or extreme-weight/expression multiplication methods.

## Related tools

- **rROMA** (R package implementing PC1-based module activity scoring with L1/Median Exp statistics and random-gene-set significance testing) — https://github.com/sysbio-curie/rROMA
- **irlba** (Computes only the first principal component (first column of V, first value of S) via truncated SVD for efficiency)
- **mice** (Imputes missing values in the global expression matrix via iterative PCA before activity scoring)
- **edgeR** (Provides cpm normalization of RNA-seq counts prior to rRoma analysis)
- **DESeq2** (Alternative normalization (vst) accepted as input to rRoma)
- **Roma** (Java implementation of the original ROMA algorithm by Zinovyev and co-authors) — https://github.com/sysbio-curie/Roma

## Examples

```
set.seed(69); rRoma.output <- rRoma.R(MatData, AllHall)
```

## Evaluation signals

- Run rRoma.R on MatData with the 50 MSigDB v6.2 Hallmark sets and reproduce the reported result: 3 of 50 pathways (FATTY_ACID_METABOLISM, APICAL_SURFACE, COAGULATION) significant at p < 0.05
- Check ModuleMatrix columns exist: L1, Median Exp, ppv L1, ppv Median Exp, q L1, q Median Exp
- Verify classification logic: modules flagged shifted have 'ppv Median Exp' <= 0.05; modules flagged overdispersed have 'ppv L1' <= 0.05
- Confirm the expression matrix was globally centered (every gene mean zero across samples) before PCA interpretation
- Verify random gene sets are matched to module sizes and that background sets are shared across modules whose sizes differ by less than the ApproxSamples percentage

## Limitations

- PC1 sign ambiguity: which gene direction defines high activity must be resolved with orientation methods (gene signs or extreme-weight/expression multiplication); several methods exist and may disagree
- rROMA uses a random gene set procedure, so a fixed seed should be set for reproducibility of ppv values
- The CF case study lacked statistical power; authors note additional transcriptional datasets and several gene set reference databases would be needed
- Default values of hyperparameters (e.g., extreme-weight percentage, outlier z-score threshold, ApproxSample) are not reported in the article
- No explicit software versions for R, irlba, or mice are stated

## Evidence

- [intro] assess the activity of a gene set by determining the maximum amount of one-dimensional variance, which is represented by the first principal component (PC1): "The underlying hypothesis of rROMA is to assess the activity of a gene set by determining the maximum amount of one-dimensional variance, which is represented by the first principal component (PC1)"
- [discussion] rROMA uses the R irlba package36,37, which allows to focus on the first principal component: "rROMA uses the R irlba package36,37, which allows to focus on the first principal component, by computing only the first column of V and the first value of S."
- [discussion] uses a random gene set procedure to generate a null distribution for the L1 amount of variance explained by the PC1: "uses a random gene set procedure to generate a null distribution for the L1 amount of variance explained by the PC1 and calculates the p-value by comparing the observed L1 to the null distribution."
- [readme] rRoma works with **normalized** data. Here cpm from edgeR was used for normalisation: "Warning: rRoma works with **normalized** data. Here cpm from edgeR was used for normalisation, but you could also use vst from DeSeq2 or any other method you deem interesting."
- [discussion] out of the 50 hallmark pathways tested, 3 were significantly active: FATTY_ACID_METABOLISM, APICAL_SURFACE, and COAGULATION.: "out of the 50 hallmark pathways tested, 3 were significantly active: FATTY_ACID_METABOLISM, APICAL_SURFACE, and COAGULATION."
