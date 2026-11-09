---
name: cross-embedding-dr-quality-comparison
description: Use when two or more dimensionality-reduction algorithms (e.g. t-SNE
  vs UMAP) have been run on the same dataset and you need a statistical, per-cell
  measure of which embedding and which hyperparameter setting better preserves high-dimensional
  neighborhoods.
when_to_use_negative:
- The goal is a single global embedding-quality score rather than local, per-cell
  p-values against an empirical null — EMBEDR's value is per-sample statistical comparison,
  not a global metric.
- The dataset has not been normalized/processed consistently with EMBEDR's marginal-resampling
  null generation, which has not been extended to non-normalized datasets.
- Only one DR algorithm at one hyperparameter setting is of interest with no comparison
  planned — running the full sweep and null machinery is unnecessary overhead.
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0091
tools:
- name: EMBEDR
  role: Generates data and null embeddings via marginal resampling and computes per-cell
    EMBEDR p-values used to compare t-SNE and UMAP quality; EMBEDR_sweep performs
    the perplexity/n_neighbors sweep and fit_samplewise_optimal yields cell-wise optimal
    embeddings.
  repo: github.com/ejohnson643/EMBEDR
- name: openTSNE/FIt-SNE
  role: Backend implementation used for the t-SNE embeddings (DRA='tsne'); fast FIt-SNE
    requires fftw.
  repo: github.com/KlugerLab/FIt-SNE
- name: UMAP
  role: DRA='umap' backend generating the UMAP embeddings at default (kEff=15) and
    EMBEDR-optimized n_neighbors.
- name: numpy
  role: Loading the count matrix and handling arrays; required dependency of EMBEDR.
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.patter.2022.100443
    title: 'EMBEDR: Distinguishing signal from noise in single-cell omics data'
  source_content_hash: sha256:bd0cdaf4d0161be85398e13612cd5da446dbaf8f574410690e2c866eb41b8236
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/ejohnson643/EMBEDR
  tools:
  - EMBEDR
  - Python
  - UMAP
  - numpy
  - openTSNE
  - openTSNE/FIt-SNE
  - scikit-learn
  - scipy
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8125
n_claims: 16
---

# cross-embedding-dr-quality-comparison

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Compares the neighborhood-structure quality of t-SNE and UMAP embeddings of the same scRNA-seq data at default versus EMBEDR-optimized hyperparameters, using per-cell EMBEDR p-values as the comparison metric.

## When to use

Use when two or more dimensionality-reduction algorithms (e.g. t-SNE vs UMAP) have been run on the same dataset and you need a statistical, per-cell measure of which embedding and which hyperparameter setting better preserves high-dimensional neighborhoods.

## When NOT to use

- The goal is a single global embedding-quality score rather than local, per-cell p-values against an empirical null — EMBEDR's value is per-sample statistical comparison, not a global metric.
- The dataset has not been normalized/processed consistently with EMBEDR's marginal-resampling null generation, which has not been extended to non-normalized datasets.
- Only one DR algorithm at one hyperparameter setting is of interest with no comparison planned — running the full sweep and null machinery is unnecessary overhead.

## Inputs

- scRNA-seq count matrix (~4,800 Tabula Muris bone marrow cells) from the Tabula Muris Consortium deposit (FACS counts CSV)
- EMBEDR package installed via pip (requires numpy, scikit-learn, scipy, conda, numba; fftw recommended for FIt-SNE)

## Outputs

- Four 2D embeddings of the marrow data: t-SNE default (perplexity 15), t-SNE sample-wise-optimal, UMAP default (n_neighbors 15), UMAP sample-wise-optimal
- Per-cell EMBEDR p-values for each embedding
- Counts-and-percentages table (rows: embeddings and p-value thresholds; columns: cell counts and percentages), e.g. figure5_pvalue_threshold_table.csv

## How to apply

