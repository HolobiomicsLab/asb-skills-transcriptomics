---
name: mlp-baseline-model-comparison
description: Use when evaluating whether a neural ODE transcriptomic forecaster genuinely
  outperforms a standard multi-layer MLP on paired two-time-point single-cell data,
  i.e., a controlled prediction-accuracy (MSE) ablation on held-out cells is needed.
when_to_use_negative:
- Input is unpaired standard scRNA-seq with no two-time-point measurements of the
  same cell population, since neither the neural ODE nor the paired benchmark can
  be trained on t=0 -> t=1 transitions
- The goal is mechanistic gene regulatory network inference rather than forecasting
  accuracy; the article states network inference from RNAForecaster remains future
  work
- Predictions must extend beyond short time scales without validation data; the article
  has not evaluated limits of predictability over diverse timescales and biological
  conditions
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: RNAForecaster
  role: Neural ODE forecaster being benchmarked; trained with trainRNAForecaster and
    recursively forecast with predictCellFutures
  repo: https://github.com/rossinerbe/RNAForecaster.jl
- name: Flux.jl
  role: Machine-learning framework used to define and train both the neural ODE and
    the five-hidden-layer MLP baseline
- name: DiffEqFlux.jl
  role: Provides the neural ODE training layer used with the Tsit5 ODE solver
- name: Julia 1.6
  role: Implementation language for RNAForecaster and the benchmark comparison scripts
  repo: https://julialang.org/
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.patter.2023.100793
    title: Transcriptomic forecasting with neural ordinary differential equations
  source_content_hash: sha256:7d9501b69fd9ed195aa1c13b108c90f5456ff68787b73efe842d038a71267c18
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# mlp-baseline-model-comparison

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmarking procedure in which the RNAForecaster neural ODE (trained with Flux.jl/DiffEqFlux.jl via the Tsit5 solver) is compared against a five-hidden-layer MLP baseline on matched training/validation splits, using validation-set MSE at t=1 and across recursive future predictions to quantify the accuracy benefit of the neural ODE formulation.

## When to use

Use when evaluating whether a neural ODE transcriptomic forecaster genuinely outperforms a standard multi-layer MLP on paired two-time-point single-cell data, i.e., a controlled prediction-accuracy (MSE) ablation on held-out cells is needed.

## When NOT to use

- Input is unpaired standard scRNA-seq with no two-time-point measurements of the same cell population, since neither the neural ODE nor the paired benchmark can be trained on t=0 -> t=1 transitions
- The goal is mechanistic gene regulatory network inference rather than forecasting accuracy; the article states network inference from RNAForecaster remains future work
- Predictions must extend beyond short time scales without validation data; the article has not evaluated limits of predictability over diverse timescales and biological conditions

## Inputs

- BoolODE simulated paired t=0/t=1 count matrices (genes-as-rows, cells-as-columns; random 10-gene networks)
- Preprocessed Float32, log1p-normalized expression matrices split 80-20 into training and validation sets

## Outputs

- Validation-set MSE at t=1 for the neural ODE and the five-hidden-layer MLP
- Recursive prediction MSE across 50 future time points for both models
- MSE comparison table (boolode_mse_comparison.csv) with significance of the difference

## How to apply

Load the paired t=0/t=1 count matrices (genes as rows, cells as columns), preprocess to Float32 with log1p normalization, and partition into the default 80-20 training/validation split so both models see identical data. Train the RNAForecaster neural ODE with trainRNAForecaster (Tsit5 solver, default 10 epochs, MSE loss) against the t=1 target matrix, and independently train a five-hidden-layer MLP on the same training split. Compute validation-set MSE at t=1 for both models, then use predictCellFutures to recursively forecast 50 future time points by feeding each model's outputs back into its input nodes, computing MSE at every predicted step. Tabulate the MSE comparison (e.g., boolode_mse_comparison.csv) and assess significance; in the paper's BoolODE random 10-gene-network simulations the neural ODE significantly outperformed the MLP at t=1 (p < 1e-16) and at all 50 recursive steps (p < 1e-11), with error propagating faster and extreme outliers appearing earlier in MLP predictions.

## Related tools

- **RNAForecaster** (Neural ODE forecaster being benchmarked; trained with trainRNAForecaster and recursively forecast with predictCellFutures) — https://github.com/rossinerbe/RNAForecaster.jl
- **Flux.jl** (Machine-learning framework used to define and train both the neural ODE and the five-hidden-layer MLP baseline)
- **DiffEqFlux.jl** (Provides the neural ODE training layer used with the Tsit5 ODE solver)
- **Julia 1.6** (Implementation language for RNAForecaster and the benchmark comparison scripts) — https://julialang.org/

## Examples

```
julia -e 'using Pkg; Pkg.add("RNAForecaster")'
```

## Evaluation signals

- Both models are trained and evaluated on the identical 80-20 train/validation split of the same preprocessed matrix
- Neural ODE validation MSE is significantly lower than the five-hidden-layer MLP at t=1 (paper reports p < 1e-16)
- Neural ODE MSE remains significantly lower at every one of the 50 recursive prediction steps (paper reports p < 1e-11)
- MLP error grows faster over recursive steps, with extreme outliers appearing earlier than in the neural ODE predictions (Figures 2A-2C)
- Comparison table (boolode_mse_comparison.csv) contains per-step MSE rows for both models

## Limitations

- Ensembling network predictions (median across networks) increases the computational resources required
- Expression predictions above an allowed maximum (default two times the maximum observed in training data, in log space) are capped post hoc rather than learned
- Benchmarked only on BoolODE simulations of random 10-gene networks (111 of 117 simulations passing stability checks) and short-term biological data; predictability limits over longer timescales were not evaluated

## Evidence
