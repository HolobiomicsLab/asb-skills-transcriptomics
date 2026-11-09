---
name: cluster-subtype-correspondence-mapping
description: Use when samples have both an unsupervised grouping (e.g., hierarchical
  clustering over rROMA pathway activity scores) and independent clinical/molecular
  subtype labels (e.g., PAM50), and you need to establish or validate the cluster-to-subtype
  correspondence.
when_to_use_negative:
- No independent subtype or clinical labels are available — the mapping step cannot
  be performed and only the unsupervised clustering is meaningful
- The input is raw/unnormalized count data — rRoma requires normalized matrices (e.g.,
  cpm from edgeR or vst from DESeq2) before any clustering
- Only one or two significant modules are detected, making activity-profile-based
  clustering unstable or uninformative
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: rROMA
  role: Computes sample-wise module activity scores (PC1 per Hallmark gene set) that
    form the feature matrix for the hierarchical clustering
  repo: https://github.com/sysbio-curie/rROMA
- name: R
  role: Environment in which rRoma is run and hierarchical clustering/heatmap annotation
    is performed
- name: irlba
  role: Efficiently computes only the first principal component (PC1) per module for
    the activity scores
- name: mice
  role: Imputes missing values in the global expression matrix via iterative PCA before
    activity computation
- name: MSigDB
  role: Source of the Hallmark gene sets (v6.2 symbols) tested to generate pathway
    activity profiles
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41540-024-00331-x
    title: Representation and quantification of module activity from omics data with
      rROMA
  source_content_hash: sha256:5e85316bbd5dccbb117833d9e640b4d59ff2a6b5a24e8e7970bf8a48338f1182
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
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# cluster-subtype-correspondence-mapping

## Summary

Maps an unsupervised hierarchical clustering of samples, built on rROMA module activity scores, onto clinical molecular subtype labels (e.g., PAM50 breast cancer subtypes) to check whether pathway-activity-based groupings recapitulate known biology, as demonstrated on a 9771-protein × 122-sample breast cancer proteomic dataset.

## When to use

Use when samples have both an unsupervised grouping (e.g., hierarchical clustering over rROMA pathway activity scores) and independent clinical/molecular subtype labels (e.g., PAM50), and you need to establish or validate the cluster-to-subtype correspondence.

## When NOT to use

- No independent subtype or clinical labels are available — the mapping step cannot be performed and only the unsupervised clustering is meaningful
- The input is raw/unnormalized count data — rRoma requires normalized matrices (e.g., cpm from edgeR or vst from DESeq2) before any clustering
- Only one or two significant modules are detected, making activity-profile-based clustering unstable or uninformative

## Inputs

- rRoma output object with sample-level module activity scores (rRoma.output$ModuleMatrix), restricted to significant modules (e.g., ppv Median Exp < 0.05 shifted, ppv L1 < 0.05 overdispersed)
- Normalized gene/protein expression matrix (genes as rows, samples as columns)
- Clinical/molecular subtype labels per sample (e.g., PAM50 labels for the 122 breast cancer samples)

## Outputs

- Six-cluster partition of the 122 samples based on hierarchical clustering of pathway activity profiles
- Cluster-to-subtype correspondence table (cluster vs PAM50 label counts)
- Heatmap visualization of samples annotated by cluster and subtype

## How to apply

First run rRoma.R on the normalized expression matrix (here the mass-spectrometry breast cancer matrix with 9771 proteins and 122 treatment-naive primary samples) against the 50 MSigDB Hallmark gene sets (v6.2, symbols), centering the global matrix, imputing missing values with the mice low-rank PCA procedure, and computing PC1 per module with irlba to obtain shifted and overdispersed modules (34 shifted and 6 overdispersed at p < 0.05 in the reported run). Extract the sample-wise module activity scores from rRoma.output$ModuleMatrix for the significant modules only. Perform unsupervised hierarchical clustering of samples over these activity profiles and cut the dendrogram into the target number of clusters (six in the reported stratification). Finally, cross-tabulate clusters against PAM50 labels and report the correspondence (clusters 1/5 aligning to basal/Luminal B and clusters 4/6 to Luminal A in the reported mapping), visualizing agreement as a heatmap annotation of the cluster-dendrogram.

## Related tools

- **rROMA** (Computes sample-wise module activity scores (PC1 per Hallmark gene set) that form the feature matrix for the hierarchical clustering) — https://github.com/sysbio-curie/rROMA
- **R** (Environment in which rRoma is run and hierarchical clustering/heatmap annotation is performed)
- **irlba** (Efficiently computes only the first principal component (PC1) per module for the activity scores)
- **mice** (Imputes missing values in the global expression matrix via iterative PCA before activity computation)
- **MSigDB** (Source of the Hallmark gene sets (v6.2 symbols) tested to generate pathway activity profiles)

## Examples

```
rRoma.output <- rRoma.R(MatData, AllHall)
```

## Evaluation signals

- The number of significantly active pathways matches the reported result: 34 shifted and 6 overdispersed Hallmark pathways at p < 0.05
- The hierarchical clustering cut into six clusters reproduces the reported correspondence: clusters 1/5 mapping to basal/Luminal B and clusters 4/6 to Luminal A against PAM50 labels
- Every sample in the expression matrix appears in exactly one cluster and carries a PAM50 label for the cross-tabulation
- The activity-score matrix used for clustering has one row/column per significant module and one entry per sample (122 samples), with no missing values after mice imputation
- The heatmap visualization correctly annotates each sample's dendrogram leaf with both its cluster and its PAM50 subtype

## Limitations

- Unsupervised clustering of activity profiles depends on the number of significant modules; if few pathways pass p < 0.05 the stratification may not align with subtypes
- Cluster numbering is arbitrary and must be mapped to subtype labels post hoc; the correspondence (e.g., 1/5 → basal/Luminal B) is dataset-specific and not portable
- rRoma recommends using datasets whose missing values have already been imputed; reliance on the mice low-rank PCA imputation adds variability
- The authors note that statistical power can be limiting for some case studies, potentially requiring additional datasets and multiple gene set reference databases

## Evidence

- [intro] breast cancer dataset applied to clinical subtypes: "we show application of rROMA to a mass-spectrometry-based breast cancer proteomic dataset, showing its capability of identifying significantly dysregulated pathways in clinical breast cancer subtypes"
- [discussion] breast cancer proteomic dataset: "rROMA was applied to a proteomic dataset consisting of 9771 proteins from 122 treatment-naive primary breast cancer samples."
- [readme] input requirements for expression matrix: "rRoma requires a gene expression matrix, with column names indicating samples and row names indicating gene names."
- [discussion] activity computation via PC1: "rROMA uses the R irlba package36,37, which allows to focus on the first principal component, by computing only the first column of V and the first value of S."
- [discussion] missing value imputation: [evidence span withheld — anchor status: not_found]
