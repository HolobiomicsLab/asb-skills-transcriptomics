---
name: synthetic-single-cell-simulation-training
description: Use when you need a ground-truth benchmark of transcriptomic forecasting
  accuracy where the true future expression states of cells are known, using simulated
  single-cell data (e.g. BoolODE random 10-gene networks with paired t=0/t=1 count
  matrices).
when_to_use_negative:
- The goal is forecasting real biological data without known future states — this
  skill is specifically for simulated data where ground-truth future expression is
  available (biological applications used scEU-seq or murine pancreatic development
  data instead).
- Only a single simulation/network is available and no error distribution across networks
  can be formed, preventing the paired significance testing used in the paper.
- The input lacks paired time points (earlier and later counts for the same cells),
  which the neural ODE training setup requires.
edam_operation: http://edamontology.org/operation_3936
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3474
tools:
- name: RNAForecaster
  role: Neural ODE model trained on two simulated time points (trainRNAForecaster,
    Tsit5 solver, MSE loss) and recursively forecast forward with predictCellFutures
  repo: https://github.com/rossinerbe/RNAForecaster.jl
- name: Julia
  role: Implementation language (version 1.6) in which RNAForecaster and the benchmark
    MLP are run
- name: Flux.jl
  role: Machine-learning package used to train the neural ODE
- name: DiffEqFlux.jl
  role: Package used with Flux.jl to train the neural ODE with the Tsit5 ODE solver
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.patter.2023.100793
    title: Transcriptomic forecasting with neural ordinary differential equations
  source_content_hash: sha256:55c8122bce152e6269329f40742caf3a98beeebd605fa7a187915057e917b62d
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
  edam_operation: http://edamontology.org/operation_3936
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# synthetic-single-cell-simulation-training

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark RNAForecaster's neural ODE against a five-hidden-layer MLP on BoolODE-simulated single-cell time-series (random 10-gene networks, paired t=0/t=1 matrices), evaluating held-out prediction MSE at t=1 and over 50 recursive future time points.

## When to use

Use when you need a ground-truth benchmark of transcriptomic forecasting accuracy where the true future expression states of cells are known, using simulated single-cell data (e.g. BoolODE random 10-gene networks with paired t=0/t=1 count matrices).

## When NOT to use

- The goal is forecasting real biological data without known future states — this skill is specifically for simulated data where ground-truth future expression is available (biological applications used scEU-seq or murine pancreatic development data instead).
- Only a single simulation/network is available and no error distribution across networks can be formed, preventing the paired significance testing used in the paper.
- The input lacks paired time points (earlier and later counts for the same cells), which the neural ODE training setup requires.

## Inputs

- BoolODE-simulated single-cell time-series dataset: random 10-gene networks with paired t=0 and t=1 count matrices (genes as rows, cells as columns)
- Preprocessed Float32 log1p-normalized expression matrices split 80-20 into training and validation sets

## Outputs

- Validation-set MSE at t=1 for the RNAForecaster neural ODE and the five-hidden-layer MLP
- Validation MSE across 50 recursively predicted future time points for both models (boolode_mse_comparison.csv)
- Statistical comparison of model errors (paired tests; reported p < 1e-16 at t=1 and p < 1e-11 at all 50 recursive steps)

## How to apply

Load BoolODE-simulated random 10-gene network data (paired t=0/t=1 count matrices, genes-as-rows/cells-as-columns), preprocess to Float32 with log1p normalization, and apply sparsity filtering (remove genes with >98% zero counts) to avoid gradient-descent problems. Partition into the default 80-20 training/validation split, then train the RNAForecaster neural ODE with trainRNAForecaster on t=0 -> t=1 using the Tsit5 ODE solver (default 10 epochs, MSE loss, error tolerance 1e-3). Train a five-hidden-layer MLP baseline on the identical training split, compute validation-set MSE at t=1 for both models, and recursively forecast 50 future time points with predictCellFutures by feeding each output back into the input nodes. The article used 117 simulations (111 retained after stability checks passed on 15 iterations) so that per-model MSE distributions could be compared with paired tests. Tabulate the MSE comparison (writing boolode_mse_comparison.csv) and confirm the neural ODE outperforms the MLP, as reported for Figures 2A-2C.

## Related tools

- **RNAForecaster** (Neural ODE model trained on two simulated time points (trainRNAForecaster, Tsit5 solver, MSE loss) and recursively forecast forward with predictCellFutures) — https://github.com/rossinerbe/RNAForecaster.jl
- **Julia** (Implementation language (version 1.6) in which RNAForecaster and the benchmark MLP are run)
- **Flux.jl** (Machine-learning package used to train the neural ODE)
- **DiffEqFlux.jl** (Package used with Flux.jl to train the neural ODE with the Tsit5 ODE solver)

## Examples

```
using RNAForecaster; network = trainRNAForecaster(train_t0, train_t1; epochs=10); preds = predictCellFutures(network, valid_t0, 50)
```

## Evaluation signals

- Validation MSE at t=1 is significantly lower for the neural ODE than the five-hidden-layer MLP across simulations (paper reports p < 1e-16).
- Validation MSE remains significantly lower for the neural ODE across all 50 recursive future time points (paper reports p < 1e-11), with MLP error propagating more quickly and extreme outliers appearing earlier.
- The 80-20 train/validation split and sparsity filter (>98% zero counts) were applied before training, per the article's preprocessing defaults.
- The simulation set passes stability checks (simulations failing checks on 15 iterations are excluded, leaving 111 of 117 simulations).
- Comparison table is written as boolode_mse_comparison.csv with per-simulation MSE for both models.

## Limitations

- Random networks can generate strange regulatory behaviors — the article generated 117 simulations to guard against this, and excluded simulations where stability checks were not passed on 15 iterations (leaving 111).
- Recursive forecasting error grows over future time points; extreme outliers appear earlier in the MLP, and error propagates more quickly in the weaker model.
- Findings on simulated data do not by themselves establish predictability limits over diverse timescales and biological conditions, which the article notes have not been evaluated.

## Evidence

- [results] On the 20% held-out validation set of BoolODE simulated data, the RNAForecaster neural ODE significantly outperforms a five-hidden-layer MLP at t=1 (p < 1e-16): [evidence span withheld — anchor status: not_found]
- [readme] RNAForecaster functionality is provided as a [Julia package](https://github.com/rossinerbe/RNAForecaster.jl) on GitHub.: [evidence span withheld — anchor status: external_doc]
