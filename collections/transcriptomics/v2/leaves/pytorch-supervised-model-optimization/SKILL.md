---
name: pytorch-supervised-model-optimization
description: Use when you already have transfer component analysis (TCA) characteristic/latent
  matrices for paired spatial transcriptomics (ST) and scRNA-seq data plus ST spot
  x-y coordinates, and you need to learn a regression mapping to produce per-cell
  pseudo-space coordinates.
when_to_use_negative:
- The input feature matrices have not yet been aligned by transfer component analysis
  (TCA) — the MLP expects the shared latent/characteristic matrices, not raw gene
  expression counts
- The goal is spatial domain identification of the ST data itself (e.g., SpaGCN, BayesSpace,
  DR-SC) rather than projecting scRNA-seq cells into a pseudo space — scSpace explicitly
  targets spatial analysis of scRNA-seq data
- The spatial reference tissue has low spatial coherence or is mismatched to the query
  tissue (e.g., SCC Patients 5 and 9, or DLPFC vs agranular dACC lacking cortical
  layer 4), which degrades reconstruction accuracy
edam_operation: http://edamontology.org/operation_0322
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: scSpace
  role: Encapsulates the TCA + MLP pipeline; this skill implements its pseudo-space
    projection stage, which regresses ST coordinates on latent features and applies
    the model to scRNA-seq latent features
  repo: https://github.com/ZJUFanLab/scSpace
- name: PyTorch
  role: Framework used to build and train the two-layer fully connected MLP with Adam
    (lr 1e-3) and MSE loss; install a version matching your machine's CUDA
  repo: https://pytorch.org/get-started/locally/
- name: pandas
  role: Reads the five formatted .csv input files (latent matrices and ST coordinates)
    and parses the final pseudo_space_coordinates.csv for validation
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-023-38121-4
    title: Reconstruction of the cell pseudo-space from single-cell RNA sequencing
      data with scSpace
  source_content_hash: sha256:e750d995e7b38535704de56cf469413d3abfcd8b316176008ba3c821f2896861
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ZJUFanLab/scSpace
  tools:
  - PyTorch
  - Python
  - pandas
  - scSpace
  edam_operation: http://edamontology.org/operation_0322
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# pytorch-supervised-model-optimization

## Summary

Trains a supervised PyTorch MLP that regresses spatial coordinates from TCA-derived shared latent features, exactly as scSpace does to project scRNA-seq cells into a pseudo space. Optimizes the mapping with Adam on an MSE loss against ST spot x-y ground truth, then reuses the frozen model for prediction on new latent features.

## When to use

Use when you already have transfer component analysis (TCA) characteristic/latent matrices for paired spatial transcriptomics (ST) and scRNA-seq data plus ST spot x-y coordinates, and you need to learn a regression mapping to produce per-cell pseudo-space coordinates.

## When NOT to use

- The input feature matrices have not yet been aligned by transfer component analysis (TCA) — the MLP expects the shared latent/characteristic matrices, not raw gene expression counts
- The goal is spatial domain identification of the ST data itself (e.g., SpaGCN, BayesSpace, DR-SC) rather than projecting scRNA-seq cells into a pseudo space — scSpace explicitly targets spatial analysis of scRNA-seq data
- The spatial reference tissue has low spatial coherence or is mismatched to the query tissue (e.g., SCC Patients 5 and 9, or DLPFC vs agranular dACC lacking cortical layer 4), which degrades reconstruction accuracy

## Inputs

- TCA-extracted characteristic (latent) matrix for the spatial transcriptomics reference (.csv)
- TCA-extracted characteristic (latent) matrix for the scRNA-seq data (.csv)
- ST spot x-y spatial coordinates table (.csv)
- Five formatted .csv scSpace input files (read in with pandas)
- Demo/toy dataset available in the vignettes/data folder of the scSpace repository

## Outputs

- Trained PyTorch MLP mapping latent features to 2D spatial coordinates
- pseudo_space_coordinates.csv: one coordinate row (finite x, y) per input scRNA-seq cell

## How to apply

Load the TCA-extracted characteristic (latent) matrices for the ST reference and the scRNA-seq data, together with the ST spot x-y coordinates, from the five formatted .csv input files with pandas. Build a two-layer fully connected MLP in PyTorch whose first layer has 128 neurons with sigmoid/ReLU activations, taking the ST latent feature matrix as input and regressing the 2D spatial coordinates. Train with the Adam optimizer (learning rate 1e-3) minimizing mean squared error (MSE) loss against the ST spatial coordinates, monitoring that the loss decreases to a converged plateau. Freeze the trained weights and apply the model to the scRNA-seq latent feature matrix so every single cell receives a predicted coordinate pair. Write the per-cell predictions to pseudo_space_coordinates.csv, preserving one row per input scRNA-seq cell. Rationale: this regression transfers the spatial coordinate system learned from the ST reference onto the scRNA-seq latent space, since TCA already aligned the two feature distributions by removing the batch effect between them.

## Related tools

- **scSpace** (Encapsulates the TCA + MLP pipeline; this skill implements its pseudo-space projection stage, which regresses ST coordinates on latent features and applies the model to scRNA-seq latent features) — https://github.com/ZJUFanLab/scSpace
- **PyTorch** (Framework used to build and train the two-layer fully connected MLP with Adam (lr 1e-3) and MSE loss; install a version matching your machine's CUDA) — https://pytorch.org/get-started/locally/
- **pandas** (Reads the five formatted .csv input files (latent matrices and ST coordinates) and parses the final pseudo_space_coordinates.csv for validation)

## Examples

```
opt = torch.optim.Adam(model.parameters(), lr=1e-3); loss = torch.nn.functional.mse_loss(model(X_st_latent), xy_st); opt.zero_grad(); loss.backward(); opt.step()
```

## Evaluation signals

- pseudo_space_coordinates.csv exists and parses with pandas without error
- Output contains exactly one coordinate row per input scRNA-seq cell (row count matches the scRNA-seq latent matrix)
- All predicted x and y coordinate values are finite (no NaN/Inf)
- Training MSE loss against ST spatial coordinates converges to a low plateau during Adam optimization
- Predicted coordinate spread for scRNA-seq cells falls within a range consistent with the ST reference spot coordinates

## Limitations

- The choice of spatial reference directly influences the spatial reconstruction; an appropriateness assessment or pre-processing of the reference should precede model fitting
- Intrinsic differences in spatial homogeneity between tissues may reduce accuracy — apply cautiously to tissues with low spatial coherence (e.g., SCC Patients 5 and 9)
- For scATAC-seq inputs the data must first be converted to a gene activity matrix (e.g., with Signac) before latent feature extraction and MLP projection

## Evidence

- [methods] MLP regression stage grounds the pseudo-space projection: "project single cells into a pseudo space via a multiple layer perceptron (MLP) model"
- [readme] TCA precondition for the supervised mapping: "Using [transfer component analysis (TCA)](https://ieeexplore.ieee.org/stamp/stamp.jsp?tp=&arnumber=5640675&tag=1), scSpace could extract the characteristic matrixes of spatial transcriptomics and"
- [readme] Purpose of the jointly embedded representation: "so that gene expression and spatial weight of cells can be embedded jointly for the further cell typing with higher accuracy and precision"
- [discussion] Reference choice caveat that bounds correct application: "Since the choice of spatial reference would directly influence the spatial reconstruction"
