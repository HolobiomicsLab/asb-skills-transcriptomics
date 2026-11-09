---
name: single-cell-expression-matrix-subsetting
description: Use when validating predicted cell–cell communication by visualizing
  a specific gene's expression in two selected cell types from a spatial dataset (Seq-Scope/seqFISH+
  .zip archives containing expression matrices and spatial coordinates).
when_to_use_negative:
- Input is raw sequencing data or aggregated bulk expression without per-cell spatial
  coordinates — subsetting and spatial plotting require single-cell resolution coordinates
- Goal is to construct or compare communication networks (CPPLS vs CCPLS) rather than
  to visually validate a specific predicted communication via gene expression
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0102
tools:
- name: ComplexHeatmap
  role: R plotting package used to render the spatial expression plot of GPX1 in subsetted
    cell types; installed via BiocManager
  repo: https://github.com/wuzhenao/CPPLS-MLP
- name: Seurat
  role: R package used for Seurat-based handling of the single-cell expression data
    during subsetting
  repo: https://github.com/wuzhenao/CPPLS-MLP
- name: CPPLS-MLP
  role: Provides the Seq-Scope dataset archive (2023_01_Seq-Scope.zip) and the communication
    predictions being validated by the spatial expression visualization
  repo: https://github.com/wuzhenao/CPPLS-MLP
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bib/bbae198
    title: 'CPPLS-MLP: a method for constructing cell–cell communication networks
      and identifying related highly variable genes based on single-cell sequencing
      and spatial transcriptomics data'
  source_content_hash: sha256:a368b2443e8b7431157bd92cdfb296767ae51b44d21bb716fb74695a141c3221
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/wuzhenao/CPPLS-MLP
  tools:
  - CPPLS-MLP
  - ComplexHeatmap
  - R
  - Seurat
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# single-cell-expression-matrix-subsetting

## Summary

Subsets a single-cell/spatial transcriptomics gene expression matrix and its spatial coordinate matrix to specific cell types (e.g. B.cell_IgA and fibroblasts) and extracts expression values for a target gene (e.g. GPX1) for downstream spatial validation plots.

## When to use

Use when validating predicted cell–cell communication by visualizing a specific gene's expression in two selected cell types from a spatial dataset (Seq-Scope/seqFISH+ .zip archives containing expression matrices and spatial coordinates).

## When NOT to use

- Input is raw sequencing data or aggregated bulk expression without per-cell spatial coordinates — subsetting and spatial plotting require single-cell resolution coordinates
- Goal is to construct or compare communication networks (CPPLS vs CCPLS) rather than to visually validate a specific predicted communication via gene expression

## Inputs

- Dataset archive from CPPLS-MLP repo (e.g. 2023_01_Seq-Scope.zip) containing gene expression matrix and spatial coordinate matrix
- Cell type label vector distinguishing B.cell_IgA cells and fibroblasts
- Target gene name (GPX1)

## Outputs

- Per-cell GPX1 expression values for the subsetted cell types
- Two-dimensional spatial plot of cell positions colored by GPX1 expression (e.g. gpx1_spatial_expression.png, PNG/PDF)

## How to apply

Obtain the dataset archive (e.g. 2023_01_Seq-Scope.zip) from the CPPLS-MLP GitHub repository (https://github.com/wuzhenao/CPPLS-MLP) and extract the gene expression matrix and spatial coordinate data. Subset both matrices to the cell types of interest (here B.cell_IgA cells and fibroblasts) and extract the GPX1 expression value per cell. Using R (version <= 4.2.3) with Seurat-based data handling and ComplexHeatmap (version 2.16, installed via BiocManager), plot the two-dimensional spatial positions of the cells colored by GPX1 expression level for both cell types, saving the combined plot (e.g. gpx1_spatial_expression.png) for comparison against the article's Figure 4C and D. Rationale: the paper confirms predicted B cell–fibroblast communication by showing that immature B cells adjacent to fibroblasts exhibit high GPX1 expression in their spatial distribution.

## Related tools

- **ComplexHeatmap** (R plotting package used to render the spatial expression plot of GPX1 in subsetted cell types; installed via BiocManager) — https://github.com/wuzhenao/CPPLS-MLP
- **Seurat** (R package used for Seurat-based handling of the single-cell expression data during subsetting) — https://github.com/wuzhenao/CPPLS-MLP
- **CPPLS-MLP** (Provides the Seq-Scope dataset archive (2023_01_Seq-Scope.zip) and the communication predictions being validated by the spatial expression visualization) — https://github.com/wuzhenao/CPPLS-MLP

## Examples

```
BiocManager::install("ComplexHeatmap"); run 2023_01_Seq-Scope.RMD
```

## Evaluation signals

- Output image file (gpx1_spatial_expression.png) exists and is a valid PNG or PDF
- Spatial distribution pattern visually matches Figure 4C and D: B.cell_IgA cells adjacent to fibroblasts show high GPX1 expression
- Both subsetted cell types (B.cell_IgA and fibroblasts) are represented in the combined spatial plot, colored by GPX1 expression level
- Subsetted matrices retain only cells with the specified cell type labels and the GPX1 column from the Seq-Scope expression matrix

## Limitations

- No accession IDs or download sources beyond the GitHub repository are provided for the Seq-Scope dataset; the archive must be fetched from the CPPLS-MLP repo
- R version constraint of <= 4.2.3 applies (Seurat, ComplexHeatmap and other dependencies were pinned to this environment)
- Validation is visual comparison against a published figure rather than a quantitative similarity metric

## Evidence

- [discussion] we visualized the expression levels of the GPX1 gene in both cell types and its spatial distribution. This visualization confirmed the accuracy of our predictions: "we visualized the expression levels of the GPX1 gene in both cell types and its spatial distribution. This visualization confirmed the accuracy of our predictions"
- [readme] BiocManager::install("ComplexHeatmap"): "BiocManager::install("ComplexHeatmap")"
- [readme] If you want to use CPPLS to build a cell-to-cell communication network, you can decompress the 2023_01_seqFISH+.zip and 2023_01_Seq-Scope.zip packages and run them in R: "If you want to use CPPLS to build a cell-to-cell communication network, you can decompress the 2023_01_seqFISH+.zip and 2023_01_Seq-Scope.zip packages and run them in R"
- [intro] dependence: R version <= 4.2.3.: "dependence: R version <= 4.2.3."
