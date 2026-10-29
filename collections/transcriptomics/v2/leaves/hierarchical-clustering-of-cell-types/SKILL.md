---
name: hierarchical-clustering-of-cell-types
description: Use when deconvolving bulk RNA-seq cell type proportions with MuSiC and
  the scRNA-seq reference contains closely related cell types (e.g., PT vs DCT, CD-PC
  vs CD-IC) where collinearity destabilizes single-stage W-NNLS estimates.
when_to_use_negative:
- Cell types in the reference are clearly distinct with low expression correlation,
  so single-stage music_prop without clustering is sufficient
- Bulk input is TPM-normalized — cell type proportions cannot be estimated with TPM
  as the input; use raw read counts
- The task only needs cluster-level grouping without the recursive two-stage deconvolution
  (plain music_prop on all cell types)
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: MuSiC
  role: Provides music_basis to compute the design matrix used as input to clustering,
    and music_prop.cluster to run the tree-guided recursive two-stage W-NNLS deconvolution
    using the derived cluster groups
  repo: https://github.com/xuranw/MuSiC
- name: R
  role: Environment for running MuSiC's hclust-based clustering (complete linkage
    on log design matrix) and annotating the ExpressionSet with clusterType
provenance:
  source_task_ids:
  - task_003
  - task_004
  - task_006
  source_papers:
  - doi: 10.1038/s41467-018-08023-x
    title: Bulk tissue cell type deconvolution with multi-subject single-cell expression
      reference
  source_content_hash: sha256:caf1104c681337b03d965db88f8fbddfac89745058d4d74f5372f8cf472ad72d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/xuranw/MuSiC
  tools:
  - Biobase
  - MuSiC
  - R
  - ggplot2
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# hierarchical-clustering-of-cell-types

## Summary

Groups closely related cell types (e.g., the 13 mouse kidney cell types from the Park et al. GSE107585 reference) into high-level clusters via complete-linkage hierarchical clustering on the MuSiC design matrix, to feed the tree-guided recursive two-stage deconvolution that mitigates collinearity.

## When to use

Use when deconvolving bulk RNA-seq cell type proportions with MuSiC and the scRNA-seq reference contains closely related cell types (e.g., PT vs DCT, CD-PC vs CD-IC) where collinearity destabilizes single-stage W-NNLS estimates.

## When NOT to use

- Cell types in the reference are clearly distinct with low expression correlation, so single-stage music_prop without clustering is sufficient
- Bulk input is TPM-normalized — cell type proportions cannot be estimated with TPM as the input; use raw read counts
- The task only needs cluster-level grouping without the recursive two-stage deconvolution (plain music_prop on all cell types)

## Inputs

- Single-cell reference ExpressionSet (e.g., Mousesubeset.rds, GSE107585 subset, 16273 genes x 10000 cells) with cellType and sampleID pheno labels
- Bulk RNA-seq ExpressionSet with read counts (e.g., Mousebulkeset.rds from GSE81492, GSE56743 rat tubule counts)
- Intra-cluster differential expression markers (IEmarkers.RData) for the epithelial and immune clusters
- Chosen number of dendrogram cuts (4 groups in the vignette)

## Outputs

- MuSiC design matrix (Disgn.mtx) and cross-subject mean of relative abundance (M.theta) from music_basis
- clusterType factor assigning each of the 13 cell types to one of 4 clusters
- Estimated cluster-level and cell-type-level proportion matrix from music_prop.cluster (e.g., written to music_cluster_proportions.csv)

## How to apply

Compute the design matrix and cross-subject mean of relative abundance with music_basis (clusters='cellType', samples='sampleID') on the single-cell ExpressionSet, excluding novel and transition cell types (13 kidney types remain: Endo, Podo, PT, LOH, DCT, CD-PC, CD-IC, Fib, Macro, Neutro, B lymph, T lymph, NK). Hierarchically cluster cell types with complete linkage on the Euclidean distance of the log design matrix (log + 1e-6), cutting the dendrogram into 4 groups: Group 1 Neutro; Group 2 Podo; Group 3 epithelial (Endo, CD-PC, CD-IC, LOH, DCT, PT); Group 4 immune (Fib, Macro, NK, B lymph, T lymph). Annotate the single-cell data with a clusterType factor and supply cluster-specific markers (IEmarkers for epithelial C3 and immune C4 clusters) to music_prop.cluster, which estimates cluster proportions in stage 1 and within-cluster cell type proportions via W-NNLS constrained by the stage-1 cluster proportions in stage 2. The rationale is that correlation in gene expression across similar cell types causes collinearity, so grouping them first and recursing stabilizes estimates, especially for rare cell types like Podo and Neutro.

## Related tools

- **MuSiC** (Provides music_basis to compute the design matrix used as input to clustering, and music_prop.cluster to run the tree-guided recursive two-stage W-NNLS deconvolution using the derived cluster groups) — https://github.com/xuranw/MuSiC
- **R** (Environment for running MuSiC's hclust-based clustering (complete linkage on log design matrix) and annotating the ExpressionSet with clusterType)

## Examples

```
Dismtx <- music_basis(Mousesub.eset, clusters='cellType', samples='sampleID'); ct.clusters <- cutree(hclust(dist(t(log(Dismtx$Disgn.mtx+1e-6))), method='complete'), k=4)
```

## Evaluation signals

- The clustering dendrogram on log(design matrix + 1e-6) with complete linkage reproduces the 4 vignette groups: Neutro alone, Podo alone, epithelial (Endo, CD-PC, CD-IC, LOH, DCT, PT), and immune (Fib, Macro, NK, B lymph, T lymph)
- The music_prop.cluster output contains all 13 kidney cell types and resolves PT from DCT as separate components despite their proximity on the dendrogram
- Estimated DCT proportions fall in the expected 8-20% range across the UUO, folic acid, and APOL1 mouse CKD bulk datasets
- Stage-1 cluster proportions are consistent with stage-2 within-cluster proportions (within-cluster estimates sum to the pre-estimated cluster proportion)
- For the rat GSE56743 application, the dominant deconvolved cell type agrees with each tubule segment's anatomical position along the nephron

## Limitations

- Correlation in gene expression can lead to collinearity, making it difficult to reliably estimate cell type proportions, especially for less frequent and rare cell types — clustering mitigates but does not eliminate this
- Only trends of cell type proportion change across subjects can be recovered, not absolute cell type proportions (assumptions A1-A3)
- Cell type proportions cannot be estimated with TPM as the input; raw read counts must be used

## Evidence

- [discussion] Stage 1 recursive clustering: "In stage 1 of this procedure, cell types in the design matrix are divided into high-level clusters by hierarchical clustering with closely related cell types clustered together."
- [discussion] Stage 2 constrained W-NNLS: "the cell type proportions are estimated using W-NNLS with genes displaying small intra-cell type variance, subject to the constraint on the pre-estimated cluster proportions."
- [discussion] Collinearity rationale: "Correlation in gene expression can lead to collinearity, making it difficult to reliably estimate cell type proportions, especially for less frequent and rare cell types."
- [methods] 4-group cut in vignette: "Here we cut 13 cell types into 4 groups: - Group 1: Neutro - Group 2: Podo - Group 3: Endo, CD-PC, CD-IC, LOH, DCT, PT - Group 4: Fib, Macro, NK, B lymph, T lymph"
- [methods] music_basis function role: "`music_basis` is the function to produce the design matrix (`Disgn.mtx`), cross-subject mean of relative abundance (`M.theta`)"
