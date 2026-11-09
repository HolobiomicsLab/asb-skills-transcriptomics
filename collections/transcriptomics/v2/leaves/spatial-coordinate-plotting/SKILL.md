---
name: spatial-coordinate-plotting
description: Use when you have a spatial transcriptomics gene expression matrix plus
  per-cell spatial coordinates and need a 2D scatter/heatmap of one gene's expression
  across specified cell types to validate predicted cell–cell communication.
when_to_use_negative:
- The goal is constructing the cell–cell communication network itself — use the CPPLS-MLP
  CPPLS regression workflow (2023_01_*.RMD scripts) rather than its downstream plotting
  validation step
- Input data lack per-cell spatial coordinates (e.g., dissociated single-cell or bulk
  RNA-seq), since the visualization is inherently a 2D spatial coordinate plot
- A quantitative, statistically tested validation of communication-associated genes
  is required — the article's visualization is a qualitative confirmation, not a statistical
  test
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0092
tools:
- name: ComplexHeatmap
  role: R/Bioconductor package used to render the combined 2D spatial expression plot
    colored by GPX1/CPLX1 expression level
- name: Seurat
  role: Seurat-based data handling of the expression matrix and coordinate matrix
    prior to plotting
- name: CPPLS-MLP
  role: Source of the Seq-Scope and seqFISH+ dataset archives and the R scripts whose
    predictions the spatial plots validate
  repo: https://github.com/wuzhenao/CPPLS-MLP
- name: R
  role: Runtime environment (version <= 4.2.3) in which the spatial expression visualization
    is produced
provenance:
  source_task_ids:
  - task_005
  - task_006
  source_papers:
  - doi: 10.1093/bib/bbae198
    title: 'CPPLS-MLP: a method for constructing cell–cell communication networks
      and identifying related highly variable genes based on single-cell sequencing
      and spatial transcriptomics data'
  source_content_hash: sha256:a2a2e5eb584d3bbe8e768cb481a30e3cde5fb7fbd23bde147cc22c7f4544043b
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
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# spatial-coordinate-plotting

## Summary

Renders the 2D spatial positions of cells from spatial transcriptomics datasets (Seq-Scope, seqFISH+) colored by a specific gene's expression level, to visually validate cell–cell communication predictions (e.g., GPX1 in B.cell_IgA/fibroblasts, CPLX1 in L5/L6 eNeurons) made by CPPLS-MLP.

## When to use

Use when you have a spatial transcriptomics gene expression matrix plus per-cell spatial coordinates and need a 2D scatter/heatmap of one gene's expression across specified cell types to validate predicted cell–cell communication.

## When NOT to use

- The goal is constructing the cell–cell communication network itself — use the CPPLS-MLP CPPLS regression workflow (2023_01_*.RMD scripts) rather than its downstream plotting validation step
- Input data lack per-cell spatial coordinates (e.g., dissociated single-cell or bulk RNA-seq), since the visualization is inherently a 2D spatial coordinate plot
- A quantitative, statistically tested validation of communication-associated genes is required — the article's visualization is a qualitative confirmation, not a statistical test

## Inputs

- Gene expression matrix from a Seq-Scope or seqFISH+ dataset archive (2023_01_Seq-Scope.zip / 2023_01_seqFISH+.zip)
- Per-cell two-dimensional spatial coordinate matrix
- Cell type label annotations (e.g., B.cell_IgA, fibroblasts, L5 eNeuron, L6 eNeuron)
- Target gene identifier whose spatial expression is being validated (e.g., GPX1, CPLX1)

## Outputs

- 2D spatial expression scatter/heatmap image colored by target-gene expression per cell type (e.g., gpx1_spatial_expression.png, cplx1_spatial_expression.png)
- PNG or PDF figure suitable for side-by-side comparison with the article's figures

## How to apply

Extract the gene expression matrix and spatial coordinate matrix from the dataset archive (e.g., 2023_01_Seq-Scope.zip or 2023_01_seqFISH+.zip from the CPPLS-MLP GitHub repository). Subset both matrices to the communicating cell types of interest (e.g., B.cell_IgA cells and fibroblasts on Seq-Scope; L5 eNeuron and L6 eNeuron on seqFISH+) and extract the target gene's expression values (e.g., GPX1 or CPLX1). Join expression values with the two-dimensional spatial coordinates per cell, then plot spatial positions colored by expression level using ComplexHeatmap (version 2.16, installed via BiocManager) with Seurat-based data handling in R (version <= 4.2.3). Save the combined spatial expression plot as a PNG or PDF, and judge correctness by visual comparison against the article's figure (Figure 4C/D for GPX1; Supplementary Figure S7B for CPLX1), checking that high expression co-localizes with the predicted communicating cell pairs.

## Related tools

- **ComplexHeatmap** (R/Bioconductor package used to render the combined 2D spatial expression plot colored by GPX1/CPLX1 expression level)
- **Seurat** (Seurat-based data handling of the expression matrix and coordinate matrix prior to plotting)
- **CPPLS-MLP** (Source of the Seq-Scope and seqFISH+ dataset archives and the R scripts whose predictions the spatial plots validate) — https://github.com/wuzhenao/CPPLS-MLP
- **R** (Runtime environment (version <= 4.2.3) in which the spatial expression visualization is produced)

## Examples

```
BiocManager::install("ComplexHeatmap")
```

## Evaluation signals

- Output image file exists and is a valid PNG or PDF
- For GPX1 (Seq-Scope): B.cell_IgA cells adjacent to fibroblasts show high GPX1 expression, visually matching Figure 4C and D of the article
- For CPLX1 (seqFISH+): CPLX1 expression in L5 eNeuron is higher where L5 eNeuron cells are closer to L6 eNeuron cells, matching Supplementary Figure S7B
- Cells are positioned at their correct two-dimensional spatial coordinates from the coordinate matrix, not clustered arbitrarily
- Both communicating cell types are plotted in the combined figure so adjacency of the predicted pair is visually checkable

## Limitations

- Validation is qualitative (visual comparison against published figures) — no quantitative similarity metric for the spatial pattern is specified
- Only demonstrated on two datasets (Seq-Scope mouse colon and seqFISH+ mouse cortex); the article acknowledges 'Despite the limited number of datasets, we chose these due to their representativeness'
- Requires R version <= 4.2.3 for the dependent packages (Seurat, ComplexHeatmap 2.16), constraining the runtime environment
- The article notes cell-location extraction from tissue images is only a proposal: 'We propose utilizing image processing techniques to extract precise cell location information from tissue slice images in ST data'

## Evidence

- [discussion] we visualized the expression levels of the GPX1 gene in both cell types and its spatial distribution. This visualization confirmed the accuracy of our predictions: "we visualized the expression levels of the GPX1 gene in both cell types and its spatial distribution. This visualization confirmed the accuracy of our predictions"
- [intro] BiocManager::install("ComplexHeatmap"): "BiocManager::install("ComplexHeatmap")"
- [readme] If you want to use CPPLS to build a cell-to-cell communication network, you can decompress the 2023_01_seqFISH+.zip and 2023_01_Seq-Scope.zip packages: "If you want to use CPPLS to build a cell-to-cell communication network, you can decompress the 2023_01_seqFISH+.zip and 2023_01_Seq-Scope.zip packages"
- [methods] the seqFISH+ mouse cortex dataset and the Seq-Scope mouse colon dataset: "the seqFISH+ mouse cortex dataset and the Seq-Scope mouse colon dataset"
