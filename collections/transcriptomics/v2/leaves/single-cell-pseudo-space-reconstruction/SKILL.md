---
name: single-cell-pseudo-space-reconstruction
description: Use when you have paired scRNA-seq and ST data (Visium, STARmap, Slide-seq)
  and need to map single cells into spatial coordinates or identify spatially variated
  cell subpopulations that traditional single-cell pipelines miss.
when_to_use_negative:
- Tissues with low spatial coherence, such as Patients 5 and 9 of SCC evaluated in
  the paper — the paper states scSpace should be applied cautiously there
- No appropriate spatial reference is available for the tissue (e.g. spatial ATAC-seq
  data was unavailable for the scATAC-seq attempt in the paper)
- Spatial domain identification of the ST data itself is the goal — scSpace targets
  spatial analysis of scRNA-seq data, unlike SpaGCN/BayesSpace/DR-SC which target
  ST spatial domains
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3934
- http://edamontology.org/topic_3170
tools:
- name: scSpace
  role: 'Core algorithm: TCA feature extraction, MLP pseudo-space projection, and
    space-informed clustering of scRNA-seq cells'
  repo: https://github.com/ZJUFanLab/scSpace
- name: PyTorch
  role: Implements the two-layer MLP (128-neuron first layer) trained with Adam (lr
    1e-3) and MSE loss to regress spatial coordinates
- name: scanpy
  role: Joint embedding of gene expression and spatial weight followed by Leiden space-informed
    clustering
  repo: https://github.com/scverse/scanpy
- name: Python >=3.8
  role: Runtime environment; installed via conda (python=3.8) with numpy, pandas,
    scikit-learn, scipy, igraph, leidenalg dependencies
  repo: https://github.com/ZJUFanLab/scSpace
provenance:
  source_task_ids:
  - task_003
  - task_007
  source_papers:
  - doi: 10.1038/s41467-023-38121-4
    title: Reconstruction of the cell pseudo-space from single-cell RNA sequencing
      data with scSpace
  source_content_hash: sha256:74b456b099929c1796b96d469a1ccbc4fadf78c828f875b43a8a95b035fe8521
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ZJUFanLab/scSpace
  tools:
  - PyTorch
  - Python
  - Python >=3.8
  - Slide-seq
  - Visium
  - scSpace
  - scanpy
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3934
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9523809523809523
n_claims: 21
---

# single-cell-pseudo-space-reconstruction

## Summary

Reconstruct spatial associations of single cells within scRNA-seq data by integrating a spatial transcriptomics (ST) reference: scSpace uses transfer component analysis (TCA) to extract shared latent features, an MLP to project cells into a pseudo-space, and space-informed clustering to identify spatially heterogeneous subpopulations.

## When to use

Use when you have paired scRNA-seq and ST data (Visium, STARmap, Slide-seq) and need to map single cells into spatial coordinates or identify spatially variated cell subpopulations that traditional single-cell pipelines miss.

## When NOT to use

- Tissues with low spatial coherence, such as Patients 5 and 9 of SCC evaluated in the paper — the paper states scSpace should be applied cautiously there
- No appropriate spatial reference is available for the tissue (e.g. spatial ATAC-seq data was unavailable for the scATAC-seq attempt in the paper)
- Spatial domain identification of the ST data itself is the goal — scSpace targets spatial analysis of scRNA-seq data, unlike SpaGCN/BayesSpace/DR-SC which target ST spatial domains

## Inputs

- scRNA-seq count matrix (.csv, read by pandas)
- Spatial transcriptomics reference count matrix (.csv)
- ST spot x-y spatial coordinate matrix (.csv)
- Five formatted .csv input files total, as required by scSpace

## Outputs

- Per-cell pseudo-space coordinates (pseudo_space_coordinates.csv, one finite x,y row per scRNA-seq cell)
- Space-informed Leiden cluster assignments labeling spatially variated subpopulations
- ARI and NMI metrics (ari_nmi_metrics.csv) comparing subpopulation assignments

## How to apply

