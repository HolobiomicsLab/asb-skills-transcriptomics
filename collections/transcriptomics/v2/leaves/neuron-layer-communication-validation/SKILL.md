---
name: neuron-layer-communication-validation
description: Use when a CPPLS-MLP cell–cell communication prediction on the seqFISH+
  mouse cortex dataset links two cortical layer cell types (e.g., L5 eNeuron → L6
  eNeuron) and a spatial expression visualization of a target gene is needed to confirm
  the predicted communication.
when_to_use_negative:
- The communication prediction involves non-neuronal or non-cortex cell types (e.g.,
  Seq-Scope mouse colon immature B cells vs fibroblasts, where GPX1 is the target
  gene instead of CPLX1).
- Only bulk or dissociated single-cell data without spatial coordinates is available
  — the validation depends on 2D spatial positions of individual cells.
- The gene of interest was not classified by CPPLS-MLP as involved in cell communication
  (use the search_gene criterion script instead of spatial validation).
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: CPPLS-MLP
  role: Source repository providing the seqFISH+ data package and the 2023_01_seqFISH+.RMD
    script that builds the L5/L6 eNeuron communication network being validated
  repo: https://github.com/wuzhenao/CPPLS-MLP
- name: Python
  role: Python (>= 3.7) environment with networkx 3.2.1 and pandas 2.1.0 used to filter
    L5/L6 eNeurons, join CPLX1 expression with spatial coordinates, and render the
    spatial plot
  repo: https://github.com/wuzhenao/CPPLS-MLP
- name: Seurat
  role: R dependency (installed alongside pls, circlize, ComplexHeatmap) used in the
    2023_01_seqFISH+.RMD script that produces the communication network upstream of
    the visualization
  repo: https://github.com/wuzhenao/CPPLS-MLP
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bib/bbae198
    title: 'CPPLS-MLP: a method for constructing cell–cell communication networks
      and identifying related highly variable genes based on single-cell sequencing
      and spatial transcriptomics data'
  source_content_hash: sha256:3d9607b6284933404f525e2125c27f24891156b460e8d62be395defc07a86029
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/wuzhenao/CPPLS-MLP
  tools:
  - CPPLS-MLP
  - Python
  - Seurat
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# neuron-layer-communication-validation

## Summary

Validates predicted cortical layer-to-layer neuron communication by visualizing the spatial expression of a communication-associated gene (e.g., CPLX1) in L5 and L6 eNeurons from the seqFISH+ mouse cortex dataset, mirroring the article's Supplementary Figure S7B.

## When to use

Use when a CPPLS-MLP cell–cell communication prediction on the seqFISH+ mouse cortex dataset links two cortical layer cell types (e.g., L5 eNeuron → L6 eNeuron) and a spatial expression visualization of a target gene is needed to confirm the predicted communication.

## When NOT to use

- The communication prediction involves non-neuronal or non-cortex cell types (e.g., Seq-Scope mouse colon immature B cells vs fibroblasts, where GPX1 is the target gene instead of CPLX1).
- Only bulk or dissociated single-cell data without spatial coordinates is available — the validation depends on 2D spatial positions of individual cells.
- The gene of interest was not classified by CPPLS-MLP as involved in cell communication (use the search_gene criterion script instead of spatial validation).

## Inputs

