---
name: batch-effect-removal-cross-assay
description: Use when integrating scRNA-seq and ST count matrices measured on the
  same shared gene space from different experiments or patients, and a domain-invariant
  latent feature is needed before spatial projection or joint clustering.
when_to_use_negative:
- When the scRNA-seq and ST data do not share a common gene space or are from tissues
  with low spatial coherence, where scSpace should be applied cautiously (e.g., Patients
  5 and 9 of SCC evaluated in the paper)
- When the goal is spatial domain identification of ST data itself rather than cross-assay
  integration — scSpace targets spatial analysis of scRNA-seq data, unlike SpaGCN,
  BayesSpace, or DR-SC
- When data come from the same assay with no cross-experiment batch effect to remove
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scSpace
  role: Hosts the TCA-based characteristic-matrix extraction that eliminates batch
    effect between scRNA-seq and ST data before pseudo-space reconstruction
  repo: https://github.com/ZJUFanLab/scSpace
- name: Python >=3.8
  role: Runtime environment (with numpy 1.23.4, pandas 1.5.0, scikit-learn 1.1.2,
    scipy 1.9.2) for implementing the TCA kernel construction and projection solve
  repo: https://www.python.org/
- name: scanpy
  role: Single-cell preprocessing ecosystem in the scSpace requirements used for handling
    scRNA-seq matrices
  repo: https://github.com/scverse/scanpy
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-023-38121-4
    title: Reconstruction of the cell pseudo-space from single-cell RNA sequencing
      data with scSpace
  source_content_hash: sha256:09ff4e9d7750bd40fe172c42bfc994815b58d559309ab90a05e4b218a82f1628
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ZJUFanLab/scSpace
  tools:
  - Python
  - Python >=3.8
  - scSpace
  - scanpy
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# batch-effect-removal-cross-assay

## Summary

Removes batch effect between scRNA-seq and spatial transcriptomics (ST) data and extracts a shared latent feature space using transfer component analysis (TCA) as implemented in the scSpace workflow.

## When to use

Use when integrating scRNA-seq and ST count matrices measured on the same shared gene space from different experiments or patients, and a domain-invariant latent feature is needed before spatial projection or joint clustering.

## When NOT to use

- When the scRNA-seq and ST data do not share a common gene space or are from tissues with low spatial coherence, where scSpace should be applied cautiously (e.g., Patients 5 and 9 of SCC evaluated in the paper)
- When the goal is spatial domain identification of ST data itself rather than cross-assay integration — scSpace targets spatial analysis of scRNA-seq data, unlike SpaGCN, BayesSpace, or DR-SC
- When data come from the same assay with no cross-experiment batch effect to remove

## Inputs

- scRNA-seq count matrix (.csv, cells x shared genes) from the source domain
- spatial transcriptomics count matrix (.csv, spots x shared genes) from the target domain, e.g. Visium/DLPFC data
- shared gene space over both matrices
- TCA hyperparameters (number of components, regularization lambda)

## Outputs

- shared latent feature matrix X'_S for the spatial transcriptomics samples
- shared latent feature matrix X'_T for the scRNA-seq samples
- tca_features.csv with cell/spot identifiers preserved

## How to apply

Load the single-cell and spatial transcriptomics count matrices (pandas-readable .csv files, e.g. from the scSpace vignettes/data folder) and restrict both to a shared gene space. Project the concatenated source (scRNA-seq) and target (ST) samples into a reproducing kernel Hilbert space (RKHS) by constructing the cross-kernel matrix K. Solve the TCA objective (Eqs. 1-6, following Pan et al.) by computing the projection A = (KLK + lambda*I)^-1 K' L X, which minimizes the maximum mean discrepancy (MMD) between source and target distributions while penalizing lambda*tr(A'KLKA); choose the number of latent components and lambda as hyperparameters. Apply A to obtain the shared latent feature matrices X'_S (ST) and X'_T (scRNA-seq) with cell/spot identifiers preserved, writing them to tca_features.csv. These latent features then feed scSpace's downstream MLP-based projection of single cells into a pseudo space and joint space-informed clustering.

## Related tools

- **scSpace** (Hosts the TCA-based characteristic-matrix extraction that eliminates batch effect between scRNA-seq and ST data before pseudo-space reconstruction) — https://github.com/ZJUFanLab/scSpace
- **Python >=3.8** (Runtime environment (with numpy 1.23.4, pandas 1.5.0, scikit-learn 1.1.2, scipy 1.9.2) for implementing the TCA kernel construction and projection solve) — https://www.python.org/
- **scanpy** (Single-cell preprocessing ecosystem in the scSpace requirements used for handling scRNA-seq matrices) — https://github.com/scverse/scanpy

## Examples

```
A = np.linalg.inv(K @ L @ K + lam * np.eye(n)) @ K.T @ L @ X; Xs, Xt = A.T @ Xs_raw, A.T @ Xt_raw
```

## Evaluation signals

- tca_features.csv exists and contains two blocks of features whose row counts equal the input scRNA-seq and ST sample counts
- MMD between the projected source and target distributions is lower than the MMD computed on the untransformed inputs
- Cell/spot identifiers are preserved in the output feature matrices and align with the input .csv files
- Downstream MLP pseudo-space projection and space-informed clustering run on the latent features without domain-separation artifacts (e.g., no clustering by assay instead of cell type)

## Limitations

- The choice of spatial reference directly influences the spatial reconstruction, so an appropriateness assessment or pre-processing of the reference should be performed (e.g., dACC is agranular and lacks cortical layer 4 versus DLPFC)
- Intrinsic differences in spatial homogeneity between tissues may influence the accuracy of scSpace; apply cautiously to tissues with low spatial coherence (e.g., SCC Patients 5 and 9)
- No appropriate spatial ATAC-seq data was available for scATAC-seq reconstruction; scATAC-seq must first be converted to a gene activity matrix using Signac

## Evidence

- [methods] TCA first projects the scRNA-seq and spatial transcriptomics data into a Reproducing Kernel Hilbert Space (RKHS), and then reduces the difference in the distribution of the transformed two domain data by minimizing the maximum mean discrepancy (MMD) between them.: [evidence span withheld — anchor status: not_found]
- [intro] scSpace enables eliminating the batch effect between single-cell and ST data and extracting the shared latent feature across these two types of data using transfer component analysis (TCA).: "scSpace enables eliminating the batch effect between single-cell and ST data and extracting the shared latent feature across these two types of data"
- [readme] Using transfer component analysis (TCA), scSpace could extract the characteristic matrixes of spatial transcriptomics and scRNA-seq, and project single cells into a pseudo space via a multiple layer perceptron (MLP) model: "Using [transfer component analysis (TCA)](https://ieeexplore.ieee.org/stamp/stamp.jsp?tp=&arnumber=5640675&tag=1), scSpace could extract the characteristic matrixes"
- [readme] We have included a toy dataset in the vignettes/data folder of this repository as examples to show how to use scSpace: "We have included a toy dataset 
in the [vignettes/data folder](vignettes/data) of this repository as examples to show how to use scSpace"
