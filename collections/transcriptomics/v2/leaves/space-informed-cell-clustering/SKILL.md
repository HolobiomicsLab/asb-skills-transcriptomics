---
name: space-informed-cell-clustering
description: Use when scRNA-seq cells must be partitioned into subpopulations reflecting
  spatial variation, and a spatial transcriptomics reference (Visium, STARmap, Slide-seq)
  is available to reconstruct a pseudo-space.
when_to_use_negative:
- The task is spatial domain identification of ST data itself — scSpace targets spatial
  analysis of scRNA-seq data, unlike SpaGCN, BayesSpace, or DR-SC which target ST
  spatial domains
- The tissue has low spatial coherence (e.g., Patients 5 and 9 of SCC), where the
  article advises applying scSpace cautiously
- No appropriate spatial reference exists or the reference is mismatched to the tissue
  architecture (e.g., dACC vs DLPFC, where dACC is agranular and lacks cortical layer
  4)
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: scSpace
  role: Runs TCA batch correction, trains the MLP pseudo-space projector, and provides
    the joint embedding used for space-informed Leiden clustering
  repo: https://github.com/ZJUFanLab/scSpace
- name: scanpy
  role: Preprocessing/normalization of count matrices and scanpy-based Leiden clustering
    on the jointly embedded gene expression + spatial weight representation
  repo: https://github.com/scverse/scanpy
- name: PyTorch
  role: Trains the multiple layer perceptron (MLP) model that projects single cells
    into the pseudo space
- name: leidenalg
  role: Leiden community detection (v0.9.0) used for space-informed clustering of
    the embedded cells
- name: igraph
  role: Graph backend (v0.10.2) underlying the Leiden clustering
provenance:
  source_task_ids:
  - task_005
  - task_003
  - task_004
  source_papers:
  - doi: 10.1038/s41467-023-38121-4
    title: Reconstruction of the cell pseudo-space from single-cell RNA sequencing
      data with scSpace
  source_content_hash: sha256:3690bc51e42fb08c013116802494bbbc21826739c9bd55e237bca73bda96aa85
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ZJUFanLab/scSpace
  tools:
  - PyTorch
  - Python >=3.8
  - Slide-seq
  - Visium
  - igraph
  - leidenalg
  - matplotlib
  - pandas
  - scSpace
  - scanpy
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9375
n_claims: 16
---

# space-informed-cell-clustering

## Summary

Clusters single-cell RNA-seq data in a reconstructed pseudo-space by jointly embedding gene expression and spatial weight, as implemented in scSpace, to resolve spatially heterogeneous cell subpopulations that traditional transcriptome-only pipelines miss.

## When to use

Use when scRNA-seq cells must be partitioned into subpopulations reflecting spatial variation, and a spatial transcriptomics reference (Visium, STARmap, Slide-seq) is available to reconstruct a pseudo-space.

## When NOT to use

- The task is spatial domain identification of ST data itself — scSpace targets spatial analysis of scRNA-seq data, unlike SpaGCN, BayesSpace, or DR-SC which target ST spatial domains
- The tissue has low spatial coherence (e.g., Patients 5 and 9 of SCC), where the article advises applying scSpace cautiously
- No appropriate spatial reference exists or the reference is mismatched to the tissue architecture (e.g., dACC vs DLPFC, where dACC is agranular and lacks cortical layer 4)

## Inputs

- Formatted .csv count matrix of the spatial transcriptomics reference (with spatial coordinates), read with pandas
- Formatted .csv count matrix of the scRNA-seq cells to be clustered, read with pandas
- Gene-harmonized expression matrices across modalities

## Outputs

- Pseudo-space coordinates for each projected single cell
- Leiden cluster labels per cell from space-informed clustering
- Subpopulation assignments for spatially variated cell types (e.g., C4/C9/C12 cardiomyocytes, T cell C1–C5)
- Clustering agreement metrics (ARI/NMI) against reference annotations

## How to apply

