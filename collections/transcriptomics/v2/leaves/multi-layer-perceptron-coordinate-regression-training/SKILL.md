---
name: multi-layer-perceptron-coordinate-regression-training
description: 'Use when reconstructing cell pseudo-space from scRNA-seq data: TCA characteristic
  matrices for ST reference and scRNA-seq exist, plus ST spot x-y coordinates, and
  per-cell spatial coordinates must be predicted.'
when_to_use_negative:
- The goal is spatial domain identification of ST data itself (e.g., SpaGCN, BayesSpace,
  DR-SC territory) rather than projecting scRNA-seq cells into a pseudo space
- The tissue reference has low spatial coherence or differs architecturally from the
  scRNA-seq tissue (e.g., dACC lacking cortical layer 4) without an appropriateness
  assessment or pre-processing of the spatial reference
- Input data are raw expression matrices rather than TCA-extracted shared latent characteristic
  matrices — the transfer component analysis step must come first
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: scSpace
  role: Integrative algorithm whose pseudo-space reconstruction stage trains the MLP
    to regress ST coordinates from shared latent features and applies it to scRNA-seq
    cells
  repo: https://github.com/ZJUFanLab/scSpace
- name: PyTorch
  role: Framework used to build and train the two-layer fully connected MLP (Adam
    optimizer, lr 1e-3, MSE loss)
  repo: https://pytorch.org/get-started/locally/
- name: scanpy
  role: Single-cell analysis toolkit in scSpace's Python environment supporting preprocessing
    of the scRNA-seq inputs
  repo: https://github.com/scverse/scanpy
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-023-38121-4
    title: Reconstruction of the cell pseudo-space from single-cell RNA sequencing
      data with scSpace
  source_content_hash: sha256:857320552e175ad51797bef074005ce45511b0fb26f1dfe4fb4885952cf53e1b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ZJUFanLab/scSpace
  tools:
  - PyTorch
  - Python
  - scSpace
  - scanpy
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# multi-layer-perceptron-coordinate-regression-training

## Summary

Trains a two-layer fully connected MLP in PyTorch that regresses spatial x-y coordinates from TCA-extracted latent features of a spatial transcriptomics reference, then applies the trained model to scRNA-seq latent features to project single cells into a pseudo space (scSpace).

## When to use

Use when reconstructing cell pseudo-space from scRNA-seq data: TCA characteristic matrices for ST reference and scRNA-seq exist, plus ST spot x-y coordinates, and per-cell spatial coordinates must be predicted.

## When NOT to use

- The goal is spatial domain identification of ST data itself (e.g., SpaGCN, BayesSpace, DR-SC territory) rather than projecting scRNA-seq cells into a pseudo space
- The tissue reference has low spatial coherence or differs architecturally from the scRNA-seq tissue (e.g., dACC lacking cortical layer 4) without an appropriateness assessment or pre-processing of the spatial reference
- Input data are raw expression matrices rather than TCA-extracted shared latent characteristic matrices — the transfer component analysis step must come first

## Inputs

- TCA-extracted characteristic (latent) matrix for the spatial transcriptomics reference (.csv)
- TCA-extracted characteristic (latent) matrix for the scRNA-seq data (.csv)
- ST spot x-y spatial coordinates (.csv)
- Five formatted .csv input files read in with pandas

## Outputs

- Trained PyTorch MLP coordinate-regression model
- pseudo_space_coordinates.csv with one predicted x-y coordinate row per input scRNA-seq cell

## How to apply

Load the TCA-extracted characteristic (latent) matrices for the spatial transcriptomics reference and the scRNA-seq data, together with the ST spot x-y coordinates, from the five formatted .csv input files with pandas. Build a two-layer fully connected MLP in PyTorch whose first layer has 128 neurons with sigmoid/ReLU activations, taking ST latent features as input and regressing spatial coordinates. Train the MLP with the Adam optimizer (learning rate 1e-3) minimizing mean squared error (MSE) loss against the ST spatial coordinates, since coordinates are the outcome and latent features are the predictors. Then apply the trained MLP to the scRNA-seq latent feature matrix so every single cell is projected into the pseudo space, and write the per-cell predicted coordinates to pseudo_space_coordinates.csv. This projection step feeds the jointly embedded gene expression and spatial weights used by scSpace for space-informed cell typing.

## Related tools

- **scSpace** (Integrative algorithm whose pseudo-space reconstruction stage trains the MLP to regress ST coordinates from shared latent features and applies it to scRNA-seq cells) — https://github.com/ZJUFanLab/scSpace
- **PyTorch** (Framework used to build and train the two-layer fully connected MLP (Adam optimizer, lr 1e-3, MSE loss)) — https://pytorch.org/get-started/locally/
- **scanpy** (Single-cell analysis toolkit in scSpace's Python environment supporting preprocessing of the scRNA-seq inputs) — https://github.com/scverse/scanpy

## Examples

```
import torch, torch.nn as nn; model = nn.Sequential(nn.Linear(n_latent,128), nn.Sigmoid(), nn.Linear(128,2)); opt = torch.optim.Adam(model.parameters(), lr=1e-3); loss = nn.MSELoss()(model(st_latent), st_xy); loss.backward(); opt.step()
```

## Evaluation signals

- pseudo_space_coordinates.csv exists and parses with pandas
- The output contains exactly one coordinate row per input scRNA-seq cell
- All predicted x and y coordinate values are finite (no NaN/inf)
- Training MSE loss against ST spot coordinates decreases and converges under Adam with learning rate 1e-3
- The number of latent-feature columns consumed by the MLP matches the TCA characteristic matrix dimensionality of both datasets

## Limitations

- The choice of spatial reference directly influences the spatial reconstruction; scSpace should be applied cautiously to tissues with low spatial coherence (e.g., Patients 5 and 9 of SCC)
- Intrinsic differences in spatial homogeneity between tissues may influence the accuracy of scSpace
- Spatial architectures differ across regions of the same tissue (e.g., DLPFC vs dACC, which lacks cortical layer 4), so an appropriateness assessment or pre-processing of the spatial reference should be considered before applying scSpace

## Evidence

- [methods] MLP trained with spatial coordinates as outcome: "project single cells into a pseudo space via a [multiple layer perceptron (MLP)](https://en.wikipedia.org/wiki/Multilayer_perceptron) model"
- [readme] TCA characteristic matrix extraction precedes projection: "Using [transfer component analysis (TCA)](https://ieeexplore.ieee.org/stamp/stamp.jsp?tp=&arnumber=5640675&tag=1), scSpace could extract the characteristic matrixes of spatial transcriptomics and"
- [readme] CSV inputs read with pandas: "To use scSpace we require five formatted `.csv` files as input (i.e. read in by pandas)."
- [discussion] Spatial reference choice caveat: "Since the choice of spatial reference would directly influence the spatial reconstruction"
- [discussion] Low spatial coherence limitation: "scSpace should be applied cautiously to tissues with low spatial coherence, such as Patients 5 and 9 of SCC evaluated in this paper"
