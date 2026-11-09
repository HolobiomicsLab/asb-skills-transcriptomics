---
name: forecast-mse-benchmarking
description: Use when a forecaster's accuracy must be quantified against known future
  expression states — e.g., recursive predictions over BoolODE trajectories, comparing
  two forecasting variants, or testing whether KO forecasts degrade.
when_to_use_negative:
- Data is real scRNA-seq without paired time points or simulated ground truth — no
  reference trajectory exists to compute MSE against
- Input is already a lower-dimensional embedding with no gene-space decoding path,
  so decoded MSE in gene space cannot be interpreted
- The question is ordering cells along a trajectory rather than validating numeric
  forecast accuracy at specific future time points
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3518
tools:
- name: RNAForecaster
  role: Neural ODE forecaster whose recursive predictions are scored by MSE against
    simulated ground truth
  repo: https://github.com/rossinerbe/RNAForecaster.jl
- name: Flux.jl
  role: Training the neural ODE and the VAE variant whose MSE time courses are compared
- name: DiffEqFlux.jl
  role: Neural ODE training with the Tsit5 solver at 1e-3 error tolerance
- name: BoolODE
  role: Generates the simulated expression trajectories serving as ground truth for
    MSE computation
  repo: https://github.com/FertigLab/RNAForecasterPaperCode
- name: Seurat
  role: UMAP embedding of predicted vs. simulated KO profiles to confirm co-clustering
    alongside MSE
provenance:
  source_task_ids:
  - task_005
  - task_004
  source_papers:
  - doi: 10.1016/j.patter.2023.100793
    title: Transcriptomic forecasting with neural ordinary differential equations
  source_content_hash: sha256:06f39513b714ec1bcb85d2b0baa975ceae2cf97701e462a31ca2fdac09bb67a9
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
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3518
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# forecast-mse-benchmarking

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmarks recursive neural-ODE transcriptomic forecasts by computing mean squared error of predicted expression against simulated ground-truth trajectories (e.g., BoolODE), enabling variant comparisons such as VAE-embedding input vs. by-gene input, or KO vs. non-KO forecasting, over all simulated time points.

## When to use

Use when a forecaster's accuracy must be quantified against known future expression states — e.g., recursive predictions over BoolODE trajectories, comparing two forecasting variants, or testing whether KO forecasts degrade.

## When NOT to use

- Data is real scRNA-seq without paired time points or simulated ground truth — no reference trajectory exists to compute MSE against
- Input is already a lower-dimensional embedding with no gene-space decoding path, so decoded MSE in gene space cannot be interpreted
- The question is ordering cells along a trajectory rather than validating numeric forecast accuracy at specific future time points

## Inputs

- BoolODE simulated count matrices (genes x cells) for random 10-gene networks with 801 time points and 2,000 cells per simulation
- log1p-normalized Float32 expression matrices (trainRNAForecaster requirement)
- Trained RNAForecaster ensemble (.jld2 file saved via saveForecaster)
- Optional VAE-encoded latent representation trained on the first two time points
- Non-KO control simulation trajectories for MSE comparison

## Outputs

- MSE time course (one MSE value per simulated time point) per forecasting variant
- Significance test comparing variant MSE to control MSE (e.g., p < 1e-7)
- UMAP embeddings (Seurat 4.0.1) of predicted vs. simulated post-KO profiles for qualitative co-clustering check

## How to apply

Load the simulated expression matrices (genes as rows, cells as columns; e.g., the 111 retained BoolODE 10-gene-network simulations out of 117, 801 time points, 2,000 cells each, with stability-failing simulations excluded), log1p-normalize to Float32, and train the RNAForecaster ensemble (Tsit5 solver, 1e-3 error tolerance, default 10 epochs) using an earlier time point as input and a later point as target. Recursively feed each prediction back through the network to forecast forward across all remaining time points, combining the 10-network ensemble via the median prediction. Compute the MSE between forecasts and the simulated expression at every simulated time point, then compare MSE time courses between the variant and control (e.g., VAE-encoded input vs. by-gene ensemble, or post-KO vs. non-KO forecasts). Acceptance for a variant is defined by the paper's reported behavior: the VAE variant should show significantly higher MSE than the by-gene ensemble (reported p < 1e-7, Figure S6), while the KO ensemble should have MSE comparable to the non-KO control with predicted profiles co-clustering with simulated KO cells in UMAP (Seurat 4.0.1) space.

## Related tools

- **RNAForecaster** (Neural ODE forecaster whose recursive predictions are scored by MSE against simulated ground truth) — https://github.com/rossinerbe/RNAForecaster.jl
- **Flux.jl** (Training the neural ODE and the VAE variant whose MSE time courses are compared)
- **DiffEqFlux.jl** (Neural ODE training with the Tsit5 solver at 1e-3 error tolerance)
- **BoolODE** (Generates the simulated expression trajectories serving as ground truth for MSE computation) — https://github.com/FertigLab/RNAForecasterPaperCode
- **Seurat** (UMAP embedding of predicted vs. simulated KO profiles to confirm co-clustering alongside MSE)

## Examples

```
julia> ensemble = trainRNAForecaster(t0Data, t1Data; perturbationLevels=[0.0]); preds = predictCellFutures(ensemble, t0Data, 50); mse = mean((medianPreds .- t1Data).^2)
```

## Evaluation signals

- MSE is computed at every simulated time point, not just the training pair, so the full MSE time course must be produced
- VAE-input variant should show significantly higher MSE than the 10-network by-gene ensemble (paper reports p < 1e-7; Figure S6), especially at early time points
- KO-forecast MSE should be comparable to (not worse than) the identically trained non-KO control ensemble's MSE over 50 post-KO time points
- Predicted post-KO profiles co-cluster with simulated KO cells in UMAP space, matching Figure 4 behavior
- Simulations failing stability checks on 15 iterations were excluded, leaving 111 of 117 simulations — the benchmark set size must match

## Limitations

- Encoding input through a VAE loses information and significantly degrades accuracy, especially at early time points — benchmarks must not assume compression is neutral
- Ensembling median predictions across 10 networks increases required computational resources
- Predicted expression above the allowed maximum (default 2x max observed in training data, log space) is capped post hoc, which can mask unbounded forecast divergence in MSE
- Benchmarks are grounded in simulated BoolODE data; limits of predictability over diverse timescales and biological conditions have not been evaluated

## Evidence
