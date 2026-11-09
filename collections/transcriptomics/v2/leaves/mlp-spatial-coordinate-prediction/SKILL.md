---
name: mlp-spatial-coordinate-prediction
description: Use when you have paired scRNA-seq count data and a spatial transcriptomics
  reference and need to assign spatial coordinates (pseudo-space) to individual single
  cells, e.g. to recover spatially heterogeneous subpopulations like atrial/ventricular
  cardiomyocytes.
when_to_use_negative:
- Input is ST data and the goal is spatial domain identification of the ST slide itself
  — scSpace targets spatial analysis of scRNA-seq data, whereas SpaGCN/BayesSpace/DR-SC
  target spatial domain identification of ST data
- Tissue has low spatial coherence (e.g. Patients 5 and 9 of SCC in the paper) — apply
  cautiously since intrinsic differences in spatial homogeneity influence accuracy
- The spatial reference is from a region with different spatial architecture than
  the scRNA-seq sample (e.g. agranular dACC lacking cortical layer 4 vs DLPFC) without
  an appropriateness assessment or pre-processing of the reference
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_2229
- http://edamontology.org/topic_3308
tools:
- name: scSpace
  role: 'Core algorithm: TCA feature extraction + MLP projection of single cells into
    pseudo-space with spatial reference from ST data'
  repo: https://github.com/ZJUFanLab/scSpace
- name: scanpy
  role: Space-informed Leiden clustering on the joint gene-expression/spatial-weight
    embedding for downstream cell typing
  repo: https://github.com/scverse/scanpy
- name: Visium
  role: Spatial transcriptomics reference platform used for pseudo-space reconstruction
- name: Slide-seq
  role: Alternative spatial transcriptomics reference platform for pseudo-space reconstruction
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-023-38121-4
    title: Reconstruction of the cell pseudo-space from single-cell RNA sequencing
      data with scSpace
  source_content_hash: sha256:906224e51fc6736a80cb13a2f2872028b40804dc5c3b370aac016c6245e7bf46
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ZJUFanLab/scSpace
  tools:
  - Python >=3.8
  - Slide-seq
  - Visium
  - scSpace
  - scanpy
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_2229
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# MLP-based pseudo-space coordinate prediction

## Summary

Train scSpace's multiple layer perceptron (MLP) to project scRNA-seq cells into a reconstructed pseudo-space using spatial transcriptomics references (Visium, STARmap, Slide-seq), after transfer component analysis (TCA) removes batch effects between the single-cell and ST feature matrices.

## When to use

Use when you have paired scRNA-seq count data and a spatial transcriptomics reference and need to assign spatial coordinates (pseudo-space) to individual single cells, e.g. to recover spatially heterogeneous subpopulations like atrial/ventricular cardiomyocytes.

## When NOT to use

- Input is ST data and the goal is spatial domain identification of the ST slide itself — scSpace targets spatial analysis of scRNA-seq data, whereas SpaGCN/BayesSpace/DR-SC target spatial domain identification of ST data
- Tissue has low spatial coherence (e.g. Patients 5 and 9 of SCC in the paper) — apply cautiously since intrinsic differences in spatial homogeneity influence accuracy
- The spatial reference is from a region with different spatial architecture than the scRNA-seq sample (e.g. agranular dACC lacking cortical layer 4 vs DLPFC) without an appropriateness assessment or pre-processing of the reference

## Inputs

- scRNA-seq count matrix formatted as .csv (read by pandas)
- Spatial transcriptomics reference data (Visium, STARmap, or Slide-seq) formatted as .csv
- Python >=3.8 environment with scSpace and its dependencies (numpy, pandas, scanpy, scikit-learn, pytorch)

## Outputs

- Pseudo-space coordinates for each single cell
- Joint embedding of gene expression and spatial weight per cell
- Space-informed Leiden cluster assignments identifying spatially variated subpopulations (e.g. ari_nmi_metrics.csv with ARI/NMI values)

## How to apply

