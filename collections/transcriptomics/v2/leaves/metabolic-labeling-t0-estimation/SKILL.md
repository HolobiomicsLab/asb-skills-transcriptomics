---
name: metabolic-labeling-t0-estimation
description: Use when scRNA-seq data come from a metabolic labeling assay (e.g., scEU-seq)
  with separate labeled, unlabeled, and total count matrices, and you need a t=0 expression
  estimate to train RNAForecaster on paired time points.
when_to_use_negative:
- Input is standard scRNA-seq without labeled/unlabeled count matrices — the same
  cell cannot be measured at two time points, so no t=0 estimate can be constructed
- Data come from trajectory-inference or RNA velocity settings where only spliced/unspliced
  counts are available rather than metabolic labeling counts
- A t0Estimate matrix has already been produced and validated; re-running estimation
  would be redundant
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: RNAForecaster
  role: Provides estimateT0LabelingData, which fits the labeled-vs-total regression,
    computes degradation rate as −log(1 − slope), and returns the t0Estimate matrix
  repo: https://github.com/rossinerbe/RNAForecaster.jl
- name: Julia
  role: Implementation language for RNAForecaster and estimateT0LabelingData; requires
    Julia 1.6 or later
  repo: https://julialang.org/
- name: JLD2
  role: Used to load and save the labeled/unlabeled/total matrices and the resulting
    t0_estimate.jld2
provenance:
  source_task_ids:
  - task_006
  - task_007
  source_papers:
  - doi: 10.1016/j.patter.2023.100793
    title: Transcriptomic forecasting with neural ordinary differential equations
  source_content_hash: sha256:e683d33f085d1e69a470cf11d255f427ab3fc3aaa9fea0944a10db7ef2e44f27
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
  - JLD2
  - Julia
  - RNAForecaster
  - scEU-seq
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# metabolic-labeling-t0-estimation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Constructs the t=0 total-expression input matrix for RNAForecaster from metabolic labeling scRNA-seq data (e.g., scEU-seq) by estimating per-gene degradation rates from labeled-vs-total count regressions and adding estimated degraded transcripts to the unlabeled counts matrix.

## When to use

Use when scRNA-seq data come from a metabolic labeling assay (e.g., scEU-seq) with separate labeled, unlabeled, and total count matrices, and you need a t=0 expression estimate to train RNAForecaster on paired time points.

## When NOT to use

- Input is standard scRNA-seq without labeled/unlabeled count matrices — the same cell cannot be measured at two time points, so no t=0 estimate can be constructed
- Data come from trajectory-inference or RNA velocity settings where only spliced/unspliced counts are available rather than metabolic labeling counts
- A t0Estimate matrix has already been produced and validated; re-running estimation would be redundant

## Inputs

- Labeled counts matrix (genes x cells) from metabolic labeling scRNA-seq (e.g., scEU-seq)
- Unlabeled counts matrix (genes x cells)
- Total counts matrix (genes x cells)
- Labeling time vector (e.g., labelingTime == 60.0 minutes for 4sU-treated hTERT RPE cells)
- rpeExampleData.jld2 / rpeLabeling.h5ad from Battich et al. scEU-seq dataset

## Outputs

- t0Estimate matrix (genes x cells) of estimated total transcripts at t=0, saved as t0_estimate.jld2

## How to apply

Load the labeled, unlabeled, and total count matrices plus the labeling-time vector (e.g., from rpeExampleData.jld2 via JLD2 or rpeLabeling.h5ad) and convert matrices to Float32. For each gene, fit a linear regression of labeled counts against total counts across cells to estimate the slope, then compute the degradation rate as −log(1 − slope), following Qiu et al. 2022. Using the fitted degradation rate and the labeling time (e.g., the 60-minute 4sU-labeled hTERT RPE cells), estimate the number of degraded transcripts per gene per cell and add them to the unlabeled count matrix via estimateT0LabelingData. The rationale is that standard scRNA-seq destroys the cell, so RNAForecaster requires the t=0 matrix to be reconstructed as unlabeled counts plus degraded transcripts, with degradation estimated in real time from the labeling data itself. Save the resulting genes x cells t0Estimate matrix for downstream RNAForecaster training (e.g., trainRNAForecaster).

## Related tools

- **RNAForecaster** (Provides estimateT0LabelingData, which fits the labeled-vs-total regression, computes degradation rate as −log(1 − slope), and returns the t0Estimate matrix) — https://github.com/rossinerbe/RNAForecaster.jl
- **Julia** (Implementation language for RNAForecaster and estimateT0LabelingData; requires Julia 1.6 or later) — https://julialang.org/
- **JLD2** (Used to load and save the labeled/unlabeled/total matrices and the resulting t0_estimate.jld2)

## Examples

```
t0Estimate = estimateT0LabelingData(labeledData, totalData, unlabeledData, labelingTime)
```

## Evaluation signals

- Output t0Estimate has the same gene-by-cell dimensions as the unlabeled counts matrix
- Every entry of t0Estimate is greater than or equal to the corresponding unlabeled count (degraded transcripts are added, never subtracted)
- No non-finite (NaN/Inf) values in the output matrix
- Fitted per-gene regression slopes fall in [0, 1] so degradation rates −log(1 − slope) are finite and non-negative
- Downstream RNAForecaster training (e.g., trainRNAForecaster with trainingProp = 1.0) accepts the matrix without dimension or type errors

## Limitations

- Metabolic labeling protocols are largely confined to in vitro studies, limiting application to in vivo contexts
- The degradation rate is estimated per gene from a linear regression, so genes with noisy or low labeled-vs-total signal yield unstable rate estimates
- Downstream forecasts are limited by training-set size (only 405 one-hour-labeled scEU-seq RPE cells were available)

## Evidence
