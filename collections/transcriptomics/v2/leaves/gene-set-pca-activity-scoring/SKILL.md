---
name: gene-set-pca-activity-scoring
description: Use when you have a normalized gene-by-sample expression matrix (RNA-seq
  cpm/vst or proteomics) plus gene sets (GMT/MSigDB or custom signatures) and need
  per-sample module activity scores with statistical significance.
when_to_use_negative:
- The expression matrix is raw counts or otherwise unnormalized — rRoma explicitly
  requires normalized data before PC1 computation
- The question is differential expression of individual genes rather than coordinated
  module/sample-level activity — DE tools like edgeR/DESeq2 testing are the wrong
  instrument
- Datasets contain many missing values that have not been pre-imputed — the README
  recommends using datasets for which missing values have already been imputed rather
  than relying on the mice fallback
edam_operation: http://edamontology.org/operation_0571
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3523
- http://edamontology.org/topic_0091
tools:
- name: rROMA
  role: Core R package implementing the PC1-based module activity algorithm with random
    gene set null distributions
  repo: github.com/sysbio-curie/rROMA
- name: irlba
  role: Computes only the first principal component (first column of V, first value
    of S) for each gene set for efficiency
- name: mice
  role: Iterative PCA method used to impute missing values in the global expression
    matrix (FillNAMethod)
- name: edgeR
  role: cpm normalization of count data before running rRoma.R
- name: DESeq2
  role: Alternative vst normalization for count data prior to rROMA
provenance:
  source_task_ids:
  - task_003
  - task_004
  source_papers:
  - doi: 10.1038/s41540-024-00331-x
    title: Representation and quantification of module activity from omics data with
      rROMA
  source_content_hash: sha256:b4ae80a27e4678bae197f2a826cef82685cf1e72e8145b59a4a46886751630ad
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DESeq2
  - R
  - edgeR
  - irlba
  - mice
  - rROMA
  edam_operation: http://edamontology.org/operation_0571
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3523
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# gene-set-pca-activity-scoring

## Summary

Quantify gene-set (module) activity in a normalized omics expression matrix by computing the first principal component (PC1) of each gene set with rROMA, and assess significance of shifted vs over-dispersed modules against random gene-set null distributions.

## When to use

Use when you have a normalized gene-by-sample expression matrix (RNA-seq cpm/vst or proteomics) plus gene sets (GMT/MSigDB or custom signatures) and need per-sample module activity scores with statistical significance.

## When NOT to use

- The expression matrix is raw counts or otherwise unnormalized — rRoma explicitly requires normalized data before PC1 computation
- The question is differential expression of individual genes rather than coordinated module/sample-level activity — DE tools like edgeR/DESeq2 testing are the wrong instrument
- Datasets contain many missing values that have not been pre-imputed — the README recommends using datasets for which missing values have already been imputed rather than relying on the mice fallback

## Inputs

- Normalized gene-by-sample expression matrix (row names = gene names, column names = samples), e.g. edgeR cpm or DESeq2 vst output for GSE176121 airway epithelial RNA-seq
- Module list / GMT file of gene sets, e.g. MSigDB Hallmark v6.2 (50 HALLMARK sets) or custom cell-type signatures loaded with ReadGMTFile
- Sample labels (e.g. sample_labels with Type CF/non-CF or culture-medium group) for grouping and heatmaps

## Outputs

- rRoma.output object with ModuleMatrix containing per-module PC1 sample scores, L1 and Median Exp ppv p/q values
- Classification of modules as shifted (ppv Median Exp < 0.05) or over-dispersed (ppv L1 < 0.05)
- Sample-score heatmaps per module (Plot.Genesets.Samples, cluster_cols=TRUE, GroupInfo labels), exportable as PNG
- Top contributing genes per module (GetTopContrib) and outlier visualizations (PlotOutliers)

## How to apply

