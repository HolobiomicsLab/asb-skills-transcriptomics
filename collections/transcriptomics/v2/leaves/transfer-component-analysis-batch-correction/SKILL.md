---
name: transfer-component-analysis-batch-correction
description: Use when integrating scRNA-seq or scATAC-seq-derived gene activity data
  with an ST reference (Visium, Slide-seq, STARmap) whose platforms and batch effects
  prevent direct joint analysis for pseudo-space reconstruction.
when_to_use_negative:
- The input data is already from the same platform and batch (no cross-modality/intersample
  batch effect to remove between scRNA-seq and ST)
- The target tissue has low spatial coherence (e.g., Patients 5 and 9 of SCC in the
  paper), where scSpace reconstruction including TCA alignment should be applied cautiously
- The available spatial reference is anatomically mismatched to the target tissue
  (e.g., dACC, which lacks cortical layer 4, for a DLPFC target) without appropriateness
  assessment or pre-processing
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2229
tools:
- name: scSpace
  role: Implements the transfer component analysis (TCA) step that eliminates batch
    effects and extracts shared characteristic matrices between scRNA-seq and ST data
  repo: github.com/ZJUFanLab/scSpace
- name: scanpy
  role: Normalizes and preprocesses both expression matrices before TCA alignment
  repo: scverse/scanpy
- name: Signac
  role: Converts scATAC-seq data into a gene activity matrix so it can enter TCA alignment
    against an ST reference
- name: PyTorch
  role: Dependency supporting the scSpace MLP model trained on the TCA-aligned features
    (Python >=3.8 environment)
provenance:
  source_task_ids:
  - task_002
  - task_003
  - task_004
  - task_005
  - task_008
  source_papers:
  - doi: 10.1038/s41467-023-38121-4
    title: Reconstruction of the cell pseudo-space from single-cell RNA sequencing
      data with scSpace
  source_content_hash: sha256:cc057060a6cc10043ef7c6873530e4eb079ca7a563677ebfdc3e35a1000c9874
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - PyTorch
  - Python
  - Python >=3.8
  - Signac
  - Slide-seq
  - Visium
  - igraph
  - leidenalg
  - matplotlib
  - pandas
  - scSpace
  - scanpy
  - scikit-learn
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9411764705882353
n_claims: 17
---

# transfer-component-analysis-batch-correction

## Summary

Apply transfer component analysis (TCA), as implemented in scSpace, to eliminate the batch effect between scRNA-seq (or gene-activity-matrix) data and a spatial transcriptomics reference and extract their shared characteristic matrices, which are then used to train an MLP projector into pseudo-space.

## When to use

Use when integrating scRNA-seq or scATAC-seq-derived gene activity data with an ST reference (Visium, Slide-seq, STARmap) whose platforms and batch effects prevent direct joint analysis for pseudo-space reconstruction.

## When NOT to use

- The input data is already from the same platform and batch (no cross-modality/intersample batch effect to remove between scRNA-seq and ST)
- The target tissue has low spatial coherence (e.g., Patients 5 and 9 of SCC in the paper), where scSpace reconstruction including TCA alignment should be applied cautiously
- The available spatial reference is anatomically mismatched to the target tissue (e.g., dACC, which lacks cortical layer 4, for a DLPFC target) without appropriateness assessment or pre-processing

## Inputs

- Spatial transcriptomics reference count matrix with spatial coordinates (Visium/Slide-seq/STARmap), as a formatted .csv readable by pandas
- Target scRNA-seq count matrix (e.g., control and COVID-19 lung, embryonic heart, melanoma T cells), as a formatted .csv
- Optional: scATAC-seq data pre-converted to a gene-by-cell gene activity matrix with Signac
- Preprocessed/normalized expression matrices prepared with scanpy 1.9.1

## Outputs

- Shared characteristic matrices of the spatial transcriptomics and single-cell modalities after batch-effect removal
- Aligned feature space consumed by the scSpace MLP pseudo-space projector

## How to apply

Load the spatial reference and target single-cell count matrices as formatted .csv files with pandas (scSpace requires five input files, e.g. as in the DLPFC slice 151674 vignette). Normalize and preprocess both matrices with scanpy (scanpy 1.9.1), harmonizing genes across modalities; for scATAC-seq input, first convert to a gene-by-cell gene activity matrix with Signac. Run scSpace's TCA step to remove the batch effect between the ST reference and the single-cell data and extract the shared characteristic matrices of both modalities — this alignment is what makes cross-platform projection possible. Decide whether the ST reference is appropriate for the target tissue (e.g., DLPFC vs agranular dACC lacks cortical layer 4) and preprocess or swap the reference before TCA, since the choice of reference directly influences the reconstruction. The TCA outputs feed the scSpace MLP model (PyTorch) that projects target cells into pseudo-space coordinates.

## Related tools

- **scSpace** (Implements the transfer component analysis (TCA) step that eliminates batch effects and extracts shared characteristic matrices between scRNA-seq and ST data) — github.com/ZJUFanLab/scSpace
- **scanpy** (Normalizes and preprocesses both expression matrices before TCA alignment) — scverse/scanpy
- **Signac** (Converts scATAC-seq data into a gene activity matrix so it can enter TCA alignment against an ST reference)
- **PyTorch** (Dependency supporting the scSpace MLP model trained on the TCA-aligned features (Python >=3.8 environment))

## Examples

```
conda create -n scspace python=3.8; conda activate scspace; cd scSpace-master; pip install -r requirements.txt; python setup.py install
```

## Evaluation signals

- The shared characteristic matrices are produced for both modalities with genes harmonized across the scRNA-seq and ST inputs
- Downstream MLP projection of target cells yields pseudo-space coordinates with significant positive pairwise-distance correlation to the reference spatial positions
- On human DLPFC data, the reconstructed pseudo-space preserves cortical layer ordering and layer-specific marker-gene expression patterns consistent with Fig. 2a-d
- On embryonic heart data (14 clusters), space-informed clustering after TCA achieves cardiomyocyte subpopulation separation of ARI 0.91 / NMI 0.85 (vs Seurat ARI 0.16 / NMI 0.18)
- Batch effect between modalities is reduced so jointly embedded gene expression and spatial weight allow Leiden clustering that resolves spatially variated subpopulations (e.g., melanoma C1–C5)

## Limitations

- The choice of spatial reference directly influences the spatial reconstruction; an appropriateness assessment or pre-processing of the reference should be performed before applying scSpace
- Intrinsic differences in spatial homogeneity between tissues may influence accuracy; apply cautiously to tissues with low spatial coherence (e.g., SCC Patients 5 and 9)
- Because appropriate spatial ATAC-seq data is not available, scATAC-seq must be converted to a gene activity matrix and an ST reference used instead, losing assay-specific signal

## Evidence

- [intro] TCA eliminates batch effect and extracts shared features: [evidence span withheld — anchor status: not_found]
- [methods] Methods description of TCA: "Using [transfer component analysis (TCA)](https://ieeexplore.ieee.org/stamp/stamp.jsp?tp=&arnumber=5640675&tag=1), scSpace could extract the characteristic matrixes of spatial transcriptomics and"
- [readme] Readme requirements: "To use scSpace we require five formatted `.csv` files as input (i.e. read in by pandas). We have included a toy dataset"
- [discussion] Reference choice caveat: "Since the choice of spatial reference would directly influence the spatial reconstruction"
- [discussion] ATAC conversion requirement: "Because the appropriate spatial ATAC-seq data is not available, we chose the spatial reference with the ST data and convert the scATAC-seq data into a gene activity matrix using Signac82 in advance"
