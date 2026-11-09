---
name: single-cell-rna-seq-dimensionality-reduction-embedding
description: Use when a raw scRNA-seq count matrix (e.g., Tabula Muris FACS marrow
  counts) needs 2D embedding with t-SNE/UMAP and a statistically grounded, per-cell
  assessment of whether the visible structure is real rather than stochastic.
when_to_use_negative:
- The data are not single-cell count matrices — EMBEDR's null-generation process has
  not been extended to non-normalized datasets, so whole-pipeline quality analyses
  of arbitrary feature tables are out of scope.
- Only a single global quality estimate is needed rather than local, per-cell error
  estimates against a null hypothesis.
- The downstream task requires strictly 'nearest' neighbors rather than approximate
  ones (use vantage-point trees instead of Annoy-based FIt-SNE nearest neighbors when
  subtle detail is required).
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: EMBEDR
  role: Wraps t-SNE/UMAP to generate data and marginally-resampled null embeddings,
    compute per-cell EMBEDR p-values, and support hyperparameter sweeps via the EMBEDR_sweep
    class
  repo: https://github.com/ejohnson643/EMBEDR
- name: openTSNE
  role: Custom variation of the openTSNE implementation (FIt-SNE) used as the t-SNE
    dimensionality reduction algorithm (DRA='tsne')
  repo: https://github.com/KlugerLab/FIt-SNE
- name: numpy
  role: Loads and casts the counts matrix (e.g., np.loadtxt(...).astype(float)) required
    for EMBEDR installation and fitting
- name: scikit-learn
  role: Required dependency for EMBEDR package installation and nearest-neighbor computations
- name: scipy
  role: Required dependency for EMBEDR package installation
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.patter.2022.100443
    title: 'EMBEDR: Distinguishing signal from noise in single-cell omics data'
  source_content_hash: sha256:7f59b02af200a2ab7270b0d6f69719409ab3a532f7edbdb8bfd601d509a04341
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
  - numpy
  - openTSNE
  - scikit-learn
  - scipy
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# single-cell-rna-seq-dimensionality-reduction-embedding

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Embed scRNA-seq count data (e.g., ~4,800 Tabula Muris bone marrow cells) in 2D with t-SNE (FIt-SNE/openTSNE) and UMAP, and use EMBEDR's marginal-resampling nulls to assign per-cell p-values that quantify which parts of the embedding reflect real structure versus algorithm-induced noise.

## When to use

Use when a raw scRNA-seq count matrix (e.g., Tabula Muris FACS marrow counts) needs 2D embedding with t-SNE/UMAP and a statistically grounded, per-cell assessment of whether the visible structure is real rather than stochastic.

## When NOT to use

- The data are not single-cell count matrices — EMBEDR's null-generation process has not been extended to non-normalized datasets, so whole-pipeline quality analyses of arbitrary feature tables are out of scope.
- Only a single global quality estimate is needed rather than local, per-cell error estimates against a null hypothesis.
- The downstream task requires strictly 'nearest' neighbors rather than approximate ones (use vantage-point trees instead of Annoy-based FIt-SNE nearest neighbors when subtle detail is required).

## Inputs

- scRNA-seq count matrix (~4,800 Tabula Muris bone marrow cells; e.g., FACS*-counts.csv from the Tabula Muris Figshare deposit)
- DRA choice and hyperparameters (DRA='tsne' or 'umap'; perplexity or nearest_neighbors, default k/kEff=15)
- n_data_embed (3-10 recommended) and n_null_embed counts for empirical p-value resolution

## Outputs

- 2D embeddings colored by per-cell EMBEDR p-value (default and sample-wise-optimal)
- EMBEDR p-value per cell for each data embedding
- Sweep summaries (sweep boxplots/lineplots) of p-values vs. perplexity or n_neighbors
- Counts-and-percentages table of cells below p-value thresholds (e.g., figure5_pvalue_threshold_table.csv)

## How to apply

Load the raw counts matrix (e.g., Tabula Muris FACS bone marrow, ~4,800 cells) with numpy/pandas, then instantiate the EMBEDR class with DRA='tsne' (FIt-SNE/openTSNE implementation) or DRA='umap' and call fit to generate multiple data embeddings and marginally-resampled null embeddings; set n_data_embed to 3-10 (the README-recommended practice) and enough null embeddings for adequate empirical p-value resolution. Compare algorithms and scales by wrapping EMBEDR in the EMBEDR_sweep class with sweep_type='perplexity' (t-SNE) or 'n_neighbors' (UMAP), visualizing with sweep_boxplot/sweep_lineplot to identify EMBEDR-optimal neighborhood sizes. Call fit_samplewise_optimal to select each cell's optimal perplexity/n_neighbors and generate the 'cell-wise optimal' embedding, choosing optimal values only from those used in the sweep. Finally, count cells whose EMBEDR p-value falls below chosen thresholds for each embedding (e.g., t-SNE/UMAP default vs. optimized) and convert to percentages of all marrow cells; with EMBEDR-optimized hyperparameters, t-SNE embeds more than 50% of marrow cells far better than the random null.

## Related tools

- **EMBEDR** (Wraps t-SNE/UMAP to generate data and marginally-resampled null embeddings, compute per-cell EMBEDR p-values, and support hyperparameter sweeps via the EMBEDR_sweep class) — https://github.com/ejohnson643/EMBEDR
- **openTSNE** (Custom variation of the openTSNE implementation (FIt-SNE) used as the t-SNE dimensionality reduction algorithm (DRA='tsne')) — https://github.com/KlugerLab/FIt-SNE
- **numpy** (Loads and casts the counts matrix (e.g., np.loadtxt(...).astype(float)) required for EMBEDR installation and fitting)
- **scikit-learn** (Required dependency for EMBEDR package installation and nearest-neighbor computations)
- **scipy** (Required dependency for EMBEDR package installation)

## Examples

```
from EMBEDR import EMBEDR, EMBEDR_sweep; embObj = EMBEDR(project_dir='./projects/', DRA='tsne', n_data_embed=3, n_null_embed=1); embObj.fit(X)
```

## Evaluation signals

- The output CSV contains a row for each embedding/p-value-threshold combination, with percentages between 0 and 100 and counts not exceeding the total number of marrow cells (~4,800).
- EMBEDR p-values have finite resolution bounded by the number of null embeddings generated — verify n_null_embed was set so p-values reach the desired thresholds.
- With EMBEDR-optimized hyperparameters, the optimized t-SNE should embed more than 50% of marrow cells far better than the random null, improving over UMAP at the same neighborhood size and over both methods' defaults.
- Re-running EMBEDR with a larger n_data_embed should reuse cached embeddings (file caching), generating only the additional embeddings needed.
- Sample-wise-optimal embedding can only be produced after a sweep has run; optimal perplexity/n_neighbors values must come from the values used in the sweep.

## Limitations

- Calculated p values have finite resolution because null distributions are computed empirically; the number of nulls bounds the lower limit of p values.
- No theoretical description of the tails of the null distributions exists, and no principled method for parameterizing the null distribution exists.
- Runtime depends on the speed of current DR algorithm implementations used to generate many (null) embeddings; GPU extensions and quadratic rate optimization schemes were not included.
- EMBEDR's null-generation process has not been extended to non-normalized datasets, limiting quality analyses of entire data-processing pipelines.
- The optimal perplexity values for the sample-wise optimal embedding will only be selected from those used in the sweep.

## Evidence
