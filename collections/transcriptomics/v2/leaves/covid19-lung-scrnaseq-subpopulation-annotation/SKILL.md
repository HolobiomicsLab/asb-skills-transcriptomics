---
name: covid19-lung-scrnaseq-subpopulation-annotation
description: Use when control vs COVID-19 lung scRNA-seq cells must be spatially localized
  against a 10X Visium normal-lung reference to detect spatially variated subpopulations
  (e.g. MDM/TMDM macrophages) and quantify myeloid–epithelial proximity.
when_to_use_negative:
- The goal is spatial domain identification of the ST data itself — scSpace focuses
  on the spatial analysis of scRNA-seq data, whereas methods like SpaGCN, BayesSpace,
  and DR-SC target ST domains
- The tissue has low spatial coherence (e.g. Patients 5 and 9 of SCC), where scSpace
  should be applied cautiously
- No appropriate spatial reference is available (analogous to the missing spatial
  ATAC-seq reference in the scATAC-seq attempt), so the choice of spatial reference
  cannot be validated
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scSpace
  role: 'Core integrative algorithm: TCA-based batch-effect removal between ST reference
    and scRNA-seq, MLP projection into pseudo-space, and space-informed clustering'
  repo: https://github.com/ZJUFanLab/scSpace
- name: Scanpy
  role: Preprocessing and normalization of the harmonized ST and scRNA-seq count matrices
    before TCA/projection
  repo: https://github.com/scverse/scanpy
- name: PyTorch
  role: Trains the multiple layer perceptron (MLP) projector on the ST reference;
    install a version matching the machine's CUDA (e.g. cu116)
  repo: https://pytorch.org/get-started/locally/
- name: Leidenalg
  role: With igraph 0.10.2, performs the Leiden space-informed clustering on the joint
    gene-expression/spatial-weight embedding to resolve MDM and TMDM subpopulations
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-023-38121-4
    title: Reconstruction of the cell pseudo-space from single-cell RNA sequencing
      data with scSpace
  source_content_hash: sha256:2e95415cbc647f8d8ab1eaf3bf9e9e4746272322cbb686a497b028efc3cd1698
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ZJUFanLab/scSpace
  tools:
  - Leidenalg
  - PyTorch
  - Scanpy
  - Visium
  - igraph
  - scSpace
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9545454545454546
n_claims: 22
---

# covid19-lung-scrnaseq-subpopulation-annotation

## Summary

Reconstructs a pseudo-space for control and COVID-19 lung scRNA-seq cells by using a 10X Visium ST reference of normal lung in scSpace, then annotates spatially variated myeloid subpopulations (MDM and TMDM macrophages) and quantifies myeloid-to-epithelial pseudo-space distances to reveal severe immune infiltration.

## When to use

Use when control vs COVID-19 lung scRNA-seq cells must be spatially localized against a 10X Visium normal-lung reference to detect spatially variated subpopulations (e.g. MDM/TMDM macrophages) and quantify myeloid–epithelial proximity.

## When NOT to use

- The goal is spatial domain identification of the ST data itself — scSpace focuses on the spatial analysis of scRNA-seq data, whereas methods like SpaGCN, BayesSpace, and DR-SC target ST domains
- The tissue has low spatial coherence (e.g. Patients 5 and 9 of SCC), where scSpace should be applied cautiously
- No appropriate spatial reference is available (analogous to the missing spatial ATAC-seq reference in the scATAC-seq attempt), so the choice of spatial reference cannot be validated

## Inputs

- 10X Visium spatial transcriptomics count matrix of normal lung with spatial coordinates (scSpace-formatted .csv, read in by pandas)
- Control lung scRNA-seq count matrix (gene-harmonized .csv)
- COVID-19 lung scRNA-seq count matrix (gene-harmonized .csv)
- Cell type labels for myeloid cells and alveolar/airway epithelial cells

## Outputs

- Pseudo-space coordinates of control and COVID-19 scRNA-seq cells projected via the MLP
- Space-informed Leiden clusters resolving spatially variated subpopulations
- MDM and TMDM macrophage subpopulation annotations
- Normalized myeloid-to-epithelial pseudo-space distances per condition (control vs COVID-19)

