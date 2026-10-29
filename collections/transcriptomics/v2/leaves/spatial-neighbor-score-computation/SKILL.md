---
name: spatial-neighbor-score-computation
description: Use when you have single-cell-resolution spatial transcriptomics coordinates
  (e.g., Seq-Scope mouse colon) plus a cell type label vector and need neighbor-influence
  scores per cell type to feed CPPLS cell-cell communication modeling.
when_to_use_negative:
- Data lacks single-cell spatial coordinates (e.g., dissociated single-cell RNA-seq
  without spatial information) — neighbor scores cannot be computed from O
- The goal is ligand-receptor database-based communication inference rather than spatial-neighbor
  regression modeling in CPPLS-MLP
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0092
tools:
- name: CPPLS-MLP
  role: Implements raw adjacent cell type score computation from coordinate matrix
    O and cell type label vector D as a preprocessing step of CPPLS regression modeling
  repo: https://github.com/wuzhenao/CPPLS-MLP
- name: R
  role: Runtime environment (version <= 4.2.3) in which the Seq-Scope coordinate matrix
    is loaded and scores are computed
  repo: https://github.com/wuzhenao/CPPLS-MLP
- name: Seurat
  role: R dependency installed alongside the CPPLS-MLP pipeline that processes the
    spatial dataset
  repo: https://github.com/wuzhenao/CPPLS-MLP
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bib/bbae198
    title: 'CPPLS-MLP: a method for constructing cell–cell communication networks
      and identifying related highly variable genes based on single-cell sequencing
      and spatial transcriptomics data'
  source_content_hash: sha256:9da67eeeab1ae873a05cdf06883b8eeea0f44bc641500553bac9e16b4bf68900
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/wuzhenao/CPPLS-MLP
  tools:
  - CPPLS-MLP
  - R
  - Seurat
  - circlize
  - pls
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# spatial-neighbor-score-computation

## Summary

Computes raw adjacent cell type scores from a spatial transcriptomics coordinate matrix and cell type labels, using log10(dist/dist0) distance transformation followed by per-cell-type z-score transformation, as the spatial-neighbor input to CPPLS regression modeling in CPPLS-MLP.

## When to use

Use when you have single-cell-resolution spatial transcriptomics coordinates (e.g., Seq-Scope mouse colon) plus a cell type label vector and need neighbor-influence scores per cell type to feed CPPLS cell-cell communication modeling.

## When NOT to use

- Data lacks single-cell spatial coordinates (e.g., dissociated single-cell RNA-seq without spatial information) — neighbor scores cannot be computed from O
- The goal is ligand-receptor database-based communication inference rather than spatial-neighbor regression modeling in CPPLS-MLP

## Inputs

- Spatial coordinate matrix O (single-cell-resolution cell positions, e.g., from the Seq-Scope mouse colon dataset)
- Cell type label vector D (per-cell cell type assignments)
- Gene expression matrix U (for companion per-cell-type preprocessing: zero-gene removal and z-score normalization)

## Outputs

- Raw adjacent cell type scores (log10(dist/dist0) per cell and neighboring cell type)
- Per-cell-type z-score-transformed adjacent cell type score matrix used as CPPLS regression independent variables

## How to apply

From the CPPLS-MLP workflow, take the spatial coordinate matrix O and cell type label vector D loaded in R (version <= 4.2.3). Compute raw adjacent cell type scores from O and D by transforming inter-cell distances as log10(dist/dist0), capturing how strongly each neighboring cell type impinges on a given cell. Then apply a z-score transformation to these raw scores separately within each cell type, so scores are comparable across cell types before regression. The resulting z-scored neighbor scores serve as independent variables alongside z-score normalized gene expression (genes with zero expression across all cells of each type removed) in the CPPLS 10-fold cross-validated regression that estimates the coefficient matrix W of neighboring-cell-type regulation on HVGs.

## Related tools

- **CPPLS-MLP** (Implements raw adjacent cell type score computation from coordinate matrix O and cell type label vector D as a preprocessing step of CPPLS regression modeling) — https://github.com/wuzhenao/CPPLS-MLP
- **R** (Runtime environment (version <= 4.2.3) in which the Seq-Scope coordinate matrix is loaded and scores are computed) — https://github.com/wuzhenao/CPPLS-MLP
- **Seurat** (R dependency installed alongside the CPPLS-MLP pipeline that processes the spatial dataset) — https://github.com/wuzhenao/CPPLS-MLP

## Examples

```
unzip 2023_01_Seq-Scope.zip; run 2023_01_Seq-Scope.RMD  # computes log10(dist/dist0) adjacent cell type scores in R (<= 4.2.3)
```

## Evaluation signals

- Raw scores are finite and non-constant after log10(dist/dist0) transformation; zero-distance degenerate cases do not produce NaN/Inf
- Per-cell-type z-scored neighbor scores have mean approximately 0 and standard deviation approximately 1 within each cell type
- Downstream CPPLS 10-fold cross-validation runs to completion on the z-scored scores plus normalized gene expression, yielding a coefficient matrix W
- Resulting MIMO network (after two-step BH FDR = 0.05 coefficient filtering) contains edges connecting biologically validated pairs such as immature B/IgA B cells and fibroblasts, e.g. renderable as Supplementary Figure S8A

## Limitations

- Scores depend on the dist0 reference distance choice; the article provides no sensitivity analysis for this parameter
- Only demonstrated on two spatial datasets (Seq-Scope mouse colon and seqFISH+ mouse cortex); generalization to other ST platforms is untested
- Image-processing extraction of cell locations from tissue slice images is only a proposal in the paper, not an implemented component — coordinates must already exist

## Evidence

- [methods] Computation of raw adjacent cell type scores from coordinate matrix and cell type labels: [evidence span withheld — anchor status: not_found]
- [methods] Per-cell-type z-score transformation of neighbor scores: [evidence span withheld — anchor status: not_found]
- [methods] 10-fold CV input of gene expression and spatial coordinates: "we input gene expression data and spatial coordinate information into the model and divide these data into 10 subsets of approximately equal size"
- [readme] Run instruction for the Seq-Scope script: "you can decompress the 2023_01_seqFISH+.zip and 2023_01_Seq-Scope.zip packages and run them in R"
- [discussion] Unannotated cell locations in proposed image extraction: "We propose utilizing image processing techniques to extract precise cell location information from tissue slice images in ST data"
