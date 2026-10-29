---
name: single-cell-embeddr-pvalue-estimation
description: Use when a dimensionality-reduced single-cell embedding (t-SNE/UMAP of
  scRNA-seq or scATAC-seq counts) needs per-cell statistical quality assessment, hyperparameter
  (perplexity/kEff) optimization, or separation of biological signal from embedding
  artifacts.
when_to_use_negative:
- The dataset is not normalized — the article notes EMBEDR's null-generation process
  'has not been extended to non-normalized datasets', so entire raw pre-normalization
  pipelines cannot be quality-assessed
- You need a single global quality score rather than local per-cell error estimates;
  EMBEDR is explicitly designed for local (per cell) estimates against an empirical
  null
- Very fine p-value resolution is required — p values are bounded below by the number
  of null embeddings since null distributions are computed empirically
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0091
tools:
- name: EMBEDR
  role: Python package providing the EMBEDR and EMBEDR_sweep classes that generate
    data/null embeddings, compute per-sample p values, and visualize sweeps
  repo: https://github.com/ejohnson643/EMBEDR
- name: openTSNE
  role: Custom variation used as the t-SNE dimensionality-reduction algorithm (DRA='tsne')
    whose embeddings EMBEDR evaluates
- name: FIt-SNE
  role: Fast FFT-accelerated t-SNE implementation used by EMBEDR when fftw is installed
  repo: https://github.com/KlugerLab/FIt-SNE
- name: numpy
  role: Loads and formats the count matrix passed to EMBEDR.fit (e.g., np.loadtxt(...).astype(float))
- name: scikit-learn
  role: Installation dependency of the EMBEDR package
- name: scipy
  role: Installation dependency of the EMBEDR package
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.patter.2022.100443
    title: 'EMBEDR: Distinguishing signal from noise in single-cell omics data'
  source_content_hash: sha256:23d9fe5cee3eb21cc1f5d162e2cbe2bc2ad2aa464939a2058e3e32f24e7c5379
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/ejohnson643/EMBEDR
  repo_url: https://github.com/ejohnson643/EMBEDR
  tools:
  - EMBEDR
  - FIt-SNE
  - Python
  - numpy
  - openTSNE
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
yes_rate: 0.7857142857142857
n_claims: 14
---

# single-cell-embeddr-pvalue-estimation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Estimates a per-sample EMBEDR p value that tests whether each cell's neighborhood structure in a 2D embedding (e.g., openTSNE-based t-SNE of Tabula Muris scRNA-seq data) reflects real high-dimensional signal rather than algorithm-induced noise, using an empirical null built from marginally-resampled data.

## When to use

Use when a dimensionality-reduced single-cell embedding (t-SNE/UMAP of scRNA-seq or scATAC-seq counts) needs per-cell statistical quality assessment, hyperparameter (perplexity/kEff) optimization, or separation of biological signal from embedding artifacts.

## When NOT to use

- The dataset is not normalized — the article notes EMBEDR's null-generation process 'has not been extended to non-normalized datasets', so entire raw pre-normalization pipelines cannot be quality-assessed
- You need a single global quality score rather than local per-cell error estimates; EMBEDR is explicitly designed for local (per cell) estimates against an empirical null
- Very fine p-value resolution is required — p values are bounded below by the number of null embeddings since null distributions are computed empirically

## Inputs

- High-dimensional feature matrix of single-cell counts as a numpy array (e.g., np.loadtxt of a tab-delimited matrix, or Tabula Muris FACS tissue counts CSV such as Diaphragm-counts.csv)
- EMBEDR parameters: DRA ('tsne' or 'umap'), perplexity or nearest_neighbors (default 10% of samples), n_data_embed, n_null_embed, project_dir/project_name

## Outputs

- Per-sample EMBEDR p values indicating which cells are better embedded than expected by chance
- 2D data embeddings and null embeddings cached in the project directory
- Plots: embedding colored by EMBEDR p value, sweep_boxplot and sweep_lineplot summaries across perplexity/kEff values
- Optionally a sample-wise (cell-wise) optimal embedding via fit_samplewise_optimal

## How to apply

Install EMBEDR by cloning github.com/ejohnson643/EMBEDR and running `pip install .` (requires numpy, scikit-learn, scipy, conda, numba; seaborn for figures; fftw for the fast FIt-SNE t-SNE implementation). Load the count matrix (e.g., ~4,800 Tabula Muris bone marrow cells from the FACS raw data CSVs), then instantiate the EMBEDR class with a project_dir for file caching, DRA='tsne', and n_data_embed of 3-10 (recommended over the default of 1) plus at least one marginally-resampled null embedding. Calling `fit` generates data and null embeddings and computes each sample's p value by comparing neighborhood affinities in the high- and low-dimensional spaces against the empirical null distribution. For scale sweeps, wrap this in the EMBEDR_sweep class with sweep_type='perplexity' and the desired sweep_values, then summarize with sweep_boxplot/sweep_lineplot; EMBEDR_sweep reuses cached embeddings so changing n_data_embed only generates the additional runs.

## Related tools

- **EMBEDR** (Python package providing the EMBEDR and EMBEDR_sweep classes that generate data/null embeddings, compute per-sample p values, and visualize sweeps) — https://github.com/ejohnson643/EMBEDR
- **openTSNE** (Custom variation used as the t-SNE dimensionality-reduction algorithm (DRA='tsne') whose embeddings EMBEDR evaluates)
- **FIt-SNE** (Fast FFT-accelerated t-SNE implementation used by EMBEDR when fftw is installed) — https://github.com/KlugerLab/FIt-SNE
- **numpy** (Loads and formats the count matrix passed to EMBEDR.fit (e.g., np.loadtxt(...).astype(float)))
- **scikit-learn** (Installation dependency of the EMBEDR package)
- **scipy** (Installation dependency of the EMBEDR package)

## Examples

```
from EMBEDR import EMBEDR_sweep; sweepObj = EMBEDR_sweep(DRA='tsne', n_data_embed=3, n_null_embed=1, sweep_type='perplexity', sweep_values=[25, 100, 250], project_dir='./projects/'); sweepObj.fit(X)
```

## Evaluation signals

- The fraction of cells with minimal (lowest) EMBEDR p values should be interpretable per kEff; for Tabula Muris marrow this peaks at kEff≈1200 with a secondary plateau near kEff≈150
- The sweep_boxplot of p-value distributions across perplexity values should reproduce the expected Figure 4D pattern (median fraction of minimal p-values peaking at a single kEff)
- Changing n_data_embed after a cached run should generate only the additional embeddings, confirming file caching and nearest-neighbor/affinity reuse are working
- The embedding colored by EMBEDR p value should show low p values (real structure) concentrated in biologically interpretable clusters rather than uniformly or randomly distributed
- Null embeddings from marginally resampled data should yield p values consistent with the empirical null distribution across samples

## Limitations

- Runtime depends on the speed of current DR algorithm implementations since many (null) embeddings of the same data must be generated; GPU extensions and quadratic-rate optimization schemes were not included
- Calculated p values have finite resolution because null distributions are empirical — the number of nulls bounds the lower limit of achievable p values
- No theoretical description of the tails of the null distributions exists, and there is no principled method for parameterizing the null distribution
- The null-generation process has not been extended to non-normalized datasets
- For subtle cluster detail, 'near' neighbors from approximate NN methods may be insufficient; vantage-point trees (exact nearest neighbors) are recommended

## Evidence
