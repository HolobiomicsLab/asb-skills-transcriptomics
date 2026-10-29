---
name: mse-loss-regressor-validation
description: Use when single-cell latent features (e.g. TCA characteristic matrices)
  must be projected into spatial coordinates by regression against a spatial transcriptomics
  reference, and per-cell coordinate outputs need validation.
when_to_use_negative:
- Input is raw gene expression rather than TCA-extracted shared characteristic matrices
  — the MLP operates on the latent features scSpace extracts via transfer component
  analysis
- The goal is spatial domain identification of ST data itself (e.g. with SpaGCN, BayesSpace,
  DR-SC) rather than reconstructing pseudo-space for scRNA-seq cells
- The tissue has low spatial coherence or the spatial reference is anatomically mismatched
  to the scRNA-seq tissue (e.g. agranular dACC vs granular DLPFC), which undermines
  reconstruction accuracy
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: scSpace
  role: Integrative algorithm whose pseudo-space reconstruction step trains the MLP
    regressor from ST latent features to spatial coordinates and applies it to scRNA-seq
    cells
  repo: github.com/ZJUFanLab/scSpace
- name: PyTorch
  role: Framework used to build the two-layer fully connected MLP and run Adam optimization
    with MSE loss
  repo: https://pytorch.org/get-started/locally/
- name: pandas
  role: Reads the five formatted .csv input files and parses the output coordinate
    file
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-023-38121-4
    title: Reconstruction of the cell pseudo-space from single-cell RNA sequencing
      data with scSpace
  source_content_hash: sha256:e5b72557755784afc1fe5599273df156f9072eae913c99ad4aaf53dab239382e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - PyTorch
  - Python
  - pandas
  - scSpace
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# MSE-loss regressor validation

## Summary

Trains and validates a multi-layer perceptron regressor that maps shared latent features to spatial coordinates, using Adam with mean squared error loss, as done in scSpace's pseudo-space reconstruction of scRNA-seq data.

## When to use

Use when single-cell latent features (e.g. TCA characteristic matrices) must be projected into spatial coordinates by regression against a spatial transcriptomics reference, and per-cell coordinate outputs need validation.

## When NOT to use

- Input is raw gene expression rather than TCA-extracted shared characteristic matrices — the MLP operates on the latent features scSpace extracts via transfer component analysis
- The goal is spatial domain identification of ST data itself (e.g. with SpaGCN, BayesSpace, DR-SC) rather than reconstructing pseudo-space for scRNA-seq cells
- The tissue has low spatial coherence or the spatial reference is anatomically mismatched to the scRNA-seq tissue (e.g. agranular dACC vs granular DLPFC), which undermines reconstruction accuracy

## Inputs

- Five formatted .csv files readable by pandas: TCA characteristic matrix for the ST reference, TCA characteristic matrix for scRNA-seq, and ST spot x-y coordinates
- ST spatial coordinates (x, y per spot) as regression targets
- scRNA-seq latent feature matrix to which the trained model is applied

## Outputs

- pseudo_space_coordinates.csv with one x, y coordinate row per input scRNA-seq cell
- Trained PyTorch MLP model mapping latent features to spatial coordinates

## How to apply

Load the TCA-extracted characteristic (latent) matrices for the spatial transcriptomics reference and the scRNA-seq data, together with ST spot x-y coordinates, from the five formatted .csv input files with pandas. Build a two-layer fully connected MLP in PyTorch whose first layer has 128 neurons with sigmoid/ReLU activations, taking ST latent features as input and regressing spatial coordinates. Train with the Adam optimizer (learning rate 1e-3) minimizing mean squared error (MSE) loss against the ST spatial coordinates, then apply the trained model to the scRNA-seq latent feature matrix to project every single cell into the pseudo space. The rationale is that the MLP learns the latent-feature-to-coordinate mapping from ST spots where coordinates are known, then transfers it to cells where they are not.

## Related tools

- **scSpace** (Integrative algorithm whose pseudo-space reconstruction step trains the MLP regressor from ST latent features to spatial coordinates and applies it to scRNA-seq cells) — github.com/ZJUFanLab/scSpace
- **PyTorch** (Framework used to build the two-layer fully connected MLP and run Adam optimization with MSE loss) — https://pytorch.org/get-started/locally/
- **pandas** (Reads the five formatted .csv input files and parses the output coordinate file)

## Examples

```
import torch; model = torch.nn.Sequential(torch.nn.Linear(128, 128), torch.nn.Sigmoid(), torch.nn.Linear(128, 2)); opt = torch.optim.Adam(model.parameters(), lr=1e-3); loss = torch.nn.MSELoss()(model(X_st), y_xy)
```

## Evaluation signals

- pseudo_space_coordinates.csv exists and parses with pandas
- The output contains exactly one coordinate row per input scRNA-seq cell
- All predicted x and y coordinate values are finite (no NaN/inf)
- Training MSE loss against ST spatial coordinates decreases and converges under Adam (lr 1e-3)
- Predicted coordinates for ST spots held as targets are consistent with the known x-y layout of the reference

## Limitations

- Accuracy depends on the choice of spatial reference; scSpace should be applied cautiously to tissues with low spatial coherence (e.g. Patients 5 and 9 of SCC)
- Intrinsic differences in spatial homogeneity between tissues may influence the accuracy of the projection
- Spatial architectures can differ across regions of the same tissue (e.g. DLPFC vs agranular dACC lacking cortical layer 4), so appropriateness assessment of the reference is required before applying the model

## Evidence

- [methods] MLP pseudo-space projection with MSE training: "project single cells into a pseudo space via a [multiple layer perceptron (MLP)](https://en.wikipedia.org/wiki/Multilayer_perceptron) model"
- [methods] Latent feature extraction feeding the regressor: "Using [transfer component analysis (TCA)](https://ieeexplore.ieee.org/stamp/stamp.jsp?tp=&arnumber=5640675&tag=1), scSpace could extract the characteristic matrixes of spatial transcriptomics and"
- [readme] Input format requirement: "To use scSpace we require five formatted `.csv` files as input (i.e. read in by pandas). We have included a toy dataset"
- [discussion] Tissue coherence caveat: "scSpace should be applied cautiously to tissues with low spatial coherence, such as Patients 5 and 9 of SCC evaluated in this paper"
- [discussion] Scope distinction from spatial domain methods: "scSpace focuses on the spatial analysis of scRNA-seq data whereas other methods target the spatial domain identification of ST data"