First run scSpace's transfer component analysis (TCA) to eliminate batch effect between the ST reference and scRNA-seq data and extract shared characteristic matrices. Train the multiple layer perceptron (MLP) projector (PyTorch) on the ST reference, then project scRNA-seq cells into pseudo-space coordinates. Jointly embed gene expression and spatial weight of the projected cells and perform Leiden clustering (leidenalg 0.9.0 / igraph 0.10.2 via scanpy 1.9.1 neighbors) at the desired resolution — e.g., 14 clusters to match embryonic heart annotations, or 5 clusters to resolve melanoma T cell subpopulations C1–C5. Inputs are five formatted .csv files read with pandas; validate by comparing cluster assignments to known annotations (ARI/NMI) or by checking that spatially distinct subpopulations form separate clusters.

## Related tools

- **scSpace** (Runs TCA batch correction, trains the MLP pseudo-space projector, and provides the joint embedding used for space-informed Leiden clustering) — https://github.com/ZJUFanLab/scSpace
- **scanpy** (Preprocessing/normalization of count matrices and scanpy-based Leiden clustering on the jointly embedded gene expression + spatial weight representation) — https://github.com/scverse/scanpy
- **PyTorch** (Trains the multiple layer perceptron (MLP) model that projects single cells into the pseudo space)
- **leidenalg** (Leiden community detection (v0.9.0) used for space-informed clustering of the embedded cells)
- **igraph** (Graph backend (v0.10.2) underlying the Leiden clustering)

## Examples

```
conda create -n scspace python=3.8; conda activate scspace; cd scSpace-master && pip install -r requirements.txt && python setup.py install
```

## Evaluation signals

- On embryonic heart scRNA-seq clustered into 14 groups, ARI vs original annotations should reach ~0.91 and NMI ~0.85 (vs Seurat's ARI 0.16 / NMI 0.18)
- Spatially variated cardiomyocyte subpopulations C4 (ventricular), C9 (atrial), and C12 (Myoz2-enriched) form distinct clusters; metrics reported in ari_nmi_metrics.csv
- In the melanoma T cell analysis, all 2064 T cells receive pseudo-space coordinates and five-cluster labels, with C5 nearer malignant cells and showing higher T-cell exhaustion score than distant C3 (Fig. 5c-f)
- In the COVID-19 lung reconstruction, normalized myeloid-to-epithelial pseudo-space distances are reduced in COVID-19 vs control and MDM/TMDM macrophage subpopulations form distinct clusters (Fig. 6a-k)
- Clustering output schema contains per-cell pseudo-space coordinates, cluster labels, and gene-expression + spatial-weight joint embedding

## Limitations

- The choice of spatial reference directly influences the spatial reconstruction; an appropriateness assessment or pre-processing of the reference should be performed first
- Intrinsic differences in spatial homogeneity between tissues may influence accuracy — apply cautiously to tissues with low spatial coherence (e.g., SCC Patients 5 and 9)
- Spatially heterogeneous subpopulations are hard to identify by traditional single-cell pipelines due to transcriptome similarity, so plain transcriptome-only clustering is an inadequate substitute

## Evidence

- [methods] joint embedding defines clustering: "so that gene expression and spatial weight of cells can be embedded jointly for the further cell typing with higher accuracy and precision"
- [methods] project cells via MLP: "project single cells into a pseudo space via a [multiple layer perceptron (MLP)](https://en.wikipedia.org/wiki/Multilayer_perceptron) model"
- [methods] TCA extracts shared features: "Using [transfer component analysis (TCA)](https://ieeexplore.ieee.org/stamp/stamp.jsp?tp=&arnumber=5640675&tag=1), scSpace could extract the characteristic matrixes of spatial transcriptomics and"
- [abstract] benchmark performance claim: "We benchmark scSpace with both simulated and biological datasets, and demonstrate that scSpace can accurately and robustly identify spatially variated cell subpopulations."
- [discussion] scSpace scope vs ST methods: "scSpace focuses on the spatial analysis of scRNA-seq data whereas other methods target the spatial domain identification of ST data"
- [readme] input format requirement: "To use scSpace we require five formatted `.csv` files as input (i.e. read in by pandas). We have included a toy dataset"
- [discussion] reference choice caveat: "performing an appropriateness assessment or necessary pre-processing steps of the spatial reference should be considered before applying scSpace"
