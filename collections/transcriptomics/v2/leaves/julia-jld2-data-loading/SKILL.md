---
name: julia-jld2-data-loading
description: Use when the input is a JLD2 (.jld2) file containing labeled/unlabeled/total
  scRNA-seq count matrices (e.g. scEU-seq rpeExampleData) and they must be loaded
  into Julia for RNAForecaster training or t0 estimation.
when_to_use_negative:
- The data are in AnnData (.h5ad) or other HDF5 layouts produced by the Dynamo python
  package rather than JLD2 — conversion happens outside this skill.
- The pipeline is in Python/R (e.g. Seurat 4.0.1 for UMAP visualization) and a Julia-native
  JLD2 load would add a needless cross-language hop.
- The matrices are already the t0Estimate output of estimateT0LabelingData and only
  need downstream sparsity/log-normalization filtering.
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: RNAForecaster
  role: Consumer of the JLD2-loaded Float32 count matrices; loading must produce gene-by-cell
    arrays compatible with estimateT0LabelingData and neural ODE training
  repo: https://github.com/rossinerbe/RNAForecaster.jl
- name: JLD2
  role: Julia package used to deserialize the rpeExampleData.jld2 container holding
    labeled/unlabeled/total matrices and labeling times
- name: Julia 1.6
  role: Runtime language in which RNAForecaster (with Flux 0.13.0 and DiffEqFlux 1.46.0)
    is implemented and in which JLD2 loading is performed
  repo: https://julialang.org/
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.patter.2023.100793
    title: Transcriptomic forecasting with neural ordinary differential equations
  source_content_hash: sha256:7066b8417b1de3bf4c7f8ed2a86250bc6d9d2250e34b2ce4bc792bdd714d016e
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
  - Julia 1.6
  - RNAForecaster
  - scEU-seq
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# julia-jld2-data-loading

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Load metabolic-labeling scRNA-seq count matrices (labeled, unlabeled, total) and labeling-time vectors stored in JLD2 format into Julia as Float32 arrays, the required first step before RNAForecaster degradation-rate estimation and t=0 matrix construction.

## When to use

Use when the input is a JLD2 (.jld2) file containing labeled/unlabeled/total scRNA-seq count matrices (e.g. scEU-seq rpeExampleData) and they must be loaded into Julia for RNAForecaster training or t0 estimation.

## When NOT to use

- The data are in AnnData (.h5ad) or other HDF5 layouts produced by the Dynamo python package rather than JLD2 — conversion happens outside this skill.
- The pipeline is in Python/R (e.g. Seurat 4.0.1 for UMAP visualization) and a Julia-native JLD2 load would add a needless cross-language hop.
- The matrices are already the t0Estimate output of estimateT0LabelingData and only need downstream sparsity/log-normalization filtering.

## Inputs

- JLD2 (.jld2) file such as data/rpeExampleData.jld2 containing labeled counts matrix, unlabeled counts matrix, total counts matrix, and labeling-time vector (rpeExampleData[1..5])
- Julia 1.6+ environment with the JLD2 package

## Outputs

- In-memory Julia arrays of the labeled, unlabeled, and total count matrices converted to Float32 (genes x cells)
- Labeling-time vector passed to estimateT0LabelingData

## How to apply

Load the .jld2 container with the JLD2 Julia package (e.g. rpeExampleData.jld2) and extract the labeled, unlabeled, and total count matrices plus the labeling-time vector (indexed as rpeExampleData[1..5]). Convert every matrix to Float32 so downstream RNAForecaster steps — the linear regression of labeled vs. total counts, the degradation rate computed as -log(1 - slope), and estimateT0LabelingData — operate on a consistent numeric type. Verify the loaded objects are gene-by-cell matrices matching the dimensions of the unlabeled counts, since these matrices feed directly into estimateT0LabelingData(labeledData, totalData, unlabeledData, labelingTime). Note the article's ecosystem runs on Julia 1.6 with Flux 0.13.0 and DiffEqFlux 1.46.0, so JLD2 loading is performed inside that Julia environment rather than in Python/Seurat.

## Related tools

- **RNAForecaster** (Consumer of the JLD2-loaded Float32 count matrices; loading must produce gene-by-cell arrays compatible with estimateT0LabelingData and neural ODE training) — https://github.com/rossinerbe/RNAForecaster.jl
- **JLD2** (Julia package used to deserialize the rpeExampleData.jld2 container holding labeled/unlabeled/total matrices and labeling times)
- **Julia 1.6** (Runtime language in which RNAForecaster (with Flux 0.13.0 and DiffEqFlux 1.46.0) is implemented and in which JLD2 loading is performed) — https://julialang.org/

## Examples

```
using JLD2; d = load("data/rpeExampleData.jld2"); labeledData = Float32.(d["rpeExampleData"][1]); unlabeledData = Float32.(d["rpeExampleData"][2]); totalData = Float32.(d["rpeExampleData"][3])
```

## Evaluation signals

- All expected keys (labeled, unlabeled, total, labeling time; rpeExampleData[1..5]) resolve from the .jld2 file without load errors.
- Every loaded count matrix converts to Float32 with identical gene-by-cell dimensions across the three matrices.
- Loaded matrices contain only finite, non-negative counts and no missing entries.
- Dimensions of loaded matrices match the downstream t0Estimate output dimensions required by RNAForecaster training.
- Subsequent estimateT0LabelingData(labeledData, totalData, unlabeledData, labelingTime) call type-checks without Float64/Float32 promotion issues.

## Limitations

- JLD2 loading is Julia-native; the source scEU-seq data (rpeLabeling.h5ad from Battich et al. via the Dynamo python package) must first be converted to .jld2, which this skill does not cover.
- Float32 conversion may lose precision relative to stored counts, which is acceptable here but should be checked if matrices are reused for other numeric analyses.
- Labeling-time heterogeneity must already be resolved (cells subset to 4sU uridine 60 min treatment) before loading, since the workflow assumes a single labelingTime.

## Evidence
