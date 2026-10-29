---
name: tsne-umap-neighborhood-hyperparameter-optimization
description: Use when comparing t-SNE and UMAP embedding quality at default (k=15)
  versus optimized neighborhood sizes on single-cell data, or when per-cell optimal
  perplexity/n_neighbors values are needed to distinguish real structure from algorithm-induced
  noise.
when_to_use_negative:
- Input is not a normalized scRNA-seq or similar high-dimensional feature matrix —
  the paper notes EMBEDR's null-generation process has not been extended to non-normalized
  datasets
- The goal is a single global DR quality score rather than local (per-cell) statistical
  assessment against an empirical null
- The DR algorithm is neither t-SNE nor UMAP — the EMBEDR package currently supports
  only 'tSNE' and 'UMAP' as DRA options
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3678
- http://edamontology.org/topic_3170
tools:
- name: EMBEDR
  role: Generates data and marginal-resampling null embeddings, computes per-sample
    EMBEDR p-values, and sweeps t-SNE perplexity / UMAP n_neighbors via the EMBEDR
    and EMBEDR_sweep classes (fit, fit_samplewise_optimal, sweep_boxplot, sweep_lineplot).
  repo: https://github.com/ejohnson643/EMBEDR
- name: openTSNE
  role: Custom variation of the openTSNE implementation (with the FIt-SNE backend)
    used as the t-SNE dimensionality reduction algorithm inside EMBEDR.
  repo: https://github.com/KlugerLab/FIt-SNE
- name: UMAP
  role: The second dimensionality reduction algorithm (DRA='umap') whose n_neighbors
    hyperparameter is swept and optimized against EMBEDR p-values.
- name: numpy
  role: Loads and formats the count matrix (e.g. np.loadtxt of the data file) before
    fitting EMBEDR.
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.patter.2022.100443
    title: 'EMBEDR: Distinguishing signal from noise in single-cell omics data'
  source_content_hash: sha256:c0a236a31896565b9d63401183dc1f631acf190c39a6764581e7ca807631acad
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
  - Python
  - UMAP
  - numpy
  - openTSNE
  - scikit-learn
  - scipy
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3678
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8947368421052632
n_claims: 19
---

# tsne-umap-neighborhood-hyperparameter-optimization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Uses EMBEDR's empirical marginal-resampling p-values to sweep t-SNE perplexity and UMAP n_neighbors on Tabula Muris bone marrow scRNA-seq data, identifying EMBEDR-optimal neighborhood sizes and generating sample-wise ('cell-wise') optimal embeddings that preserve local neighborhood structure better than default settings.

## When to use

Use when comparing t-SNE and UMAP embedding quality at default (k=15) versus optimized neighborhood sizes on single-cell data, or when per-cell optimal perplexity/n_neighbors values are needed to distinguish real structure from algorithm-induced noise.

## When NOT to use

- Input is not a normalized scRNA-seq or similar high-dimensional feature matrix — the paper notes EMBEDR's null-generation process has not been extended to non-normalized datasets
- The goal is a single global DR quality score rather than local (per-cell) statistical assessment against an empirical null
- The DR algorithm is neither t-SNE nor UMAP — the EMBEDR package currently supports only 'tSNE' and 'UMAP' as DRA options

## Inputs

- Tabula Muris FACS bone marrow scRNA-seq count matrix (~4,800 cells), e.g. from the Tabula Muris Consortium Figshare deposit
- EMBEDR package installed via pip (requires numpy, scikit-learn, scipy, conda, numba; fftw recommended for FIt-SNE)
- Sweep hyperparameter grids: perplexity values for t-SNE, n_neighbors values for UMAP

## Outputs

- Data and null (marginally resampled) embeddings at default and swept neighborhood sizes
- Per-cell EMBEDR p-values for t-SNE and UMAP at default (k=15) and optimized settings
- Sample-wise (cell-wise) optimal t-SNE and UMAP embeddings via fit_samplewise_optimal
- Counts-and-percentages table of cells below each p-value threshold per embedding (e.g. figure5_pvalue_threshold_table.csv)
- Sweep visualizations (sweep_boxplot, sweep_lineplot) identifying EMBEDR-optimal perplexity/n_neighbors

