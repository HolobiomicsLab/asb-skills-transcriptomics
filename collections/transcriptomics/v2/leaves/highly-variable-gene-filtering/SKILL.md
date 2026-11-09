---
name: highly-variable-gene-filtering
description: Use when preparing single-cell count matrices (e.g. scEU-seq rpeLabeling.h5ad
  or .jld2) for RNAForecaster neural-ODE training, when high gene sparsity or dimensionality
  would destabilize gradient descent.
when_to_use_negative:
- The matrices are already sparsity-filtered and restricted to highly variable genes
  (re-filtering would discard informative genes)
- The downstream analysis needs the full gene universe, e.g. unbiased marker discovery
  or pathway-level quantification rather than neural ODE forecasting
- The data are not single-cell counts but bulk or already-normalized low-dimensional
  embeddings, where the 98% zero cutoff and top-quartile variance rule do not apply
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: RNAForecaster
  role: Julia package whose training step requires the sparsity-filtered, highly-variable-gene,
    log-normalized input matrices produced by this skill
  repo: https://github.com/rossinerbe/RNAForecaster.jl
- name: Julia 1.6
  role: Language in which the filtering and downstream RNAForecaster preprocessing/training
    are implemented
  repo: https://github.com/rossinerbe/RNAForecaster.jl
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.patter.2023.100793
    title: Transcriptomic forecasting with neural ordinary differential equations
  source_content_hash: sha256:021acc9478a3512d739817afe7761ae57d2117d4da0f8354b17ee0856cc99b96
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
  - Julia 1.6
  - RNAForecaster
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# highly-variable-gene-filtering

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Preprocessing skill from the RNAForecaster neural-ODE paper: remove genes with extreme sparsity (>98% zero counts) and restrict to the top-quartile variance genes before training neural ODEs on scRNA-seq data, because spurious zeroes destabilize gradient descent.

## When to use

Use when preparing single-cell count matrices (e.g. scEU-seq rpeLabeling.h5ad or .jld2) for RNAForecaster neural-ODE training, when high gene sparsity or dimensionality would destabilize gradient descent.

## When NOT to use

- The matrices are already sparsity-filtered and restricted to highly variable genes (re-filtering would discard informative genes)
- The downstream analysis needs the full gene universe, e.g. unbiased marker discovery or pathway-level quantification rather than neural ODE forecasting
- The data are not single-cell counts but bulk or already-normalized low-dimensional embeddings, where the 98% zero cutoff and top-quartile variance rule do not apply

## Inputs

- Cell-by-gene count matrices from metabolic labeling scRNA-seq (e.g. scEU-seq rpeLabeling.h5ad / rpeExampleData.jld2, or estimated t0/t1 total-expression matrices)
- Gene name vector aligned to matrix columns

## Outputs

- Filtered log1p-normalized Float32 expression matrices restricted to top-quartile variance genes (zero-count genes >98% removed)
- Filtered gene name list used consistently for t0 and t1 matrices

## How to apply

After estimating the clean t=0 matrix (e.g. with estimateT0LabelingData on labeled/unlabeled/total counts from metabolic labeling data), first drop genes whose zero-count proportion exceeds the 98% default, since these cause problems with gradient descent. Then compute per-gene variance across cells and retain only genes in the top quartile, as was done for the scEU-seq hTERT RPE 405-cell 60-minute-labeling subset. Apply log1p normalization to both the t0 and t1 matrices, casting to Float32 for training. The rationale is that sparsity filtering plus log normalization are the two preprocessing steps RNAForecaster requires before neural ODE training with the Tsit5 solver.

## Related tools

- **RNAForecaster** (Julia package whose training step requires the sparsity-filtered, highly-variable-gene, log-normalized input matrices produced by this skill) — https://github.com/rossinerbe/RNAForecaster.jl
- **Julia 1.6** (Language in which the filtering and downstream RNAForecaster preprocessing/training are implemented) — https://github.com/rossinerbe/RNAForecaster.jl

## Evaluation signals

- No retained gene has a zero-count proportion greater than 98% across cells
- Retained gene set equals the top variance quartile (e.g. ~25% of genes from the scEU-seq t0/t1 matrices) and is identical across the t0 and t1 matrices
- Matrices are log1p-normalized and typed Float32 before trainRNAForecaster is called
- Downstream check: recursive 72-hour forecasts scored with the tricycle 0–2π cell-cycle embedding should be significantly more ordered than random (p < 1e-16 at all hours), confirming the filtered inputs support reproducible training

## Limitations

- Filtering to highly variable genes was a recommendation applied to the paper's biological datasets; the specific top-quartile cutoff may need adjustment for datasets with different sparsity profiles
- Even after filtering, the biological application trained on only 405 cells, which the authors note may be too few to capture rare cell-cycle events
- Genes are removed before training, so any cell-cycle genes outside the top variance quartile are excluded from forecasting and downstream interpretation

## Evidence
