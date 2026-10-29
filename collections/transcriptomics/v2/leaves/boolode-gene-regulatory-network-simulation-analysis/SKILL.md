---
name: boolode-gene-regulatory-network-simulation-analysis
description: Use when benchmarking transcriptomic forecasting methods on simulated
  gene regulatory networks where true future cell states and lineage splits are known,
  or when determining how far ahead recursive predictions stay accurate before diverging
  from simulated dynamics.
when_to_use_negative:
- Input is real scRNA-seq data without paired time points from the same cells — RNAForecaster
  requires dynamic measurements such as metabolic labeling data, not standard protocols
- The simulation has no known future trajectory or bifurcation, so there is no ground
  truth to validate recursive forecasts against
- Goal is long-horizon prediction far beyond the training window — the paper shows
  forecasts break down at regulatory transitions outside the training window's vicinity
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0605
tools:
- name: RNAForecaster
  role: Neural ODE forecaster trained on consecutive simulated time points and used
    for recursive prediction of future expression states
  repo: https://github.com/rossinerbe/RNAForecaster.jl
- name: Flux
  role: Machine-learning framework underlying RNAForecaster network training (version
    0.13.0)
- name: DiffEqFlux
  role: Neural ODE training with the Tsit5 ODE solver (version 1.46.0)
- name: Seurat
  role: UMAP visualization of predicted expression states for qualitative comparison
    with simulated lineages (version 4.0.1)
- name: BoolODE
  role: Generator of the bifurcating-lineage simulation count matrices used as ground-truth
    benchmark data
  repo: https://github.com/FertigLab/RNAForecasterPaperCode
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.patter.2023.100793
    title: Transcriptomic forecasting with neural ordinary differential equations
  source_content_hash: sha256:5b2b2008928fe435d2f924a82709d2c2f9d14b0d6932b6a2da613d69fe6cd6b5
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
  - Flux
  - RNAForecaster
  - Seurat
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# boolode-gene-regulatory-network-simulation-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Use BoolODE-simulated bifurcating-lineage single-cell count matrices as ground-truth benchmarks for testing whether RNAForecaster's recursive neural ODE forecasts reproduce lineage commitment, and characterize the temporal horizon over which predictions remain valid.

## When to use

Use when benchmarking transcriptomic forecasting methods on simulated gene regulatory networks where true future cell states and lineage splits are known, or when determining how far ahead recursive predictions stay accurate before diverging from simulated dynamics.

## When NOT to use

- Input is real scRNA-seq data without paired time points from the same cells — RNAForecaster requires dynamic measurements such as metabolic labeling data, not standard protocols
- The simulation has no known future trajectory or bifurcation, so there is no ground truth to validate recursive forecasts against
- Goal is long-horizon prediction far beyond the training window — the paper shows forecasts break down at regulatory transitions outside the training window's vicinity

## Inputs

- BoolODE bifurcating-lineage simulation count matrices (genes x cells) at consecutive simulated time points (e.g., 365/366 and 250/251)
- Gene names from the simulated 10-gene regulatory network
- Known bifurcation time point in the simulation

## Outputs

- 100 recursively predicted future expression states per cell from the trained neural ODE
- UMAP projections of predicted expression states showing lineage split (or failure) relative to simulated lineages
- Qualitative comparison of forecast behavior for training windows immediately before vs. well before bifurcation (as in Figure 3)

## How to apply

Load the BoolODE bifurcating-lineage simulation count matrices (genes as rows, cells as columns), remove genes with >98% zero counts, log1p-transform, and cast to Float32. Train the RNAForecaster neural ODE (Flux 0.13.0 / DiffEqFlux 1.46.0, Tsit5 solver, 80-20 train/validation split) using the expression matrix at simulated time point 365 as input and time point 366 as target, then call predictCellFutures to recursively predict 100 future time points by feeding each output back into the input layer. Project predicted expression states with UMAP (Seurat 4.0.1) and check that predicted cells split into the two simulated lineages after bifurcation. Repeat training with matrices from time points 250/251, well before the bifurcation, to demonstrate the failure mode: recursive forecasts break down at the bifurcation point and form a spurious new cluster absent from the simulation, establishing that accuracy holds only over a limited prediction horizon.

## Related tools

- **RNAForecaster** (Neural ODE forecaster trained on consecutive simulated time points and used for recursive prediction of future expression states) — https://github.com/rossinerbe/RNAForecaster.jl
- **Flux** (Machine-learning framework underlying RNAForecaster network training (version 0.13.0))
- **DiffEqFlux** (Neural ODE training with the Tsit5 ODE solver (version 1.46.0))
- **Seurat** (UMAP visualization of predicted expression states for qualitative comparison with simulated lineages (version 4.0.1))
- **BoolODE** (Generator of the bifurcating-lineage simulation count matrices used as ground-truth benchmark data) — https://github.com/FertigLab/RNAForecasterPaperCode

## Examples

```
using RNAForecaster; network = trainRNAForecaster(expressionT365, expressionT366); futurePreds = predictCellFutures(expressionT365, network, 100)
```

## Evaluation signals

- UMAP of the 365/366-trained 100-step recursive forecasts reproduces the two-lineage differentiation seen in the BoolODE simulation (Figure 3 comparison)
- UMAP of the 250/251-trained forecasts fails to anticipate lineage commitment and forms a new cluster absent from the simulation
- Predicted expression values stay within the allowed maximum (default two times the maximum observed in training data, in log space) rather than diverging
- Training converges under the default 10 epochs with the 80-20 train/validation split and Tsit5 solver error tolerance of 1e-3

## Limitations

- Accuracy holds only over a limited prediction horizon — forecasts trained well before the bifurcation (250/251) break down at the bifurcation point and invent a spurious cluster
- Ensembling networks (median prediction) increases required computational resources
- Limits of predictability over diverse timescales and biological conditions have not been evaluated; this benchmark covers one bifurcating 10-gene network
- Simulations failing stability checks on 15 iterations were excluded (111 of 117 simulations retained), so network topologies with unstable dynamics are not represented

## Evidence

- [discussion] Sparsity filtering and log normalization preprocessing of input count matrices: "The main preprocessing steps needed are sparsity filtering and log normalization."
- [discussion] Recursive prediction mechanism: "the output is recorded and fed back into the input nodes, allowing for recursive predictions forward in time."
- [discussion] Filtering threshold for zero-count genes: "Including genes that have high proportions of zeroes (default greater than 98%) can cause problems with gradient descent, and thus these genes must be removed."
- [discussion] Prediction cap safeguard: "expression level predictions that are higher than an allowed maximum are set to the maximum value (by default two times the maximum observed in the training data, in log space)"
- [discussion] Implementation stack: "RNAForecaster is implemented using Julia version 1.6 with the machine-learning packages Flux (version 0.13.0) and DiffEqFlux (version 1.46.0)."
