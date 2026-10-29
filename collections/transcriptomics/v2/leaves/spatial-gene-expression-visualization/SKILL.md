---
name: spatial-gene-expression-visualization
description: Use when a predicted cell–cell communication pair (e.g., immature B cells–fibroblasts,
  L5–L6 eNeurons) needs spatial validation by overlaying a specific gene's expression
  on cell coordinates from spatial transcriptomics data.
when_to_use_negative:
- The goal is to construct or score the cell–cell communication network itself rather
  than validate an already-predicted communication pair with a gene-expression overlay
- The input data lack single-cell spatial coordinates (e.g., bulk or non-spatial scRNA-seq),
  so no 2D spatial distribution can be plotted
- The candidate gene has zero expression across all cells of the target cell type,
  which the pipeline removes before downstream analysis
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: CPPLS-MLP
  role: Source of the predicted cell–cell communications and the dataset archives
    (2023_01_Seq-Scope.zip, 2023_01_seqFISH+.zip) whose communication predictions
    the visualization validates
  repo: https://github.com/wuzhenao/CPPLS-MLP
- name: ComplexHeatmap
  role: R package used to render the spatial expression plot of GPX1 (version 2.16,
    installed via BiocManager)
  repo: https://github.com/wuzhenao/CPPLS-MLP
- name: Seurat
  role: Seurat-based data handling of the expression and coordinate matrices prior
    to plotting
  repo: https://github.com/wuzhenao/CPPLS-MLP
- name: R
  role: Runtime environment (version <= 4.2.3) for the Seq-Scope visualization workflow,
    including the 2023_01_Seq-Scope.RMD script
  repo: https://github.com/wuzhenao/CPPLS-MLP
- name: Python
  role: Runtime environment (>= 3.7) for the seqFISH+ CPLX1 workflow with networkx
    3.2.1 and pandas 2.1.0
  repo: https://github.com/wuzhenao/CPPLS-MLP
provenance:
  source_task_ids:
  - task_005
  - task_006
  source_papers:
  - doi: 10.1093/bib/bbae198
    title: 'CPPLS-MLP: a method for constructing cell–cell communication networks
      and identifying related highly variable genes based on single-cell sequencing
      and spatial transcriptomics data'
  source_content_hash: sha256:7e0623cce400a977b5f29f2930ae13203b716effda8f6968c6693c0559bf7c8d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/wuzhenao/CPPLS-MLP
  tools:
  - CPPLS-MLP
  - ComplexHeatmap
  - Python
  - R
  - Seurat
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# spatial-gene-expression-visualization

## Summary

Two-dimensional spatial scatter/heatmap plots of a candidate communication-associated gene (e.g., GPX1 on Seq-Scope, CPLX1 on seqFISH+) colored by expression level in interacting cell types, used to visually validate cell–cell communication networks predicted by CPPLS-MLP.

## When to use

Use when a predicted cell–cell communication pair (e.g., immature B cells–fibroblasts, L5–L6 eNeurons) needs spatial validation by overlaying a specific gene's expression on cell coordinates from spatial transcriptomics data.

## When NOT to use

- The goal is to construct or score the cell–cell communication network itself rather than validate an already-predicted communication pair with a gene-expression overlay
- The input data lack single-cell spatial coordinates (e.g., bulk or non-spatial scRNA-seq), so no 2D spatial distribution can be plotted
- The candidate gene has zero expression across all cells of the target cell type, which the pipeline removes before downstream analysis

## Inputs

- Gene expression matrix from a spatial transcriptomics dataset (Seq-Scope mouse colon or seqFISH+ mouse cortex)
- Spatial coordinate matrix (2D cell positions) for the same cells
- Cell type label vector (e.g., B.cell_IgA, fibroblasts, L5 eNeuron, L6 eNeuron)
- Target gene name (e.g., GPX1, CPLX1) predicted to be communication-associated

## Outputs

- Two-dimensional spatial scatter/heatmap plot colored by target-gene expression level per cell type (PNG or PDF, e.g., gpx1_spatial_expression.png, cplx1_spatial_expression.png)
- Combined figure suitable for side-by-side comparison against article Figure 4C/D or Supplementary Figure S7B

