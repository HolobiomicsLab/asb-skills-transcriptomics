---
name: seqfish-dataset-processing
description: Use when a spatial transcriptomics task requires reproducing seqFISH+
  mouse cortex communication analysis or validating that a communication-predicted
  gene (e.g., CPLX1) shows elevated expression where two cell types (L5/L6 eNeurons)
  are spatially adjacent.
when_to_use_negative:
- The target dataset is bulk RNA-seq or scRNA-seq without single-cell spatial coordinates
  — the validation plot depends on the spatial coordinate matrix
- The goal is Seq-Scope mouse colon analysis (e.g., GPX1 in immature B cells/fibroblasts)
  rather than seqFISH+ cortex — use the 2023_01_Seq-Scope.RMD entry point instead
- Input is a pre-computed communication network or feature table with no raw expression/coordinate
  matrices to reload and filter
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: CPPLS-MLP
  role: Provides the repository, data package (2023_01_seqFISH+.zip), and 2023_01_seqFISH+.RMD
    script used to construct the cell-to-cell communication network being validated
  repo: https://github.com/wuzhenao/CPPLS-MLP
- name: R
  role: Runtime (<= 4.2.3) in which 2023_01_seqFISH+.RMD is executed to build the
    communication network
  repo: https://github.com/wuzhenao/CPPLS-MLP
- name: Python
  role: Runtime (>= 3.7) used to filter L5/L6 eNeuron cells, join CPLX1 expression
    with spatial coordinates, and render the validation plot
  repo: https://github.com/wuzhenao/CPPLS-MLP
- name: Seurat
  role: R dependency of the CPPLS-MLP pipeline, supplying FindVariableFeatures()-style
    single-cell preprocessing used in the network workflow
  repo: https://github.com/wuzhenao/CPPLS-MLP
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bib/bbae198
    title: 'CPPLS-MLP: a method for constructing cell–cell communication networks
      and identifying related highly variable genes based on single-cell sequencing
      and spatial transcriptomics data'
  source_content_hash: sha256:860987e15991b1bbfc08986a5a694206a657766c996588c00e653c0e28a17cf3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/wuzhenao/CPPLS-MLP
  tools:
  - CPPLS-MLP
  - Python
  - R
  - Seurat
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# seqfish-dataset-processing

## Summary

Processes the seqFISH+ mouse cortex dataset from the CPPLS-MLP repository to rebuild the cell-to-cell communication network and produce a spatial gene-expression validation plot (e.g., CPLX1 in L5/L6 eNeurons, mirroring Supplementary Figure S7B). It operationalizes the article's claim that CPPLS-MLP unveils how intercellular communication regulates HVG expression in spatial transcriptomics data.

## When to use

Use when a spatial transcriptomics task requires reproducing seqFISH+ mouse cortex communication analysis or validating that a communication-predicted gene (e.g., CPLX1) shows elevated expression where two cell types (L5/L6 eNeurons) are spatially adjacent.

## When NOT to use

- The target dataset is bulk RNA-seq or scRNA-seq without single-cell spatial coordinates — the validation plot depends on the spatial coordinate matrix
- The goal is Seq-Scope mouse colon analysis (e.g., GPX1 in immature B cells/fibroblasts) rather than seqFISH+ cortex — use the 2023_01_Seq-Scope.RMD entry point instead
- Input is a pre-computed communication network or feature table with no raw expression/coordinate matrices to reload and filter

## Inputs

- 2023_01_seqFISH+.zip data package (seqFISH+ mouse cortex dataset) from the CPPLS-MLP GitHub repository
- Gene expression matrix for seqFISH+ mouse cortex cells
- Two-dimensional spatial coordinate matrix with 12-type cortex cell type annotations

## Outputs

- Cell-to-cell communication network for seqFISH+ cortex cell types (from 2023_01_seqFISH+.RMD)
- cplx1_spatial_expression.png (or PDF): two-dimensional spatial plot of CPLX1 expression in L5 eNeuron and L6 eNeuron, mirroring Supplementary Figure S7B

## How to apply

