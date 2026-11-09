---
name: umap-embedding-of-predicted-expression-profiles
description: Use when RNAForecaster (or another gene-dimensional forecaster) has produced
  future/perturbed expression profiles and you need to verify they occupy biologically
  plausible expression space by checking co-clustering with real or simulated post-perturbation
  cells.
when_to_use_negative:
- The predictor already operates in a low-dimensional embedding (e.g., the VAE-input
  variant) whose outputs are not directly comparable in full gene-dimensional UMAP
  space
- You need a quantitative accuracy verdict rather than a structural/visual check —
  use the forecast-vs-simulated MSE comparison against a non-KO control ensemble instead
- Prediction and observation matrices do not share the same gene set, normalization
  (log1p Float32), or scale, since joint UMAP requires a common expression space
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: Seurat
  role: Version 4.0.1 used to produce the UMAP visualizations jointly embedding predicted
    and simulated expression profiles
- name: RNAForecaster
  role: Produces the predicted post-KO expression profiles (via predictCellFutures,
    perturbationLevels = [0.0], median ensemble combination) that are embedded in
    UMAP space
  repo: https://github.com/rossinerbe/RNAForecaster.jl
- name: BoolODE
  role: Generates the simulated single-gene-KO count matrices used as the ground-truth
    co-clustering reference in the joint UMAP
  repo: https://github.com/FertigLab/RNAForecasterPaperCode
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.patter.2023.100793
    title: Transcriptomic forecasting with neural ordinary differential equations
  source_content_hash: sha256:b44f5a8747fe0c75c5d967d7139486c09d65367ba0c453f5d6fa4155a5856ccb
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# umap-embedding-of-predicted-expression-profiles

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Jointly embeds RNAForecaster-predicted post-perturbation expression profiles and ground-truth (e.g., simulated post-KO) profiles in UMAP space using Seurat 4.0.1 to visually and structurally assess whether predictions co-cluster with real expression states outside the training data.

## When to use

Use when RNAForecaster (or another gene-dimensional forecaster) has produced future/perturbed expression profiles and you need to verify they occupy biologically plausible expression space by checking co-clustering with real or simulated post-perturbation cells.

## When NOT to use

- The predictor already operates in a low-dimensional embedding (e.g., the VAE-input variant) whose outputs are not directly comparable in full gene-dimensional UMAP space
- You need a quantitative accuracy verdict rather than a structural/visual check — use the forecast-vs-simulated MSE comparison against a non-KO control ensemble instead
- Prediction and observation matrices do not share the same gene set, normalization (log1p Float32), or scale, since joint UMAP requires a common expression space

## Inputs

- Matrix of RNAForecaster-predicted post-perturbation expression profiles (genes x cells, log1p-normalized Float32, ensemble-median combined)
- Matrix of observed or BoolODE-simulated post-perturbation expression trajectories (genes x cells, same gene space and normalization)
- Gene name vector shared across predicted and simulated matrices

## Outputs

- Joint UMAP embedding of predicted and simulated post-KO expression profiles
- Qualitative co-clustering assessment of predicted vs. simulated KO cells in UMAP space

## How to apply

Take the log1p-normalized Float32 expression matrices for both the RNAForecaster forecasts (e.g., 50 recursively forecast post-KO time points, combined across the 10-network ensemble via the median prediction) and the corresponding simulated post-KO trajectories, and embed them jointly in UMAP space using Seurat 4.0.1 so shared genes and comparable scaling define a common manifold. Joint embedding is essential: UMAP computed separately on predictions and observations cannot be compared, because the forecast lives in full gene-dimensional space (RNAForecaster takes input in gene-dimensional space rather than a lower-dimensional embedding). The decision point is qualitative-structural: accept the forecast if the predicted profiles cluster with the simulated KO cells in the UMAP, reproducing the Figure 4 behavior in which RNAForecaster accurately predicts future expression states in simulated data. Pair this visual assessment with the quantitative MSE comparison against a non-KO control ensemble, since co-clustering alone does not certify point-wise accuracy.

## Related tools

- **Seurat** (Version 4.0.1 used to produce the UMAP visualizations jointly embedding predicted and simulated expression profiles)
- **RNAForecaster** (Produces the predicted post-KO expression profiles (via predictCellFutures, perturbationLevels = [0.0], median ensemble combination) that are embedded in UMAP space) — https://github.com/rossinerbe/RNAForecaster.jl
- **BoolODE** (Generates the simulated single-gene-KO count matrices used as the ground-truth co-clustering reference in the joint UMAP) — https://github.com/FertigLab/RNAForecasterPaperCode

## Examples

```
library(Seurat); seu <- CreateSeuratObject(predicted_counts); seu <- RunUMAP(seu, dims = 1:30)
```

## Evaluation signals

- Predicted post-KO profiles form clusters that overlap or sit adjacent to the simulated KO cells in the joint UMAP rather than separating into a distinct island
- The predicted and simulated matrices were embedded jointly (single UMAP object), not as two separately computed UMAPs
- Both matrices entered the embedding on the same gene set after log1p normalization to Float32, with genes >98% zero counts removed
- The co-clustering conclusion agrees with the direction of the quantitative check: forecast MSE is comparable to (not worse than) the non-KO control ensemble
- The embedding reproduces the paper's Figure 4 pattern in which RNAForecaster forecasts track simulated expression states over time

## Limitations

- Co-clustering in UMAP is a qualitative, visual criterion; the paper pairs it with MSE comparison because embedding geometry alone does not quantify forecast error
- RNAForecaster caps expression predictions post hoc at a maximum (default 2x the max observed in training data, in log space), so extreme out-of-distribution states may be truncated before embedding
- UMAP visualizations were produced with a specific tool version (Seurat 4.0.1); results may differ across UMAP parameterizations or versions

## Evidence
