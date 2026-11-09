---
name: seed-varied-ensemble-network-training
description: Use when evaluating whether ensembling seed-varied neural ODE forecasts
  improves accuracy over a single RNAForecaster network, or when forecasting simulated/BoolODE
  single-cell expression futures where single-network log MSE must be reduced at later
  time points.
when_to_use_negative:
- When only a single trained network is affordable and the ensemble's increased computational
  cost is prohibitive — the article notes 'the increased computational resources required'
  as the downside of ensembling
- When the dataset lacks paired time-point measurements of the same cells (e.g., standard
  scRNA-seq), since RNAForecaster training requires t0/t1 matrix pairs
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_2269
tools:
- name: RNAForecaster
  role: Neural ODE forecaster whose trainRNAForecaster and predictCellFutures functions
    train the seed-varied networks and generate recursive forecasts combined by median
  repo: https://github.com/rossinerbe/RNAForecaster.jl
- name: Julia
  role: Implementation language for RNAForecaster (version 1.6 or later)
- name: Flux.jl
  role: Machine-learning package used to train the neural ODE networks with different
    gradient-descent seeds
- name: DiffEqFlux.jl
  role: Differential-equation package used with the Tsit5 ODE solver during network
    training
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.patter.2023.100793
    title: Transcriptomic forecasting with neural ordinary differential equations
  source_content_hash: sha256:0c86fea5aaa0858f7f809053313971e2ee63b088d79754ad7377194296900fe8
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
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# seed-varied-ensemble-network-training

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Train multiple RNAForecaster neural ODEs on the same paired time-point expression matrices, each initialized with a different gradient-descent seed (ensembles of 10 and 25 networks), and combine member predictions by taking the median to improve forecast accuracy over a single network on simulated data.

## When to use

Use when evaluating whether ensembling seed-varied neural ODE forecasts improves accuracy over a single RNAForecaster network, or when forecasting simulated/BoolODE single-cell expression futures where single-network log MSE must be reduced at later time points.

## When NOT to use

- When only a single trained network is affordable and the ensemble's increased computational cost is prohibitive — the article notes 'the increased computational resources required' as the downside of ensembling
- When the dataset lacks paired time-point measurements of the same cells (e.g., standard scRNA-seq), since RNAForecaster training requires t0/t1 matrix pairs

## Inputs

- BoolODE simulated expression matrices from random 10-gene networks (801 time points, 2,000 cells per simulation; 111 simulations after stability filtering)
- Paired t0/t1 expression matrices formatted with genes as rows and cells as columns, log1p-transformed Float32

## Outputs

- Recursive forecasts (50 time points) from single-network and each median-combined 10- and 25-network ensemble
- Per-time-point log MSE values and paired significance test statistics written to ensemble_log_mse_results.csv

## How to apply

Load BoolODE simulations of random 10-gene networks (801 time points, 2,000 cells each; exclude simulations failing stability checks on 15 iterations, leaving 111), format paired t0/t1 matrices (genes as rows, cells as columns) as log1p-transformed Float32, and apply sparsity filtering (remove genes with >98% zero counts). Train a single RNAForecaster network with trainRNAForecaster (Tsit5 solver, error tolerance 1e-3, default 10 epochs, 80-20 train/validation split), then train additional ensembles of 10 and 25 networks on the same data, each with a different gradient-descent seed. Generate recursive forecasts with predictCellFutures for 50 time points from the single network and every ensemble member, combining each ensemble's predictions by taking the median across networks. Compute log mean squared error against the simulated future expression states per forecast time point, and test significance of the ensemble advantage by time point (paired tests per simulation), writing per-time-point log MSE values and test statistics to ensemble_log_mse_results.csv.

## Related tools

- **RNAForecaster** (Neural ODE forecaster whose trainRNAForecaster and predictCellFutures functions train the seed-varied networks and generate recursive forecasts combined by median) — https://github.com/rossinerbe/RNAForecaster.jl
- **Julia** (Implementation language for RNAForecaster (version 1.6 or later))
- **Flux.jl** (Machine-learning package used to train the neural ODE networks with different gradient-descent seeds)
- **DiffEqFlux.jl** (Differential-equation package used with the Tsit5 ODE solver during network training)

## Examples

```
julia> using RNAForecaster; networks = [trainRNAForecaster(t0, t1; seed=s) for s in 1:25]; preds = predictCellFutures(median_ensemble(networks), 50)
```

## Evaluation signals

- Ensemble log MSE is significantly lower than single-network log MSE by t = 10 (p < 1e-6), with no significant difference at t = 1
- Log MSE of the median ensemble remains below 0.7 out to 200 forecast time points
- MSE gap between ensemble and single network grows monotonically at later forecast time points, reproducing Figures 2D-2E and S3-S4
- ensemble_log_mse_results.csv contains per-time-point log MSE values and test statistics for single, 10-network, and 25-network ensembles
- All ensemble member networks converge (training uses default 10 epochs with 80-20 train/validation split) before median combination

## Limitations

- Ensembling increases required computational resources, scaling linearly with the number of networks (10 or 25)
- No significant accuracy advantage over a single network at the earliest forecast time point (t = 1); the benefit only emerges by t = 10 and later
- Results demonstrated on BoolODE simulated 10-gene networks; stability-check failures excluded 6 of 117 simulations, so pathological random networks must be filtered before training

## Evidence
