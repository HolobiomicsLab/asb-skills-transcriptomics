---
name: ensemble-median-aggregation
description: Use when forecasting future transcriptomic states from paired time-point
  single-cell expression matrices with RNAForecaster and single-network predictions
  are unstable or degrade over recursive time steps.
when_to_use_negative:
- 'When only one network''s prediction is needed at very short horizons: there is
  no significant accuracy difference between ensemble and single network at t = 1'
- When computational resources are constrained, since ensembling incurs the increased
  computational resources the paper flags as the main downside
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: RNAForecaster
  role: Provides trainRNAForecaster to train the seeded neural ODE ensemble and predictCellFutures
    for recursive forecasting; the median is taken across member predictions
  repo: https://github.com/rossinerbe/RNAForecaster.jl
- name: Flux
  role: Machine-learning backend (v0.13.0) used to train each ensemble network with
    a different gradient-descent seed
- name: DiffEqFlux
  role: Provides the neural ODE training layer (v1.46.0) with the Tsit5 ODE solver
    used for each ensemble member
- name: BoolODE
  role: Generates the simulated 10-gene-network time-series data (801 time points,
    2,000 cells) on which ensemble vs single-network MSE is benchmarked
  repo: https://github.com/FertigLab/RNAForecasterPaperCode
provenance:
  source_task_ids:
  - task_005
  - task_004
  - task_002
  source_papers:
  - doi: 10.1016/j.patter.2023.100793
    title: Transcriptomic forecasting with neural ordinary differential equations
  source_content_hash: sha256:338cc11b04c472a81aaffc350784f1aa923270a4e5ff8b6691310ccba40519da
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
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8235294117647058
n_claims: 17
---

# ensemble-median-aggregation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Trains multiple RNAForecaster neural ODE networks on the same paired t0/t1 expression matrices with different gradient-descent seeds and combines their recursive forecasts by taking the median prediction across networks, reducing forecast MSE relative to any single network.

## When to use

Use when forecasting future transcriptomic states from paired time-point single-cell expression matrices with RNAForecaster and single-network predictions are unstable or degrade over recursive time steps.

## When NOT to use

- When only one network's prediction is needed at very short horizons: there is no significant accuracy difference between ensemble and single network at t = 1
- When computational resources are constrained, since ensembling incurs the increased computational resources the paper flags as the main downside

## Inputs

- Paired log1p-transformed Float32 expression matrices from two consecutive time points (genes as rows, cells as columns), e.g. BoolODE simulated data or metabolic-labeling scRNA-seq
- Sparsity-filtered gene matrices (genes with >98% zero counts removed; filtering to top-quartile variance genes recommended)
- Number of ensemble members and per-member gradient-descent seeds

## Outputs

- A trained ensemble of 10 (or 25) neural ODE networks, saveable as a .jld2 file via saveForecaster
- Median-aggregated recursive forecasts per cell per gene across 50+ future time points (predictCellFutures)
- Per-time-point log MSE values comparing ensemble vs single-network accuracy (e.g. written to ensemble_log_mse_results.csv)

## How to apply

Format the input data as log1p-transformed Float32 count matrices with genes as rows and cells as columns, then train 10 (or up to 25) RNAForecaster neural ODEs on the same t0/t1 pair using trainRNAForecaster, each initialized with a different gradient-descent seed (Tsit5 solver, error tolerance 1e-3, default 10 epochs, 80-20 train/validation split). Generate recursive forecasts with predictCellFutures for every ensemble member, feeding each network's output back into its input layer, and combine member predictions at each time point via the median. The rationale is that gradient descent from different seeds explores different local optima, so the median suppresses individual-network noise; the accuracy advantage is not significant at t=1 but becomes significant by t=10 (p < 1e-6) and grows at later time points. Expect log MSE to remain below 0.7 out to 200 forecast time points on BoolODE-style simulations. Note the trade-off that ensembling increases required computational resources.

## Related tools

- **RNAForecaster** (Provides trainRNAForecaster to train the seeded neural ODE ensemble and predictCellFutures for recursive forecasting; the median is taken across member predictions) — https://github.com/rossinerbe/RNAForecaster.jl
- **Flux** (Machine-learning backend (v0.13.0) used to train each ensemble network with a different gradient-descent seed)
- **DiffEqFlux** (Provides the neural ODE training layer (v1.46.0) with the Tsit5 ODE solver used for each ensemble member)
- **BoolODE** (Generates the simulated 10-gene-network time-series data (801 time points, 2,000 cells) on which ensemble vs single-network MSE is benchmarked) — https://github.com/FertigLab/RNAForecasterPaperCode

## Examples

```
julia -e 'using Pkg; Pkg.add(url="https://github.com/rossinerbe/RNAForecaster.jl"); using RNAForecaster; forecaster = trainRNAForecaster(t0Matrix, t1Matrix)'
```

## Evaluation signals

- Median-ensemble log MSE is significantly lower than single-network log MSE by t = 10 (p < 1e-6), with no significant difference at t = 1
- Ensemble log MSE stays below 0.7 out to 200 recursive forecast time points
- The MSE gap between ensemble and single network grows with forecast horizon, reproducing Figures 2D-2E and S3-S4
- Every ensemble member is trained with a distinct gradient-descent seed on the same t0/t1 matrices; the combined prediction equals the element-wise median across members
- Reproducing Figure S6's baseline (by-gene ensemble) forecasts requires the same 10-network median aggregation before computing decoded MSE

## Limitations

- Ensembling increases required computational resources
- The ensemble advantage only becomes significant after multiple recursive steps; it does not improve t=1 predictions
- Expression predictions above an allowed maximum (default two times the maximum observed in the training data, in log space) are capped, bounding long-horizon forecasts

## Evidence
