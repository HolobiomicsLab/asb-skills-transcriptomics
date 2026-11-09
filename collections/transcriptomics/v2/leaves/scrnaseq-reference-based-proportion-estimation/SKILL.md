---
name: scrnaseq-reference-based-proportion-estimation
description: Use when you have bulk RNA-seq read counts from a complex tissue and
  a matching scRNA-seq reference with cell type annotations, and need cell type proportion
  estimates without pre-selected marker genes.
when_to_use_negative:
- Input bulk data are TPM-normalized rather than raw read counts — cell type proportions
  cannot be estimated with TPM as the input
- The bulk samples come from multiple clinical conditions where at least one condition
  differs from the scRNA-seq reference — use MuSiC2 instead
- Estimates of cell type specific total RNA abundance are unavailable and only RPKM
  data exist, with no way to provide cell size/RNA-content corrections
edam_operation: http://edamontology.org/operation_3634
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3518
tools:
- name: MuSiC
  role: 'Core deconvolution method: music_basis builds the design matrix, music_prop
    and music_prop.cluster estimate cell type proportions via weighted NNLS'
  repo: https://github.com/xuranw/MuSiC
- name: R
  role: Implementation environment; MuSiC is an R package installed via devtools,
    used with Biobase ExpressionSet objects
- name: Biobase
  role: Provides the ExpressionSet container for bulk and single-cell reference data
- name: ggplot2
  role: Render the pairwise sample correlation heatmap of estimated proportions
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-018-08023-x
    title: Bulk tissue cell type deconvolution with multi-subject single-cell expression
      reference
  source_content_hash: sha256:4067869dd51a2fde62f35a581d8dd59b2339b00743d05982a1fa970a162f3b58
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/xuranw/MuSiC
  tools:
  - Biobase
  - MuSiC
  - R
  - ggplot2
  edam_operation: http://edamontology.org/operation_3634
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3518
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# scrnaseq-reference-based-proportion-estimation

## Summary

Estimate cell type proportions in bulk RNA-seq data using a multi-subject single-cell RNA-seq reference with MuSiC's weighted non-negative least squares, which weights genes by cross-subject and cross-cell consistency instead of relying on pre-selected marker genes.

## When to use

Use when you have bulk RNA-seq read counts from a complex tissue and a matching scRNA-seq reference with cell type annotations, and need cell type proportion estimates without pre-selected marker genes.

## When NOT to use

- Input bulk data are TPM-normalized rather than raw read counts — cell type proportions cannot be estimated with TPM as the input
- The bulk samples come from multiple clinical conditions where at least one condition differs from the scRNA-seq reference — use MuSiC2 instead
- Estimates of cell type specific total RNA abundance are unavailable and only RPKM data exist, with no way to provide cell size/RNA-content corrections

## Inputs

- Bulk RNA-seq raw read count matrix (e.g., GSE56743 rat micro-dissected renal tubule segments, 105 samples) as an ExpressionSet or count matrix
- Single-cell RNA-seq reference with cell type annotations (e.g., GSE107585 mouse kidney 'Mousesub.eset' ExpressionSet or SingleCellExperiment)
- One-to-one ortholog gene mapping table (for cross-species deconvolution)
- Cell type grouping/clustering specification (e.g., clusters.type list from hierarchical clustering of the design matrix)

## Outputs

- Estimated cell type proportion matrix per bulk sample (Est.prop.weighted from music_prop)
- MuSiC first-step design matrix and cross-subject mean of relative abundance (Disgn.mtx, M.theta from music_basis)
- Sample-to-sample correlation heatmap of estimated proportions (e.g., rat_segment_proportion_heatmap.png rendered with ggplot2)

## How to apply

Download the bulk read counts (e.g., rat micro-dissected renal tubule segments from GSE56743) and the single-cell reference ExpressionSet (e.g., mouse kidney scRNA-seq GSE107585 'Mousesub.eset'). Keep raw read counts, never TPM, since cell type proportions cannot be estimated with TPM as the input; if the data are cross-species, map genes to one-to-one orthologs and intersect on the shared homologous gene set. Build the design matrix with music_basis (clusters='cellType', samples='sampleID'), excluding novel and transition cell types, and hierarchically cluster closely related cell types with complete linkage on the log design matrix, grouping them (e.g., immune, epithelial, endothelial, podocyte clusters) to mitigate collinearity. Estimate proportions for all bulk samples with music_prop (or music_prop.cluster with clusters.type for tree-guided two-stage estimation) and return Est.prop.weighted. Validate by computing pairwise sample-to-sample correlations of estimated proportions and checking that the dominant deconvolved cell types agree with known anatomical or compositional ground truth.

## Related tools

- **MuSiC** (Core deconvolution method: music_basis builds the design matrix, music_prop and music_prop.cluster estimate cell type proportions via weighted NNLS) — https://github.com/xuranw/MuSiC
- **R** (Implementation environment; MuSiC is an R package installed via devtools, used with Biobase ExpressionSet objects)
- **Biobase** (Provides the ExpressionSet container for bulk and single-cell reference data)
- **ggplot2** (Render the pairwise sample correlation heatmap of estimated proportions)

## Examples

```
library(MuSiC); Est.prop.weighted = music_prop(bulk.mtx = rat_bulk_counts, sc.sce = Mousesub.eset, clusters = 'cellType', samples = 'sampleID', clusters.type = list)
```

## Evaluation signals

- Samples from each renal segment cluster together in the pairwise sample-to-sample correlation heatmap of estimated proportions
- The dominant deconvolved cell type for each segment agrees with its anatomical position along the nephron (e.g., DCT cells recovered from the DCT segment)
- Estimated proportions are only returned for non-excluded cell types (novel and transition cell types removed) and each sample's proportions are non-negative and sum to ~1
- High within-segment sample correlation of estimated proportions, consistent with the reported figure

## Limitations

- Collinearity among closely related cell types makes it difficult to reliably estimate proportions, especially for less frequent and rare cell types — hence the need for hierarchical clustering and grouping (e.g., 13 cell types cut into 4 groups) before music_prop.cluster
- Under assumptions A1–A3 only trends of cell type proportion change across subjects can be recovered, not absolute cell type proportions
- Most scRNA-seq data do not reflect true cell type proportions in intact tissues; they only provide cell type-specific gene expression
- MuSiC can utilize RPKM only if estimates of cell type specific total RNA abundance are provided from another dataset

## Evidence

- [discussion] MuSiC is a weighted non-negative least squares regression (W-NNLS), which does not require pre-selected marker genes.: "MuSiC is a weighted non-negative least squares regression (W-NNLS), which does not require pre-selected marker genes."
- [discussion] TPM not usable: "However, cell type proportions cannot be estimated with TPM as the input."
- [methods] music_basis function: "`music_basis` is the function to produce the design matrix (`Disgn.mtx`), cross-subject mean of relative abundance (`M.theta`)"
- [methods] clustered grouping: "Here we cut 13 cell types into 4 groups"
- [discussion] collinearity limitation: "Correlation in gene expression can lead to collinearity, making it difficult to reliably estimate cell type proportions, especially for less frequent and rare cell types."
- [abstract] cross-species validation: "When applied to pancreatic islet and whole kidney expression data in human, mouse, and rats, MuSiC outperformed existing methods, especially for tissues with closely related cell types."
