---
name: intra-cluster-gene-selection
description: Use when deconvolving bulk RNA-seq with a multi-subject scRNA-seq reference
  in which closely related cell types cause collinearity (e.g., 13 mouse kidney cell
  types), so proportions must be resolved hierarchically rather than in a single NNLS
  fit.
when_to_use_negative:
- When cell types are well separated and not collinear, a single-stage music_prop
  W-NNLS fit without clustering is sufficient
- When the input is TPM-normalized expression — cell type proportions cannot be estimated
  with TPM as the input
- When no intra-cluster differential expression markers or scRNA-seq reference with
  replicate subjects are available for the tissue
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3170
tools:
- name: MuSiC
  role: R package providing music_basis for the design matrix and music_prop.cluster
    for the two-stage recursive cluster-constrained W-NNLS deconvolution
  repo: https://github.com/xuranw/MuSiC
- name: R
  role: Execution environment; MuSiC is installed as a GitHub R package (devtools::install_github('xuranw/MuSiC'))
provenance:
  source_task_ids:
  - task_006
  - task_003
  source_papers:
  - doi: 10.1038/s41467-018-08023-x
    title: Bulk tissue cell type deconvolution with multi-subject single-cell expression
      reference
  source_content_hash: sha256:7ea297b7001eeedc7733708115a66437eb209c91a2efe430d02eb57b8c464575
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/xuranw/MuSiC
  tools:
  - MuSiC
  - R
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# intra-cluster-gene-selection

## Summary

MuSiC's tree-guided recursive two-stage deconvolution resolves collinear, closely related cell types (e.g., PT vs DCT in the mouse kidney) by first clustering cell types and estimating cluster proportions with cluster-consistent genes / group markers, then estimating within-cluster cell type proportions via W-NNLS constrained by the stage-1 cluster proportions.

## When to use

Use when deconvolving bulk RNA-seq with a multi-subject scRNA-seq reference in which closely related cell types cause collinearity (e.g., 13 mouse kidney cell types), so proportions must be resolved hierarchically rather than in a single NNLS fit.

## When NOT to use

- When cell types are well separated and not collinear, a single-stage music_prop W-NNLS fit without clustering is sufficient
- When the input is TPM-normalized expression — cell type proportions cannot be estimated with TPM as the input
- When no intra-cluster differential expression markers or scRNA-seq reference with replicate subjects are available for the tissue

## Inputs

- Bulk RNA-seq read-count ExpressionSet (e.g., Mousebulkeset.rds from GSE81492)
- Multi-subject single-cell reference ExpressionSet (e.g., Mousesubeset.rds, 16273 genes x 10000 cells from GSE107585)
- Hierarchical clustering of cell types cut into 4 groups (clusters.type list, C1-C4)
- Intra-cluster differential expression markers (IEmarkers.RData) for the epithelial and immune clusters

## Outputs

- Matrix of estimated cell type proportions for all 13 kidney cell types (resolving PT from DCT)
- Estimated cluster-level proportions from stage 1
- Proportion table (e.g., music_cluster_proportions.csv / mouse_kidney_deconvolution.csv)

## How to apply

First run music_basis on the single-cell ExpressionSet (e.g., Mousesubeset.rds, GSE107585 subset of 16273 genes x 10000 cells, clusters='cellType', samples='sampleID') to get the design matrix and cross-subject mean of relative abundance. Hierarchically cluster the 13 kidney cell types with complete linkage on Euclidean distance of log(design matrix + 1e-6) and cut into 4 groups (Group 1: Neutro; Group 2: Podo; Group 3: Endo, CD-PC, CD-IC, LOH, DCT, PT; Group 4: Fib, Macro, NK, B lymph, T lymph), annotating the sc eset with a clusterType factor. Load intra-cluster differential expression markers (IEmarkers.RData) for the epithelial and immune clusters and pass them as group.markers. In stage 1 of music_prop.cluster, cluster proportions are estimated with cluster-consistent genes; in stage 2, within-cluster cell type proportions are estimated by W-NNLS using genes with small intra-cell type variance, subject to the constraint on the pre-estimated cluster proportions.

## Related tools

- **MuSiC** (R package providing music_basis for the design matrix and music_prop.cluster for the two-stage recursive cluster-constrained W-NNLS deconvolution) — https://github.com/xuranw/MuSiC
- **R** (Execution environment; MuSiC is installed as a GitHub R package (devtools::install_github('xuranw/MuSiC')))

## Examples

```
est.prop <- music_prop.cluster(bulk.eset = Mouse.bulk.eset, sc.eset = Mousesub.eset, group.markers = IEmarkers, clusters = 'cellType', groups = 'clusterType', samples = 'sampleID', clusters.type = clusters.type)
```

## Evaluation signals

- The estimated proportions matrix contains all 13 kidney cell types and resolves the closely related PT and DCT types as separate components
- DCT proportions fall in the expected 8-20% range across the UUO, folic acid, and APOL1 mouse CKD bulk datasets
- PT proportions decrease with CKD progression while the immune signature (Macro, Fib, B lymph, T lymph) increases with disease
- Grouping of the 4 clusters (Neutro; Podo; epithelial; immune) is consistent with the complete-linkage dendrogram cut of the design matrix
- Results are saved as a proportion table (e.g., music_cluster_proportions.csv) with one column per cell type

## Limitations

- Correlation in gene expression among closely related cell types leads to collinearity, making proportions hard to estimate reliably, especially for rare cell types; methods like CIBERSORT and BSEQ-sc have limited power to resolve similar cell types
- MuSiC requires count or count-like input; proportions cannot be estimated with TPM, and RPKM is usable only if cell-type-specific total RNA abundance estimates are supplied
- MuSiC outperforms existing methods especially for tissues with closely related cell types, but for multi-condition bulk data diverging from the reference, MuSiC2 is the recommended iterative variant

## Evidence

- [discussion] Stage 1 of recursive tree-guided deconvolution: "In stage 1 of this procedure, cell types in the design matrix are divided into high-level clusters by hierarchical clustering with closely related cell types clustered together."
- [discussion] Stage 2 W-NNLS within clusters: "the cell type proportions are estimated using W-NNLS with genes displaying small intra-cell type variance, subject to the constraint on the pre-estimated cluster proportions."
- [methods] music_prop.cluster function: "Function `music_prop.cluster` is used for estimation with cluster."
- [methods] 4-group cut of 13 kidney cell types: "Here we cut 13 cell types into 4 groups: - Group 1: Neutro - Group 2: Podo - Group 3: Endo, CD-PC, CD-IC, LOH, DCT, PT"
- [abstract] MuSiC advantage for closely related cell types: "When applied to pancreatic islet and whole kidney expression data in human, mouse, and rats, MuSiC outperformed existing methods, especially for tissues with closely related cell types."
