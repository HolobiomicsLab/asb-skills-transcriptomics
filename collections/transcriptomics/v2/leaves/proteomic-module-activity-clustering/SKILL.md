---
name: proteomic-module-activity-clustering
description: Use when you have a normalized proteomic expression matrix (proteins
  x samples) and want to identify significantly shifted or overdispersed pathways
  and stratify samples into clusters based on their module activity profiles.
when_to_use_negative:
- The expression matrix contains missing values that have not been imputed — the README
  recommends using datasets for which missing values have already been imputed rather
  than relying on the mice fill step
- The data are raw (unnormalized) counts — rRoma requires normalized data (e.g. cpm
  from edgeR or vst from DESeq2)
- The goal is differential expression of individual genes rather than coordinated
  gene set (module) activity quantification
edam_operation: http://edamontology.org/operation_3929
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0602
tools:
- name: rROMA
  role: R package that computes module activity (PC1 per gene set), generates random-gene-set
    null distributions for L1 and Median Exp scores, and produces the activity matrix
    used for sample clustering
  repo: https://github.com/sysbio-curie/rROMA
- name: irlba
  role: Computes only the first principal component (first column of V and first singular
    value) for efficiency
- name: mice
  role: Imputes missing values in the global expression matrix via iterative (low-rank)
    PCA when datasets contain missing values
- name: MSigDB
  role: Source of the 50 Hallmark gene sets tested (v6.2, symbols) used as the module
    list
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41540-024-00331-x
    title: Representation and quantification of module activity from omics data with
      rROMA
  source_content_hash: sha256:8ff13c203f8a2c856a234d38ac52e6000dbb81634a0f1790876ccf030945be42
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sysbio-curie/rROMA
  tools:
  - MSigDB
  - R
  - irlba
  - mice
  - rROMA
  edam_operation: http://edamontology.org/operation_3929
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6428571428571429
n_claims: 14
---

# proteomic-module-activity-clustering

## Summary

Quantifies pathway activity in a mass-spectrometry proteomic dataset using rROMA (PC1-based module activity with random gene set null distributions) and then stratifies samples by unsupervised clustering of module activity profiles. Applied to the 9771-protein, 122-sample breast cancer proteomic matrix with the 50 MSigDB Hallmark gene sets (v6.2).

## When to use

Use when you have a normalized proteomic expression matrix (proteins x samples) and want to identify significantly shifted or overdispersed pathways and stratify samples into clusters based on their module activity profiles.

## When NOT to use

- The expression matrix contains missing values that have not been imputed — the README recommends using datasets for which missing values have already been imputed rather than relying on the mice fill step
- The data are raw (unnormalized) counts — rRoma requires normalized data (e.g. cpm from edgeR or vst from DESeq2)
- The goal is differential expression of individual genes rather than coordinated gene set (module) activity quantification

## Inputs

- Normalized proteomic expression matrix (proteins as rows, samples as columns), e.g. 9771 proteins x 122 treatment-naive primary breast cancer samples
- Gene set module list as GMT or via SelectFromMSIGdb, e.g. MSigDB Hallmark gene sets (v6.2, symbols)
- Sample clinical subtype labels (e.g. PAM50) for cluster-to-subtype comparison

## Outputs

- rRoma.output$ModuleMatrix with ppv L1 (overdispersion) and ppv Median Exp (shift) p and q values per module
- Counts of significant pathways (34 shifted and 6 overdispersed at p<0.05 in the reference analysis)
- Sample module activity scores and a six-cluster unsupervised hierarchical clustering partition aligned to clinical subtypes, visualized as a heatmap

## How to apply

Load the expression matrix (e.g. 9771 proteins x 122 treatment-naive primary breast cancer samples) with sample labels, and retrieve the 50 MSigDB Hallmark gene sets (v6.2, symbols) via SelectFromMSIGdb("HALLMARK"). Run rRoma.R with centering of the global matrix, missing-value imputation via the mice low-rank PCA procedure, PC1 computation per module with irlba, and null distributions generated from random gene sets of matched size to obtain L1 (overdispersion) and Median Exp (shift) p and q values. From rRoma.output$ModuleMatrix, classify modules as shifted (ppv Median Exp < 0.05) or overdispersed (ppv L1 < 0.05); for the breast cancer data this yields 34 shifted and 6 overdispersed pathways at p<0.05. Then perform unsupervised hierarchical clustering of the sample module activity scores across the significant pathways and cut into six clusters, mapping clusters to clinical subtypes (e.g. clusters 1/5 to basal/Luminal B and clusters 4/6 to Luminal A against PAM50 labels) with a heatmap visualization.

## Related tools

- **rROMA** (R package that computes module activity (PC1 per gene set), generates random-gene-set null distributions for L1 and Median Exp scores, and produces the activity matrix used for sample clustering) — https://github.com/sysbio-curie/rROMA
- **irlba** (Computes only the first principal component (first column of V and first singular value) for efficiency)
- **mice** (Imputes missing values in the global expression matrix via iterative (low-rank) PCA when datasets contain missing values)
- **MSigDB** (Source of the 50 Hallmark gene sets tested (v6.2, symbols) used as the module list)

## Examples

```
AllHall <- SelectFromMSIGdb("HALLMARK"); rRoma.output <- rRoma.R(MatData, AllHall)
```

## Evaluation signals

- rRoma.output$ModuleMatrix contains 34 pathways with ppv Median Exp < 0.05 (shifted) and 6 pathways with ppv L1 < 0.05 (overdispersed) at p<0.05, matching the reported breast cancer result
- All 50 Hallmark gene sets are processed without errors and activity scores are produced for all 122 samples
- Hierarchical clustering of activity scores cuts cleanly into six clusters
- Cluster-to-subtype mapping reproduces the reported correspondence: clusters 1/5 align to basal/Luminal B and clusters 4/6 to Luminal A against PAM50 labels
- Fixing the random seed before rRoma.R yields reproducible significance classifications

## Limitations

- rRoma relies on the mice package for missing-value imputation, but the README recommends using datasets for which missing values have already been imputed
- Default values of hyperparameters (e.g. extreme-weight percentage, outlier z-score threshold, distance threshold, ApproxSample) are not reported and must be tuned or accepted as defaults
- Random permutations affect p-value estimation; fixing the seed (as in the vignette) is needed for reproducibility
- Results depend on the gene set reference database; the README recommends using rRoma with multiple databases combined for a more complete biological view

## Evidence

- [discussion] rROMA was applied to a proteomic dataset consisting of 9771 proteins from 122 treatment-naive primary breast cancer samples.: "rROMA was applied to a proteomic dataset consisting of 9771 proteins from 122 treatment-naive primary breast cancer samples."
- [discussion] imputation method: "rROMA utilizes the iterative PCA method implemented in the mice package40 on the global expression matrix, imputing any missing values within the dataset."
- [discussion] null distribution computation: "uses a random gene set procedure to generate a null distribution for the L1 amount of variance explained by the PC1 and calculates the p-value by comparing the observed L1 to the null distribution."
- [methods] module selection: "Here, we extract all the "HALLMARK" genesets from MSigDB (Liberzon A et al, Cell Syst 2015)"
- [readme] running rROMA: "You can now run rRoma on our dataset by simply specifying the expression dataset and the modules you want to test. Here we are fixing the seed"
- [readme] normalized data requirement: "Warning: rRoma works with **normalized** data. Here cpm from edgeR was used for normalisation"