## How to apply

Load the normal-lung 10X Visium ST count matrix (with spatial coordinates) and the control and COVID-19 lung scRNA-seq count matrices with pandas, harmonizing genes across modalities, then normalize with Scanpy (scanpy 1.9.1). Run transfer component analysis (TCA) in scSpace to eliminate the batch effect between the ST reference and the scRNA-seq data and extract shared characteristic matrixes. Train the multiple layer perceptron (MLP) projector in scSpace on the ST reference using PyTorch (CUDA-appropriate build), then project the control and COVID-19 scRNA-seq cells into the pseudo space. Jointly embed gene expression and spatial weights and perform space-informed clustering with the Leiden algorithm (leidenalg 0.9.0 / igraph 0.10.2) to resolve spatially variable subpopulations. Finally, compute pairwise myeloid-to-epithelial pseudo-space distances in control versus COVID-19 conditions and identify MDM and TMDM macrophage subpopulations among the clusters; validate by requiring reduced myeloid-to-epithelial distance in COVID-19 and distinct MDM/TMDM clusters, reproducing the Fig. 6a–k pattern.

## Related tools

- **scSpace** (Core integrative algorithm: TCA-based batch-effect removal between ST reference and scRNA-seq, MLP projection into pseudo-space, and space-informed clustering) — https://github.com/ZJUFanLab/scSpace
- **Scanpy** (Preprocessing and normalization of the harmonized ST and scRNA-seq count matrices before TCA/projection) — https://github.com/scverse/scanpy
- **PyTorch** (Trains the multiple layer perceptron (MLP) projector on the ST reference; install a version matching the machine's CUDA (e.g. cu116)) — https://pytorch.org/get-started/locally/
- **Leidenalg** (With igraph 0.10.2, performs the Leiden space-informed clustering on the joint gene-expression/spatial-weight embedding to resolve MDM and TMDM subpopulations)

## Examples

```
conda create -n scspace python=3.8; conda activate scspace; pip install torch --extra-index-url https://download.pytorch.org/whl/cu116; pip install -r requirements.txt; python setup.py install
```

## Evaluation signals

- The normalized distance between myeloid cells and alveolar/airway epithelial cells is significantly reduced in the COVID-19 group compared to the control, indicating severe immune infiltration
- MDM and TMDM macrophage subpopulations form distinct clusters in the Leiden space-informed clustering, reproducing the Fig. 6a–k pattern
- TCA removes the cross-platform batch effect such that characteristic matrixes of spatial transcriptomics and scRNA-seq are extracted as shared latent features
- Projected cells carry spatial weights that, when jointly embedded with gene expression, yield higher-accuracy cell typing than expression-only clustering
- Genes are harmonized across the Visium ST and scRNA-seq modalities before TCA, so the input .csv files share a consistent gene axis

## Limitations

- scSpace should be applied cautiously to tissues with low spatial coherence, such as Patients 5 and 9 of SCC evaluated in this paper
- Since the choice of spatial reference would directly influence the spatial reconstruction, an appropriateness assessment or pre-processing of the reference should be considered (e.g. dACC is agranular and lacks cortical layer 4 versus DLPFC)
- The intrinsic difference in the spatial homogeneity between tissues may influence the accuracy of scSpace

## Evidence

- [intro] TCA cross-modality integration: "scSpace enables eliminating the batch effect between single-cell and ST data and extracting the shared latent feature across these two types of data"
- [readme] MLP projection into pseudo-space: "project single cells into a pseudo space via a [multiple layer perceptron (MLP)](https://en.wikipedia.org/wiki/Multilayer_perceptron) model"
- [results] COVID-19 myeloid infiltration finding: [evidence span withheld — anchor status: not_found]
- [discussion] Scope: scRNA-seq spatial analysis, not ST domain detection: "scSpace focuses on the spatial analysis of scRNA-seq data whereas other methods target the spatial domain identification of ST data"
- [discussion] Spatial coherence caveat: "scSpace should be applied cautiously to tissues with low spatial coherence, such as Patients 5 and 9 of SCC evaluated in this paper"
