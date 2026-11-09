---
name: forecast-log-mse-error-analysis
description: Use when comparing forecast accuracy of single-network vs ensemble neural
  ODE predictions over recursive time steps, or when validating that forecast error
  stays within a bounded threshold over long horizons.
when_to_use_negative:
- The input is real biological scRNA-seq data without simulated ground truth — log
  MSE against true future states requires simulated data with known future expression
- The goal is biological interpretation or gene regulatory network inference rather
  than forecast accuracy benchmarking
- Only a single network is available with no ensemble members — the ensemble-vs-single
  comparison cannot be performed
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_2269
tools:
- name: RNAForecaster
  role: Trains single and ensemble neural ODEs and generates recursive forecasts via
    predictCellFutures whose accuracy is scored by log MSE
  repo: https://github.com/rossinerbe/RNAForecaster.jl
- name: Julia
  role: Implementation language (version 1.6+) in which the training, forecasting,
    and error analysis are executed
  repo: https://github.com/FertigLab/RNAForecasterPaperCode
- name: Flux.jl
  role: Machine-learning package used to train the neural ODE networks being benchmarked
- name: DiffEqFlux.jl
  role: Provides the neural ODE training with the Tsit5 ODE solver used in forecast
    generation
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.patter.2023.100793
    title: Transcriptomic forecasting with neural ordinary differential equations
  source_content_hash: sha256:7726fd037aab88c25d42e147fd12f07c07b575dc48d31dde83ed0bf71967ce72
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
  - Julia
  - RNAForecaster
  edam_operation: http://edamontology.org/operation_2428
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# forecast-log-mse-error-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Evaluates RNAForecaster neural ODE forecasts by computing log mean squared error of single-network versus median-pooled ensemble predictions against simulated future expression states (BoolODE 10-gene networks), and tests when the ensemble advantage becomes statistically significant.

## When to use

Use when comparing forecast accuracy of single-network vs ensemble neural ODE predictions over recursive time steps, or when validating that forecast error stays within a bounded threshold over long horizons.

## When NOT to use

- The input is real biological scRNA-seq data without simulated ground truth — log MSE against true future states requires simulated data with known future expression
- The goal is biological interpretation or gene regulatory network inference rather than forecast accuracy benchmarking
- Only a single network is available with no ensemble members — the ensemble-vs-single comparison cannot be performed

## Inputs

- BoolODE simulated dataset: 111 stable simulations of random 10-gene networks with 801 time points and 2,000 cells each (excludes simulations failing stability checks on 15 iterations)
- Paired t0/t1 expression matrices, genes as rows and cells as columns, log1p-transformed Float32
- Simulated future expression states at each forecast time point (ground truth for log MSE)

## Outputs

- Per-time-point log MSE values for single-network and median-ensemble predictions (ensemble_log_mse_results.csv)
- Statistical test statistics comparing ensemble vs single-network log MSE by forecast time point
- Verification that log MSE remains below 0.7 out to 200 time points (reproducing Figures 2D-2E and S3-S4)

## How to apply

Format paired t0/t1 expression matrices (genes as rows, cells as columns) as log1p-transformed Float32 from the BoolODE simulated dataset (111 stable simulations of random 10-gene networks, 801 time points, 2,000 cells each). Train a single RNAForecaster network with trainRNAForecaster (Tsit5 solver, error tolerance 1e-3, 10 epochs, 80-20 train/validation split), plus ensembles of 10 and 25 networks each initialized with different gradient-descent seeds. Generate recursive forecasts with predictCellFutures for 50 time points from the single network and every ensemble member, combining ensemble predictions by taking the median across networks. Compute log mean squared error of each prediction against the simulated future expression states at each forecast time point, then test significance of the ensemble advantage per time point (paired comparison across simulations) and write per-time-point log MSE values and test statistics to ensemble_log_mse_results.csv.

## Related tools

- **RNAForecaster** (Trains single and ensemble neural ODEs and generates recursive forecasts via predictCellFutures whose accuracy is scored by log MSE) — https://github.com/rossinerbe/RNAForecaster.jl
- **Julia** (Implementation language (version 1.6+) in which the training, forecasting, and error analysis are executed) — https://github.com/FertigLab/RNAForecasterPaperCode
- **Flux.jl** (Machine-learning package used to train the neural ODE networks being benchmarked)
- **DiffEqFlux.jl** (Provides the neural ODE training with the Tsit5 ODE solver used in forecast generation)

## Examples

```
julia> forecaster = trainRNAForecaster(t0, t1; solver=Tsit5(), relTol=1e-3); futures = predictCellFutures(forecaster, 50)
```

## Evaluation signals

- Median-prediction ensemble log MSE is not significantly different from single-network log MSE at t = 1 but becomes significantly lower by t = 10 (p < 1e-6)
- Log MSE gap between single-network and ensemble predictions grows at later forecast time points
- Ensemble log MSE stays below 0.7 up to 200 forecast time points
- Results reproduce Figures 2D-2E and S3-S4 of the paper
- ensemble_log_mse_results.csv contains per-time-point log MSE values and test statistics

## Limitations

- Ensembling increases the required computational resources proportionally to the number of networks trained
- The evaluation relies on BoolODE-simulated ground truth; the same bounded-error behavior on real biological data is not established here
- Some random networks generate strange regulatory behaviors — simulations failing stability checks on 15 iterations are excluded, reducing 117 simulations to 111

## Evidence

- [readme] RNAForecaster functionality is provided as a [Julia package](https://github.com/rossinerbe/RNAForecaster.jl) on GitHub.: [evidence span withheld — anchor status: external_doc]
