---
name: signature-matrix-construction-from-marker-genes
description: Use when you have a curated marker gene set (e.g., from G-PC/phenotype-cover,
  RankCorr, or scGeneFit) plus a scRNA-seq dataset with cell-type labels, and you
  need to assess how well those markers support deconvolution of bulk or pseudo-bulk
  mixtures.
when_to_use_negative:
- When the input is already bulk data without single-cell reference profiles to build
  a signature matrix
- When the goal is marker selection itself rather than evaluating a marker set — use
  GreedyPC/CEMPC selection first
- When you need an optimal (not approximate) marker set — G-PC does not provide an
  optimal solution
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: phenotype-cover (GreedyPC)
  role: Selects the marker gene set (phenotype cover solution) from which the signature
    matrix is built
  repo: https://github.com/euxhenh/phenotype-cover
- name: CIBERSORT
  role: Performs deconvolution of pseudo-bulk mixtures against the signature matrix
    using linear ν-SVR (ν ∈ {0.25, 0.5, 0.75})
- name: scikit-learn
  role: Supports the ν-SVR regression and classification utilities used in the deconvolution
    pipeline
- name: numpy
  role: Numerical computation for averaging expression profiles and computing divergences
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.crmeth.2022.100332
    title: Multiset multicover methods for discriminative marker selection
  source_content_hash: sha256:1b9d67baaca85094aa179c93ab78ea55e13ef0bad459ab308ccce4293c0ac3c4
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/euxhenh/phenotype-cover
  repo_url: https://github.com/euxhenh/phenotype-cover
  tools:
  - CIBERSORT
  - GreedyPC
  - Python
  - numpy
  - phenotype-cover (GreedyPC)
  - scikit-learn
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# Signature matrix construction from marker genes

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Builds a genes × cell types signature matrix from a selected marker gene set by averaging per-cell-type expression profiles, then uses it as the basis for pseudo-bulk deconvolution with CIBERSORT (linear ν-SVR) evaluated by Jensen-Shannon divergence between predicted and true cell-type proportions.

## When to use

Use when you have a curated marker gene set (e.g., from G-PC/phenotype-cover, RankCorr, or scGeneFit) plus a scRNA-seq dataset with cell-type labels, and you need to assess how well those markers support deconvolution of bulk or pseudo-bulk mixtures.

## When NOT to use

- When the input is already bulk data without single-cell reference profiles to build a signature matrix
- When the goal is marker selection itself rather than evaluating a marker set — use GreedyPC/CEMPC selection first
- When you need an optimal (not approximate) marker set — G-PC does not provide an optimal solution

## Inputs

- scRNA-seq expression matrix with cell-type annotations (e.g., IPF lung dataset from Adams et al.)
- Marker gene sets of matched size per method (e.g., 163 genes each from G-PC, RankCorr, scGeneFit)
- Known cell-type proportions for each pseudo-bulk mixture (ground truth)

## Outputs

- Genes × cell types signature matrix built from per-cell-type average expression of marker genes
- Pseudo-bulk expression mixtures with ground-truth proportions
- Per-method average Jensen-Shannon divergence (e.g., js_divergences.csv with one row per method and a JS field)

## How to apply

From each marker set (e.g., the 163-gene G-PC set on the IPF Adams et al. scRNA-seq data), build a signature matrix of average expression profiles per cell type restricted to the marker genes. Construct pseudo-bulk mixtures by averaging expression across all single cells in the test set, recording the known cell-type proportions as ground truth. Run CIBERSORT deconvolution on each pseudo-bulk mixture: initialize three linear ν-SVR instances with ν ∈ {0.25, 0.5, 0.75} and save the model achieving the lowest root-mean-square error. Compare predicted versus true proportions per mixture and compute the average Jensen-Shannon divergence across mixtures for each method (expected on IPF at 163 genes: G-PC 0.045, RankCorr 0.056, scGeneFit 0.062). The rationale is that deconvolution needs discriminatory markers between all cell-type pairs, not just good markers per type — which is what phenotype-cover-style marker selection optimizes.

## Related tools

- **phenotype-cover (GreedyPC)** (Selects the marker gene set (phenotype cover solution) from which the signature matrix is built) — https://github.com/euxhenh/phenotype-cover
- **CIBERSORT** (Performs deconvolution of pseudo-bulk mixtures against the signature matrix using linear ν-SVR (ν ∈ {0.25, 0.5, 0.75}))
- **scikit-learn** (Supports the ν-SVR regression and classification utilities used in the deconvolution pipeline)
- **numpy** (Numerical computation for averaging expression profiles and computing divergences)

## Examples

```
from phenotype_cover import GreedyPC; gpc = GreedyPC(); gpc.fit(X, y); features = gpc.select(163)
```

## Evaluation signals

- Average Jensen-Shannon divergence on IPF at 163 genes reproduces the reported ranking: G-PC 0.045 < RankCorr 0.056 < scGeneFit 0.062
- js_divergences.csv contains one row per marker-selection method with the JS field present
- Signature matrix has dimensions (number of marker genes) × (number of cell types) with no missing average-expression entries
- For each mixture, the retained ν-SVR model is the one of the three ν ∈ {0.25, 0.5, 0.75} instances with the lowest RMSE
- Predicted proportions per mixture are non-negative and, after normalization, comparable against the recorded ground-truth proportions

## Limitations

- G-PC does not provide an optimal solution to the phenotype cover problem, so the resulting signature matrix may be suboptimal
- The biological analysis relied on overlaps with existing gene marker lists that may be incomplete
- Pseudo-bulk evaluation uses mixtures averaged from single cells of test sets, which may not reflect real bulk tissues; stability checks (half-data resampling over 5 runs) are needed for the underlying marker sets

## Evidence
