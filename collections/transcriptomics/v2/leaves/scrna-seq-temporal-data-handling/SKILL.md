---
name: scrna-seq-temporal-data-handling
description: Use when you have metabolic labeling scRNA-seq data (e.g., scEU-seq rpeLabeling.h5ad
  with separate labeled, unlabeled, and total count matrices plus a labeling-time
  vector) and need paired time-point inputs for RNAForecaster.
when_to_use_negative:
- Input is standard scRNA-seq without paired time-point measurements of the same cell,
  since 'This sort of data is not available when using standard scRNA-seq protocols'
- Data lacks separate labeled/unlabeled count matrices or a defined labeling time,
  so degradation cannot be estimated in real time
- The goal is ordering cells along a trajectory rather than forecasting future expression
  states with a neural ODE
edam_operation: http://edamontology.org/operation_0314
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: RNAForecaster
  role: Consumes the reconstructed t=0 matrix as the earlier-time-point input to train
    a neural ODE that predicts expression at the later time point
  repo: https://github.com/rossinerbe/RNAForecaster.jl
- name: JLD2
  role: Loads the labeled/unlabeled/total count matrices and labeling-time vector
    from data/rpeExampleData.jld2 and saves the t0Estimate output
- name: Julia
  role: Implementation environment (Julia 1.6) in which estimateT0LabelingData and
    the RNAForecaster training are executed
- name: scEU-seq
  role: Source metabolic labeling assay providing the labeled/unlabeled/total count
    structure this handling skill operates on
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.patter.2023.100793
    title: Transcriptomic forecasting with neural ordinary differential equations
  source_content_hash: sha256:c6aaabe3f0d4ffc19e0def195135444cc2a232f299c8e3b84a48b1bda1066952
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
  edam_operation: http://edamontology.org/operation_0314
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# scRNA-seq-temporal-data-handling

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Converts metabolic labeling scRNA-seq counts (e.g., scEU-seq labeled/unlabeled/total matrices) into paired time-point expression matrices suitable for training RNAForecaster neural ODEs, by reconstructing a t=0 expression estimate that includes degraded transcripts using per-gene degradation rates.

## When to use

Use when you have metabolic labeling scRNA-seq data (e.g., scEU-seq rpeLabeling.h5ad with separate labeled, unlabeled, and total count matrices plus a labeling-time vector) and need paired time-point inputs for RNAForecaster.

## When NOT to use

- Input is standard scRNA-seq without paired time-point measurements of the same cell, since 'This sort of data is not available when using standard scRNA-seq protocols'
- Data lacks separate labeled/unlabeled count matrices or a defined labeling time, so degradation cannot be estimated in real time
- The goal is ordering cells along a trajectory rather than forecasting future expression states with a neural ODE

## Inputs

- Labeled counts matrix (genes x cells) from metabolic labeling scRNA-seq (e.g., scEU-seq)
- Unlabeled counts matrix (genes x cells)
- Total counts matrix (genes x cells)
- Labeling-time vector (e.g., 4sU labeling duration in minutes)
- rpeLabeling.h5ad AnnData from Battich et al. (via Dynamo) or equivalent JLD2 file such as data/rpeExampleData.jld2

## Outputs

- t0Estimate matrix (genes x cells): unlabeled counts plus estimated degraded transcripts, used as the t=0 neural ODE training input
- Per-gene real-time degradation rates estimated as -log(1 - slope) from labeled-vs-total linear regression
- Sparsity- and variance-filtered, log-normalized count matrices for RNAForecaster training

## How to apply

Load the labeled, unlabeled, and total count matrices plus the labeling-time vector (e.g., from data/rpeExampleData.jld2 with JLD2) and convert all matrices to Float32. Subset to cells with a uniform labeling treatment (e.g., 4sU uridine for 60 min) so the labeling time is equal for all input cells, and preprocess with sparsity filtering (remove genes >98% zero counts) and top-quartile variance gene filtering. For each gene, fit a linear regression of labeled counts against total counts across cells and compute the degradation rate as -log(1 - slope) following Qiu et al. 2022. Call estimateT0LabelingData(labeledData, totalData, unlabeledData, labelingTime) to add the estimated degraded transcripts to the unlabeled matrix, yielding the t=0 expression estimate; the t=1 matrix is the later-time-point (total/labeled) counts. Save the resulting t0Estimate (genes x cells) for downstream neural ODE training with an 80-20 train/validation split.

## Related tools

- **RNAForecaster** (Consumes the reconstructed t=0 matrix as the earlier-time-point input to train a neural ODE that predicts expression at the later time point) — https://github.com/rossinerbe/RNAForecaster.jl
- **JLD2** (Loads the labeled/unlabeled/total count matrices and labeling-time vector from data/rpeExampleData.jld2 and saves the t0Estimate output)
- **Julia** (Implementation environment (Julia 1.6) in which estimateT0LabelingData and the RNAForecaster training are executed)
- **scEU-seq** (Source metabolic labeling assay providing the labeled/unlabeled/total count structure this handling skill operates on)

## Examples

```
t0Estimate = estimateT0LabelingData(labeledData, totalData, unlabeledData, labelingTime)
```

## Evaluation signals

- The t0Estimate matrix has the same gene-by-cell dimensions as the unlabeled counts matrix
- Every entry of the t0Estimate matrix is greater than or equal to the corresponding unlabeled count, since it adds estimated degraded transcripts
- No non-finite values in the t0Estimate matrix
- Per-gene degradation rates are valid for -log(1 - slope), i.e., slopes fall in (0, 1); the linear regression of labeled vs. total counts yields finite slopes
- All retained cells share a single labeling time (e.g., 4sU 60 min subset) and matrices are Float32 after conversion

## Limitations

- Metabolic labeling protocols are largely confined to in vitro studies, limiting in vivo application
- Training data may be small (the biological application used only 405 cells), which can limit capture of rare events such as cell cycle checkpoints
- Genes with high proportions of zeroes (default >98%) cause problems with gradient descent and must be removed before training
- Expression predictions are capped post hoc at two times the maximum observed in the training data (in log space) rather than learning a prior for out-of-range expression

## Evidence