Read five formatted .csv files with pandas: scRNA-seq count matrix, ST count matrix, and ST spot x-y coordinates. First run transfer component analysis (TCA) to eliminate the batch effect between the scRNA-seq and ST data and extract their shared characteristic (latent) matrices. Then train a two-layer fully connected MLP in PyTorch (first layer 128 neurons, Adam optimizer, learning rate 1e-3, MSE loss) regressing ST latent features onto ST spatial coordinates, and apply the trained model to the scRNA-seq latent matrix to project every cell into the pseudo-space. Finally, jointly embed gene expression and spatial weight of the projected cells and perform space-informed Leiden clustering (scanpy, leidenalg) at the desired cluster number K — e.g. K=14 for embryonic heart — to identify spatially heterogeneous subpopulations.

## Related tools

- **scSpace** (Core algorithm: TCA feature extraction, MLP pseudo-space projection, and space-informed clustering of scRNA-seq cells) — https://github.com/ZJUFanLab/scSpace
- **PyTorch** (Implements the two-layer MLP (128-neuron first layer) trained with Adam (lr 1e-3) and MSE loss to regress spatial coordinates)
- **scanpy** (Joint embedding of gene expression and spatial weight followed by Leiden space-informed clustering) — https://github.com/scverse/scanpy
- **Python >=3.8** (Runtime environment; installed via conda (python=3.8) with numpy, pandas, scikit-learn, scipy, igraph, leidenalg dependencies) — https://github.com/ZJUFanLab/scSpace

## Examples

```
conda create -n scspace python=3.8 && conda activate scspace && cd scSpace-master && python setup.py install
```

## Evaluation signals

- pseudo_space_coordinates.csv exists, parses with pandas, and contains one coordinate row per input scRNA-seq cell with finite x and y values
- On embryonic heart data clustered into 14 groups, cardiomyocyte subpopulations (atrial C9, ventricular C4, Myoz2-enriched C12) reach ARI 0.91 and NMI 0.85, far above Seurat's ARI 0.16 and NMI 0.18
- TCA output latent/characteristic matrices exist for both scRNA-seq and ST data with a shared feature space dimensionality
- ARI and NMI values are reported in ari_nmi_metrics.csv and are consistent with the scSpace-vs-Seurat comparison shown in Fig. 3b-i
- Increasing Seurat's K from 14 to 23 still fails to separate ventricular and Myoz2-enriched cardiomyocytes, while scSpace separates them at K=14

## Limitations

- The choice of spatial reference directly influences the spatial reconstruction; an appropriateness assessment or pre-processing of the reference should be performed first (e.g. DLPFC vs dACC — dACC is agranular and lacks cortical layer 4)
- Intrinsic differences in spatial homogeneity between tissues may influence accuracy; scSpace should be applied cautiously to tissues with low spatial coherence (SCC Patients 5 and 9)
- Intrinsic difference in spatial homogeneity between tissues may influence accuracy
- scATAC-seq input must first be converted to a gene activity matrix using Signac, and appropriate spatial ATAC-seq data may not be available

## Evidence

- [readme] Using transfer component analysis (TCA), scSpace could extract the characteristic matrixes of spatial transcriptomics and: "Using [transfer component analysis (TCA)](https://ieeexplore.ieee.org/stamp/stamp.jsp?tp=&arnumber=5640675&tag=1), scSpace could extract the characteristic matrixes of spatial transcriptomics and"
- [methods] project single cells into a pseudo space via a multiple layer perceptron (MLP) model: "project single cells into a pseudo space via a [multiple layer perceptron (MLP)](https://en.wikipedia.org/wiki/Multilayer_perceptron) model"
- [readme] Set up the required Python >=3.8 environment with conda and install scSpace from the Zenodo deposit: "For scSpace, the python version need is over 3.8. If you have installed Python3.6 or Python3.7, consider installing Anaconda, and then you can create a new environment."
- [results] scSpace precisely distinguishes atrial (C9), ventricular (C4), and Myoz2-enriched (C12) cardiomyocytes with ARI 0.91 vs Seurat 0.16: [evidence span withheld — anchor status: not_found]
- [discussion] scSpace should be applied cautiously to tissues with low spatial coherence, such as Patients 5 and 9 of SCC evaluated in this paper: "scSpace should be applied cautiously to tissues with low spatial coherence, such as Patients 5 and 9 of SCC evaluated in this paper"
