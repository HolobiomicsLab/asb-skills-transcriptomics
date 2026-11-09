---
name: weighted-nonnegative-least-squares-deconvolution
description: Use when estimating cell type proportions in bulk RNA-seq from a multi-subject
  scRNA-seq reference, especially when closely related cell types cause collinearity
  and methods like CIBERSORT or BSEQ-sc lack power to resolve them.
when_to_use_negative:
- Input bulk data is TPM — cell type proportions cannot be estimated with TPM as the
  input; use read counts (RPKM only if cell-type-specific total RNA abundance estimates
  are available)
- The bulk samples span multiple clinical conditions where at least one condition
  differs from the scRNA-seq reference — use MuSiC2, the iterative variant, instead
- No single-cell reference with cell type labels and replicate subjects is available,
  since MuSiC requires cross-subject scRNA-seq to weight genes
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: MuSiC
  role: R package implementing W-NNLS deconvolution via music_basis, music_prop, and
    the recursive music_prop.cluster
  repo: https://github.com/xuranw/MuSiC
- name: R
  role: Runtime environment (>= 3.0) in which the MuSiC GitHub package is installed
    and the deconvolution is executed
- name: CIBERSORT
  role: Benchmark comparator; resolves collinearity via support vector regression
    marker selection but has limited power for similar cell types
- name: BSEQ-sc
  role: Benchmark comparator extending CIBERSORT to scRNA-seq references; limited
    power to resolve similar cell types
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-018-08023-x
    title: Bulk tissue cell type deconvolution with multi-subject single-cell expression
      reference
  source_content_hash: sha256:84583f5d57ed7a3abee77f533b9def6403b3943f36cf9d558f5e349c98290183
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/xuranw/MuSiC
  tools:
  - BSEQ-sc
  - CIBERSORT
  - MuSiC
  - R
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# weighted-nonnegative-least-squares-deconvolution

## Summary

MuSiC's W-NNLS deconvolution estimates cell type proportions in bulk RNA-seq using multi-subject scRNA-seq references, weighting genes by cross-subject and cross-cell consistency instead of pre-selected marker genes, with a tree-guided recursive two-stage variant for collinear (closely related) cell types.

## When to use

Use when estimating cell type proportions in bulk RNA-seq from a multi-subject scRNA-seq reference, especially when closely related cell types cause collinearity and methods like CIBERSORT or BSEQ-sc lack power to resolve them.

## When NOT to use

- Input bulk data is TPM — cell type proportions cannot be estimated with TPM as the input; use read counts (RPKM only if cell-type-specific total RNA abundance estimates are available)
- The bulk samples span multiple clinical conditions where at least one condition differs from the scRNA-seq reference — use MuSiC2, the iterative variant, instead
- No single-cell reference with cell type labels and replicate subjects is available, since MuSiC requires cross-subject scRNA-seq to weight genes

## Inputs

- Bulk RNA-seq read counts as an ExpressionSet (e.g., Mousebulkeset.rds, GSE81492)
- Single-cell RNA-seq reference as an ExpressionSet or SingleCellExperiment (e.g., Mousesub.eset.rds, 16273 genes x 10000 cells from GSE107585)
- Cell type and sample ID phenotype labels ('cellType', 'sampleID')
- Hierarchical clustering grouping of cell types (clusterType factor) and intra-cluster differential expression markers (IEmarkers.RData) for the recursive variant

## Outputs

- Matrix/data frame of estimated cell type proportions for each bulk sample (Est.prop.weighted / Est.prop.all_gene)
- Estimated cluster proportions and cluster-wise weighted mean relative abundance (M.theta) from music_basis
- CSV table of deconvolved proportions (e.g., music_cluster_proportions.csv)

## How to apply

