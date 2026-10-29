---
name: two-stage-recursive-cell-type-estimation
description: Use when deconvolving bulk RNA-seq from tissues containing closely related
  cell types whose expression collinearity breaks single-stage NNLS deconvolution,
  and a multi-subject scRNA-seq reference is available.
when_to_use_negative:
- Single-cell proportions themselves are the target — scRNA-seq data do not reflect
  true cell type proportions in intact tissues due to biased cell dissociation
- Input bulk data are TPM-normalized — cell type proportions cannot be estimated with
  TPM as the input; use read counts (RPKM only with cell type-specific total RNA abundance
  estimates)
- Bulk samples come from multiple clinical conditions mismatched to the reference
  — use MuSiC2's iterative algorithm instead of the two-stage recursive procedure
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3518
tools:
- name: MuSiC
  role: R package providing music_basis for the design matrix, hierarchical clustering
    guidance, and music_prop.cluster for the two-stage recursive deconvolution
  repo: https://github.com/xuranw/MuSiC
- name: R
  role: Runtime environment; MuSiC is installed via devtools::install_github('xuranw/MuSiC')
    and used as an R/Bioconductor-style ExpressionSet workflow
  repo: https://github.com/xuranw/MuSiC
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-018-08023-x
    title: Bulk tissue cell type deconvolution with multi-subject single-cell expression
      reference
  source_content_hash: sha256:66162bda8a25ed53bf0b02df955c3186f843a5f1098b1f5c4335c2242a438150
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/xuranw/MuSiC
  tools:
  - MuSiC
  - R
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3518
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# two-stage-recursive-cell-type-estimation

## Summary

MuSiC's tree-guided recursive procedure for estimating proportions of closely related cell types: hierarchical clustering groups similar cell types, cluster proportions are estimated in stage 1, and within-cluster cell type proportions are recovered in stage 2 via constrained W-NNLS.

## When to use

Use when deconvolving bulk RNA-seq from tissues containing closely related cell types whose expression collinearity breaks single-stage NNLS deconvolution, and a multi-subject scRNA-seq reference is available.

## When NOT to use

- Single-cell proportions themselves are the target — scRNA-seq data do not reflect true cell type proportions in intact tissues due to biased cell dissociation
- Input bulk data are TPM-normalized — cell type proportions cannot be estimated with TPM as the input; use read counts (RPKM only with cell type-specific total RNA abundance estimates)
- Bulk samples come from multiple clinical conditions mismatched to the reference — use MuSiC2's iterative algorithm instead of the two-stage recursive procedure

## Inputs

- Bulk RNA-seq read counts as an ExpressionSet (e.g., Mousebulkeset.rds, GSE81492)
- Multi-subject single-cell reference ExpressionSet (e.g., Mousesubeset.rds, GSE107585 subset, 16273 genes x 10000 cells) with cellType and sampleID phenotype metadata
- Hierarchical clustering cut of cell types into groups (e.g., 4 groups over 13 mouse kidney cell types) annotated as a clusterType factor
- Intra-cluster differential expression markers (IEmarkers.RData) for clusters containing multiple related cell types

## Outputs

- Estimated cell type proportion matrix covering all 13 kidney cell types (music_cluster_proportions.csv)
- Estimated cluster-level proportions from stage 1
- Hierarchical clustering dendrogram of cell types used to define the groups

## How to apply

