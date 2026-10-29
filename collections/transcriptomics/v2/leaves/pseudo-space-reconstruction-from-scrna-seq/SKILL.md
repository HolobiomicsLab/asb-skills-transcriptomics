---
name: pseudo-space-reconstruction-from-scrna-seq
description: Use when you have a scRNA-seq count matrix and a matched spatial transcriptomics
  reference from the same tissue type, and need to identify spatially heterogeneous
  cell subpopulations or pairwise cellular spatial associations from single-cell-only
  data.
when_to_use_negative:
- Tissues with low spatial coherence (e.g., Patients 5 and 9 of SCC in the paper)
  — scSpace should be applied cautiously there
- The goal is spatial domain identification of ST data itself — scSpace targets spatial
  analysis of scRNA-seq data, unlike SpaGCN, BayesSpace, or DR-SC
- No appropriate spatial reference is available for the modality — e.g., no spatial
  ATAC-seq data existed for the scATAC-seq pseudo-space reconstruction attempt
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2229
tools:
- name: scSpace
  role: 'Core algorithm: TCA-based batch-effect removal and shared feature extraction,
    MLP-based projection of cells into pseudo-space, and space-informed clustering'
  repo: https://github.com/ZJUFanLab/scSpace
- name: scanpy
  role: Nearest-neighbor graph for Leiden clustering and gene-signature scoring (e.g.,
    T-cell exhaustion scores) on pseudo-space clusters
  repo: https://github.com/scverse/scanpy
- name: PyTorch
  role: Training backend for the scSpace multiple-layer perceptron model
  repo: https://pytorch.org/get-started/locally/
- name: leidenalg
  role: Leiden community detection used for space-informed clustering of T cells into
    subpopulations
- name: matplotlib
  role: Visualization of cluster-level exhaustion scores and proximity comparisons
    (Fig. 5c–f panels)
- name: pandas
  role: Loading the formatted .csv count matrices and ST reference data
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-023-38121-4
    title: Reconstruction of the cell pseudo-space from single-cell RNA sequencing
      data with scSpace
  source_content_hash: sha256:d743483013e50eb8fee3bea1ba20047ba7c2cb1993a57fe4c91cacd69b0deec2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ZJUFanLab/scSpace
  tools:
  - PyTorch
  - igraph
  - leidenalg
  - matplotlib
  - pandas
  - scSpace
  - scanpy
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8823529411764706
n_claims: 17
---

# pseudo-space-reconstruction-from-scrna-seq

## Summary

Reconstructs pseudo-space coordinates for scRNA-seq cells by using spatial transcriptomics (Visium, STARmap, Slide-seq) as a spatial reference with scSpace, then performs space-informed clustering to identify spatially variable cell subpopulations.

## When to use

Use when you have a scRNA-seq count matrix and a matched spatial transcriptomics reference from the same tissue type, and need to identify spatially heterogeneous cell subpopulations or pairwise cellular spatial associations from single-cell-only data.

## When NOT to use

- Tissues with low spatial coherence (e.g., Patients 5 and 9 of SCC in the paper) — scSpace should be applied cautiously there
- The goal is spatial domain identification of ST data itself — scSpace targets spatial analysis of scRNA-seq data, unlike SpaGCN, BayesSpace, or DR-SC
- No appropriate spatial reference is available for the modality — e.g., no spatial ATAC-seq data existed for the scATAC-seq pseudo-space reconstruction attempt

## Inputs

- Preprocessed scRNA-seq count matrix (.csv, e.g., 2064 melanoma T cells)
- Spatial transcriptomics reference dataset (.csv, e.g., melanoma ST data)
- Formatted .csv input files (five required by scSpace, read in by pandas)

## Outputs

- Pseudo-space coordinates for every single cell
- Space-informed Leiden cluster labels for all cells (e.g., five subpopulations C1–C5)
- Per-cell gene-signature scores (e.g., T-cell exhaustion score via scanpy)
- Comparison of signature scores and spatial proximity between subpopulations (e.g., C5 vs C3, Fig. 5c–f)

## How to apply

Load the preprocessed scRNA-seq count matrix and the ST reference as five formatted .csv files with pandas (e.g., the melanoma T-cell matrix with 2064 T cells and the melanoma ST reference from the Google Drive tutorial package). Apply scSpace's transfer component analysis (TCA) to eliminate the batch effect between the scRNA-seq and ST modalities and extract shared characteristic matrices, then train the scSpace multiple-layer perceptron (MLP) model with PyTorch to project single cells into pseudo-space coordinates. Jointly embed gene expression and spatial weights and run space-informed Leiden clustering (leidenalg/igraph via scanpy neighbors) to partition cells into subpopulations (e.g., C1–C5). Downstream, score cells for signatures such as T-cell exhaustion with scanpy and compare scores and proximity to malignant cells across clusters to identify tumor-infiltrating identities.

## Related tools

- **scSpace** (Core algorithm: TCA-based batch-effect removal and shared feature extraction, MLP-based projection of cells into pseudo-space, and space-informed clustering) — https://github.com/ZJUFanLab/scSpace
- **scanpy** (Nearest-neighbor graph for Leiden clustering and gene-signature scoring (e.g., T-cell exhaustion scores) on pseudo-space clusters) — https://github.com/scverse/scanpy
- **PyTorch** (Training backend for the scSpace multiple-layer perceptron model) — https://pytorch.org/get-started/locally/
- **leidenalg** (Leiden community detection used for space-informed clustering of T cells into subpopulations)
- **matplotlib** (Visualization of cluster-level exhaustion scores and proximity comparisons (Fig. 5c–f panels))
- **pandas** (Loading the formatted .csv count matrices and ST reference data)

## Examples

```
conda create -n scspace python=3.8; conda activate scspace; pip install -r requirements.txt; python setup.py install
```

## Evaluation signals

- Output table contains pseudo-space coordinates and cluster labels for all 2064 T cells with five-cluster labels (C1–C5)
- C5 subpopulation shows higher T-cell exhaustion score than the distant C3 subpopulation, matching Fig. 5c–d
- C5 shows higher proximity to malignant cells in the pseudo-space than C3, matching Fig. 5e–f
- Performance on spatial variability identification is conserved when spatial references come from different biological replicates (melanoma replicate check)
- TCA output yields comparable shared characteristic matrices across the scRNA-seq and ST modalities (batch effect eliminated)

## Limitations

- Intrinsic difference in spatial homogeneity between tissues may influence the accuracy of scSpace
- Choice of spatial reference directly influences the spatial reconstruction; an appropriateness assessment or pre-processing of the reference should be considered (e.g., DLPFC vs agranular dACC lacking cortical layer 4)
- MERFISH and seqFISH fail to detect gene variations that directly reflect gene function; FISSEQ and ISS sequence very short transcript segments, leading to bias

## Evidence

- [intro] TCA eliminates batch effect between modalities: [evidence span withheld — anchor status: not_found]
- [methods] TCA + MLP workflow: [evidence span withheld — anchor status: not_found]
- [methods] Space-informed cell typing: "so that gene expression and spatial weight of cells can be embedded jointly for the further cell typing with higher accuracy and precision"
- [readme] Tutorial data and melanoma vignette: "Additional step-by-step tutorials now available! Preprocessed datasets used can be downloaded from [Google Drive]"
- [discussion] Low spatial coherence caution: "scSpace should be applied cautiously to tissues with low spatial coherence, such as Patients 5 and 9 of SCC evaluated in this paper"
