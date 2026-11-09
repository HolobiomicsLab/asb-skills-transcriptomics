---
name: mlp-neural-projection-training
description: Use when scRNA-seq cells must be spatially mapped using an ST reference
  (Visium, STARmap, Slide-seq) — e.g., to place myeloid cells relative to epithelial
  cells in normal vs COVID-19 lung pseudo-space.
when_to_use_negative:
- The goal is spatial domain identification of ST data itself — scSpace focuses on
  spatial analysis of scRNA-seq data, whereas methods like SpaGCN, BayesSpace, and
  DR-SC target ST spatial domains
- The tissue has low spatial coherence (e.g., SCC Patients 5 and 9 evaluated in the
  paper), where accuracy of the pseudo-space is compromised
- No suitable spatial reference is available or the reference is anatomically mismatched
  to the scRNA-seq tissue and cannot be pre-processed or assessed for appropriateness
  (as with spatial ATAC-seq in the paper)
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scSpace
  role: Implements TCA feature extraction and the MLP projector that maps scRNA-seq
    cells into the pseudo-space built from the ST reference
  repo: github.com/ZJUFanLab/scSpace
- name: PyTorch
  role: Trains the multiple layer perceptron projector on the spatial transcriptomics
    reference before projecting query cells
- name: Scanpy
  role: Preprocesses and normalizes ST and scRNA-seq count matrices (scanpy 1.9.1)
    to prepare expression data for cross-platform projection
  repo: https://github.com/scverse/scanpy
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-023-38121-4
    title: Reconstruction of the cell pseudo-space from single-cell RNA sequencing
      data with scSpace
  source_content_hash: sha256:0d796c8fd28fc3510dd30e1043a841426e10d8bfdac093a10dd1bfc7c12a82be
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
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
yes_rate: 1.0
n_claims: 12
---

# mlp-neural-projection-training

## Summary

Train scSpace's multiple layer perceptron (MLP) projector on a spatial transcriptomics reference (e.g., 10X Visium) so that scRNA-seq cells can be projected into a reconstructed pseudo-space. This is the core supervised step after transfer component analysis (TCA) removes the batch effect between the two modalities.

## When to use

Use when scRNA-seq cells must be spatially mapped using an ST reference (Visium, STARmap, Slide-seq) — e.g., to place myeloid cells relative to epithelial cells in normal vs COVID-19 lung pseudo-space.

## When NOT to use

- The goal is spatial domain identification of ST data itself — scSpace focuses on spatial analysis of scRNA-seq data, whereas methods like SpaGCN, BayesSpace, and DR-SC target ST spatial domains
- The tissue has low spatial coherence (e.g., SCC Patients 5 and 9 evaluated in the paper), where accuracy of the pseudo-space is compromised
- No suitable spatial reference is available or the reference is anatomically mismatched to the scRNA-seq tissue and cannot be pre-processed or assessed for appropriateness (as with spatial ATAC-seq in the paper)

## Inputs

- Spatial transcriptomics count matrix with spatial coordinates (e.g., 10X Visium normal-lung ST data), as formatted .csv read by pandas
- scRNA-seq count matrices for the query condition(s) (e.g., control and COVID-19 lung), as formatted .csv read by pandas
- TCA-derived shared characteristic matrixes of spatial transcriptomics and scRNA-seq (output of the preceding scSpace TCA step)

## Outputs

- Trained MLP projector (PyTorch model) fitted on the ST reference
- Pseudo-space coordinates (spatial weights) for each projected scRNA-seq cell
- Joint embedding of gene expression and spatial weight feeding Leiden space-informed clustering

## How to apply

Load the ST reference count matrix (with spatial coordinates) and the query scRNA-seq count matrices as five formatted .csv files with pandas, harmonizing genes across modalities. Preprocess and normalize with Scanpy (scanpy 1.9.1), then run transfer component analysis (TCA) in scSpace to eliminate the batch effect between ST and scRNA-seq and extract the shared characteristic matrixes of both modalities. Train the MLP projector with PyTorch on the TCA-transformed ST reference, so the network learns to map expression features into pseudo-space coordinates; then apply the trained projector to the scRNA-seq cells. Jointly embed gene expression and spatial weight of the projected cells for downstream space-informed clustering (Leiden via leidenalg 0.9.0 / igraph 0.10.2), which yields spatially variable subpopulations with higher cell-typing accuracy. The choice of spatial reference directly influences the reconstruction, so assess reference appropriateness (e.g., DLPFC vs agranular dACC) before training.

## Related tools

- **scSpace** (Implements TCA feature extraction and the MLP projector that maps scRNA-seq cells into the pseudo-space built from the ST reference) — github.com/ZJUFanLab/scSpace
- **PyTorch** (Trains the multiple layer perceptron projector on the spatial transcriptomics reference before projecting query cells)
- **Scanpy** (Preprocesses and normalizes ST and scRNA-seq count matrices (scanpy 1.9.1) to prepare expression data for cross-platform projection) — https://github.com/scverse/scanpy

## Examples

```
conda create -n scspace python=3.8; conda activate scspace; pip install -r requirements.txt; python setup.py install
```

## Evaluation signals

- The trained MLP, applied to scRNA-seq cells, reproduces the paper's pairwise spatial associations — e.g., normalized myeloid-to-epithelial pseudo-space distances are significantly reduced in COVID-19 vs control (Fig. 6a-k pattern)
- Space-informed Leiden clustering of the joint expression-plus-spatial-weight embedding yields distinct MDM and TMDM macrophage subpopulation clusters
- TCA-shared features show the batch effect between ST reference and scRNA-seq removed before MLP training (modalities mix in the characteristic matrix space)
- The projector transfers across modalities: the same MLP trained only on the ST reference assigns coherent spatial weights to control and COVID-19 scRNA-seq cells
- Spatial-variability identification remains conserved when references from different biological replicates are used (as benchmarked on melanoma)

## Limitations

- Since the choice of spatial reference directly influences the spatial reconstruction, an inappropriate reference distorts all downstream distances and clusters
- The intrinsic difference in spatial homogeneity between tissues influences accuracy; apply cautiously to tissues with low spatial coherence (SCC Patients 5 and 9)
- An appropriateness assessment or necessary pre-processing of the spatial reference should be considered before applying scSpace (e.g., dACC is agranular and lacks cortical layer 4 relative to DLPFC)
- Appropriate spatial ATAC-seq references may be unavailable, forcing ST-based reconstruction with a gene activity matrix converted via Signac

## Evidence

- [intro] we introduce scSpace, an integrative method that uses ST data as a spatial reference to reconstruct the pseudo-space of scRNA-seq data: "we introduce scSpace, an integrative method that uses ST data as a spatial reference to reconstruct the pseudo-space of scRNA-seq data"
- [readme] TCA extraction of shared features: "scSpace could extract the characteristic matrixes of spatial transcriptomics and scRNA-seq, and project single cells into a pseudo space"
- [methods] MLP projection step: "project single cells into a pseudo space via a [multiple layer perceptron]"
- [methods] joint embedding for cell typing: "so that gene expression and spatial weight of cells can be embedded jointly for the further cell typing with higher accuracy and precision"
- [discussion] reference choice caveat: "Since the choice of spatial reference would directly influence the spatial reconstruction"
- [discussion] low-coherence tissue caveat: "scSpace should be applied cautiously to tissues with low spatial coherence, such as Patients 5 and 9 of SCC"
- [readme] environment setup: [evidence span withheld — anchor status: not_found]
