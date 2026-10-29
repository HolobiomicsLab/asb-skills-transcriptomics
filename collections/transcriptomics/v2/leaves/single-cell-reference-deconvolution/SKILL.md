---
name: single-cell-reference-deconvolution
description: Use when bulk RNA-seq ExpressionSet data must be deconvolved into cell
  type proportions and a multi-subject scRNA-seq reference (ExpressionSet or SingleCellExperiment)
  with cell type and subject annotations is available.
when_to_use_negative:
- Input bulk data are TPM-normalized — cell type proportions cannot be estimated with
  TPM as the input; use read counts (RPKM only if cell-type-specific total RNA abundance
  estimates are provided)
- Only a single-subject scRNA-seq reference is available — MuSiC relies on cross-subject
  scRNA-seq to capture subject-to-subject variation in cell-type-specific expression
- The goal is absolute cell type proportions under unsatisfied efficiency assumptions
  — only the fraction of RNA molecules from each cell type, or trends of proportion
  change across subjects, can be recovered
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3518
tools:
- name: MuSiC
  role: 'Core deconvolution method: music_prop for W-NNLS proportion estimation, music_basis
    for the design matrix, music_prop.cluster for tree-guided recursive two-stage
    deconvolution of closely related cell types'
  repo: https://github.com/xuranw/MuSiC
- name: BSEQ-sc
  role: 'Baseline comparator: bseqsc_basis and bseqsc_proportions with pancreasMarkers
    marker lists build an SVR-selected marker basis for proportion estimation to benchmark
    against MuSiC'
- name: CIBERSORT
  role: Baseline comparator referenced for support-vector-regression-based marker
    gene selection used to address collinearity
- name: R
  role: Implementation environment; MuSiC is an R package installed from GitHub and
    applied to ExpressionSet objects
  repo: https://github.com/xuranw/MuSiC
provenance:
  source_task_ids:
  - task_001
  - task_006
  - task_003
  source_papers:
  - doi: 10.1038/s41467-018-08023-x
    title: Bulk tissue cell type deconvolution with multi-subject single-cell expression
      reference
  source_content_hash: sha256:6aa0d8e2156cb86ab689aa00847d145216a4156d87da370af63967a7456fbb2f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/xuranw/MuSiC
  tools:
  - BSEQ-sc
  - CIBERSORT
  - MuSiC
  - R
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3518
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# single-cell-reference-deconvolution

## Summary

Estimate cell type proportions in bulk RNA-seq data using cross-subject scRNA-seq references with MuSiC, a weighted non-negative least squares (W-NNLS) method that weights genes by cross-subject and cross-cell consistency instead of pre-selected markers.

## When to use

Use when bulk RNA-seq ExpressionSet data must be deconvolved into cell type proportions and a multi-subject scRNA-seq reference (ExpressionSet or SingleCellExperiment) with cell type and subject annotations is available.

## When NOT to use

- Input bulk data are TPM-normalized — cell type proportions cannot be estimated with TPM as the input; use read counts (RPKM only if cell-type-specific total RNA abundance estimates are provided)
- Only a single-subject scRNA-seq reference is available — MuSiC relies on cross-subject scRNA-seq to capture subject-to-subject variation in cell-type-specific expression
- The goal is absolute cell type proportions under unsatisfied efficiency assumptions — only the fraction of RNA molecules from each cell type, or trends of proportion change across subjects, can be recovered

## Inputs

- Bulk RNA-seq read counts ExpressionSet (e.g., GSE50244bulkeset.rds, Mousebulkeset.rds from GSE81492)
- Single-cell reference ExpressionSet or SingleCellExperiment with cellType and sampleID annotations (e.g., EMTABesethealthy.rds E-MTAB-5061, XinT2Deset.rds GSE81608, Mousesubeset.rds subset of GSE107585 with 16273 genes x 10000 cells)
- Optional: cell type grouping list (clusters.type) and intra-cluster differential expression markers (IEmarkers.RData) for recursive two-stage deconvolution

## Outputs

- Estimated cell type proportion matrix (Est.prop.weighted from music_prop, W-NNLS estimates)
- All-gene NNLS proportion estimates (Est.prop.allgene)
- Design matrix, cross-subject mean and variance of relative abundance from music_basis
- Benchmark metric table of RMSD, mAD, and Pearson correlation per method (e.g., benchmark_evaluation_metrics.csv)
- Cluster-level and within-cluster proportion estimates from music_prop.cluster (e.g., music_cluster_proportions.csv)

## How to apply

