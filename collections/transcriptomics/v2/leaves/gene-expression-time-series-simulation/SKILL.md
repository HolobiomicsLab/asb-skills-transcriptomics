---
name: gene-expression-time-series-simulation
description: Use when a forecasting or trajectory method needs ground-truth future
  expression states for benchmarking, by simulating noisy single-cell time-series
  data with BoolODE from random gene networks.
when_to_use_negative:
- You already have real metabolic-labeling time-course data (e.g., scEU-seq 4sU-labeled
  cells) that directly provides paired temporal measurements.
- The goal is ordering cells along a trajectory rather than predicting concrete future
  expression values against known ground truth.
- Only a single static scRNA-seq snapshot is available with no temporal structure
  to emulate or validate.
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: BoolODE
  role: Generates simulated single-cell expression time series from random gene regulatory
    networks for benchmarking RNAForecaster
  repo: https://github.com/FertigLab/RNAForecasterPaperCode
- name: RNAForecaster
  role: Neural ODE forecaster trained on the first two simulated time points and evaluated
    recursively across all simulated time points
  repo: https://github.com/rossinerbe/RNAForecaster.jl
- name: Julia
  role: Language (version 1.6) in which RNAForecaster, Flux, and DiffEqFlux implement
    and run the forecasting evaluation on simulated data
  repo: https://github.com/rossinerbe/RNAForecaster.jl
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.patter.2023.100793
    title: Transcriptomic forecasting with neural ordinary differential equations
  source_content_hash: sha256:657ce018b81939102bd62259fe9057cbe962bd923ad5fc5cbc3b6e07a6688b18
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/FertigLab/RNAForecaster.jl
  repo_url: https://github.com/FertigLab/RNAForecasterPaperCode
  tools:
  - BoolODE
  - DiffEqFlux.jl
  - Flux.jl
  - Julia
  - RNAForecaster
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# gene-expression-time-series-simulation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Generate ground-truth simulated single-cell gene expression time series with BoolODE from random regulatory networks (117 random 10-gene networks, 801 time points, 2,000 cells each) so forecasting methods like RNAForecaster can be benchmarked against known future expression states.

## When to use

Use when a forecasting or trajectory method needs ground-truth future expression states for benchmarking, by simulating noisy single-cell time-series data with BoolODE from random gene networks.

## When NOT to use

- You already have real metabolic-labeling time-course data (e.g., scEU-seq 4sU-labeled cells) that directly provides paired temporal measurements.
- The goal is ordering cells along a trajectory rather than predicting concrete future expression values against known ground truth.
- Only a single static scRNA-seq snapshot is available with no temporal structure to emulate or validate.

## Inputs

- Random gene regulatory networks (10 genes per network in the paper's design)
- BoolODE simulation configuration (801 time points, 2,000 cells per simulation)
- Simulated expression matrices from the FertigLab/RNAForecasterPaperCode repository

## Outputs

- 111 stability-validated simulated expression time series (from 117 generated networks)
- Paired t=0 and t=1 expression matrices for neural ODE training
- Per-time-point MSE trajectories comparing forecast variants (e.g., by-gene ensemble vs VAE-input)

## How to apply

Sample random 10-gene regulatory networks and run BoolODE to simulate expression over 801 time points with 2,000 cells per simulation; 117 simulations are generated to guarantee at least 100 usable networks. Apply stability checks to each simulation and exclude any that fail stability checks on 15 iterations, leaving 111 simulations for downstream evaluation. Use the first two simulated time points as the t=0 and t=1 training pair for models like the RNAForecaster neural ODE (Tsit5 solver, 1e-3 error tolerance, 10 epochs), then recursively feed predictions forward and compute MSE of decoded forecasts at every remaining simulated time point. The rationale is that simulations provide the paired per-cell temporal measurements that standard scRNA-seq destroys, enabling controlled evaluation of prediction accuracy, including of variants such as VAE-encoded input.

## Related tools

- **BoolODE** (Generates simulated single-cell expression time series from random gene regulatory networks for benchmarking RNAForecaster) — https://github.com/FertigLab/RNAForecasterPaperCode
- **RNAForecaster** (Neural ODE forecaster trained on the first two simulated time points and evaluated recursively across all simulated time points) — https://github.com/rossinerbe/RNAForecaster.jl
- **Julia** (Language (version 1.6) in which RNAForecaster, Flux, and DiffEqFlux implement and run the forecasting evaluation on simulated data) — https://github.com/rossinerbe/RNAForecaster.jl

## Examples

```
julia -e 'using Pkg; Pkg.add("https://github.com/rossinerbe/RNAForecaster.jl")'
```

## Evaluation signals

- At least 100 usable simulations after filtering: 117 networks generated, 6 excluded, 111 remain.
- Every retained simulation passes stability checks (failures on 15 iterations are excluded).
- Each simulation contains 801 time points and 2,000 cells.
- Forecast MSE is computed at every simulated time point for each variant, enabling significance testing against the baseline ensemble (reported p < 1e-7 for the VAE-input comparison).

## Limitations

- Some random networks may generate strange regulatory behaviors, motivating the stability-check filter and generation of more than the minimum 100 simulations.
- Simulations exclude networks failing stability checks, so results may not generalize to unstable regulatory dynamics.
- The paper's simulated benchmark uses 10-gene networks, which is far smaller than genome-scale transcriptomic datasets.

## Evidence