## How to apply

Obtain the dataset archive from the CPPLS-MLP GitHub repository (e.g., 2023_01_Seq-Scope.zip or 2023_01_seqFISH+.zip) and extract the gene expression matrix and the spatial coordinate matrix. Subset both matrices to the two communicating cell types (e.g., B.cell_IgA and fibroblasts, or L5 eNeuron and L6 eNeuron) and extract the target gene's expression per cell. In R, join expression values to the 2D coordinates and render spatial positions colored by expression level using ComplexHeatmap (installed via BiocManager) with Seurat-based data handling; alternatively use Python (>= 3.7) with networkx 3.2.1/pandas 2.1.0 for the seqFISH+ workflow. Save the figure as a PNG or PDF (e.g., gpx1_spatial_expression.png, cplx1_spatial_expression.png). The rationale is that a communication prediction is considered accurate when the ligand/receiver cell shows high target-gene expression spatially adjacent to its predicted partner, mirroring the article's Figure 4C/D or Supplementary Figure S7B layout.

## Related tools

- **CPPLS-MLP** (Source of the predicted cell–cell communications and the dataset archives (2023_01_Seq-Scope.zip, 2023_01_seqFISH+.zip) whose communication predictions the visualization validates) — https://github.com/wuzhenao/CPPLS-MLP
- **ComplexHeatmap** (R package used to render the spatial expression plot of GPX1 (version 2.16, installed via BiocManager)) — https://github.com/wuzhenao/CPPLS-MLP
- **Seurat** (Seurat-based data handling of the expression and coordinate matrices prior to plotting) — https://github.com/wuzhenao/CPPLS-MLP
- **R** (Runtime environment (version <= 4.2.3) for the Seq-Scope visualization workflow, including the 2023_01_Seq-Scope.RMD script) — https://github.com/wuzhenao/CPPLS-MLP
- **Python** (Runtime environment (>= 3.7) for the seqFISH+ CPLX1 workflow with networkx 3.2.1 and pandas 2.1.0) — https://github.com/wuzhenao/CPPLS-MLP

## Examples

```
run 2023_01_Seq-Scope.RMD
```

## Evaluation signals

- Output image file exists and is a valid PNG or PDF (e.g., gpx1_spatial_expression.png, cplx1_spatial_expression.png)
- For Seq-Scope: the plot's GPX1 spatial distribution visually matches Figure 4C/D, where B.cell_IgA cells adjacent to fibroblasts show high GPX1 expression
- For seqFISH+: CPLX1 expression in L5 eNeuron is higher where L5 eNeuron cells are closer to L6 eNeuron cells, matching Supplementary Figure S7B
- Both communicating cell types are present as separate colored populations in the plot, not merged into one layer
- Expression-to-coordinate join covers all cells of the two selected cell types (no dropped cells after subsetting)

## Limitations

- No accession IDs or download sources are provided for the Seq-Scope and seqFISH+ datasets beyond the GitHub-hosted zip archives
- R version must be <= 4.2.3 for the R-based workflow; Python must be >= 3.7 for the seqFISH+ workflow
- Validation is visual comparison against a specific article figure, so assessment is subjective and figure-layout dependent

## Evidence

- [discussion] we visualized the expression levels of the GPX1 gene in both cell types and its spatial distribution. This visualization confirmed the accuracy of our predictions: "we visualized the expression levels of the GPX1 gene in both cell types and its spatial distribution. This visualization confirmed the accuracy of our predictions"
- [intro] Applied to Seq-scope and seqFISH+ datasets, CPPLS-MLP unveils the regulatory role of intercellular communication across various cell types on HVG expression.: "Applied to Seq-scope and seqFISH+ datasets, CPPLS-MLP unveils the regulatory role of intercellular communication across various cell types on HVG expression."
- [readme] If you want to use CPPLS to build a cell-to-cell communication network, you can decompress the 2023_01_seqFISH+.zip and 2023_01_Seq-Scope.zip packages and run them in R: "If you want to use CPPLS to build a cell-to-cell communication network, you can decompress the 2023_01_seqFISH+.zip and 2023_01_Seq-Scope.zip packages and run them in R"
