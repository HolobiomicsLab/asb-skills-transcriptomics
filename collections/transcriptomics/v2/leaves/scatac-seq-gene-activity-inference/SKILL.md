---
name: scatac-seq-gene-activity-inference
description: Use when the input is scATAC-seq data and pseudo-space reconstruction
  is needed but no spatial ATAC-seq reference is available, requiring conversion to
  a gene activity matrix (via Signac) so ST data can serve as the spatial reference
  in scSpace.
when_to_use_negative:
- Input is already a gene-expression matrix (scRNA-seq or ST) — no gene activity conversion
  via Signac is needed before scSpace
- An appropriate spatial ATAC-seq reference exists for the tissue — direct spatial
  ATAC-seq referencing is preferable to substituting an ST reference
- The tissue has low spatial coherence or weak correspondence with the chosen ST reference
  — scSpace should be applied cautiously in such cases
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_3173
- http://edamontology.org/topic_0203
tools:
- name: Signac
  role: Converts the scATAC-seq data into a gene activity matrix with default parameters
    prior to pseudo-space reconstruction
- name: scSpace
  role: Consumes the gene activity matrix, uses TCA to align it with the ST spatial
    reference, and projects cells into pseudo space via an MLP model
  repo: https://github.com/ZJUFanLab/scSpace
- name: scanpy
  role: Python single-cell analysis toolkit within the scSpace environment (Python
    >=3.8) used for handling gene-by-cell matrices and downstream analysis
  repo: https://github.com/scverse/scanpy
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-023-38121-4
    title: Reconstruction of the cell pseudo-space from single-cell RNA sequencing
      data with scSpace
  source_content_hash: sha256:15409598b8ace19e4f7d6b124988dd893425e7ff796190af7ecf93aee13a2900
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ZJUFanLab/scSpace
  tools:
  - Python
  - Signac
  - scSpace
  - scanpy
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_3173
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# scatac-seq-gene-activity-inference

## Summary

Converts scATAC-seq data (e.g., mouse cortex ISSAAC-seq) into a gene-by-cell gene activity matrix using Signac so that the data becomes compatible with scSpace, which uses spatial transcriptomics (ST) data as a reference to reconstruct the pseudo-space of single cells.

## When to use

Use when the input is scATAC-seq data and pseudo-space reconstruction is needed but no spatial ATAC-seq reference is available, requiring conversion to a gene activity matrix (via Signac) so ST data can serve as the spatial reference in scSpace.

## When NOT to use

- Input is already a gene-expression matrix (scRNA-seq or ST) — no gene activity conversion via Signac is needed before scSpace
- An appropriate spatial ATAC-seq reference exists for the tissue — direct spatial ATAC-seq referencing is preferable to substituting an ST reference
- The tissue has low spatial coherence or weak correspondence with the chosen ST reference — scSpace should be applied cautiously in such cases

## Inputs

- scATAC-seq peak/cell count matrix (e.g., mouse cortex ISSAAC-seq data)
- Fragments file / peak annotations needed by Signac to compute the gene activity matrix
- Spatial transcriptomics (ST) reference dataset (e.g., mouse cortex Visium) used by scSpace as the spatial reference

## Outputs

- Gene-by-cell gene activity matrix derived from the scATAC-seq data via Signac
- scSpace pseudo-space coordinates for each scATAC-seq cell (MLP projection)
- Space-informed clustering labels identifying spatially stratified cell subpopulations
- Visualization of the reconstructed pseudo-space (e.g., supplementary_fig31_scatac_pseudospace.png)

## How to apply

Load the scATAC-seq data (e.g., mouse cortex ISSAAC-seq) and the ST spatial reference dataset (e.g., mouse cortex Visium) into Python (>=3.8). Convert the scATAC-seq data into a gene-by-cell gene activity matrix using Signac with default parameters — the rationale is that scSpace operates on gene expression matrices, so the peak matrix must first be collapsed into gene-space features. Run scSpace with the ST data as the spatial reference: transfer component analysis (TCA) extracts the shared characteristic matrices of the gene activity matrix and the ST expression data while eliminating the batch effect between them. Project the scATAC-seq cells into pseudo space with scSpace's multi-layer perceptron (MLP) model, then jointly embed gene expression and spatial weights and perform space-informed clustering to identify spatially stratified subpopulations. In the source application, this recovered the layered spatial distribution of mouse cortex cells (Supplementary Fig. 31).

## Related tools

- **Signac** (Converts the scATAC-seq data into a gene activity matrix with default parameters prior to pseudo-space reconstruction)
- **scSpace** (Consumes the gene activity matrix, uses TCA to align it with the ST spatial reference, and projects cells into pseudo space via an MLP model) — https://github.com/ZJUFanLab/scSpace
- **scanpy** (Python single-cell analysis toolkit within the scSpace environment (Python >=3.8) used for handling gene-by-cell matrices and downstream analysis) — https://github.com/scverse/scanpy

## Examples

```
conda create -n scspace python=3.8; conda activate scspace; pip install -r requirements.txt; python setup.py install
```

## Evaluation signals

- The intermediate object must be a gene-by-cell (not peak-by-cell) matrix after Signac conversion, with default parameters
- TCA should remove the batch effect so that the gene activity matrix and ST expression data share a comparable latent feature space
- The MLP projection should place scATAC-seq cells into the ST-derived pseudo space consistent with known tissue architecture
- In the mouse cortex case, the reconstruction should recover the layered cortical spatial distribution of cells as in Supplementary Fig. 31
- The scSpace environment must be Python >=3.8 (e.g., 'conda create -n scspace python=3.8') with the documented dependency versions installed

## Limitations

- Because appropriate spatial ATAC-seq data was not available, the ST data served as a surrogate spatial reference; the choice of spatial reference directly influences the spatial reconstruction
- An appropriateness assessment or pre-processing of the spatial reference should be performed first, since spatial architectures differ across regions of the same tissue (e.g., dACC is agranular and lacks cortical layer 4)
- Intrinsic differences in spatial homogeneity between tissues may influence scSpace accuracy; apply cautiously to tissues with low spatial coherence (e.g., Patients 5 and 9 of SCC)

## Evidence

- [discussion] Signac converts scATAC-seq to gene activity matrix before reconstruction: "we chose the spatial reference with the ST data and convert the scATAC-seq data into a gene activity matrix using Signac82 in advance"
- [intro] TCA extracts shared features and eliminates batch effect: "scSpace enables eliminating the batch effect between single-cell and ST data and extracting the shared latent feature across these two types of data"
- [readme] MLP projection into pseudo space (README): "project single cells into a pseudo space via a [multiple layer perceptron (MLP)](https://en.wikipedia.org/wiki/Multilayer_perceptron) model"
- [discussion] Assess the spatial reference before applying scSpace: "performing an appropriateness assessment or necessary pre-processing steps of the spatial reference should be considered before applying scSpace"
- [readme] Python version requirement (README): "For scSpace, the python version need is over 3.8."
