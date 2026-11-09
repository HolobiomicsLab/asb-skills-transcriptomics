---
name: empirical-null-hypothesis-testing-via-marginal-resampling
description: Use when a t-SNE or UMAP embedding of single-cell omics data (e.g. ~4,800
  Tabula Muris marrow cells) must be judged for real vs. algorithm-induced structure,
  requiring a per-sample significance test rather than a global quality score.
when_to_use_negative:
- 'The dataset is not normalized: EMBEDR''s null-generation process has not been extended
  to non-normalized datasets'
- You need exact tail p-values beyond the empirical null resolution, since no theoretical
  description of the null distribution tails exists and p-value lower bounds are set
  by the number of nulls embedded
- You want a single global DR-quality metric rather than local, per-cell statistical
  assessment against a null hypothesis
tools:
- name: EMBEDR
  role: Python package implementing the marginal-resampling null generation and per-sample
    p-value calculation; provides EMBEDR and EMBEDR_sweep classes
  repo: github.com/ejohnson643/EMBEDR
- name: openTSNE
  role: Custom variation of the openTSNE implementation used as the t-SNE DRA whose
    embeddings are tested against the null
  repo: github.com/KlugerLab/FIt-SNE
- name: UMAP
  role: Alternative DRA compared against t-SNE at default and EMBEDR-optimized n_neighbors
- name: numpy
  role: Loads and formats the input matrix (e.g. np.loadtxt of MNIST or marrow count
    data)
- name: scikit-learn
  role: Required dependency for EMBEDR installation and neighborhood computations
- name: scipy
  role: Required dependency for EMBEDR installation and statistical calculations
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.patter.2022.100443
    title: 'EMBEDR: Distinguishing signal from noise in single-cell omics data'
  source_content_hash: sha256:356d076cb0833b61b347cce27bc84dc408a0696c366497baada9442d8f45ec01
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
  - scikit-learn
  - scipy
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7894736842105263
n_claims: 19
---

# Empirical null hypothesis testing via marginal resampling

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A statistical skill from EMBEDR: assess whether a dimensionality-reduction embedding (e.g. t-SNE or UMAP) contains structure more faithful to the high-dimensional data than expected by chance, by generating null datasets via marginal resampling and comparing neighborhood-preservation statistics to obtain a p-value for each sample.

## When to use

Use when a t-SNE or UMAP embedding of single-cell omics data (e.g. ~4,800 Tabula Muris marrow cells) must be judged for real vs. algorithm-induced structure, requiring a per-sample significance test rather than a global quality score.

## When NOT to use

- The dataset is not normalized: EMBEDR's null-generation process has not been extended to non-normalized datasets
- You need exact tail p-values beyond the empirical null resolution, since no theoretical description of the null distribution tails exists and p-value lower bounds are set by the number of nulls embedded
- You want a single global DR-quality metric rather than local, per-cell statistical assessment against a null hypothesis

## Inputs

- High-dimensional data matrix X (e.g. Tabula Muris bone marrow scRNA-seq counts, ~4,800 cells, loaded via numpy/pandas from the Tabula Muris Consortium deposit)
- Choice of dimensionality reduction algorithm (DRA='tsne' or 'umap') and its hyperparameter (perplexity or n_neighbors, e.g. k/kEff=15)
- Number of data embeddings (n_data_embed) and null embeddings (n_null_embed)

## Outputs

- Per-sample EMBEDR p-values quantifying whether each cell is better embedded than random chance
- 2D embedding colored by EMBEDR p-value
- Sweep diagnostics (sweep_boxplot/sweep_lineplot) of p-value quality vs perplexity or n_neighbors
- Sample-wise (cell-wise) optimal embedding from fit_samplewise_optimal
- Counts/percentages of cells below p-value thresholds per embedding (e.g. figure5_pvalue_threshold_table.csv)

## How to apply

Embed the data matrix X (e.g. the Tabula Muris FACS marrow count matrix) multiple times with the chosen DRA (tSNE via openTSNE/FIt-SNE, or UMAP) and embed one or more marginally-resampled null datasets the same way. Set DRA='tsne' or 'umap' on the EMBEDR class, choose perplexity/nearest_neighbors (default k/kEff=15 for the marrow data) and use n_data_embed of 3-10 and enough null embeddings to resolve small p-values empirically. For each sample, compute a neighborhood-correspondence statistic from the data embeddings and compare it against the empirical null distribution from the resampled embeddings to obtain a per-sample EMBEDR p-value; color the embedding by these p-values. To optimize hyperparameters or compare algorithms (e.g. t-SNE vs UMAP at default vs EMBEDR-optimized neighborhood sizes), run an EMBEDR_sweep over perplexity or n_neighbors, visualize with sweep_boxplot/sweep_lineplot, and call fit_samplewise_optimal for a per-cell optimal embedding. The null-based hypothesis test is what separates signal from DRA-induced noise, since DR distortions are inhomogeneous, stochastic, and hyperparameter-dependent.

## Related tools

- **EMBEDR** (Python package implementing the marginal-resampling null generation and per-sample p-value calculation; provides EMBEDR and EMBEDR_sweep classes) — github.com/ejohnson643/EMBEDR
- **openTSNE** (Custom variation of the openTSNE implementation used as the t-SNE DRA whose embeddings are tested against the null) — github.com/KlugerLab/FIt-SNE
- **UMAP** (Alternative DRA compared against t-SNE at default and EMBEDR-optimized n_neighbors)
- **numpy** (Loads and formats the input matrix (e.g. np.loadtxt of MNIST or marrow count data))
- **scikit-learn** (Required dependency for EMBEDR installation and neighborhood computations)
- **scipy** (Required dependency for EMBEDR installation and statistical calculations)

## Examples

```
from EMBEDR import EMBEDR; embObj = EMBEDR(project_dir='./projects/', DRA='tsne', n_data_embed=3); embObj.fit(X); embObj.plot()
```

## Evaluation signals

- Per-sample p-values are produced for every cell, not just a global score; the embedding colored by EMBEDR p-value should show high-p regions corresponding to visually distinct clusters
- Percentages of cells below any p-value threshold fall between 0 and 100, and cell counts never exceed the total number of samples (e.g. ~4,800 marrow cells)
- With enough nulls, data embeddings yield p-values significantly below 1 while null embeddings cluster near the null expectation (p ~ uniform/null-centered)
- t-SNE with EMBEDR-optimized hyperparameters embeds >50% of marrow cells far better than the random null, outperforming UMAP at the same neighborhood size and both methods' defaults
- Changing n_data_embed (e.g. 3 to 5) reuses cached embeddings so only the difference in embeddings is recomputed; file caching avoids recalculating nearest-neighbor graphs and affinity matrices

## Limitations

- There is a finite resolution to the calculated p values since the null distributions are computed empirically; the number of nulls bounds the lower limit of achievable p-values
- No theoretical description of the tails of the null distributions exists, and there is no principled method for parameterizing the null distribution
- EMBEDR's null-generation process has not been extended to non-normalized datasets
- Runtime depends on the speed of current DR algorithm implementations, since many (null) embeddings must be chained together

## Evidence
