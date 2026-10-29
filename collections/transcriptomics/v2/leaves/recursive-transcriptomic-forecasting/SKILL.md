---
name: recursive-transcriptomic-forecasting
description: Use when you have paired single-cell expression measurements of the same
  cells at two time points (e.g., metabolic labeling scEU-seq) and need to predict
  future expression states several steps beyond the observed horizon.
when_to_use_negative:
- Input is standard scRNA-seq without metabolic labeling, since paired earlier/later
  measurements of the same cell are required but not available from destructive protocols
- The goal is cell ordering or pseudotime trajectory inference alone, not forward
  modeling of gene expression values over time
- Predictions are needed for rare events such as cell-cycle checkpoints, which the
  training population may not contain (the scEU-seq training set had only 405 cells)
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: RNAForecaster
  role: Julia package providing trainRNAForecaster (neural ODE ensemble training)
    and predictCellFutures (recursive multi-step forecasting with median ensemble
    combination and prediction capping)
  repo: https://github.com/rossinerbe/RNAForecaster.jl
- name: Flux
  role: Machine-learning package (version 0.13.0) used to train the neural ODE networks
- name: DiffEqFlux
  role: Package (version 1.46.0) providing the Tsit5 ODE solver used to train the
    neural ODE
- name: tricycle
  role: Cell-cycle embedding (circular 0 to 2π coordinate) used to score predicted
    states for ordering relative to randomized controls
- name: Julia
  role: Runtime (version 1.6 or later) in which RNAForecaster is implemented and executed
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.patter.2023.100793
    title: Transcriptomic forecasting with neural ordinary differential equations
  source_content_hash: sha256:ef64d0330fea33142928219d4cebd4df768bf36f395c0b677e9014aa9a3d1f1b
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
  - DiffEqFlux
  - Flux
  - Julia
  - RNAForecaster
  - tricycle
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# recursive-transcriptomic-forecasting

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Train a neural ODE ensemble (RNAForecaster) on paired earlier/later single-cell expression matrices, then recursively feed predictions back through the network to forecast transcriptomic states multiple time steps ahead. The skill was validated on scEU-seq metabolic labeling data by scoring 72-hour forecasts against tricycle cell-cycle ordering.

## When to use

Use when you have paired single-cell expression measurements of the same cells at two time points (e.g., metabolic labeling scEU-seq) and need to predict future expression states several steps beyond the observed horizon.

## When NOT to use

- Input is standard scRNA-seq without metabolic labeling, since paired earlier/later measurements of the same cell are required but not available from destructive protocols
- The goal is cell ordering or pseudotime trajectory inference alone, not forward modeling of gene expression values over time
- Predictions are needed for rare events such as cell-cycle checkpoints, which the training population may not contain (the scEU-seq training set had only 405 cells)

## Inputs

- Metabolic labeling scRNA-seq matrices (labeled, unlabeled, and total counts) from constitutively dividing cells, e.g., the Battich et al. hTERT RPE scEU-seq dataset (rpeExampleData.jld2 / rpeLabeling.h5ad)
- Paired t0/t1 single-cell expression matrices for the same cells, log1p-normalized as Float32 and filtered to the top variance quartile

## Outputs

- Recursively forecast expression matrices for each future time step (e.g., 72 hourly steps)
- Per-forecast-hour scoring table (e.g., tricycle_ordering_metrics.csv) comparing predicted cell-cycle ordering against randomized controls
- Per-gene time-variability ranking via mostTimeVariableGenes(futureExpressionPreds, geneNames)

## How to apply

Estimate the clean t=0 matrix from labeled, unlabeled, and total counts using estimateT0LabelingData, which fits the labeled-vs-total slope to account for degraded transcripts (degradation rate = -log(1 - slope)). Preprocess by removing genes with >98% zero counts, filtering to the top-quartile variance genes, and log1p-normalizing both t0 and t1 matrices as Float32. Train an ensemble of 10 neural ODE networks with trainRNAForecaster (e.g., trainingProp = 1.0, learningRate = 0.0001, batchsize = 25), using the Tsit5 solver and MSE loss against the later time-point matrix. Forecast by calling predictCellFutures, feeding each output back into the input layer recursively for the desired number of steps, combining the ensemble via the median, and capping any prediction above two times the maximum observed training value in log space. Rationale: RNAForecaster operates in gene-dimensional space rather than a lower-dimensional embedding, so predictions are not constrained to the observed input expression space, which is why the post hoc cap is required.

## Related tools

- **RNAForecaster** (Julia package providing trainRNAForecaster (neural ODE ensemble training) and predictCellFutures (recursive multi-step forecasting with median ensemble combination and prediction capping)) — https://github.com/rossinerbe/RNAForecaster.jl
- **Flux** (Machine-learning package (version 0.13.0) used to train the neural ODE networks)
- **DiffEqFlux** (Package (version 1.46.0) providing the Tsit5 ODE solver used to train the neural ODE)
- **tricycle** (Cell-cycle embedding (circular 0 to 2π coordinate) used to score predicted states for ordering relative to randomized controls)
- **Julia** (Runtime (version 1.6 or later) in which RNAForecaster is implemented and executed)

## Examples

```
forecaster = trainRNAForecaster(t0_60min, t1_60min, trainingProp = 1.0, learningRate = 0.0001, batchsize = 25); futures = predictCellFutures(forecaster, t0_60min, 72)
```

## Evaluation signals

- Forecast states scored with the tricycle 0–2π cell-cycle embedding should be significantly more ordered than randomly generated scores at every forecast hour (p < 1e-16 in the reference reproduction), with ordering declining per day
- No forecast expression value should exceed the cap of two times the maximum observed training value (in log space)
- Training loss (MSE against the t=1 matrix, Tsit5 solver, default error tolerance 1e-3) should decrease over epochs under the default 80-20 train/validation split
- Ensemble outputs must be combined via the median across the 10 networks, not the mean
- Genes removed during preprocessing should match the sparsity filter (default >98% zero counts) and top-quartile variance criterion

## Limitations

- Standard scRNA-seq protocols destroy the cell, so paired time-point training data are unavailable; metabolic labeling protocols are largely confined to in vitro studies, limiting in vivo application
- The biological training set contained only 405 cells; a larger set may be needed to capture rare cell-cycle events, and the architecture may lack the capability to predict checkpoints or rare regulatory exceptions
- Ensembling improves predictions but increases required computational resources
- The prior probability for expression levels outside the observed range is enforced post hoc (capping) rather than learned by the network
- Limits of predictability over diverse timescales and biological conditions have not been evaluated

## Evidence
