---
name: recursive-multi-step-expression-forecasting
description: Use when you have paired single-cell count matrices (genes as rows, cells
  as columns) from two consecutive time points and need to predict future expression
  states beyond that interval, e.g., forecasting 50-100 recursive steps or anticipating
  lineage bifurcations.
when_to_use_negative:
- Only a single static scRNA-seq snapshot is available with no paired time points
  — standard protocols destroy the cell, so the required t0/t1 pairing does not exist
- The goal is ordering cells along a trajectory rather than forecasting future expression
  values forward in time (trajectory inference methods address the former)
- Long-horizon forecasting is required well beyond the trained interval, e.g., training
  before a bifurcation and expecting fate prediction across it — accuracy holds only
  over a limited prediction horizon
edam_operation: http://edamontology.org/operation_0314
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3474
tools:
- name: RNAForecaster
  role: Neural ODE forecaster trained on t0->t1 expression pairs and used to recursively
    predict future expression states via predictCellFutures
  repo: https://github.com/rossinerbe/RNAForecaster.jl
- name: Flux.jl
  role: Machine-learning framework used to construct and train the neural ODE network
- name: DiffEqFlux.jl
  role: Provides the neural ODE layer solved with the Tsit5 ODE solver during training
- name: Seurat
  role: Used to produce UMAP visualizations of recursively predicted expression states
    for validation (e.g., lineage-split inspection)
- name: BoolODE
  role: Generates simulated single-cell time-series datasets with known future states
    used to quantify recursive forecast accuracy
  repo: https://github.com/FertigLab/RNAForecasterPaperCode
provenance:
  source_task_ids:
  - task_002
  - task_003
  - task_001
  source_papers:
  - doi: 10.1016/j.patter.2023.100793
    title: Transcriptomic forecasting with neural ordinary differential equations
  source_content_hash: sha256:8a88befe4597934f37f4f6a79d8b44f57ae2aa6a7e91006cd7109a4c16798623
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
  edam_operation: http://edamontology.org/operation_0314
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8823529411764706
n_claims: 17
---

# Recursive multi-step expression forecasting

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Trains a neural ODE (RNAForecaster, Tsit5 solver) on paired t0/t1 single-cell expression matrices and then feeds each prediction back into the input layer to forecast expression over many future time points (e.g., 50-200 recursive steps).

## When to use

Use when you have paired single-cell count matrices (genes as rows, cells as columns) from two consecutive time points and need to predict future expression states beyond that interval, e.g., forecasting 50-100 recursive steps or anticipating lineage bifurcations.

## When NOT to use

- Only a single static scRNA-seq snapshot is available with no paired time points — standard protocols destroy the cell, so the required t0/t1 pairing does not exist
- The goal is ordering cells along a trajectory rather than forecasting future expression values forward in time (trajectory inference methods address the former)
- Long-horizon forecasting is required well beyond the trained interval, e.g., training before a bifurcation and expecting fate prediction across it — accuracy holds only over a limited prediction horizon

## Inputs

- Paired single-cell count matrices from two consecutive time points (genes as rows, cells as columns), e.g., BoolODE simulated t=0/t=1 matrices
- Bifurcating-lineage simulation count matrices for fate-forecasting tests (e.g., BoolODE time points 365/366)
- Metabolic-labeling or other temporally resolved scRNA-seq data compatible with RNAForecaster input

## Outputs

- Recursive expression forecasts per cell for a chosen number of future time points (e.g., 50 or 100 steps from predictCellFutures)
- Per-time-point log MSE values against simulated future expression states (e.g., ensemble_log_mse_results.csv)
- UMAP projections of recursively predicted expression states for qualitative trajectory/fate validation

## How to apply

Preprocess the paired t0/t1 matrices by removing genes with >98% zero counts, log1p-transforming, and casting to Float32; optionally filter to top-quartile variance genes. Train the RNAForecaster neural ODE with trainRNAForecaster (Flux.jl/DiffEqFlux.jl, Tsit5 ODE solver, error tolerance 1e-3, default 10 epochs, 80-20 train/validation split, MSE loss), giving the earlier time point as input and the later as target. Then call predictCellFutures to recursively forecast (e.g., 50 or 100 time points), recording each output and feeding it back into the input nodes; expression predictions above the allowed maximum (default two times the maximum observed in the training data, in log space) are capped. If accuracy is insufficient, train an ensemble of 10-25 networks with different gradient-descent seeds and combine member predictions by taking the median. Validate by comparing predicted states against known simulated future states (log MSE) or by qualitative UMAP projection (Seurat) of predicted cells against expected lineage structure.

## Related tools

- **RNAForecaster** (Neural ODE forecaster trained on t0->t1 expression pairs and used to recursively predict future expression states via predictCellFutures) — https://github.com/rossinerbe/RNAForecaster.jl
- **Flux.jl** (Machine-learning framework used to construct and train the neural ODE network)
- **DiffEqFlux.jl** (Provides the neural ODE layer solved with the Tsit5 ODE solver during training)
- **Seurat** (Used to produce UMAP visualizations of recursively predicted expression states for validation (e.g., lineage-split inspection))
- **BoolODE** (Generates simulated single-cell time-series datasets with known future states used to quantify recursive forecast accuracy) — https://github.com/FertigLab/RNAForecasterPaperCode

## Examples

```
using RNAForecaster; futures = predictCellFutures(trainedNetwork, inputMatrix, 50)
```

## Evaluation signals

- On BoolODE simulated data, recursive forecasts stay accurate: ensemble log MSE remains below 0.7 out to 200 time points
- Neural ODE validation MSE at t=1 is significantly better than a 5-hidden-layer MLP benchmark (p < 1e-16) and at all 50 recursive steps (p < 1e-11)
- When trained at time points 365/366 immediately before a simulated bifurcation, 100-step recursive forecasts reproduce the two-lineage split in UMAP; training at 250/251 should NOT anticipate the bifurcation
- Predicted expression values never exceed the allowed cap (default two times the maximum observed in training data, in log space)
- Ensemble median forecasts of 10-25 differently seeded networks show no significant advantage at t=1 but significant improvement by t=10 (p < 1e-6)

## Limitations

- Accuracy holds only over a limited prediction horizon: forecasts trained well before a bifurcation break down at the bifurcation point and can form clusters absent from the simulation
- Genes with high zero proportions (default >98%) cause problems with gradient descent and must be removed before training
- Ensembling improves accuracy but increases the required computational resources
- Standard scRNA-seq protocols cannot provide the paired time-point measurements of the same cell this forecasting requires; metabolic labeling protocols are largely confined to in vitro studies

## Evidence