Load the bulk and single-cell ExpressionSets (e.g., Mousebulkeset.rds from GSE81492 and Mousesub.eset from GSE107585, 16273 genes x 10000 cells) into R and build the design matrix and cross-subject mean of relative abundance with music_basis (clusters='cellType', samples='sampleID'). MuSiC performs weighted non-negative least squares (W-NNLS) without pre-selected marker genes; gene weights are estimated iteratively, initialized by NNLS, and a gene- and subject-specific intercept adjusts for protocol bias between bulk and single-cell data. For closely related cell types (e.g., 13 mouse kidney cell types), first hierarchical-cluster the cell types (complete linkage, Euclidean distance on log(design matrix + 1e-6)) and cut into groups (e.g., 4 groups: Neutro; Podo; Endo/CD-PC/CD-IC/LOH/DCT/PT; Fib/Macro/NK/B lymph/T lymph), annotate clusters as clusterType, load intra-cluster markers (IEmarkers.RData), then run the two-stage recursive procedure with music_prop.cluster: stage 1 estimates cluster proportions with cluster-consistent genes, stage 2 estimates within-cluster cell type proportions via W-NNLS constrained by the pre-estimated cluster proportions. Input read counts (TPM cannot be used; RPKM only if cell-type-specific total RNA abundance estimates are provided). Save the estimated proportions table (e.g., music_cluster_proportions.csv).

## Related tools

- **MuSiC** (R package implementing W-NNLS deconvolution via music_basis, music_prop, and the recursive music_prop.cluster) — https://github.com/xuranw/MuSiC
- **R** (Runtime environment (>= 3.0) in which the MuSiC GitHub package is installed and the deconvolution is executed)
- **CIBERSORT** (Benchmark comparator; resolves collinearity via support vector regression marker selection but has limited power for similar cell types)
- **BSEQ-sc** (Benchmark comparator extending CIBERSORT to scRNA-seq references; limited power to resolve similar cell types)

## Examples

```
prop_est <- music_prop(bulk.eset = Mouse.bulk.eset, sc.eset = Mousesub.eset, clusters = 'cellType', samples = 'sampleID'); Est.prop <- prop_est$Est.prop.weighted
```

## Evaluation signals

- The estimated proportions matrix contains all 13 kidney cell types and resolves closely related PT and DCT as separate components, consistent with the clustering dendrogram grouping
- W-NNLS estimates are non-negative and stage-2 within-cluster proportions are consistent with the stage-1 cluster proportion constraints
- On simulated/benchmark data (bulk_construct from XinT2D.eset with relative.ab ground truth), Eval_multi shows MuSiC outperforming comparator methods, especially for tissues with closely related cell types
- music_basis output includes the design matrix (Disgn.mtx) and cross-subject mean of relative abundance (M.theta) for the expected cell types
- MuSiC v1.0.0 also accepts SingleCellExperiment class single-cell references without error (per README update)

## Limitations

- Correlation in gene expression leads to collinearity, making proportions hard to estimate reliably for less frequent and rare cell types — the motivation for the tree-guided recursive procedure
- Cell type proportions cannot be estimated with TPM as the input; RPKM is usable only if estimates of cell-type-specific total RNA abundance are provided from another dataset
- scRNA-seq references do not reflect true tissue proportions and biased cell dissociation; MuSiC transfers gene expression information, not true proportions
- Under assumptions A1–A3 only trends of cell type proportion change across subjects are recoverable, not absolute cell type proportions; if the efficiency assumption fails, only the fraction of RNA molecules per cell type can be estimated

## Evidence

- [discussion] MuSiC is a weighted non-negative least squares regression (W-NNLS), which does not require pre-selected marker genes.: "MuSiC is a weighted non-negative least squares regression (W-NNLS), which does not require pre-selected marker genes."
- [discussion] Stage 2 constrained W-NNLS: "the cell type proportions are estimated using W-NNLS with genes displaying small intra-cell type variance, subject to the constraint on the pre-estimated cluster proportions."
- [methods] music_prop function: "The cell type proportions is estimated by function `music_prop`"
- [methods] music_basis output: "`music_basis` is the function to produce the design matrix (`Disgn.mtx`), cross-subject mean of relative abundance (`M.theta`)"
- [readme] README scope statement: [evidence span withheld — anchor status: external_doc]
- [discussion] Collinearity limitation: "Correlation in gene expression can lead to collinearity, making it difficult to reliably estimate cell type proportions, especially for less frequent and rare cell types."
