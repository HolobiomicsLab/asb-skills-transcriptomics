---
name: spatial-feature-plot-visualization
description: Use when you have spatial transcriptomics data (e.g. 10x Genomics Visium
  spots with GetTissueCoordinates) and need to identify spatially patterned genes
  without first defining spot clusters.
when_to_use_negative:
- Your input space is an embedding such as t-SNE/UMAP/PCA from a non-spatial single-cell
  experiment rather than actual physical tissue coordinates — use the 2D/PCA workflow
  variants instead
- You need continuous expression levels (not binary detected/not-detected) to drive
  the analysis — the current implementation only accepts binary detection data with
  a hard threshold
- Genes are expressed in very few spots (fewer than 10 non-zero counts) and are not
  biologically relevant — they should be filtered out before running to save computation
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: singleCellHaystack
  role: 'Core method: computes Kullback-Leibler divergence per gene to find non-random
    detection patterns in the 2D spatial coordinate space (haystack(), show_result_haystack())'
  repo: https://github.com/alexisvdb/singleCellHaystack
- name: Seurat
  role: Loads the Visium slice, extracts counts via GetAssayData, log-normalizes with
    NormalizeData, and retrieves spatial spot coordinates via GetTissueCoordinates
  repo: https://github.com/satijalab/seurat
- name: SeuratData
  role: Distributes and installs the stxBrain anterior1 Visium dataset package used
    as the spatial input data
  repo: https://github.com/satijalab/seurat-data
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-020-17900-3
    title: A clustering-independent method for finding differentially expressed genes
      in single-cell transcriptome data
  source_content_hash: sha256:2cc2f23f194ebdc0406a6ddac1dde133ff46b70ac8dbbfa4eec3158415da9a56
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/alexisvdb/singleCellHaystack
  tools:
  - R
  - Seurat
  - SeuratData
  - singleCellHaystack
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# spatial-feature-plot-visualization

## Summary

A clustering-independent spatial DEG workflow that runs singleCellHaystack's Kullback-Leibler divergence-based haystack() on the real 2D tissue coordinates of 10x Visium spots (rather than an embedding) plus a normalized, detection-binarized expression matrix, to find genes spatially regulated across tissue structures.

## When to use

Use when you have spatial transcriptomics data (e.g. 10x Genomics Visium spots with GetTissueCoordinates) and need to identify spatially patterned genes without first defining spot clusters.

## When NOT to use

- Your input space is an embedding such as t-SNE/UMAP/PCA from a non-spatial single-cell experiment rather than actual physical tissue coordinates — use the 2D/PCA workflow variants instead
- You need continuous expression levels (not binary detected/not-detected) to drive the analysis — the current implementation only accepts binary detection data with a hard threshold
- Genes are expressed in very few spots (fewer than 10 non-zero counts) and are not biologically relevant — they should be filtered out before running to save computation

## Inputs

- Seurat object of a 10x Visium spatial slice (e.g. stxBrain anterior1) loaded via SeuratData
- Counts matrix from GetAssayData
- 2D tissue spot coordinates from GetTissueCoordinates
- Detection cutoff value (e.g. 1, counts > 0) for binarizing expression

## Outputs

- Ranked haystack result table with D_KL, log.p.vals, and log.p.adj per gene
- Top-N gene list (e.g. top 10 via show_result_haystack) of spatially regulated genes
- Full ranked results written to a TSV (e.g. spatial_deg_visium_anterior1.tsv)

## How to apply

Load the 10x Visium slice (e.g. stxBrain anterior1) via SeuratData's LoadData (installing with SeuratData::InstallData if absent) and extract counts with Seurat's GetAssayData. Filter out genes with non-zero counts in fewer than 10 spots to cut computation on very lowly expressed genes, then log-normalize with NormalizeData. Extract the actual 2D tissue spot coordinates with GetTissueCoordinates — not a t-SNE/UMAP embedding — and run singleCellHaystack's haystack() on those coordinates and the normalized expression matrix with set.seed(123) and a detection cutoff of 1 (counts > 0), since the method requires binary detection data. Rank results with show_result_haystack and inspect whether top D_KL-scoring genes map onto known anatomical structures (Gpr88, Pde10a, Rgs9 in caudate putamen; Slc17a7, Nptxr in cortex; Pcp4l1 in olfactory bulb) as validation of spatial patterning.

## Related tools

- **singleCellHaystack** (Core method: computes Kullback-Leibler divergence per gene to find non-random detection patterns in the 2D spatial coordinate space (haystack(), show_result_haystack())) — https://github.com/alexisvdb/singleCellHaystack
- **Seurat** (Loads the Visium slice, extracts counts via GetAssayData, log-normalizes with NormalizeData, and retrieves spatial spot coordinates via GetTissueCoordinates) — https://github.com/satijalab/seurat
- **SeuratData** (Distributes and installs the stxBrain anterior1 Visium dataset package used as the spatial input data) — https://github.com/satijalab/seurat-data

## Examples

```
set.seed(123); res <- haystack(spatial_coords, dat.expression, cutoff = 1); show_result_haystack(res, n = 10)
```

## Evaluation signals

- Output file (spatial_deg_visium_anterior1.tsv) exists, is a parseable TSV, and contains the expected columns D_KL, log.p.vals, and log.p.adj
- Top-ranked genes include Gpr88, Pde10a, Rgs9, Slc17a7, Nptxr, and Pcp4l1, and their known anatomical localization (caudate putamen, cerebral cortex, olfactory bulb) matches the ranked order
- Genes with non-zero counts in fewer than 10 spots are absent from the input matrix before haystack() runs
- Coordinates passed to haystack() are real tissue spot coordinates from GetTissueCoordinates, not an embedding
- run was seeded (set.seed(123)) with a detection cutoff of 1 so results are reproducible

## Limitations

- The method requires binary detection data as input; no continuous expression support is implemented, and genes with different expression levels but identical detection patterns receive identical D_KL and p-values
- P-values may be inflated because gene expression data are used twice (for coordinates/detection structure and for DEG prediction); this issue affects singleCellHaystack and is deferred to future updates
- The detection threshold definition is left to the user and is somewhat arbitrary; zero counts may reflect dropout events rather than true absence, and imputation is not applied

## Evidence

- [abstract] we present singleCellHaystack, a method that enables the prediction of DEGs without relying on explicit clustering of cells: "we present singleCellHaystack, a method that enables the prediction of DEGs without relying on explicit clustering of cells"
- [methods] spatial input space: "we are not using an embedding as input space here, but the actual 2D coordinates of spots inside the tissue"
- [methods] gene filtering: "We filter genes with less 10 cells with non-zero counts. This reduces the computational time by eliminating very lowly expressed genes."
- [readme] KL divergence method: "singleCellHaystack uses Kullback-Leibler
Divergence to find features that have patterns of activity in subsets of
samples that are non-randomly positioned inside any input space."
- [discussion] binary input requirement: "The current implementation of our method requires binary detection data as input (i.e., a gene is either detected or not in each cell)."
