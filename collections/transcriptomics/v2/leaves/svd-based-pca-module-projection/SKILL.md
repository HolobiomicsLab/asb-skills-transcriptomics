---
name: svd-based-pca-module-projection
description: Use when a normalized gene expression matrix (genes as rows, samples
  as columns) must be reduced to per-gene-set activity scores with statistical significance,
  distinguishing shifted from overdispersed modules.
when_to_use_negative:
- The expression matrix is raw/unnormalized counts — rRoma explicitly requires normalized
  data (cpm, vst, or similar) before PCA projection.
- The goal is gene-level differential expression rather than coordinated gene set
  activity — a DE tool (edgeR/DESeq2) is the appropriate choice instead.
- Datasets contain many missing values that have not been imputed — the README recommends
  using datasets for which missing values have already been imputed rather than relying
  on the mice fill step.
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_2269
tools:
- name: rROMA
  role: R package implementing the SVD-based PC1 module projection, random gene set
    null generation, and ppv/q-value computation (rRoma.R)
  repo: https://github.com/sysbio-curie/rROMA
- name: irlba
  role: Efficient truncated SVD used to compute only the first principal component
    (first column of V and first value of S)
- name: mice
  role: Imputes missing values in the global expression matrix by iterative (low-rank)
    PCA approximation before module projection
- name: edgeR
  role: cpm normalization of count data prior to rRoma analysis
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41540-024-00331-x
    title: Representation and quantification of module activity from omics data with
      rROMA
  source_content_hash: sha256:1324c5c7cf3daf664f8488784dce16ef74fad8349f2f851fe71ead9a0ad91867
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sysbio-curie/rROMA
  tools:
  - R
  - edgeR
  - irlba
  - mice
  - rROMA
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# svd-based-pca-module-projection

## Summary

Quantify gene set (module) activity by computing the first principal component via SVD on the subset of a globally centered expression matrix corresponding to each module, then assess significance of L1 (% variance explained by PC1) and Median Exp (median sample projection on PC1) against random gene set nulls. This is the core rROMA procedure for representing module activity from transcriptomic or proteomic data.

## When to use

Use when a normalized gene expression matrix (genes as rows, samples as columns) must be reduced to per-gene-set activity scores with statistical significance, distinguishing shifted from overdispersed modules.

## When NOT to use

- The expression matrix is raw/unnormalized counts — rRoma explicitly requires normalized data (cpm, vst, or similar) before PCA projection.
- The goal is gene-level differential expression rather than coordinated gene set activity — a DE tool (edgeR/DESeq2) is the appropriate choice instead.
- Datasets contain many missing values that have not been imputed — the README recommends using datasets for which missing values have already been imputed rather than relying on the mice fill step.

## Inputs

- Normalized gene expression matrix (genes as rows, samples as columns), e.g. edgeR cpm- or DESeq2 vst-normalized RNA-seq counts
- Module list (GMT file or SelectFromMSIGdb output), e.g. 50 MSigDB v6.2 HALLMARK gene sets
- Optional sample labels (e.g. sample_labels with CF vs non-CF Type)

## Outputs

- ModuleMatrix table with L1, Median Exp, ppv L1, ppv Median Exp, and BH-adjusted q L1 / q Median Exp per module
- Shifted / overdispersed classification per module (ppv <= 0.05)
- Sample projections on PC1 per module for downstream plotting (e.g. GetTopContrib, PlotOutliers)

## How to apply

First center the global expression matrix so every gene has mean zero across samples, and impute missing values with the iterative PCA method in the mice package if needed. For each module (e.g. a Hallmark gene set from MSigDB v6.2), extract its compatible genes and compute PC1 using SVD via the R irlba package (PCADims = 2), computing only the first column of V and the first value of S; derive L1 (% variance explained by PC1) and Median Exp (median of sample projections on PC1). Build null distributions from nSamples = 100 random gene sets matched in size, sharing background sets across modules whose sizes differ by less than the ApproxSamples percentage; rROMA selects representative sizes uniformly distributed on a log scale (ApproxSample parameter). Compute empirical ppv p-values by comparing observed L1 (overdispersion) and Median Exp (shift) to these nulls, then BH-correct to q values. Classify a module as shifted if 'ppv Median Exp' <= 0.05 and as overdispersed if 'ppv L1' <= 0.05, and assemble all statistics into the ModuleMatrix table. Orient PC1 using gene signs or the extreme-weight/expression multiplication methods to resolve sign ambiguity.

## Related tools

- **rROMA** (R package implementing the SVD-based PC1 module projection, random gene set null generation, and ppv/q-value computation (rRoma.R)) — https://github.com/sysbio-curie/rROMA
- **irlba** (Efficient truncated SVD used to compute only the first principal component (first column of V and first value of S))
- **mice** (Imputes missing values in the global expression matrix by iterative (low-rank) PCA approximation before module projection)
- **edgeR** (cpm normalization of count data prior to rRoma analysis)

## Examples

```
rRoma.output <- rRoma.R(MatData, AllHall)  # after AllHall <- SelectFromMSIGdb("HALLMARK")
```

## Evaluation signals

- Running rRoma.R on MatData with the 50 MSigDB v6.2 Hallmark sets reproduces the reported result: 3 of 50 pathways (FATTY_ACID_METABOLISM, APICAL_SURFACE, COAGULATION) called significant at p < 0.05.
- Every module's observed L1 and Median Exp fall within the range of their size-matched random gene set null distributions; extreme values outside the null support rejection at ppv <= 0.05.
- ppV values are BH-adjusted (q L1, q Median Exp columns present in ModuleMatrix), and classifications follow exactly: shifted iff 'ppv Median Exp' <= 0.05, overdispersed iff 'ppv L1' <= 0.05.
- The input matrix is globally centered (each gene mean zero across samples) before SVD; PC1 is computed with irlba at PCADims = 2 so only the first component is returned.
- Random background sets are shared across modules whose sizes differ by less than the ApproxSamples percentage, with nSamples = 100 random sets per size.

## Limitations

- The CF case study lacked statistical power; authors note it would require additional transcriptional datasets and testing of several gene set reference databases.
- Default values of hyperparameters (extreme-weight percentage, outlier z-score threshold, number of PCs, distance threshold, ApproxSample) are not reported in the article.
- No explicit software version stated for R, irlba, or mice; random permutations depend on fixing the seed for reproducibility.

## Evidence

- [discussion] rROMA uses the R irlba package36,37, which allows to focus on the first principal component: "rROMA uses the R irlba package36,37, which allows to focus on the first principal component"
- [discussion] random gene set null for L1: "uses a random gene set procedure to generate a null distribution for the L1 amount of variance explained by the PC1 and calculates the p-value by comparing the observed L1 to the null distribution."
- [discussion] log-spaced null sizes: "rROMA constructs null distributions for a representative list of gene set sizes. These are selected to be uniformly distributed in the logarithmic scale between the minimum and maximum size of the"
- [readme] normalized data requirement: "Warning: rRoma works with **normalized** data. Here cpm from edgeR was used for normalisation, but you could also use vst from DeSeq2"
- [readme] run rRoma on MatData: "You can now run rRoma on our dataset by simply specifying the expression dataset and the modules you want to test. Here we are fixing the seed"
- [discussion] 3 of 50 hallmark significant: "out of the 50 hallmark pathways tested, 3 were significantly active: FATTY_ACID_METABOLISM, APICAL_SURFACE, and COAGULATION."
