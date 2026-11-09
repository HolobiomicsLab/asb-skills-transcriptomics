---
name: linear-regression-transcript-counts
description: Use when you have paired labeled/unlabeled/total count matrices from
  metabolic labeling scRNA-seq (e.g., scEU-seq 4sU-treated cells) and need per-gene
  degradation rates to estimate pre-labeling expression.
when_to_use_negative:
- Input is standard scRNA-seq without metabolic labeling — no labeled/unlabeled split
  exists, so degradation rates cannot be estimated and paired time-point data required
  by RNAForecaster are unavailable
- Cells have heterogeneous labeling times — subset to a common labeling time (e.g.,
  60 min 4sU) first, as the degradation calculation assumes equal labeling duration
- The goal is forecasting future states from already-paired time-course data rather
  than constructing a t=0 matrix from labeling data
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: RNAForecaster
  role: Consumer of the t=0 matrix produced by this regression-based degradation estimation;
    estimateT0LabelingData is its helper for labeling data
  repo: https://github.com/rossinerbe/RNAForecaster.jl
- name: Julia
  role: Implementation language for the regression and t0 estimation workflow (version
    1.6)
- name: JLD2
  role: File format/package used to load the labeled/unlabeled/total matrices (rpeExampleData.jld2)
    and save the t0Estimate output
- name: scEU-seq
  role: Source assay providing labeled/unlabeled/total counts (Battich et al. rpeLabeling.h5ad)
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.patter.2023.100793
    title: Transcriptomic forecasting with neural ordinary differential equations
  source_content_hash: sha256:8bd495ecc024ec1c72ac8a521b0c7a7e3497592e3da2d7f1a359a039ce2f77ad
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
  - JLD2
  - Julia
  - RNAForecaster
  - scEU-seq
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# Linear regression transcript counts

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Estimates per-gene RNA degradation rates from metabolic labeling scRNA-seq data (e.g., scEU-seq) by fitting a linear regression of labeled counts against total counts across cells, then computing the rate as -log(1 - slope); these rates enable construction of the t=0 input matrix for RNAForecaster.

## When to use

Use when you have paired labeled/unlabeled/total count matrices from metabolic labeling scRNA-seq (e.g., scEU-seq 4sU-treated cells) and need per-gene degradation rates to estimate pre-labeling expression.

## When NOT to use

- Input is standard scRNA-seq without metabolic labeling — no labeled/unlabeled split exists, so degradation rates cannot be estimated and paired time-point data required by RNAForecaster are unavailable
- Cells have heterogeneous labeling times — subset to a common labeling time (e.g., 60 min 4sU) first, as the degradation calculation assumes equal labeling duration
- The goal is forecasting future states from already-paired time-course data rather than constructing a t=0 matrix from labeling data

## Inputs

- Labeled counts matrix (genes x cells) from metabolic labeling scRNA-seq (e.g., scEU-seq)
- Unlabeled counts matrix (genes x cells)
- Total counts matrix (genes x cells)
- Labeling time vector (e.g., 60 min 4sU)

## Outputs

- Per-gene degradation rates (-log(1 - slope) from labeled vs total regression)
- t0Estimate matrix (genes x cells): unlabeled counts plus estimated degraded transcripts, saved as t0_estimate.jld2 for RNAForecaster training

## How to apply

Load the labeled, unlabeled, and total count matrices plus the labeling-time vector (e.g., from data/rpeExampleData.jld2, converted to Float32), subsetting first to cells with a common labeling time such as 60 min of 4sU uridine. For each gene, fit a linear regression of labeled counts against total counts across cells; the slope reflects the labeled fraction. Convert the slope to a real-time degradation rate as -log(1 - slope), following Qiu et al. 2022. Use the fitted degradation rate with the labeling time to estimate degraded transcripts per gene per cell, and add these to the unlabeled counts (estimateT0LabelingData) to form the t=0 expression estimate fed to RNAForecaster training. This regression-based step is needed because metabolic labeling data reveal new transcription, but the network must be trained on the total transcripts present at t=0, including already-degraded species.

## Related tools

- **RNAForecaster** (Consumer of the t=0 matrix produced by this regression-based degradation estimation; estimateT0LabelingData is its helper for labeling data) — https://github.com/rossinerbe/RNAForecaster.jl
- **Julia** (Implementation language for the regression and t0 estimation workflow (version 1.6))
- **JLD2** (File format/package used to load the labeled/unlabeled/total matrices (rpeExampleData.jld2) and save the t0Estimate output)
- **scEU-seq** (Source assay providing labeled/unlabeled/total counts (Battich et al. rpeLabeling.h5ad))

## Examples

```
using JLD2, RNAForecaster; rpe = load("data/rpeExampleData.jld2")["rpeExampleData"]; t0Estimate = estimateT0LabelingData(Float32.(rpe[2]), Float32.(rpe[3]), Float32.(rpe[4]), rpe[5])
```

## Evaluation signals

- Output t0Estimate matrix must have the same gene-by-cell dimensions as the unlabeled counts matrix
- Every entry of t0Estimate must be greater than or equal to the corresponding unlabeled count (degraded transcripts are added, never subtracted)
- No non-finite (NaN/Inf) values in the degradation rates or t0Estimate matrix
- All matrices converted to Float32 before regression; fitted slopes must lie in [0, 1) so that -log(1 - slope) is defined and non-negative
- Downstream RNAForecaster training must succeed on the resulting t0 matrix after sparsity filtering (>98% zeros removed) and log normalization

## Limitations

- Degradation rate is estimated under a linear-model assumption on labeled vs total counts; deviations from the Qiu et al. 2022 model may bias rates
- Metabolic labeling protocols are largely confined to in vitro studies, limiting application in many in vivo contexts
- All cells must share an equal labeling time; the scEU-seq data were subset to cells treated with 4sU uridine for 60 min for this reason

## Evidence
