---
name: metabolic-labeling-scrnaseq-data-processing
description: Use when input is metabolic labeling scRNA-seq (scEU-seq h5ad like rpeLabeling.h5ad)
  with labeled/total/unlabeled counts and a labeling time, and you must prepare paired
  t0/t1 matrices for RNAForecaster forecasting.
when_to_use_negative:
- Data come from standard scRNA-seq protocols, which destroy cells to measure them
  and therefore cannot provide paired time-point measurements of the same cell required
  by RNAForecaster
- In vivo samples without metabolic labeling, since these protocols are currently
  largely confined to in vitro studies
- The data are already log-normalized and variance-filtered paired t0/t1 matrices
  ready for trainRNAForecaster — applying redundant filtering could remove signal
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: RNAForecaster
  role: Julia package providing estimateT0LabelingData and the neural ODE training
    functions that consume the preprocessed paired time-point matrices
  repo: https://github.com/rossinerbe/RNAForecaster.jl
- name: Flux.jl
  role: Machine-learning package used to train the neural ODE on the preprocessed
    labeled scRNA-seq matrices
  repo: https://github.com/FertigLab/RNAForecasterPaperCode
- name: DiffEqFlux.jl
  role: Used with the Tsit5 ODE solver for neural ODE training after data preprocessing
    is complete
  repo: https://github.com/FertigLab/RNAForecasterPaperCode
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.patter.2023.100793
    title: Transcriptomic forecasting with neural ordinary differential equations
  source_content_hash: sha256:4b4f03d692648142ed5f9ac501ee808e0378c5f7b9001f8ea676dfecbb72eaa7
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
  - DiffEqFlux.jl
  - Flux
  - Flux.jl
  - RNAForecaster
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

# metabolic-labeling-scrnaseq-data-processing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Preprocessing pipeline for metabolic labeling scRNA-seq data (e.g., scEU-seq 4sU uridine labeling) to produce log-normalized, sparsity- and variance-filtered paired time-point matrices suitable for RNAForecaster neural ODE training, including degradation rate estimation via estimateT0LabelingData.

## When to use

Use when input is metabolic labeling scRNA-seq (scEU-seq h5ad like rpeLabeling.h5ad) with labeled/total/unlabeled counts and a labeling time, and you must prepare paired t0/t1 matrices for RNAForecaster forecasting.

## When NOT to use

- Data come from standard scRNA-seq protocols, which destroy cells to measure them and therefore cannot provide paired time-point measurements of the same cell required by RNAForecaster
- In vivo samples without metabolic labeling, since these protocols are currently largely confined to in vitro studies
- The data are already log-normalized and variance-filtered paired t0/t1 matrices ready for trainRNAForecaster — applying redundant filtering could remove signal

## Inputs

- scEU-seq metabolic labeling AnnData file (e.g., rpeLabeling.h5ad from Battich et al.)
- Labeled, total, and unlabeled count matrices from metabolically labeled cells
- Labeling time metadata (e.g., 4sU uridine 60 min) and labelingTime parameter
- Gene count matrices requiring sparsity filtering and log normalization

## Outputs

- Subset, sparsity-filtered (>98% zero-count genes removed) and log1p-normalized t0/t1 expression matrices
- Top-quartile-variance-filtered gene expression matrices
- Degradation rate estimates per gene (t0Estimate from estimateT0LabelingData)
- Paired time-point matrices ready for trainRNAForecaster (e.g., trainingProp = 1.0, learningRate = 0.0001, batchsize = 25)

## How to apply

Start from the metabolic labeling dataset (e.g., the scEU-seq rpeLabeling.h5ad from Battich et al., downloaded via the Dynamo python package) and subset to cells treated with 4sU uridine for 60 min so the labeling time is equal for all input cells. Remove genes with a high proportion of zeroes (default >98% zero counts) because they cause problems with gradient descent, and log-normalize the count matrices; filtering to highly variable genes (top-quartile variance) is strongly recommended and was performed for all biological datasets. Estimate degradation rates by fitting a linear regression between the labeled and total count matrices, computing the degradation rate as -log(1 - slope), which in RNAForecaster is done with estimateT0LabelingData(labeledData, totalData, unlabeledData, labelingTime). The resulting paired earlier/later count matrices (e.g., t0/t1, log1p-normalized) then feed trainRNAForecaster for neural ODE training and downstream recursive predictCellFutures forecasting.

## Related tools

- **RNAForecaster** (Julia package providing estimateT0LabelingData and the neural ODE training functions that consume the preprocessed paired time-point matrices) — https://github.com/rossinerbe/RNAForecaster.jl
- **Flux.jl** (Machine-learning package used to train the neural ODE on the preprocessed labeled scRNA-seq matrices) — https://github.com/FertigLab/RNAForecasterPaperCode
- **DiffEqFlux.jl** (Used with the Tsit5 ODE solver for neural ODE training after data preprocessing is complete) — https://github.com/FertigLab/RNAForecasterPaperCode

## Examples

```
using RNAForecaster; t0Estimate = estimateT0LabelingData(labeledData, totalData, unlabeledData, 60.0)
```

## Evaluation signals

- No genes remain in the matrices with greater than 98% zero counts
- Genes retained are in the top quartile by variance, matching the scEU-seq processing in the paper
- All input cells share an equal labeling time (subset to 4sU uridine 60 min treatment)
- Degradation rates are finite and derived as -log(1 - slope) from a fitted linear regression of labeled vs. total counts
- Output matrices are log-normalized and can train RNAForecaster with trainingProp = 1.0 without gradient-descent failures

## Limitations

- Metabolic labeling scRNA-seq protocols required by this preprocessing are largely confined to in vitro studies, limiting in vivo application
- Standard scRNA-seq protocols cannot provide the paired time-point measurements of the same cell this pipeline requires
- The scEU-seq training subset contained only 405 cells, which may miss rare cell-cycle events even after careful preprocessing

## Evidence
