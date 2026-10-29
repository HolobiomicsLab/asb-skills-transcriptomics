---
name: cross-species-gene-orthology-mapping
description: Use when bulk RNA-seq samples from one species (e.g., rat micro-dissected
  renal tubule segments, GSE56743) must be deconvolved with a single-cell reference
  from another species (e.g., mouse kidney scRNA-seq, GSE107585), which first requires
  homologous-gene mapping.
when_to_use_negative:
- Bulk and single-cell reference are from the same species with matching gene identifiers
  — no orthology mapping is needed, proceed directly to MuSiC
- Input expression values are TPM or other normalized units — cell type proportions
  cannot be estimated with TPM as the input; RPKM is only usable if cell type-specific
  total RNA abundance estimates are provided
- Ortholog coverage is too sparse or no one-to-one orthologs exist, leaving too small
  a shared gene set to support reliable W-NNLS estimation
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: MuSiC
  role: Performs weighted non-negative least squares deconvolution (music_basis, music_prop,
    music_prop.cluster) on the orthology-intersected cross-species count matrices
  repo: https://github.com/xuranw/MuSiC
- name: R
  role: Implementation environment; MuSiC is an R package installed from GitHub, with
    Biobase used for ExpressionSet handling and ggplot2 for the correlation heatmap
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-018-08023-x
    title: Bulk tissue cell type deconvolution with multi-subject single-cell expression
      reference
  source_content_hash: sha256:707e18e9fc3ab6ba2a95763be7e55bb82e6d2223649e33be85235cb3e4bd8c67
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
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# cross-species-gene-orthology-mapping

## Summary

Enables MuSiC deconvolution of bulk RNA-seq from one species against a scRNA-seq reference from another by mapping genes to one-to-one orthologs and intersecting both datasets on the shared homologous gene set while preserving raw read counts.

## When to use

Use when bulk RNA-seq samples from one species (e.g., rat micro-dissected renal tubule segments, GSE56743) must be deconvolved with a single-cell reference from another species (e.g., mouse kidney scRNA-seq, GSE107585), which first requires homologous-gene mapping.

## When NOT to use

- Bulk and single-cell reference are from the same species with matching gene identifiers — no orthology mapping is needed, proceed directly to MuSiC
- Input expression values are TPM or other normalized units — cell type proportions cannot be estimated with TPM as the input; RPKM is only usable if cell type-specific total RNA abundance estimates are provided
- Ortholog coverage is too sparse or no one-to-one orthologs exist, leaving too small a shared gene set to support reliable W-NNLS estimation

## Inputs

- Bulk RNA-seq raw read-count matrix from the target species (e.g., rat micro-dissected tubule segments, GSE56743, 105 samples)
- scRNA-seq reference ExpressionSet from the source species (e.g., mouse kidney 'Mousesub.eset' from GSE107585) with cellType and sampleID phenoData
- One-to-one ortholog mapping table between the two species' gene identifiers

## Outputs

- Intersected bulk and single-cell count matrices restricted to the shared one-to-one orthologous gene set (raw counts, not TPM)
- Cell type proportion estimates (Est.prop.weighted) for every bulk sample via music_prop / music_prop.cluster
- Sample-to-sample correlation heatmap of estimated proportions (e.g., rat_segment_proportion_heatmap.png)

## How to apply

Download the scRNA-seq reference as an ExpressionSet (e.g., the 'Mousesub.eset' subset of GSE107585) and the bulk read-count data (e.g., rat GSE56743). Map genes of the bulk species to one-to-one orthologs of the reference species and intersect both ExpressionSets on the shared homologous gene set, keeping raw read counts — cell type proportions cannot be estimated with TPM as the input. On the intersected matrices, build the MuSiC first-step information with music_basis (clusters = 'cellType', samples = 'sampleID'), excluding novel and transition cell types, and optionally hierarchically cluster cell types with complete linkage on the log design matrix into groups (e.g., immune, epithelial, endothelial, podocyte) to mitigate collinearity. Finally, estimate proportions for all 105 rat bulk samples with music_prop or music_prop.cluster and validate that the resulting Est.prop.weighted recovers biologically expected compositions (e.g., DCT cells dominating the DCT segment), demonstrating cross-species transfer of cell type-specific expression information.

## Related tools

- **MuSiC** (Performs weighted non-negative least squares deconvolution (music_basis, music_prop, music_prop.cluster) on the orthology-intersected cross-species count matrices) — https://github.com/xuranw/MuSiC
- **R** (Implementation environment; MuSiC is an R package installed from GitHub, with Biobase used for ExpressionSet handling and ggplot2 for the correlation heatmap)

## Examples

```
devtools::install_github('xuranw/MuSiC'); library(MuSiC)
```

## Evaluation signals

- The intersected gene set is non-empty and derived strictly from one-to-one orthologs present in both the bulk and single-cell ExpressionSets
- Count matrices remain raw integer read counts after intersection (no TPM normalization)
- Samples from each renal segment cluster together in the pairwise sample-to-sample correlation heatmap of estimated proportions
- The dominant deconvolved cell type at each segment agrees with its anatomical position along the nephron (e.g., DCT cells recovered from the DCT segment)
- High within-segment sample correlation of estimated cell type proportions across the 105 rat bulk samples

## Limitations

- Cell type proportions cannot be estimated with TPM as the input; MuSiC can utilize RPKM only if estimates of cell type-specific total RNA abundance are provided
- Only trends of cell type proportion change across subjects can be recovered, not absolute proportions, under the method's assumptions
- Collinearity among closely related cell types makes it difficult to reliably estimate proportions, especially for rare cell types — mitigate by grouping cell types via hierarchical clustering before music_prop.cluster
- Most scRNA-seq references do not reflect true cell type proportions in intact tissues, and the reference species' cell types may include novel or transition states absent in the target species

## Evidence

- [other] Cross-species mapping is the enabling step: [evidence span withheld — anchor status: not_found]
- [discussion] TPM prohibition: "However, cell type proportions cannot be estimated with TPM as the input."
- [abstract] Cross-species benchmark motivation: "When applied to pancreatic islet and whole kidney expression data in human, mouse, and rats, MuSiC outperformed existing methods, especially for tissues with closely related cell types."
- [abstract] Transfer of expression information across datasets: "By appropriate weighting of genes showing cross-subject and cross-cell consistency, MuSiC enables the transfer of cell type-specific gene expression information from one dataset to another."
- [discussion] Collinearity caveat for closely related cell types: "Correlation in gene expression can lead to collinearity, making it difficult to reliably estimate cell type proportions, especially for less frequent and rare cell types."