## How to apply

Load the Tabula Muris FACS bone marrow count matrix (~4,800 cells) from the Tabula Muris Consortium deposit into a numpy array. Instantiate the EMBEDR class with DRA='tsne' (via the FIt-SNE/openTSNE implementation) and DRA='umap' at the default neighborhood size (k/kEff=15), setting n_data_embed to 3-10 and n_null_embed large enough that the empirically estimated p-values have adequate resolution. Run EMBEDR_sweep with sweep_type='perplexity' (t-SNE) or 'n_neighbors' (UMAP), visualize with sweep_boxplot/sweep_lineplot to locate the EMBEDR-optimal neighborhood size, then call fit_samplewise_optimal to select each cell's optimal parameter from the swept values and generate sample-wise optimal embeddings. Finally, for each of the four embeddings (t-SNE/UMAP x default/optimized), count cells whose EMBEDR p-value falls below each threshold and convert to percentages of the ~4,800 cells; the expected result is that >50% of marrow cells are embedded far better than the marginal-resampling null under the optimized t-SNE, outperforming UMAP at the same neighborhood size and both methods' defaults.

## Related tools

- **EMBEDR** (Generates data and marginal-resampling null embeddings, computes per-sample EMBEDR p-values, and sweeps t-SNE perplexity / UMAP n_neighbors via the EMBEDR and EMBEDR_sweep classes (fit, fit_samplewise_optimal, sweep_boxplot, sweep_lineplot).) — https://github.com/ejohnson643/EMBEDR
- **openTSNE** (Custom variation of the openTSNE implementation (with the FIt-SNE backend) used as the t-SNE dimensionality reduction algorithm inside EMBEDR.) — https://github.com/KlugerLab/FIt-SNE
- **UMAP** (The second dimensionality reduction algorithm (DRA='umap') whose n_neighbors hyperparameter is swept and optimized against EMBEDR p-values.)
- **numpy** (Loads and formats the count matrix (e.g. np.loadtxt of the data file) before fitting EMBEDR.)

## Examples

```
from EMBEDR import EMBEDR, EMBEDR_sweep; sweepObj = EMBEDR_sweep(project_name='tm_marrow', project_dir='./projects/', DRA='tsne', sweep_type='perplexity', sweep_values=[15, 25, 50, 100], n_data_embed=5, n_null_embed=30); sweepObj.fit(X); sweepObj.fit_samplewise_optimal()
```

## Evaluation signals

- The counts-and-percentages CSV contains a row for each embedding (t-SNE default, t-SNE optimized, UMAP default, UMAP optimized) x p-value threshold combination
- Percentages per embedding fall between 0 and 100, and cell counts do not exceed the total number of marrow cells (~4,800)
- The EMBEDR-optimized t-SNE embeds more than 50% of marrow cells far better than the random marginal-resampling null
- Optimized t-SNE outperforms UMAP at the same neighborhood size and both methods' default settings (k/kEff=15)
- Sample-wise optimal parameters are selected only from the values included in the EMBEDR_sweep grid (fit_samplewise_optimal can only run after a sweep)

## Limitations

- EMBEDR p-values have finite resolution because null distributions are computed empirically; the number of null embeddings bounds the lower limit of achievable p-values
- No theoretical description of the tails of the null distributions or principled parameterization of the null exists
- Runtime depends on the speed of current DR algorithm implementations, since many (null) embeddings must be generated; GPU extensions and quadratic-rate optimization schemes were not included
- EMBEDR's null-generation process has not been extended to non-normalized datasets
- Optimal perplexity/n_neighbors values can only be chosen from those used in the sweep, and fit_samplewise_optimal can only be run after a sweep has been completed

## Evidence
