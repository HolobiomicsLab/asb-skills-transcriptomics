---
name: boolode-gene-knockout-simulation-forecasting
description: Use when you have BoolODE (or similar) simulated single-cell gene-expression
  count matrices with known time points and a defined single-gene knockout, and need
  to benchmark whether a neural ODE forecaster extrapolates to post-perturbation states
  absent from training.
when_to_use_negative:
- Your data are standard scRNA-seq without paired time-point measurements of the same
  cells (or metabolic labeling), since RNAForecaster training requires earlier/later
  counts from the same cell
- You need mechanistic gene-network inference rather than expression-state forecasting
  — the paper explicitly leaves mechanistic network inference as future work
- You require predictions only within a lower-dimensional embedding space; the value
  of this skill is precisely extrapolation in gene-dimensional space
edam_operation: http://edamontology.org/operation_0438
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: RNAForecaster
  role: Julia neural ODE package providing trainRNAForecaster, saveForecaster, and
    predictCellFutures with perturbationLevels for gene-KO forecasting
  repo: https://github.com/rossinerbe/RNAForecaster.jl
- name: Flux
  role: Machine-learning backend used to train the neural ODE networks (version 0.13.0)
- name: DiffEqFlux
  role: Differential-equation integration of the neural network with the Tsit5 ODE
    solver (version 1.46.0)
- name: BoolODE
  role: Generates the simulated single-gene-KO count matrices used as training targets
    and ground truth
  repo: https://github.com/FertigLab/RNAForecasterPaperCode
- name: Seurat
  role: UMAP visualization of predicted vs simulated post-KO profiles (version 4.0.1)
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.patter.2023.100793
    title: Transcriptomic forecasting with neural ordinary differential equations
  source_content_hash: sha256:85f514e654f31314c65dadef57107f21e38ae12b393434ae2a08ed6fdf01d890
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
  edam_operation: http://edamontology.org/operation_0438
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# boolode-gene-knockout-simulation-forecasting

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Train RNAForecaster neural ODE ensembles on pre-knockout BoolODE simulated single-cell time points and forecast post-knockout expression by conditioning on perturbationLevels=[0.0], then validate against simulated KO trajectories via MSE and UMAP co-clustering. This tests whether a network trained only on pre-KO data generalizes to expression states outside its training distribution.

## When to use

Use when you have BoolODE (or similar) simulated single-cell gene-expression count matrices with known time points and a defined single-gene knockout, and need to benchmark whether a neural ODE forecaster extrapolates to post-perturbation states absent from training.

## When NOT to use

- Your data are standard scRNA-seq without paired time-point measurements of the same cells (or metabolic labeling), since RNAForecaster training requires earlier/later counts from the same cell
- You need mechanistic gene-network inference rather than expression-state forecasting — the paper explicitly leaves mechanistic network inference as future work
- You require predictions only within a lower-dimensional embedding space; the value of this skill is precisely extrapolation in gene-dimensional space

## Inputs

- BoolODE single-gene-KO simulation count matrices (genes as rows, cells as columns) from pre-KO and post-KO time points
- Identifier of the gene to be knocked out
- Matching non-KO simulated count matrices for the control ensemble

## Outputs

- Trained 10-network forecaster ensemble saved as a .jld2 file
- Median-combined 50-step post-KO expression forecasts per cell
- MSE of KO forecasts vs simulated post-KO trajectories, compared with non-KO control MSE
- Joint UMAP embedding (Seurat 4.0.1) of predicted and simulated post-KO profiles

## How to apply

Load the BoolODE single-gene-KO simulation count matrices (genes as rows, cells as columns), remove genes with >98% zero counts, and log1p-normalize to Float32 as RNAForecaster requires. Train a 10-network neural ODE ensemble with trainRNAForecaster (Flux/DiffEqFlux, Tsit5 solver, error tolerance 1e-3, 10 epochs) using the earlier pre-KO time point as input and the later pre-KO time point as target, then save the ensemble with saveForecaster to a .jld2 file. Apply the knockout by conditioning predictions on perturbationLevels=[0.0] to zero the targeted gene, and recursively forecast 50 future time points with predictCellFutures, combining ensemble networks via the median prediction. Note RNAForecaster operates in gene-dimensional space, so unlike UMAP-restricted velocity methods it can output states beyond the observed expression space. Validate by comparing mean squared error against a non-KO control ensemble trained and forecast identically (acceptance: KO MSE comparable to control), and by jointly embedding predicted and simulated post-KO profiles in UMAP (Seurat 4.0.1) to check co-clustering.

## Related tools

- **RNAForecaster** (Julia neural ODE package providing trainRNAForecaster, saveForecaster, and predictCellFutures with perturbationLevels for gene-KO forecasting) — https://github.com/rossinerbe/RNAForecaster.jl
- **Flux** (Machine-learning backend used to train the neural ODE networks (version 0.13.0))
- **DiffEqFlux** (Differential-equation integration of the neural network with the Tsit5 ODE solver (version 1.46.0))
- **BoolODE** (Generates the simulated single-gene-KO count matrices used as training targets and ground truth) — https://github.com/FertigLab/RNAForecasterPaperCode
- **Seurat** (UMAP visualization of predicted vs simulated post-KO profiles (version 4.0.1))

## Examples

```
futures = predictCellFutures(forecaster, inputMatrix, 50, perturbationLevels=[0.0])
```

## Evaluation signals

- KO-ensemble forecast MSE over the 50 post-KO time points is comparable to (not worse than) the non-KO control ensemble's MSE
- Predicted post-KO profiles co-cluster with simulated KO cells in the joint UMAP embedding
- Expression caps hold: predictions exceeding two times the maximum observed in training data (log space) are truncated to the allowed maximum
- Trained ensembles persist as .jld2 files via saveForecaster and reload reproducibly
- Input matrices are log1p-normalized to Float32 and genes with >98% zeros removed before training

## Limitations

- The prior probability for expression levels outside the observed range is enforced post hoc (cap at 2x training maximum) rather than learned by the network; the paper notes future work is needed
- Ensembling 10 networks increases the required computational resources
- Limits of predictability over diverse timescales and biological conditions have not been evaluated
- A larger training set may be needed to capture rare regulatory events; the paper reports only 405 cells for its biological application

## Evidence

- [readme] readme: "RNAForecaster is based on a neural network that is trained using ordinary differential equations solvers (neural ODEs)."
- [discussion] tsit5-training: "A neural ODE20 is then trained using Flux.jl and DiffEqFlux.jl using the Tsit5 ODE solver"
- [discussion] exp-cap: "expression level predictions that are higher than an allowed maximum are set to the maximum value (by default two times the maximum observed in the training data, in log space)"
- [intro] gene-space: "it does not depend on a particular lower-dimensional embedding of the data but takes input in the gene-dimensional space."
- [discussion] stability-filter: "Simulations where stability checks were not passed on 15 iterations were excluded, leaving 111 simulations in the final set."
- [discussion] preprocessing: "The main preprocessing steps needed are sparsity filtering and log normalization."
