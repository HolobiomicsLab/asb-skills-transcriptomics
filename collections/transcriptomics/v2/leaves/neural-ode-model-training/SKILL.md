---
name: neural-ode-model-training
description: Use when paired time-point scRNA-seq count matrices (genes as rows, cells
  as columns, e.g. BoolODE simulations or metabolic-labeling scEU-seq data) are available
  and the goal is to forecast future single-cell expression states.
when_to_use_negative:
- Only unpaired standard scRNA-seq data are available — RNAForecaster requires paired
  time-point measurements of the same cell, which standard protocols cannot provide
- The goal is cell-state ordering rather than forward expression forecasting — trajectory
  inference or RNA velocity methods are more appropriate
- Genes with very high zero fractions (>98%) cannot be removed, since including them
  causes gradient-descent problems
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2269
tools:
- name: RNAForecaster
  role: Julia package providing trainRNAForecaster and predictCellFutures for neural
    ODE training and recursive forecasting
  repo: https://github.com/rossinerbe/RNAForecaster.jl
- name: Flux.jl
  role: Machine-learning framework used to train the neural ODE
- name: DiffEqFlux.jl
  role: Provides the neural ODE layer trained with the Tsit5 ODE solver
- name: BoolODE
  role: Generates simulated single-cell time-series count matrices used for training
    and benchmarking
  repo: https://github.com/FertigLab/RNAForecasterPaperCode
provenance:
  source_task_ids:
  - task_001
  - task_006
  - task_003
  - task_002
  source_papers:
  - doi: 10.1016/j.patter.2023.100793
    title: Transcriptomic forecasting with neural ordinary differential equations
  source_content_hash: sha256:822bfcead6905f12fedae6a5da97f473c4ef2f1515df8ce9bf5d8beeb6d9ca10
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
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# neural-ode-model-training

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Trains RNAForecaster, a neural ODE implemented in Julia with Flux.jl and DiffEqFlux.jl, on paired time-point single-cell expression matrices (earlier counts as input, later counts as target) and recursively forecasts future expression states by feeding predictions back into the input layer.

## When to use

Use when paired time-point scRNA-seq count matrices (genes as rows, cells as columns, e.g. BoolODE simulations or metabolic-labeling scEU-seq data) are available and the goal is to forecast future single-cell expression states.

## When NOT to use

- Only unpaired standard scRNA-seq data are available — RNAForecaster requires paired time-point measurements of the same cell, which standard protocols cannot provide
- The goal is cell-state ordering rather than forward expression forecasting — trajectory inference or RNA velocity methods are more appropriate
- Genes with very high zero fractions (>98%) cannot be removed, since including them causes gradient-descent problems

## Inputs

- Paired t0/t1 count matrices from single-cell time-series data (genes as rows, cells as columns), e.g. BoolODE simulated 10-gene networks or scEU-seq metabolic labeling data (rpeLabeling.h5ad)
- For labeling data: labeled, unlabeled, and total count matrices plus labelingTime (e.g. 60.0 minutes)
- Optional hyperparameters: trainingProp, learningRate (e.g. 0.0001), batchsize (e.g. 25), number of ensemble networks

## Outputs

- Trained neural ODE network (or ensemble of networks) predicting each gene's expression at the later time point
- Recursive future expression predictions per cell from predictCellFutures (e.g. 50-100 time steps)
- Per-gene temporal variability ranking via mostTimeVariableGenes

## How to apply

Preprocess the paired t0/t1 matrices with sparsity filtering (remove genes with >98% zero counts, since these cause problems with gradient descent), log1p normalization, and Float32 casting; filter to top-quartile variance genes for biological datasets. Train the neural ODE with trainRNAForecaster using the Tsit5 solver at a default error tolerance of 1e-3, default 10 epochs, MSE loss, and the default 80-20 train/validation split. For metabolic labeling data, first estimate the clean t=0 matrix with estimateT0LabelingData, which computes the degradation rate as -log(1 - slope) from a labeled-vs-total count regression. Optionally train ensembles (e.g. 10 or 25 networks with different gradient-descent seeds) and combine predictions by taking the median across networks. Forecast recursively with predictCellFutures, feeding each output back into the input nodes, and cap predictions above two times the maximum observed training value in log space.

## Related tools

- **RNAForecaster** (Julia package providing trainRNAForecaster and predictCellFutures for neural ODE training and recursive forecasting) — https://github.com/rossinerbe/RNAForecaster.jl
- **Flux.jl** (Machine-learning framework used to train the neural ODE)
- **DiffEqFlux.jl** (Provides the neural ODE layer trained with the Tsit5 ODE solver)
- **BoolODE** (Generates simulated single-cell time-series count matrices used for training and benchmarking) — https://github.com/FertigLab/RNAForecasterPaperCode

## Examples

```
julia> using RNAForecaster; network = trainRNAForecaster(t0Matrix, t1Matrix); futures = predictCellFutures(network, t0Matrix, 50)
```

## Evaluation signals

- Validation-set MSE at t=1 should be computed on the held-out 20% split; the neural ODE significantly outperformed a five-hidden-layer MLP (p < 1e-16) on BoolODE data
- Recursive forecast log MSE should remain below 0.7 out to 200 time points on BoolODE simulations
- Ensemble (10 or 25 networks, median combination) log MSE should be significantly lower than a single network by t=10 (p < 1e-6), though not at t=1
- Trained on time points immediately before a bifurcation (365/366), recursive UMAP projections should reproduce the two simulated lineages; training at 250/251 should fail
- Check that all genes with >98% zero counts were removed and matrices are log1p-transformed Float32 before training

## Limitations

- Accuracy holds only over a limited prediction horizon: forecasts break down at bifurcation points when trained well before the transition
- Metabolic labeling protocols required for paired measurements are largely confined to in vitro studies, limiting in vivo application
- A training set of only 405 cells may be insufficient to capture rare cell-cycle events; the architecture may lack capacity for checkpoints or rare regulatory events
- Ensembling improves accuracy but increases required computational resources
- Expression caps (2x max observed in training, log space) are enforced post hoc rather than learned as a prior

## Evidence
