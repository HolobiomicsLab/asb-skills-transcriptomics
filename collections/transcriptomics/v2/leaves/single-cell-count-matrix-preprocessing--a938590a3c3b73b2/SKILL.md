---
name: single-cell-count-matrix-preprocessing
description: Use when raw scRNA-seq count matrices (e.g., scEU-seq or BoolODE simulated
  data, genes x cells) are about to be fed into RNAForecaster and still contain high
  sparsity, non-log-transformed counts, or Float64 values.
when_to_use_negative:
- The matrices are already sparsity-filtered, log-normalized, and Float32 — re-applying
  filtering may discard informative genes.
- The goal is constructing the t=0 input matrix from metabolic labeling data (labeled
  + degraded transcripts); that requires estimateT0LabelingData with degradation rates,
  not plain preprocessing.
- The input is bulk RNA-seq or already-dimensionality-reduced embeddings rather than
  gene-by-cell single-cell count matrices.
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: RNAForecaster
  role: Target pipeline whose gradient-descent training on count data requires the
    sparsity filtering and log normalization performed by this skill
  repo: https://github.com/rossinerbe/RNAForecaster.jl
- name: Flux.jl
  role: Julia machine-learning framework in which the neural ODE is trained, motivating
    Float32 conversion of the preprocessed matrices
- name: JLD2
  role: Loads the raw labeled/unlabeled/total count matrices (e.g., rpeExampleData[1..5]
    from data/rpeExampleData.jld2) for preprocessing
provenance:
  source_task_ids:
  - task_007
  - task_001
  source_papers:
  - doi: 10.1016/j.patter.2023.100793
    title: Transcriptomic forecasting with neural ordinary differential equations
  source_content_hash: sha256:d753a6eabcdb03a6b38b6415ba25b43bffacce59240cba211f9fd2528a4c46ab
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
  - DiffEqFlux.jl
  - Flux.jl
  - JLD2
  - RNAForecaster
  - scEU-seq
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# single-cell-count-matrix-preprocessing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Prepares single-cell RNA-seq count matrices (genes-as-rows/cells-as-columns) for RNAForecaster neural ODE training by removing genes with high zero proportions, log-normalizing, converting to Float32, and optionally filtering to top-quartile variance genes.

## When to use

Use when raw scRNA-seq count matrices (e.g., scEU-seq or BoolODE simulated data, genes x cells) are about to be fed into RNAForecaster and still contain high sparsity, non-log-transformed counts, or Float64 values.

## When NOT to use

- The matrices are already sparsity-filtered, log-normalized, and Float32 — re-applying filtering may discard informative genes.
- The goal is constructing the t=0 input matrix from metabolic labeling data (labeled + degraded transcripts); that requires estimateT0LabelingData with degradation rates, not plain preprocessing.
- The input is bulk RNA-seq or already-dimensionality-reduced embeddings rather than gene-by-cell single-cell count matrices.

## Inputs

- Single-cell count matrix formatted genes-as-rows/cells-as-columns (e.g., from scEU-seq rpeLabeling.h5ad or BoolODE simulated t=0/t=1 matrices)
- Optional labeled/unlabeled/total count matrices plus labeling-time vector stored in JLD2 format (data/rpeExampleData.jld2)

## Outputs

- Sparsity-filtered, log1p-normalized, Float32 count matrices suitable for RNAForecaster trainRNAForecaster
- Gene-masked matrices restricted to top-quartile variance genes (for biological datasets)

## How to apply

Load the count matrices (e.g., labeled/unlabeled/total matrices from data/rpeExampleData.jld2 via JLD2, or paired t=0/t=1 BoolODE matrices) and convert all matrices to Float32 for Julia/Flux.jl compatibility. Remove genes with a high proportion of zeros (default threshold: greater than 98% zeroes), because retaining such sparse genes causes problems with gradient descent. Apply log normalization (log1p) to the counts, as sparsity filtering plus log normalization are the main preprocessing steps for RNAForecaster. For biological datasets such as the scEU-seq hTERT retinal pigment epithelium subset, additionally filter to genes in the top quartile of variance, since highly variable gene filtering was strongly recommended and performed for all biological datasets. Output the filtered, log-normalized, Float32 gene-by-cell matrices that feed the 80-20 train/validation split before Tsit5-based neural ODE training.

## Related tools

- **RNAForecaster** (Target pipeline whose gradient-descent training on count data requires the sparsity filtering and log normalization performed by this skill) — https://github.com/rossinerbe/RNAForecaster.jl
- **Flux.jl** (Julia machine-learning framework in which the neural ODE is trained, motivating Float32 conversion of the preprocessed matrices)
- **JLD2** (Loads the raw labeled/unlabeled/total count matrices (e.g., rpeExampleData[1..5] from data/rpeExampleData.jld2) for preprocessing)

## Examples

```
counts = convert(Matrix{Float32}, log1p.(counts)) # after removing genes with >98% zero counts
```

## Evaluation signals

- No retained gene exceeds the 98% zero-count proportion threshold after filtering.
- All output matrices are of Julia Float32 type and share the same gene-by-cell dimensions as the input matrices.
- Values are log1p-transformed (no raw linear counts remain) and contain no non-finite entries.
- For biological datasets, the retained gene set equals the top quartile by variance.
- Downstream trainRNAForecaster runs converge without gradient-descent failures attributable to sparse genes.

## Limitations

- Genes with high proportions of zeroes (default greater than 98%) must be removed because they cause problems with gradient descent, so rare but biologically relevant very-low-detection genes are lost.
- The biological scEU-seq training subset contained only 405 cells; preprocessing cannot compensate for small training sets and may limit capture of rare cell cycle events.
- Predicted expression beyond an allowed maximum (default two times the maximum observed in the training data, in log space) must be capped post hoc — preprocessing does not teach the network this prior.

## Evidence
