---
name: pseudo-space-projection-of-scatac-cells
description: Use when scATAC-seq cells (e.g., mouse cortex ISSAAC-seq) need spatial
  localization but no spatial ATAC-seq reference exists, and a matched scRNA-seq/ST
  tissue is available.
when_to_use_negative:
- Tissues with low spatial coherence (e.g., SCC Patients 5 and 9), where scSpace should
  be applied cautiously
- No biologically matched spatial transcriptomics reference exists for the scATAC-seq
  tissue, since the choice of spatial reference directly influences the reconstruction
- Spatial domain identification of ST data itself — scSpace targets spatial analysis
  of single-cell data, not ST spatial domain callers like SpaGCN, BayesSpace, DR-SC
edam_operation: http://edamontology.org/operation_3936
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3173
- http://edamontology.org/topic_3697
tools:
- name: scSpace
  role: 'Integrative algorithm: TCA-based feature extraction, MLP pseudo-space projection,
    and space-informed clustering of the gene activity matrix against the ST reference'
  repo: https://github.com/ZJUFanLab/scSpace
- name: Signac
  role: Converts scATAC-seq data into a gene activity matrix before pseudo-space reconstruction
- name: Scanpy
  role: Single-cell preprocessing/analysis environment within which the scSpace workflow
    is executed
  repo: https://github.com/scverse/scanpy
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-023-38121-4
    title: Reconstruction of the cell pseudo-space from single-cell RNA sequencing
      data with scSpace
  source_content_hash: sha256:099b6873a3520e4c4201c6a7f35b68a807d78c7c9a16d382faa3347eefe04444
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ZJUFanLab/scSpace
  tools:
  - Python
  - Scanpy
  - Signac
  - scSpace
  edam_operation: http://edamontology.org/operation_3936
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3173
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# pseudo-space-projection-of-scatac-cells

## Summary

Extends scSpace pseudo-space reconstruction to scATAC-seq data by converting fragments into a gene activity matrix with Signac and using a matched ST dataset (Visium-style) as the spatial reference, recovering layered cortical architecture without spatial ATAC-seq data.

## When to use

Use when scATAC-seq cells (e.g., mouse cortex ISSAAC-seq) need spatial localization but no spatial ATAC-seq reference exists, and a matched scRNA-seq/ST tissue is available.

## When NOT to use

- Tissues with low spatial coherence (e.g., SCC Patients 5 and 9), where scSpace should be applied cautiously
- No biologically matched spatial transcriptomics reference exists for the scATAC-seq tissue, since the choice of spatial reference directly influences the reconstruction
- Spatial domain identification of ST data itself — scSpace targets spatial analysis of single-cell data, not ST spatial domain callers like SpaGCN, BayesSpace, DR-SC

## Inputs

- scATAC-seq data (e.g., mouse cortex ISSAAC-seq fragments/cells)
- Spatial transcriptomics (ST) dataset from the same or matched tissue (e.g., mouse cortex ST reference)
- Python >=3.8 environment with scSpace installed (numpy 1.23.4, scanpy 1.9.1, torch)

## Outputs

- Gene activity matrix (gene-by-cell) derived from scATAC-seq via Signac
- Pseudo-space coordinates of projected scATAC-seq cells
- Space-informed clustering labels identifying spatially stratified cell subpopulations
- Figure such as supplementary_fig31_scatac_pseudospace.png showing layered cortical spatial distribution

## How to apply

First convert the scATAC-seq fragment data into a gene-by-cell gene activity matrix using Signac with default parameters, since scSpace operates on gene expression-like matrices. Load the ST data from the same or a matched tissue as the spatial reference, then run scSpace: transfer component analysis (TCA) eliminates batch effects and extracts shared characteristic matrices of the gene activity matrix and ST expression data. Project the scATAC-seq cells into the pseudo space via scSpace's multiple layer perceptron (MLP) model. Finally, jointly embed gene expression and spatial weights and perform space-informed clustering to identify spatially stratified subpopulations; visualize in pseudo space and confirm recovery of the expected tissue architecture (e.g., cortical layers).

## Related tools

- **scSpace** (Integrative algorithm: TCA-based feature extraction, MLP pseudo-space projection, and space-informed clustering of the gene activity matrix against the ST reference) — https://github.com/ZJUFanLab/scSpace
- **Signac** (Converts scATAC-seq data into a gene activity matrix before pseudo-space reconstruction)
- **Scanpy** (Single-cell preprocessing/analysis environment within which the scSpace workflow is executed) — https://github.com/scverse/scanpy

## Examples

```
import pandas as pd; from scSpace.core import run; cell_coords = run(gene_activity_csv, st_coords_csv, st_expr_csv)
```

## Evaluation signals

- Projected scATAC-seq cells reproduce the layered cortical spatial distribution reported in Supplementary Fig. 31
- Space-informed clustering yields spatially stratified subpopulations consistent with known cortex anatomy
- Gene activity matrix is gene-by-cell (same gene symbols as the ST reference) so TCA can extract shared characteristic matrices
- Batch effect between gene activity matrix and ST data is visibly reduced after TCA, per scSpace's shared-latent-feature goal

## Limitations

- Because appropriate spatial ATAC-seq data is not available, the ST dataset is used as a proxy reference, and the choice of spatial reference directly influences the spatial reconstruction
- Intrinsic difference in spatial homogeneity between tissues may influence the accuracy of scSpace
- An appropriateness assessment or pre-processing of the spatial reference should be considered before applying scSpace (e.g., dACC is agranular and lacks cortical layer 4)

## Evidence

- [discussion] Signac gene activity conversion: "we chose the spatial reference with the ST data and convert the scATAC-seq data into a gene activity matrix using Signac82 in advance"
- [methods] TCA and MLP pipeline: "Using [transfer component analysis (TCA)](https://ieeexplore.ieee.org/stamp/stamp.jsp?tp=&arnumber=5640675&tag=1), scSpace could extract the characteristic matrixes of spatial transcriptomics and"
- [methods] joint embedding rationale: "so that gene expression and spatial weight of cells can be embedded jointly for the further cell typing with higher accuracy and precision"
- [discussion] spatial reference caveat: "performing an appropriateness assessment or necessary pre-processing steps of the spatial reference should be considered before applying scSpace"
- [readme] Python environment: "[![python >=3.8](https://img.shields.io/badge/python-%3E%3D3.8-brightgreen)](https://www.python.org/) [![DOI](https://zenodo.org/badge/DOI/10.5281/zenodo.7790754.svg)]"