Load the scRNA-seq count matrix (e.g. ~4,800 Tabula Muris bone marrow cells from the Tabula Muris Consortium Figshare deposit) into numpy/pandas. Run the EMBEDR class with DRA='tsne' (via openTSNE/FIt-SNE) and DRA='umap' at the default neighborhood size (k/kEff=15), using n_data_embed of 3-10 and enough n_null_embed runs to give the empirical p-values adequate resolution. Use EMBEDR_sweep with sweep_type='perplexity' for t-SNE and 'n_neighbors' for UMAP, visualize with sweep_boxplot/sweep_lineplot to identify EMBEDR-optimal neighborhood sizes, then call fit_samplewise_optimal for per-cell optimal parameters. For each of the four embeddings (t-SNE default, t-SNE optimized, UMAP default, UMAP optimized), count cells whose EMBEDR p-value falls below each threshold and convert to percentages; with EMBEDR-optimized hyperparameters, t-SNE embeds more than 50% of marrow cells far better than the random null, improving over UMAP at the same neighborhood size and over both methods' defaults.

## Related tools

- **EMBEDR** (Generates data and null embeddings via marginal resampling and computes per-cell EMBEDR p-values used to compare t-SNE and UMAP quality; EMBEDR_sweep performs the perplexity/n_neighbors sweep and fit_samplewise_optimal yields cell-wise optimal embeddings.) — github.com/ejohnson643/EMBEDR
- **openTSNE/FIt-SNE** (Backend implementation used for the t-SNE embeddings (DRA='tsne'); fast FIt-SNE requires fftw.) — github.com/KlugerLab/FIt-SNE
- **UMAP** (DRA='umap' backend generating the UMAP embeddings at default (kEff=15) and EMBEDR-optimized n_neighbors.)
- **numpy** (Loading the count matrix and handling arrays; required dependency of EMBEDR.)

## Examples

```
from EMBEDR import EMBEDR, EMBEDR_sweep; embObj = EMBEDR(project_dir='./projects/', DRA='tsne', n_data_embed=5, n_null_embed=10); embObj.fit(X)
```

## Evaluation signals

- figure5_pvalue_threshold_table.csv exists and contains a row for each embedding/p-value-threshold combination (t-SNE default, t-SNE optimized, UMAP default, UMAP optimized).
- All percentages in the table fall between 0 and 100, and cell counts do not exceed the total number of marrow cells (~4,800).
- The EMBEDR-optimized t-SNE embedding shows more than 50% of cells with p-values far better than the random null, exceeding UMAP at the same neighborhood size and both defaults.
- Sweep diagnostics (sweep_boxplot/sweep_lineplot) show a clear optimum in EMBEDR p-values across perplexity (t-SNE) and n_neighbors (UMAP) values, and fit_samplewise_optimal only selects parameter values from the sweep grid.
- EMBEDR p-value resolution is bounded by the number of null embeddings generated; verify enough nulls were run for the reported thresholds.

## Limitations

- Calculated p-values have finite resolution because null distributions are computed empirically; the number of nulls that can be embedded determines the lower bound on p-values.
- No theoretical description of the tails of the null distributions exists and no principled method for parameterizing the null distribution is available.
- EMBEDR's null-generation process has not been extended to non-normalized datasets, so full quality analyses of entire data-processing pipelines are not supported.
- Runtime depends on the speed of the underlying DR algorithm implementations used to generate many null embeddings; GPU extensions and quadratic-rate optimization schemes were not included in this work.

## Evidence

- [abstract] EMBEDR generates a "p" value for each sample, allowing for direct comparisons of DR algorithms and facilitating optimization of algorithm hyperparameters: "EMBEDR generates a "p" value for each sample, allowing for direct comparisons of DR algorithms and facilitating optimization of algorithm hyperparameters"
- [intro] scRNA-seq data from nearly 4,800 bone marrow cells from the Tabula Muris Cell Atlas have been embedded in 2D using t-SNE and UMAP: "scRNA-seq data from nearly 4,800 bone marrow cells from the Tabula Muris Cell Atlas8 have been embedded in 2D using t-SNE51 and UMAP23"
