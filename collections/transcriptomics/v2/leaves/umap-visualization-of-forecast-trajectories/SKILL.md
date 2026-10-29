---
name: umap-visualization-of-forecast-trajectories
description: Use when you have recursive future-expression predictions from RNAForecaster
  (predictCellFutures output) and need to qualitatively verify that predicted cell
  states reproduce lineage structure, e.g. bifurcation into two branches.
when_to_use_negative:
- Predictions were made in a lower-dimensional embedding rather than gene-dimensional
  space — UMAP comparison in gene space is only meaningful for RNAForecaster's gene-dimensional
  forecasts
- You need quantitative trajectory validation rather than qualitative visual comparison;
  the paper's validation for this task is a qualitative UMAP comparison (Figure 3)
- Data lack temporal pairing between consecutive time points, so no trained forecaster
  output exists to project
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0091
tools:
- name: Seurat
  role: Produces UMAP visualizations of the simulated and predicted expression states
    (version 4.0.1 used in the paper)
- name: RNAForecaster
  role: Generates the recursive 100-step future expression predictions that are projected
    onto the UMAP
  repo: https://github.com/rossinerbe/RNAForecaster.jl
- name: BoolODE
  role: Provides the bifurcating-lineage simulated count matrices (time points 365/366
    and 250/251) used as input and ground truth
  repo: https://github.com/FertigLab/RNAForecasterPaperCode
- name: Flux
  role: Machine-learning package (version 0.13.0) underlying the neural ODE training
    whose forecasts are visualized
- name: DiffEqFlux
  role: Neural ODE training package (version 1.46.0) with the Tsit5 ODE solver used
    before forecasting
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.patter.2023.100793
    title: Transcriptomic forecasting with neural ordinary differential equations
  source_content_hash: sha256:94e474bc73c3cc8c08b3887c67caa1a2ab4c076c2b91dc533dade5da90ac870c
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
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# umap-visualization-of-forecast-trajectories

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Projects RNAForecaster's recursively predicted future expression states into UMAP space (via Seurat) to visually assess whether forecasts reproduce simulated or biological structure, such as a bifurcation into two lineages over 100 recursive prediction steps.

## When to use

Use when you have recursive future-expression predictions from RNAForecaster (predictCellFutures output) and need to qualitatively verify that predicted cell states reproduce lineage structure, e.g. bifurcation into two branches.

## When NOT to use

- Predictions were made in a lower-dimensional embedding rather than gene-dimensional space — UMAP comparison in gene space is only meaningful for RNAForecaster's gene-dimensional forecasts
- You need quantitative trajectory validation rather than qualitative visual comparison; the paper's validation for this task is a qualitative UMAP comparison (Figure 3)
- Data lack temporal pairing between consecutive time points, so no trained forecaster output exists to project

## Inputs

- Predicted future expression matrix from predictCellFutures (genes x predicted cells across 100 recursive time points, log1p-transformed Float32)
- Simulated or observed expression matrices from the BoolODE bifurcating-lineage simulation (genes as rows, cells as columns, genes with >98% zero counts removed)

## Outputs

- UMAP embedding of predicted and observed expression states (Seurat object / UMAP coordinates)
- Qualitative assessment of whether predicted trajectories reproduce the two-lineage bifurcation or form spurious clusters

## How to apply

After training the neural ODE with trainRNAForecaster (Flux/DiffEqFlux, Tsit5 solver, 80-20 train/validation split) and recursively predicting 100 future time points with predictCellFutures, project both the predicted expression states and the simulated/observed states with UMAP using Seurat (version 4.0.1 in the paper). For the BoolODE bifurcating-lineage task, train on simulated time point 365 as input and 366 as target, forecast 100 steps, and confirm predicted cells split into the two simulated lineages after bifurcation. Repeat the identical workflow trained on time points 250/251 (well before bifurcation) as a negative control: correct application is evidenced by the 365/366-trained UMAP reproducing two-lineage differentiation, while the 250/251-trained UMAP shows breakdown at the bifurcation point with a new cluster absent from the simulation. Validation is a qualitative UMAP comparison as in Figure 3, since predictions are made in gene-dimensional space rather than a lower-dimensional embedding.

## Related tools

- **Seurat** (Produces UMAP visualizations of the simulated and predicted expression states (version 4.0.1 used in the paper))
- **RNAForecaster** (Generates the recursive 100-step future expression predictions that are projected onto the UMAP) — https://github.com/rossinerbe/RNAForecaster.jl
- **BoolODE** (Provides the bifurcating-lineage simulated count matrices (time points 365/366 and 250/251) used as input and ground truth) — https://github.com/FertigLab/RNAForecasterPaperCode
- **Flux** (Machine-learning package (version 0.13.0) underlying the neural ODE training whose forecasts are visualized)
- **DiffEqFlux** (Neural ODE training package (version 1.46.0) with the Tsit5 ODE solver used before forecasting)

## Evaluation signals

- UMAP of 365/366-trained forecasts shows predicted cells splitting into the two simulated lineages after the bifurcation point
- UMAP of 250/251-trained (pre-bifurcation) forecasts breaks down at the bifurcation, forming a new cluster absent from the simulation — confirming the expected limited prediction horizon
- Predicted expression values remain within the allowed maximum (by default two times the maximum observed in training data, in log space), so no artificial outliers dominate the embedding
- Input matrices were preprocessed consistently: genes with >98% zero counts removed, log1p-transformed, cast to Float32, so predicted and simulated states are comparable in the same space

## Limitations

- Validation is qualitative (visual UMAP comparison as in Figure 3), not a quantitative metric of lineage recovery
- Accuracy holds only over a limited prediction horizon — forecasts break down when trained well before the bifurcation (time points 250/251), forming a new cluster absent from the simulation
- RNA velocity vector-field-style constraints do not apply here: RNAForecaster takes input in gene-dimensional space and does not limit predictions to the UMAP or expression space observed in the dataset, so spurious predicted states can appear

## Evidence