Start from a normalized gene-by-sample matrix (edgeR cpm for RNA-seq counts, or DESeq2 vst; the breast cancer case used a 9771-protein x 122-sample mass-spec matrix) and center the global matrix; missing values are imputed with the iterative PCA method from the mice package (FillNAMethod parameters passable, though pre-imputed data is recommended). Define modules via SelectFromMSIGdb("HALLMARK") or ReadGMTFile for custom signatures (e.g., Plasschaert/Okuda cell-type sets), then run rRoma.R fixing the random seed, using recommended settings such as UseWeights=TRUE, GeneOutDetection='L1OutExpOut', PCSignMode='UseMeanExpressionAllWeights', PCSignThr=0.9, and nSamples >= 100. PC1 is computed efficiently with irlba (only the first column of V and first value of S); its sign ambiguity is resolved by PCSignMode options; outlier genes are removed via leave-one-out L1 z-scores, with hyperparameter limits on the removable gene proportion. Significance is obtained by comparing observed L1 (variance explained by PC1) and Median Exp (shift) statistics to null distributions built from random gene sets of matched size on a log-spaced grid (ApproxSample). Explore results with SelectGeneSets, GetTopContrib, PlotOutliers, and Plot.Genesets.Samples heatmaps with GroupInfo labels.

## Related tools

- **rROMA** (Core R package implementing the PC1-based module activity algorithm with random gene set null distributions) — github.com/sysbio-curie/rROMA
- **irlba** (Computes only the first principal component (first column of V, first value of S) for each gene set for efficiency)
- **mice** (Iterative PCA method used to impute missing values in the global expression matrix (FillNAMethod))
- **edgeR** (cpm normalization of count data before running rRoma.R)
- **DESeq2** (Alternative vst normalization for count data prior to rROMA)

## Examples

```
library(rRoma); set.seed(69); rRoma.output <- rRoma.R(MatData, AllHall)
```

## Evaluation signals

- rRoma.output$ModuleMatrix yields exactly 34 shifted and 6 over-dispersed Hallmark pathways at p < 0.05 on the 9771-protein x 122-sample breast cancer proteomic dataset
- On GSE176121 with Hallmark sets, exactly 3 of 50 pathways are significantly active: FATTY_ACID_METABOLISM, APICAL_SURFACE, and COAGULATION
- With Plasschaert cell-type signatures, ciliated module scores are higher in UNC medium and secretory module scores higher in SC medium, matching reported abundances
- Fixing the random seed reproduces identical module scores and ppv values across reruns
- Breast cancer six-cluster partition of sample module scores maps clusters 1/5 to basal/Luminal B and clusters 4/6 to Luminal A against PAM50 labels

## Limitations

- The CF transcriptomic case study lacks statistical power; the authors state it would require additional transcriptional datasets and testing of several gene set reference databases
- Default values of hyperparameters (extreme-weight percentage, outlier z-score threshold, number of PCs, distance threshold, ApproxSample) are not reported in the article and may need tuning
- mice-based imputation is a fallback; the README recommends datasets with pre-imputed missing values
- No software versions are stated for R, irlba, or mice, complicating exact reproduction

## Evidence

- [abstract] Here, we present the rROMA software package for fast and accurate computation of the activity of gene sets with coordinated expression.: "Here, we present the rROMA software package for fast and accurate computation of the activity of gene sets with coordinated expression."
- [methods] rRoma works with normalized data, cpm from edgeR: "Warning: rRoma works with **normalized** data. Here cpm from edgeR was used for normalisation"
- [methods] rRoma utilizes the iterative PCA method implemented in the mice package: "Roma starts by imputing them using mice. More parameters can be passed to the mice function using `FillNAMethod`"
- [discussion] rROMA uses the R irlba package to focus on the first principal component: "rROMA uses the R irlba package36,37, which allows to focus on the first principal component"
- [discussion] Random gene set null distribution for L1 p-values: "uses a random gene set procedure to generate a null distribution for the L1 amount of variance explained by the PC1 and calculates the p-value by comparing the observed L1 to the null distribution."
- [discussion] Three of 50 hallmark pathways significantly active in CF: "out of the 50 hallmark pathways tested, 3 were significantly active: FATTY_ACID_METABOLISM, APICAL_SURFACE, and COAGULATION."
- [readme] README: fixing the seed and running rRoma.R: "You can now run rRoma on our dataset by simply specifying the expression dataset and the modules you want to test. Here we are fixing the seed"