Obtain and decompress the 2023_01_seqFISH+.zip package from https://github.com/wuzhenao/CPPLS-MLP and run the 2023_01_seqFISH+.RMD script in R (<= 4.2.3, with cluster, circlize, dplyr, pls, purrr, Seurat, stringr, ComplexHeatmap) to build the cell-to-cell communication network. In Python (>= 3.7, networkx 3.2.1), load the gene expression matrix and spatial coordinate matrix, filter cells to the L5 eNeuron and L6 eNeuron types of the 12-type cortex annotation, and extract CPLX1 expression joined to the two-dimensional coordinates. Render a two-dimensional spatial scatter/heatmap colored by CPLX1 expression for both neuron types, arranged as in Supplementary Figure S7B, and save as cplx1_spatial_expression.png/PDF. The rationale is the article's validation pattern: expression of the communication-related gene should be higher where L5 eNeuron cells lie closer to L6 eNeuron cells, confirming the predicted communication.

## Related tools

- **CPPLS-MLP** (Provides the repository, data package (2023_01_seqFISH+.zip), and 2023_01_seqFISH+.RMD script used to construct the cell-to-cell communication network being validated) — https://github.com/wuzhenao/CPPLS-MLP
- **R** (Runtime (<= 4.2.3) in which 2023_01_seqFISH+.RMD is executed to build the communication network) — https://github.com/wuzhenao/CPPLS-MLP
- **Python** (Runtime (>= 3.7) used to filter L5/L6 eNeuron cells, join CPLX1 expression with spatial coordinates, and render the validation plot) — https://github.com/wuzhenao/CPPLS-MLP
- **Seurat** (R dependency of the CPPLS-MLP pipeline, supplying FindVariableFeatures()-style single-cell preprocessing used in the network workflow) — https://github.com/wuzhenao/CPPLS-MLP

## Examples

```
Rscript -e 'rmarkdown::render("2023_01_seqFISH+.RMD")'
```

## Evaluation signals

- The output cplx1_spatial_expression.png/PDF exists and opens as a valid PNG/PDF
- Visually, CPLX1 expression in L5 eNeuron is higher where L5 eNeuron cells are closer to L6 eNeuron cells, matching Supplementary Figure S7B
- The 2023_01_seqFISH+.RMD script runs to completion in R <= 4.2.3 with the declared dependencies installed
- Filtered cells resolve to exactly the L5 eNeuron and L6 eNeuron types of the 12-type cortex annotation before plotting
- The rendered plot layout (two neuron-type panels colored by CPLX1 expression over 2D coordinates) mirrors the article's Supplementary Figure S7B arrangement

## Limitations

- The README gives no accession IDs or download mirrors for the seqFISH+ package beyond the repository zip; decompression of 2023_01_seqFISH+.zip is required first
- R must be version <= 4.2.3 and Python >= 3.7 with networkx 3.2.1 — newer versions are untested per the README
- Exact per-cell CPLX1 patterns depend on the 12-type cortex annotation; mislabeled L5/L6 eNeuron cells will distort the proximity–expression relationship
- The README offers only email support (2285308398@qq.com) for replication questions; no automated validation of the figure is provided

## Evidence

- [intro] Applied to Seq-scope and seqFISH+ datasets, CPPLS-MLP unveils the regulatory role of intercellular communication across various cell types on HVG expression.: "Applied to Seq-scope and seqFISH+ datasets, CPPLS-MLP unveils the regulatory role of intercellular communication across various cell types on HVG expression."
- [readme] you can decompress the 2023_01_seqFISH+.zip and 2023_01_Seq-Scope.zip packages and run them in R: "you can decompress the 2023_01_seqFISH+.zip and 2023_01_Seq-Scope.zip packages and run them in R"
- [discussion] we visualized the expression levels of the GPX1 gene in both cell types and its spatial distribution. This visualization confirmed the accuracy of our predictions: "we visualized the expression levels of the GPX1 gene in both cell types and its spatial distribution. This visualization confirmed the accuracy of our predictions"
- [intro] dependence: R version <= 4.2.3.: "dependence: R version <= 4.2.3."
