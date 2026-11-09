---
name: reproducing-kernel-hilbert-space-embedding
description: Use when two count matrices from different platforms (scRNA-seq and spatial
  transcriptomics .csv files) must be brought into a comparable feature space before
  joint downstream analysis such as pseudo-space projection.
when_to_use_negative:
- The two datasets are already in a shared low-dimensional feature space or the downstream
  analysis does not require cross-domain integration of scRNA-seq and ST data.
- The spatial reference tissue has low spatial coherence or differs architecturally
  from the scRNA-seq tissue (the paper advises caution, e.g. SCC Patients 5 and 9,
  dACC lacking cortical layer 4), so batch correction alone would not make the integration
  meaningful.
- The goal is spatial domain identification of ST data rather than spatial analysis
  of scRNA-seq data, which scSpace explicitly does not target.
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scSpace
  role: Implements the TCA-based characteristic (latent feature) matrix extraction
    that embeds scRNA-seq and ST data into a reproducing kernel Hilbert space prior
    to MLP-based pseudo-space projection.
  repo: https://github.com/ZJUFanLab/scSpace
- name: Python >=3.8
  role: Runtime environment (conda env with numpy 1.23.4, pandas 1.5.0, scikit-learn
    1.1.2, scipy 1.9.2, scanpy 1.9.1) in which the kernel matrices and TCA solve are
    executed.
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-023-38121-4
    title: Reconstruction of the cell pseudo-space from single-cell RNA sequencing
      data with scSpace
  source_content_hash: sha256:1dcdc6359194102d219a744dde71ef79a8ded92068874ff2b761263dbfb1aec9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ZJUFanLab/scSpace
  tools:
  - Python
  - Python >=3.8
  - scSpace
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# reproducing-kernel-hilbert-space-embedding

## Summary

Kernel embedding step within scSpace's transfer component analysis (TCA): single-cell scRNA-seq and spatial transcriptomics count matrices are mapped into a Reproducing Kernel Hilbert Space so that a shared low-dimensional latent feature can be learned that eliminates the batch effect between the two domains.

## When to use

Use when two count matrices from different platforms (scRNA-seq and spatial transcriptomics .csv files) must be brought into a comparable feature space before joint downstream analysis such as pseudo-space projection.

## When NOT to use

- The two datasets are already in a shared low-dimensional feature space or the downstream analysis does not require cross-domain integration of scRNA-seq and ST data.
- The spatial reference tissue has low spatial coherence or differs architecturally from the scRNA-seq tissue (the paper advises caution, e.g. SCC Patients 5 and 9, dACC lacking cortical layer 4), so batch correction alone would not make the integration meaningful.
- The goal is spatial domain identification of ST data rather than spatial analysis of scRNA-seq data, which scSpace explicitly does not target.

## Inputs

- scRNA-seq count matrix (.csv, rows = cells)
- spatial transcriptomics count matrix (.csv, rows = spots)
- shared gene list used to restrict both matrices to a common feature space

## Outputs

- tca_features.csv containing projected latent feature matrices X'_S (ST) and X'_T (scRNA-seq) with cell/spot identifiers preserved
- lower MMD value between transformed source and target distributions than computed on the raw inputs

## How to apply

Load the scRNA-seq and spatial transcriptomics count matrices from formatted .csv files with pandas, and restrict both to a shared gene space. Concatenate the source (scRNA-seq) and target (spatial transcriptomics) samples and construct the cross-kernel matrix K over them in the reproducing kernel Hilbert space. Solve the TCA objective (Eqs. 1-6 in the paper, following Pan et al.) by computing the projection A = (KLK + lambda*I)^-1 K L X, which minimizes the maximum mean discrepancy (MMD) between the transformed source and target distributions while the penalty term lambda*tr(A'KLKA) regularizes the solution. Apply A to obtain the shared latent feature matrices X'_S and X'_T for the ST and scRNA-seq datasets, preserving cell/spot identifiers. The success criterion is a measured MMD between projected source and target distributions lower than the MMD on the untransformed inputs, since TCA's purpose is batch-effect removal across the two data types.

## Related tools

- **scSpace** (Implements the TCA-based characteristic (latent feature) matrix extraction that embeds scRNA-seq and ST data into a reproducing kernel Hilbert space prior to MLP-based pseudo-space projection.) — https://github.com/ZJUFanLab/scSpace
- **Python >=3.8** (Runtime environment (conda env with numpy 1.23.4, pandas 1.5.0, scikit-learn 1.1.2, scipy 1.9.2, scanpy 1.9.1) in which the kernel matrices and TCA solve are executed.)

## Examples

```
import pandas as pd; sc = pd.read_csv('vignettes/data/sc_count.csv', index_col=0); st = pd.read_csv('vignettes/data/st_count.csv', index_col=0); K = build_kernel(pd.concat([sc, st])); A = solve_tca(K, L, lam); pd.DataFrame({'X_S': K @ A}) .to_csv('tca_features.csv')
```

## Evaluation signals

- tca_features.csv exists and contains two blocks of features whose row counts equal the input scRNA-seq and ST sample counts respectively.
- Cell and spot identifiers from the input .csv files are preserved in the output feature matrix.
- MMD between the projected source and target distributions is lower than the MMD computed on the untransformed inputs.
- The projection is computed as A = (KLK + lambda*I)^-1 K L X, i.e. the kernel matrix K, label/constraint matrix L, and regularization lambda all enter the solution as in the TCA formulation of Pan et al.
- Both count matrices share the same gene space before the kernel matrix is constructed.

## Limitations

- Eliminating the distribution difference (batch effect) between domains via MMD minimization assumes a suitable spatial reference; the paper states the choice of spatial reference directly influences the spatial reconstruction and that pre-processing or appropriateness assessment of the reference should be considered.
- Intrinsic differences in spatial homogeneity between tissues may influence accuracy, so caution is warranted in tissues with low spatial coherence (e.g., Patients 5 and 9 of SCC).
- Kernel embedding alone produces latent features only for the cells/spots in the concatenated kernel matrix; generalizing to new samples requires the subsequent MLP model, not the TCA projection itself.

## Evidence

- [methods] TCA first projects the scRNA-seq and spatial transcriptomics data into a Reproducing Kernel Hilbert Space (RKHS), and then reduces the difference in the distribution of the transformed two domain data by minimizing the maximum mean discrepancy (MMD): [evidence span withheld — anchor status: not_found]
- [intro] scSpace enables eliminating the batch effect between single-cell and ST data and extracting the shared latent feature across these two types of data using transfer component analysis (TCA).: "scSpace enables eliminating the batch effect between single-cell and ST data and extracting the shared latent feature across these two types of data"
- [readme] To use scSpace we require five formatted `.csv` files as input (i.e. read in by pandas). We have included a toy dataset in the [vignettes/data folder](vignettes/data) of this repository as examples: "To use scSpace we require five formatted `.csv` files as input (i.e. read in by pandas). We have included a toy dataset 
in the [vignettes/data folder](vignettes/data) of this repository as examples"
- [discussion] scSpace should be applied cautiously to tissues with low spatial coherence, such as Patients 5 and 9 of SCC evaluated in this paper: "scSpace should be applied cautiously to tissues with low spatial coherence, such as Patients 5 and 9 of SCC evaluated in this paper"
