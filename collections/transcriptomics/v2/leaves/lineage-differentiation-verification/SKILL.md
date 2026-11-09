---
name: lineage-differentiation-verification
description: Use when validating that a neural ODE forecaster trained on paired time
  points of a bifurcating single-cell simulation reproduces lineage commitment, or
  when testing how far before a fate switch training data stays predictive.
when_to_use_negative:
- 'Training time points sit well before the fate switch (e.g., BoolODE time points
  250/251): recursive forecasts break down at the bifurcation and form a new cluster
  absent from the simulation, so this skill cannot certify long-horizon fate prediction.'
- Input is standard scRNA-seq without temporally paired measurements of the same cells;
  RNAForecaster requires paired time-point data that only simulations or metabolic
  labeling protocols provide.
- The analysis goal is only ordering cells along a trajectory (pseudotime); trajectory
  inference methods, not forecasting, are the appropriate choice.
edam_operation: http://edamontology.org/operation_2423
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0203
tools:
- name: RNAForecaster
  role: Provides trainRNAForecaster to fit the neural ODE on paired time points and
    predictCellFutures to recursively forecast 100 future time points for the lineage-split
    check
  repo: https://github.com/rossinerbe/RNAForecaster.jl
- name: Flux
  role: Julia machine-learning package used to train the neural ODE forecaster network
- name: DiffEqFlux
  role: Trains the neural ODE using the Tsit5 ODE solver against the t+1 expression
    matrix
- name: Seurat
  role: Produces UMAP visualizations of the recursively predicted expression states
    to confirm the two-lineage split
- name: BoolODE
  role: Generates the bifurcating-lineage simulation supplying the temporally paired
    training and target matrices at time points 365/366 and 250/251
  repo: https://github.com/FertigLab/RNAForecasterPaperCode
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.patter.2023.100793
    title: Transcriptomic forecasting with neural ordinary differential equations
  source_content_hash: sha256:327e79f665571224cadbc42e42ec4e405d25c5226947fa0f68495f0236bdeb39
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/FertigLab/RNAForecaster.jl
  repo_url: https://github.com/rossinerbe/RNAForecaster.jl
  tools:
  - BoolODE
  - DiffEqFlux
  - Flux
  - RNAForecaster
  - Seurat
  edam_operation: http://edamontology.org/operation_2423
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# lineage-differentiation-verification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Verifies that RNAForecaster neural ODE forecasts reproduce a bifurcating lineage split by training on two adjacent BoolODE simulated time points immediately before the bifurcation and recursively forecasting 100 steps, contrasted with training well before the split to expose the limited prediction horizon.

## When to use

Use when validating that a neural ODE forecaster trained on paired time points of a bifurcating single-cell simulation reproduces lineage commitment, or when testing how far before a fate switch training data stays predictive.

## When NOT to use

- Training time points sit well before the fate switch (e.g., BoolODE time points 250/251): recursive forecasts break down at the bifurcation and form a new cluster absent from the simulation, so this skill cannot certify long-horizon fate prediction.
- Input is standard scRNA-seq without temporally paired measurements of the same cells; RNAForecaster requires paired time-point data that only simulations or metabolic labeling protocols provide.
- The analysis goal is only ordering cells along a trajectory (pseudotime); trajectory inference methods, not forecasting, are the appropriate choice.

## Inputs

- BoolODE bifurcating-lineage simulated count matrices (genes as rows, cells as columns) at adjacent simulated time points (365 and 366; 250 and 251)
- Two temporally paired single-cell expression matrices (input time t, target time t+1) after >98% zero-count gene filtering and log1p normalization (Float32)

## Outputs

- 100-step recursive future expression predictions per input cell (futureExpressionPreds)
- UMAP projection (Seurat 4.0.1) of predicted states showing two-lineage separation after bifurcation, or forecast breakdown with a spurious cluster for the 250/251 control

## How to apply

Load the BoolODE bifurcating-lineage count matrices (genes as rows, cells as columns), remove genes with >98% zero counts, log1p-transform, and cast to Float32. Train the neural ODE with trainRNAForecaster (Flux 0.13.0/DiffEqFlux 1.46.0 on Julia 1.6, Tsit5 ODE solver, default 80-20 train/validation split) using the time-365 expression matrix as input and time-366 as target, then call predictCellFutures to recursively predict 100 future time points by feeding each output back into the input layer, capping predictions at the allowed maximum (2x the maximum observed in training data, in log space). Project predicted states with UMAP in Seurat 4.0.1 and confirm predicted cells split into the two simulated lineages after the bifurcation (qualitative comparison as in Figure 3). As the negative control, repeat training with matrices from time points 250/251, well before the bifurcation: forecasts should break down at the bifurcation, forming a new cluster absent from the simulation, demonstrating that accuracy holds only over a limited prediction horizon.

## Related tools

- **RNAForecaster** (Provides trainRNAForecaster to fit the neural ODE on paired time points and predictCellFutures to recursively forecast 100 future time points for the lineage-split check) — https://github.com/rossinerbe/RNAForecaster.jl
- **Flux** (Julia machine-learning package used to train the neural ODE forecaster network)
- **DiffEqFlux** (Trains the neural ODE using the Tsit5 ODE solver against the t+1 expression matrix)
- **Seurat** (Produces UMAP visualizations of the recursively predicted expression states to confirm the two-lineage split)
- **BoolODE** (Generates the bifurcating-lineage simulation supplying the temporally paired training and target matrices at time points 365/366 and 250/251) — https://github.com/FertigLab/RNAForecasterPaperCode

## Examples

```
network = trainRNAForecaster(exprT365, exprT366); futureExpressionPreds = predictCellFutures(network, exprT365, 100)
```

## Evaluation signals

- UMAP of the 365/366-trained forecasts reproduces the two-lineage differentiation seen in the simulation after the bifurcation (qualitative Figure 3 comparison).
- UMAP of the 250/251-trained forecasts breaks down at the bifurcation into a new cluster absent from the simulation, confirming the limited prediction horizon is reproduced.
- Preprocessing invariant holds: genes with >98% zero counts are removed and matrices are log1p-transformed and cast to Float32 before training.
- No recursive prediction exceeds the allowed maximum (default two times the maximum observed in the training data, in log space).
- Training uses the default 80-20 train/validation split with the Tsit5 solver and MSE loss against the later time-point matrix.

## Limitations

- Accuracy holds only over a limited prediction horizon: models trained well before the bifurcation (time points 250/251) fail to anticipate lineage commitment and produce a cluster absent from the simulation.
- Standard scRNA-seq cannot provide the paired time-point measurements of the same cell required; metabolic labeling protocols are largely confined to in vitro studies, limiting in vivo application.
- The limits of predictability over diverse timescales and biological conditions have not been evaluated.
- Ensembling networks and using the median prediction increases the required computational resources.

## Evidence
