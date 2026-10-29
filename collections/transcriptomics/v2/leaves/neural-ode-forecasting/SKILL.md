---
name: neural-ode-forecasting
description: Use when you have paired time-point scRNA-seq count matrices from the
  same cells (e.g., metabolic labeling scEU-seq) and need to predict future single-cell
  expression states beyond the immediate observed changes that RNA velocity provides.
when_to_use_negative:
- Only standard scRNA-seq data are available with no paired time-point measurements
  of the same cell (RNAForecaster requires such paired temporal data)
- The goal is a lower-dimensional embedding-based forecast — VAE-encoded input was
  shown to significantly degrade accuracy (higher MSE, p < 1e-7), especially at early
  time points
- Predictions must be constrained to remain within the UMAP or gene-expression space
  observed in the training dataset (RNAForecaster is designed to predict states outside
  the training space, e.g., post-KO)
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: RNAForecaster
  role: Core package providing trainRNAForecaster, saveForecaster, and predictCellFutures
    for training the neural ODE and recursive forecasting
  repo: https://github.com/rossinerbe/RNAForecaster.jl
- name: Flux.jl
  role: Machine-learning framework in which the neural ODE network is implemented
    and trained
- name: DiffEqFlux.jl
  role: Couples the network with the Tsit5 ODE solver for neural ODE training
- name: BoolODE
  role: Generates simulated single-cell expression trajectories used to validate forecast
    accuracy (with minor changes in RNAForecasterPaperCode)
  repo: https://github.com/FertigLab/RNAForecasterPaperCode
- name: Seurat
  role: Joint UMAP embedding of predicted and simulated expression profiles to verify
    co-clustering
provenance:
  source_task_ids:
  - task_005
  - task_008
  - task_004
  source_papers:
  - doi: 10.1016/j.patter.2023.100793
    title: Transcriptomic forecasting with neural ordinary differential equations
  source_content_hash: sha256:9ff62019650f1e7dc8423505fd06f1564115aff8ecd319498eae3f39bf027f03
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
  - DiffEqFlux.jl
  - Flux
  - Flux.jl
  - RNAForecaster
  - Seurat
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8095238095238095
n_claims: 21
---

# neural-ode-forecasting

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Train a neural ODE (RNAForecaster) on paired single-cell count matrices from two time points and recursively forecast future gene expression states, using an ensemble with median aggregation to predict transcriptomic trajectories over short-to-intermediate time scales.

## When to use

Use when you have paired time-point scRNA-seq count matrices from the same cells (e.g., metabolic labeling scEU-seq) and need to predict future single-cell expression states beyond the immediate observed changes that RNA velocity provides.

## When NOT to use

- Only standard scRNA-seq data are available with no paired time-point measurements of the same cell (RNAForecaster requires such paired temporal data)
- The goal is a lower-dimensional embedding-based forecast — VAE-encoded input was shown to significantly degrade accuracy (higher MSE, p < 1e-7), especially at early time points
- Predictions must be constrained to remain within the UMAP or gene-expression space observed in the training dataset (RNAForecaster is designed to predict states outside the training space, e.g., post-KO)

## Inputs

- Paired scRNA-seq count matrices from two time points of the same cells (e.g., scEU-seq rpeLabeling.h5ad / rpeExampleData.jld2 log1p-normalized t0/t1 matrices)
- BoolODE simulated expression matrices (e.g., 111 random 10-gene network simulations with 801 time points, 2000 cells each)
- Pre-KO and post-KO simulation count matrices for perturbation forecasting

## Outputs

- Trained RNAForecaster network or ensemble saved as a .jld2 file (via saveForecaster)
- Recursive per-cell expression forecasts for future time points (e.g., 50 post-KO time points, 72 hourly cell-cycle steps)
- Per-time-point MSE of forecasts against simulated or observed trajectories
- Median total counts per cell after N forecast steps (e.g., median_total_counts_72h.csv)

## How to apply

Preprocess counts with sparsity filtering (remove genes with >98% zero counts), log normalization, and filtering to top-quartile variance genes, converting matrices to Float32. Split data 80-20 into training/validation sets, then train the neural ODE with trainRNAForecaster (Flux.jl/DiffEqFlux.jl on Julia 1.6, Tsit5 solver, default error tolerance 1e-3, 10 epochs, e.g. learningRate=0.0001, batchsize=25) using earlier counts as input and later counts as target. Save the trained network with saveForecaster to a .jld2 file; optionally train a 10-network ensemble whose forecasts are combined via the median prediction. Forecast with predictCellFutures by feeding each step's output back into the input nodes recursively, applying the capping filter at every step (predictions above two times the maximum observed in training data in log space are set to that maximum; negative predictions clamped to zero). Keep input in gene-dimensional space rather than a lower-dimensional embedding, since VAE-encoded input was shown to significantly increase forecast MSE (p < 1e-7) due to information loss.

## Related tools

- **RNAForecaster** (Core package providing trainRNAForecaster, saveForecaster, and predictCellFutures for training the neural ODE and recursive forecasting) — https://github.com/rossinerbe/RNAForecaster.jl
- **Flux.jl** (Machine-learning framework in which the neural ODE network is implemented and trained)
- **DiffEqFlux.jl** (Couples the network with the Tsit5 ODE solver for neural ODE training)
- **BoolODE** (Generates simulated single-cell expression trajectories used to validate forecast accuracy (with minor changes in RNAForecasterPaperCode)) — https://github.com/FertigLab/RNAForecasterPaperCode
- **Seurat** (Joint UMAP embedding of predicted and simulated expression profiles to verify co-clustering)

## Examples

```
julia> using RNAForecaster; net = trainRNAForecaster(t0, t1, learningRate=0.0001, batchsize=25); saveForecaster(net, "forecaster.jld2"); preds = predictCellFutures(net, t0, 72)
```

## Evaluation signals

- Forecast MSE of the ensemble against simulated future time points should be comparable to or better than controls (e.g., KO ensemble MSE comparable to non-KO ensemble), not worsening over recursive steps
- The VAE-input variant should show significantly HIGHER MSE than the by-gene ensemble (p < 1e-7), confirming gene-space input is required
- Median total counts per cell at 72 h of recursive forecasting should remain at a realistic, non-exploding level (Figure 6B behavior), confirming the cap is applied at every step
- Predicted post-KO expression profiles should co-cluster with simulated KO cells in joint UMAP space (Seurat 4.0.1)
- Trained .jld2 forecaster files load cleanly and reproduce forecasts when re-applied to the t0 input matrix

## Limitations

- Training data of only 405 cells may fail to capture rare cell-cycle events; a larger training set might be needed
- The current network structure may lack the capability to predict cell-cycle checkpoints and rare regulatory exceptions; attention-based architectures are proposed as future work
- The prior on expression outside the observed range is enforced post hoc via capping rather than learned by the network
- Ensembling increases the computational resources required
- Metabolic labeling protocols required for paired time-point data are largely confined to in vitro studies, limiting in vivo application

## Evidence
