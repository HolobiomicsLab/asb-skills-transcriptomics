---
name: cross-subject-gene-weighting
description: Use when deconvolving bulk RNA-seq cell type proportions with a multi-subject
  scRNA-seq reference, and gene weights must reflect cross-subject consistency rather
  than pre-selected marker genes.
when_to_use_negative:
- Input bulk data are TPM-normalized — cell type proportions cannot be estimated with
  TPM as the input
- Only pre-selected marker genes are available or the reference lacks multiple subjects
  per cell type, so cross-subject variance cannot be estimated
- Bulk samples come from clinical conditions very different from the scRNA-seq reference
  — use MuSiC2 instead
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3518
tools:
- name: MuSiC
  role: R package providing music_basis for cross-subject statistics and music_prop
    / music_prop.cluster for W-NNLS deconvolution with gene weighting
  repo: https://github.com/xuranw/MuSiC
- name: R
  role: Execution environment; MuSiC is installed from GitHub via devtools::install_github('xuranw/MuSiC')
- name: Biobase
  role: Provides the ExpressionSet container for bulk and single-cell reference data
    loaded via readRDS
provenance:
  source_task_ids:
  - task_005
  - task_003
  source_papers:
  - doi: 10.1038/s41467-018-08023-x
    title: Bulk tissue cell type deconvolution with multi-subject single-cell expression
      reference
  source_content_hash: sha256:a1c92bb069ea949aa0a576076efc65790a20c26faf9570a3aeb9af77e56b8ffa
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/xuranw/MuSiC
  tools:
  - Biobase
  - MuSiC
  - R
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3518
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# cross-subject-gene-weighting

## Summary

MuSiC's weighting scheme for weighted non-negative least squares (W-NNLS) deconvolution, which uses cross-subject variance of cell-type-specific relative gene abundance from a multi-subject scRNA-seq reference to up-weight genes consistent across subjects and down-weight non-informative, highly variable genes.

## When to use

Use when deconvolving bulk RNA-seq cell type proportions with a multi-subject scRNA-seq reference, and gene weights must reflect cross-subject consistency rather than pre-selected marker genes.

## When NOT to use

- Input bulk data are TPM-normalized — cell type proportions cannot be estimated with TPM as the input
- Only pre-selected marker genes are available or the reference lacks multiple subjects per cell type, so cross-subject variance cannot be estimated
- Bulk samples come from clinical conditions very different from the scRNA-seq reference — use MuSiC2 instead

## Inputs

- Bulk RNA-seq read-count ExpressionSet (e.g., GSE50244bulkeset.rds, Mousebulkeset.rds)
- Multi-subject single-cell reference ExpressionSet with cell type and subject annotations (e.g., EMTABesethealthy.rds, Mousesubeset.rds)
- Cell type cluster and sample ID column names (clusters, samples) and selected cell types (select.ct)
- Optionally, group markers and clusters.type definitions for music_prop.cluster

## Outputs

- Per-sample estimated cell type proportions (Est.prop.weighted) from music_prop
- Design matrix (Disgn.mtx), cross-subject mean (M.theta) and variance (Sigma) of relative abundance from music_basis
- Normalized per-sample proportion matrix (e.g., music_prop_GSE50244.csv)
- Cluster proportions (stage 1) and within-cluster cell type proportions (stage 2) for music_prop.cluster

## How to apply

First compute the cross-subject statistics from the single-cell reference using music_basis (e.g., clusters = 'cellType', samples = 'sampleID', select.ct = c('alpha','beta','delta','gamma','acinar','ductal')), which returns the design matrix (Disgn.mtx), the cross-subject mean of relative abundance (M.theta), and the cross-subject variance (Sigma). Genes with low cross-subject variance of relative abundance are treated as informative and up-weighted; genes with high cross-subject variance are down-weighted, enabling transfer of cell-type-specific expression information from one dataset to another. Then run music_prop on the bulk ExpressionSet against the single-cell reference: weights are initialized by plain NNLS and the W-NNLS fit is refined iteratively, without pre-selected marker genes. For closely related cell types where collinearity is a problem, use music_prop.cluster with hierarchical clustering of the design matrix into high-level groups and stage-1 cluster proportions constrained in stage-2 W-NNLS. Input bulk data must be read counts (TPM cannot be used; RPKM only if cell-type-specific total RNA abundance is provided). Extract 'Est.prop.weighted' as the MuSiC estimate and normalize proportions over the included cell types.

## Related tools

- **MuSiC** (R package providing music_basis for cross-subject statistics and music_prop / music_prop.cluster for W-NNLS deconvolution with gene weighting) — https://github.com/xuranw/MuSiC
- **R** (Execution environment; MuSiC is installed from GitHub via devtools::install_github('xuranw/MuSiC'))
- **Biobase** (Provides the ExpressionSet container for bulk and single-cell reference data loaded via readRDS)

## Examples

```
library(MuSiC); Est.prop <- music_prop(bulk.eset = GSE50244bulkeset, sc.eset = EMTABesethealthy, clusters = 'cellType', samples = 'sampleID', select.ct = c('alpha','beta','delta','gamma','acinar','ductal'))
```

## Evaluation signals

- Output proportion table has the expected dimensions (e.g., 89 rows for GSE50244 bulk samples and 6 cell-type columns)
- All estimated proportions are non-negative and each sample's proportions sum to 1 within numerical tolerance after normalization
- music_basis returns Disgn.mtx, M.theta, and Sigma with genes × cell types consistent with the selected select.ct set
- Estimated proportions are plausible biologically (e.g., DCT proportions of 8–20% and decreased PT proportions in mouse CKD datasets)
- W-NNLS estimates (Est.prop.weighted) outperform plain NNLS estimates when benchmarked with Eval_multi against known real proportions from bulk_construct

## Limitations

- Cell type proportions cannot be estimated with TPM as the input; RPKM can only be used if cell-type-specific total RNA abundance estimates are provided
- Under assumptions A1–A3 only trends of cell type proportion change across subjects can be recovered, not absolute cell type proportions
- Collinearity among closely related cell types makes estimation unreliable, especially for rare cell types; the recursive cluster (music_prop.cluster) procedure is needed
- Existing approaches including CIBERSORT and BSEQ-sc have limited power to resolve similar cell types; scRNA-seq references do not reflect true in-tissue proportions

## Evidence

- [abstract] Weighting by cross-subject variance: "By appropriate weighting of genes showing cross-subject and cross-cell consistency, MuSiC enables the transfer of cell type-specific gene expression information from one dataset to another."
- [discussion] W-NNLS without marker genes: "MuSiC is a weighted non-negative least squares regression (W-NNLS), which does not require pre-selected marker genes."
- [discussion] Iterative weight estimation: "we will estimate the weight wjg iteratively, initialized by NNLS"
- [methods] music_basis outputs: "`music_basis` is the function to produce the design matrix (`Disgn.mtx`), cross-subject mean of relative abundance (`M.theta`)"
- [methods] Cluster-based deconvolution: "Function `music_prop.cluster` is used for estimation with cluster."
- [discussion] TPM restriction: "However, cell type proportions cannot be estimated with TPM as the input."