Load the bulk ExpressionSet (e.g., Mousebulkeset.rds from GSE81492) and single-cell reference (e.g., Mousesubeset.rds, a GSE107585 subset of 16273 genes x 10000 cells) into R. Compute the design matrix and cross-subject mean of relative abundance with music_basis (clusters='cellType', samples='sampleID'), then hierarchically cluster the 13 cell types using complete-linkage Euclidean distance on log(design matrix + 1e-6) and cut into 4 groups (Neutro; Podo; Endo/CD-PC/CD-IC/LOH/DCT/PT; Fib/Macro/NK/B lymph/T lymph). Annotate the single-cell data with these cluster labels (clusterType factor) and load intra-cluster differential expression markers (IEmarkers.RData) for the epithelial and immune clusters. Run music_prop.cluster with bulk.eset, sc.eset, group.markers, clusters, groups, samples, and the clusters.type list: stage 1 estimates cluster proportions with cluster-consistent genes; stage 2 estimates within-cluster cell type proportions via W-NNLS constrained by the stage-1 cluster proportions. Collinearity among correlated cell types otherwise makes proportions unreliable, especially for rare types — the tree-guided staging resolves this.

## Related tools

- **MuSiC** (R package providing music_basis for the design matrix, hierarchical clustering guidance, and music_prop.cluster for the two-stage recursive deconvolution) — https://github.com/xuranw/MuSiC
- **R** (Runtime environment; MuSiC is installed via devtools::install_github('xuranw/MuSiC') and used as an R/Bioconductor-style ExpressionSet workflow) — https://github.com/xuranw/MuSiC

## Examples

```
Est.prop.list = music_prop.cluster(bulk.eset = Mouse.bulk.eset, sc.eset = Mousesub.eset, group.markers = IEmarkers, clusters = 'cellType', groups = 'clusterType', samples = 'sampleID', clusters.type = clusters.type)
```

## Evaluation signals

- The estimated proportions matrix contains all 13 kidney cell types as components
- Closely related PT and DCT cell types, adjacent on the clustering dendrogram, receive distinct proportion estimates
- Stage-1 cluster proportions sum consistently with the stage-2 within-cluster estimates constrained by them
- Dendrogram grouping matches the paper's cut: Group 1 Neutro; Group 2 Podo; Group 3 Endo, CD-PC, CD-IC, LOH, DCT, PT; Group 4 Fib, Macro, NK, B lymph, T lymph
- Results are qualitatively consistent with MuSiC outperforming existing methods for tissues with closely related cell types

## Limitations

- Collinearity from correlated gene expression makes cell type proportions unreliable, especially for less frequent and rare cell types, which motivates but also constrains the method
- CIBERSORT and BSEQ-sc style marker-gene selection still has limited power to resolve similar cell types; MuSiC's improvement applies to its W-NNLS framework
- Most scRNA-seq references do not reflect true tissue proportions, so the reference informs cell type-specific expression rather than composition

## Evidence

- [discussion] In stage 1 of this procedure, cell types in the design matrix are divided into high-level clusters by hierarchical clustering with closely related cell types clustered together.: "In stage 1 of this procedure, cell types in the design matrix are divided into high-level clusters by hierarchical clustering with closely related cell types clustered together."
- [discussion] the cell type proportions are estimated using W-NNLS with genes displaying small intra-cell type variance, subject to the constraint on the pre-estimated cluster proportions.: "the cell type proportions are estimated using W-NNLS with genes displaying small intra-cell type variance, subject to the constraint on the pre-estimated cluster proportions."
- [discussion] Correlation in gene expression can lead to collinearity, making it difficult to reliably estimate cell type proportions, especially for less frequent and rare cell types.: "Correlation in gene expression can lead to collinearity, making it difficult to reliably estimate cell type proportions, especially for less frequent and rare cell types."
- [methods] `music_basis` is the function to produce the design matrix (`Disgn.mtx`), cross-subject mean of relative abundance (`M.theta`): "`music_basis` is the function to produce the design matrix (`Disgn.mtx`), cross-subject mean of relative abundance (`M.theta`)"
- [abstract] When applied to pancreatic islet and whole kidney expression data in human, mouse, and rats, MuSiC outperformed existing methods, especially for tissues with closely related cell types.: "When applied to pancreatic islet and whole kidney expression data in human, mouse, and rats, MuSiC outperformed existing methods, especially for tissues with closely related cell types."