- seqFISH+ mouse cortex gene expression matrix (from 2023_01_seqFISH+.zip in https://github.com/wuzhenao/CPPLS-MLP)
- seqFISH+ spatial coordinate matrix with 2D cell positions
- 12-type cortex cell-type annotation (L5 eNeuron, L6 eNeuron, etc.)
- CPPLS-MLP cell-to-cell communication network output (from 2023_01_seqFISH+.RMD)

## Outputs

- 2D spatial scatter/heatmap of CPLX1 expression for L5 eNeuron and L6 eNeuron, mirroring Supplementary Figure S7B
- cplx1_spatial_expression.png (or PDF) validation figure
- Qualitative judgment on whether higher CPLX1 expression coincides with L5 eNeuron proximity to L6 eNeuron

## How to apply

Decompress 2023_01_seqFISH+.zip from the CPPLS-MLP repository and run 2023_01_seqFISH+.RMD in R (≤ 4.2.3, with Seurat, pls, circlize, ComplexHeatmap) to build the cell-to-cell communication network, or consume its output directly. In Python (>= 3.7, networkx 3.2.1, pandas 2.1.0), load the gene expression matrix and spatial coordinate matrix and filter cells to the L5 eNeuron and L6 eNeuron types from the 12-type cortex annotation. Extract CPLX1 expression values and join them with the 2D spatial coordinates for both neuron types, then produce a 2D spatial scatter/heatmap colored by CPLX1 level. The rationale follows the article's validation logic: a gene predicted as communication-associated should show elevated expression in the sender cell type specifically where it is spatially proximal to the receiver type, as visualized for GPX1 in the Seq-Scope case.

## Related tools

- **CPPLS-MLP** (Source repository providing the seqFISH+ data package and the 2023_01_seqFISH+.RMD script that builds the L5/L6 eNeuron communication network being validated) — https://github.com/wuzhenao/CPPLS-MLP
- **Python** (Python (>= 3.7) environment with networkx 3.2.1 and pandas 2.1.0 used to filter L5/L6 eNeurons, join CPLX1 expression with spatial coordinates, and render the spatial plot) — https://github.com/wuzhenao/CPPLS-MLP
- **Seurat** (R dependency (installed alongside pls, circlize, ComplexHeatmap) used in the 2023_01_seqFISH+.RMD script that produces the communication network upstream of the visualization) — https://github.com/wuzhenao/CPPLS-MLP

## Examples

```
unzip 2023_01_seqFISH+.zip && Rscript -e 'rmarkdown::render("2023_01_seqFISH+.RMD")'; python -c "import pandas as pd; df=df[df.celltype.isin(['L5 eNeuron','L6 eNeuron'])]; plt.scatter(df.x,df.y,c=df.CPLX1)"
```

## Evaluation signals

- Output image cplx1_spatial_expression.png exists and is a valid PNG/PDF file.
- Plot contains exactly the two filtered cell types (L5 eNeuron and L6 eNeuron) from the 12-type cortex annotation, each colored by CPLX1 expression level.
- Visual pattern matches Supplementary Figure S7B: CPLX1 expression in L5 eNeuron is higher where L5 eNeuron cells are closer to L6 eNeuron cells.
- Upstream network was reproduced by running the README-specified entry point 2023_01_seqFISH+.RMD after decompressing 2023_01_seqFISH+.zip.
- Environment matches README constraints: R <= 4.2.3, python >= 3.7, networkx == 3.2.1, pandas 2.1.0.

## Limitations

- Article supplies no accession IDs or download sources for the seqFISH+ dataset beyond the repository zip, and no version numbers for the datasets themselves.
- The README requires manually decompressing 2023_01_seqFISH+.zip and running the RMD script in R; no scripted end-to-end pipeline is provided for the downstream Python visualization.
- Validation of the spatial expression pattern is visual (comparison against Supplementary Figure S7B), not a quantitative metric.
- The article's discussion notes only two benchmark datasets (seqFISH+ mouse cortex, Seq-Scope mouse colon) were used, so transferability to other tissues/layers is untested.

## Evidence

- [readme] decompress 2023_01_seqFISH+ then run in R: "you can decompress the 2023_01_seqFISH+.zip and 2023_01_Seq-Scope.zip packages and run them in R"
- [discussion] GPX1 spatial visualization as validation precedent: "we visualized the expression levels of the GPX1 gene in both cell types and its spatial distribution. This visualization confirmed the accuracy of our predictions"
- [methods] Dataset scope: "the seqFISH+ mouse cortex dataset and the Seq-Scope mouse colon dataset"
- [readme] Python version constraint: "dependence: python version >= 3.7"
- [intro] CPPLS-MLP combines expression and coordinates: "CPPLS-MLP combines gene expression data and cell spatial coordinates, modeling the impact of intercellular communication among multiple cell types on HVGs"
