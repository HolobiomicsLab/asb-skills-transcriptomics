---
name: holdout-validation-mse-benchmarking
description: Use when two paired single-cell count matrices (t=0/t=1, genes-as-rows)
  are available and a neural ODE forecaster must be quantitatively compared to an
  MLP baseline via held-out MSE at t=1 and recursive multi-step forecasts.
when_to_use_negative:
- No paired two-time-point counts for the same cells exist — standard scRNA-seq protocols
  destroy the cell, so RNAForecaster training and this benchmark cannot be applied
- Goal is purely cell ordering or embedding-space trajectory inference rather than
  gene-dimensional forward forecasting with an accuracy benchmark
- Only one trained model is of interest and no baseline comparison is needed — a single-model
  validation does not require this skill
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: RNAForecaster
  role: Neural ODE forecaster trained on t=0 to t=1 counts and used with predictCellFutures
    for the 50 recursive validation forecasts
  repo: https://github.com/rossinerbe/RNAForecaster.jl
- name: Flux.jl
  role: Machine-learning backend used to train the neural ODE and the five-hidden-layer
    MLP benchmark with MSE loss
- name: DiffEqFlux.jl
  role: Provides the neural ODE training machinery with the Tsit5 ODE solver (error
    tolerance 1e-3)
- name: Julia 1.6
  role: Implementation language for RNAForecaster and the benchmarking scripts
  repo: https://julialang.org/
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.patter.2023.100793
    title: Transcriptomic forecasting with neural ordinary differential equations
  source_content_hash: sha256:d355c74c1faf274d75507ee14d766a1842344c4814a39a5079f87cd476e9d249
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
  - DiffEqFlux.jl
  - Flux.jl
  - Julia 1.6
  - RNAForecaster
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# holdout-validation-mse-benchmarking

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark a neural ODE forecaster (RNAForecaster) against a five-hidden-layer MLP by comparing held-out validation MSE at the trained time step (t=1) and over 50 recursive future prediction steps, using paired two-time-point BoolODE simulated single-cell count matrices.

## When to use

Use when two paired single-cell count matrices (t=0/t=1, genes-as-rows) are available and a neural ODE forecaster must be quantitatively compared to an MLP baseline via held-out MSE at t=1 and recursive multi-step forecasts.

## When NOT to use

- No paired two-time-point counts for the same cells exist — standard scRNA-seq protocols destroy the cell, so RNAForecaster training and this benchmark cannot be applied
- Goal is purely cell ordering or embedding-space trajectory inference rather than gene-dimensional forward forecasting with an accuracy benchmark
- Only one trained model is of interest and no baseline comparison is needed — a single-model validation does not require this skill

## Inputs

- Paired t=0/t=1 BoolODE simulated single-cell count matrices (random 10-gene networks, genes-as-rows/cells-as-columns)
- Validation split (20% held-out cells) from the 80-20 partition
- Trained RNAForecaster network ensemble and a five-hidden-layer MLP benchmark trained on the identical training split

## Outputs

- Validation-set MSE at t=1 for the neural ODE and MLP
- Per-step validation MSE across 50 recursive predictCellFutures time points for both models
- Significance test results comparing the two models' errors
- boolode_mse_comparison.csv tabulating the MSE comparison (Figures 2A-2C)

## How to apply

Load the paired t=0/t=1 BoolODE count matrices (random 10-gene networks), preprocess to Float32 with log1p normalization, and partition into the default 80-20 training/validation split. Train the RNAForecaster neural ODE with trainRNAForecaster on t=0 to t=1 (Tsit5 solver, default 10 epochs, MSE loss, error tolerance 1e-3) and train a five-hidden-layer MLP baseline on the identical split. Compute validation-set MSE at t=1 for both models, then use predictCellFutures to recursively forecast 50 future time points (outputs fed back into inputs) and compute MSE at each step. Tabulate MSE across steps, test significance with paired statistics (article reports p < 1e-16 at t=1 and p < 1e-11 across recursive steps), and write boolode_mse_comparison.csv; error should propagate faster with earlier extreme outliers in the MLP.

## Related tools

- **RNAForecaster** (Neural ODE forecaster trained on t=0 to t=1 counts and used with predictCellFutures for the 50 recursive validation forecasts) — https://github.com/rossinerbe/RNAForecaster.jl
- **Flux.jl** (Machine-learning backend used to train the neural ODE and the five-hidden-layer MLP benchmark with MSE loss)
- **DiffEqFlux.jl** (Provides the neural ODE training machinery with the Tsit5 ODE solver (error tolerance 1e-3))
- **Julia 1.6** (Implementation language for RNAForecaster and the benchmarking scripts) — https://julialang.org/

## Examples

```
julia> using Pkg; Pkg.add("RNAForecaster")
```

## Evaluation signals

- The 80-20 train/validation split is applied before training and both models see the identical training cells
- Validation MSE at t=1 is computed for both models and the neural ODE significantly outperforms the MLP (article reports p < 1e-16)
- MSE is evaluated at every one of the 50 recursive time points, with the MLP's error propagating faster and extreme outliers appearing earlier
- Results are written to boolode_mse_comparison.csv and correspond to Figures 2A-2C
- Preprocessing is confirmed as Float32 with log1p normalization and genes with >98% zero counts removed before splitting

## Limitations

- Recursive forecasting accumulates error over time, so multi-step MSE diverges as predicted outputs are fed back into the inputs
- Expression predictions above an allowed maximum (default 2x the maximum observed in training data, log space) are capped post hoc, which affects extreme-error readings
- Benchmarks on BoolODE random 10-gene networks may not reflect behavior on biological data; only simulations passing stability checks on 15 iterations were kept (111 of 117)
- Training with the default 10 epochs and small cell counts (e.g., 405 cells in the biological application) may miss rare cell cycle events, limiting generalization of the benchmark conclusion

## Evidence