Set up a Python >=3.8 conda environment and install scSpace from the Zenodo deposit (10.5281/zenodo.7790754) along with numpy, pandas, scanpy, scikit-learn, scipy, igraph, leidenalg, and a PyTorch version matched to your CUDA. Load five formatted .csv files (read in by pandas) containing the scRNA-seq counts and the spatial transcriptomics reference. First run transfer component analysis (TCA) inside scSpace to eliminate the batch effect between scRNA-seq and ST data and extract their shared characteristic feature matrices; then train the MLP model to project the single cells into the reconstructed pseudo-space. Finally, jointly embed gene expression and spatial weight of the projected cells and perform space-informed Leiden clustering (scanpy) to identify spatially variated subpopulations. The rationale is that transcriptome-similar subpopulations are hard to separate by traditional pipelines, but joint gene-expression + spatial-weight embedding raises clustering accuracy and precision.

## Related tools

- **scSpace** (Core algorithm: TCA feature extraction + MLP projection of single cells into pseudo-space with spatial reference from ST data) — https://github.com/ZJUFanLab/scSpace
- **scanpy** (Space-informed Leiden clustering on the joint gene-expression/spatial-weight embedding for downstream cell typing) — https://github.com/scverse/scanpy
- **Visium** (Spatial transcriptomics reference platform used for pseudo-space reconstruction)
- **Slide-seq** (Alternative spatial transcriptomics reference platform for pseudo-space reconstruction)

## Examples

```
conda create -n scspace python=3.8; conda activate scspace; pip install torch --extra-index-url https://download.pytorch.org/whl/cu116; cd scSpace-master && pip install -r requirements.txt && python setup.py install
```

## Evaluation signals

- On embryonic heart data clustered into 14 groups, cardiomyocyte subpopulation assignments (C4 ventricular, C9 atrial, C12 Myoz2-enriched) should reach ARI ~0.91 and NMI ~0.85, versus Seurat's ARI 0.16 / NMI 0.18
- ARI and NMI metrics are reported in ari_nmi_metrics.csv and are consistent with the scSpace-vs-Seurat comparison in Fig. 3b-i
- Increasing Seurat's K from 14 to 23 should still fail to separate ventricular and Myoz2-enriched cardiomyocytes, confirming the MLP pseudo-space embedding (not cluster count) drives the improvement
- The five input .csv files load correctly with pandas and the TCA step yields shared characteristic feature matrices for both scRNA-seq and ST data before MLP training
- Spatial variability identification performance is conserved when the spatial reference comes from a different biological replicate (as shown for melanoma)

## Limitations

- scSpace should be applied cautiously to tissues with low spatial coherence (e.g. Patients 5 and 9 of SCC)
- The choice of spatial reference directly influences the spatial reconstruction; references from regions with different spatial architecture (DLPFC vs dACC, which lacks cortical layer 4) require appropriateness assessment or pre-processing
- Intrinsic differences in spatial homogeneity between tissues may influence accuracy
- Appropriate spatial ATAC-seq data may not be available, so scATAC-seq reconstruction requires converting to a gene activity matrix via Signac and using ST as reference

## Evidence

- [readme] MLP projects single cells into pseudo-space via TCA feature extraction: "project single cells into a pseudo space via a [multiple layer perceptron (MLP)](https://en.wikipedia.org/wiki/Multilayer_perceptron) model"
- [intro] TCA extracts shared characteristic matrices eliminating batch effect: "scSpace enables eliminating the batch effect between single-cell and ST data and extracting the shared latent feature across these two types of data"
- [methods] Joint embedding of gene expression and spatial weight for cell typing: "so that gene expression and spatial weight of cells can be embedded jointly for the further cell typing with higher accuracy and precision"
- [results] Embryonic heart cardiomyocyte ARI 0.91 vs Seurat 0.16: [evidence span withheld — anchor status: not_found]
- [discussion] Spatial reference choice caveat: "Since the choice of spatial reference would directly influence the spatial reconstruction"