Load the bulk and single-cell ExpressionSet objects (e.g., GSE50244bulkeset.rds, EMTABesethealthy.rds, XinT2Deset.rds, Mousebulkeset.rds, Mousesubeset.rds) via readRDSFromWeb, then run music_prop with bulk.eset, sc.eset, clusters (e.g., 'cellType'), samples (e.g., 'sampleID'), and select.ct (e.g., c('alpha','beta','delta','gamma')); extract Est.prop.weighted (MuSiC W-NNLS estimates) and Est.prop.allgene (all-gene NNLS). Use read counts, not TPM, since proportions cannot be estimated with TPM input. When closely related cell types cause collinearity, apply the tree-guided recursive procedure: compute the design matrix with music_basis, hierarchical-cluster cell types (complete linkage on Euclidean distance of log design matrix + 1e-6) into groups (e.g., 4 groups over 13 mouse kidney cell types), and run music_prop.cluster with group.markers (IEmarkers) and clusters.type, which estimates cluster proportions with cluster-consistent genes in stage 1 and within-cluster proportions via W-NNLS constrained by stage-1 cluster proportions in stage 2. Evaluate accuracy against known proportions with Eval_multi, reporting RMSD, mAD, and Pearson correlation (e.g., MuSiC RMSD 0.09881, mAD 0.06357, R 0.9378 on the Segerstolpe artificial bulk benchmark, beating NNLS R 0.8159 and BSEQ-sc R 0.7880).

## Related tools

- **MuSiC** (Core deconvolution method: music_prop for W-NNLS proportion estimation, music_basis for the design matrix, music_prop.cluster for tree-guided recursive two-stage deconvolution of closely related cell types) — https://github.com/xuranw/MuSiC
- **BSEQ-sc** (Baseline comparator: bseqsc_basis and bseqsc_proportions with pancreasMarkers marker lists build an SVR-selected marker basis for proportion estimation to benchmark against MuSiC)
- **CIBERSORT** (Baseline comparator referenced for support-vector-regression-based marker gene selection used to address collinearity)
- **R** (Implementation environment; MuSiC is an R package installed from GitHub and applied to ExpressionSet objects) — https://github.com/xuranw/MuSiC

## Examples

```
Est.prop.Xin <- music_prop(bulk.eset = XinT2D.construct.full$Bulk.eset, sc.eset = EMTAB.eset, clusters = 'cellType', samples = 'sampleID', select.ct = c('alpha','beta','delta','gamma'))
```

## Evaluation signals

- Benchmark metrics computed with Eval_multi must match reported values to within rounding: MuSiC RMSD 0.09881, mAD 0.06357, Pearson R 0.9378; NNLS RMSD 0.17161, R 0.8159; BSEQ-sc RMSD 0.21741, R 0.7880
- Estimated proportion matrix must contain all targeted cell types (e.g., all 13 kidney cell types) with non-negative values
- For recursive deconvolution, closely related PT and DCT must receive distinct proportion estimates consistent with their dendrogram grouping
- In mouse CKD applications, DCT proportions should fall in the 8–20% range and PT proportions should decrease with disease progression across UUO, folic acid, and APOL1 models
- Sum of estimated cell type proportions per sample should be constrained (NNLS simplex constraint) and correlate with true proportions from relative.ab on artificial bulk

## Limitations

- Cell type proportions cannot be estimated with TPM as the input; RPKM is usable only if cell-type-specific total RNA abundance estimates are available from another dataset
- Under assumptions A1–A3 only trends of cell type proportion change across subjects can be recovered, not absolute cell type proportions; when the efficiency assumption is not satisfied, only the fraction of RNA molecules from each cell type can be estimated
- Collinearity among closely related cell types makes proportion estimation unreliable, especially for rare cell types — requiring the recursive tree-guided procedure
- Most scRNA-seq data do not reflect true cell type proportions in intact tissues due to biased cell dissociation, so references provide cell-type-specific expression rather than true composition

## Evidence

- [discussion] W-NNLS without pre-selected markers: "MuSiC is a weighted non-negative least squares regression (W-NNLS), which does not require pre-selected marker genes."
- [methods] Artificial bulk benchmark construction: "Benchmark dataset is constructed by summing up single cell data from `XinT2D.eset`."
- [discussion] Recursive two-stage procedure: "the cell type proportions are estimated using W-NNLS with genes displaying small intra-cell type variance, subject to the constraint on the pre-estimated cluster proportions."
- [abstract] Outperforms existing methods for closely related cell types: "When applied to pancreatic islet and whole kidney expression data in human, mouse, and rats, MuSiC outperformed existing methods, especially for tissues with closely related cell types."
- [readme] R package installation: "devtools::install_github('xuranw/MuSiC')"
