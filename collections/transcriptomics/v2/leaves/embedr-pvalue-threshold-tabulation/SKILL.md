---
name: embedr-pvalue-threshold-tabulation
description: Use when you have per-cell EMBEDR p-values for dimensionality-reduced
  scRNA-seq embeddings (e.g. t-SNE vs UMAP on ~4,800 Tabula Muris bone marrow cells)
  and need to quantify, per p-value threshold, how many cells exceed the random-null
  quality expectation.
when_to_use_negative:
- The dataset is not normalized — the article's discussion notes EMBEDR's null-generation
  process has not been extended to non-normalized datasets, so p-values (and hence
  threshold tables) would not be trustworthy
- Too few null embeddings were generated — empirical p-values have finite resolution
  bounded by the number of nulls, so low-threshold percentage counts would be meaningless
- The goal is a single global DR-quality score rather than per-sample (cell-wise)
  statistical comparison against a marginal-resampling null
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0091
tools:
- name: EMBEDR
  role: Generates per-sample p-values by comparing data embeddings to marginal-resampled
    null embeddings; the EMBEDR_sweep class identifies optimal perplexity/n_neighbors
    and fit_samplewise_optimal produces cell-wise optimal embeddings
  repo: https://github.com/ejohnson643/EMBEDR
- name: FIt-SNE (via openTSNE)
  role: Fast FFT-accelerated t-SNE implementation used as the t-SNE DRA inside EMBEDR
  repo: https://github.com/KlugerLab/FIt-SNE
- name: UMAP
  role: Alternative DRA compared against t-SNE at default and EMBEDR-optimized n_neighbors
- name: numpy
  role: Loads the count matrix (np.loadtxt/CSV parsing) and drives count/percentage
    computations
- name: scipy
  role: Required EMBEDR dependency used in the statistical machinery underlying null-distribution
    p-value estimation
- name: pandas
  role: Writes the counts-and-percentages threshold table to figure5_pvalue_threshold_table.csv
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.patter.2022.100443
    title: 'EMBEDR: Distinguishing signal from noise in single-cell omics data'
  source_content_hash: sha256:ba1f99f7d67259c3cb0b4ab8ae47408b9596bebbbb2316a6d855a3e99ec51114
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
  - FIt-SNE (via openTSNE)
  - Python
  - UMAP
  - numpy
  - openTSNE
  - pandas
  - scikit-learn
  - scipy
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# embedr-pvalue-threshold-tabulation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Tabulates, per embedding and per EMBEDR p-value threshold, the counts and percentages of single-cell samples that are better embedded than the random null. It converts EMBEDR's per-sample p-values (here for t-SNE and UMAP on Tabula Muris marrow data, at default and sample-wise-optimal neighborhood sizes) into a quantitative quality table.

## When to use

Use when you have per-cell EMBEDR p-values for dimensionality-reduced scRNA-seq embeddings (e.g. t-SNE vs UMAP on ~4,800 Tabula Muris bone marrow cells) and need to quantify, per p-value threshold, how many cells exceed the random-null quality expectation.

## When NOT to use

- The dataset is not normalized — the article's discussion notes EMBEDR's null-generation process has not been extended to non-normalized datasets, so p-values (and hence threshold tables) would not be trustworthy
- Too few null embeddings were generated — empirical p-values have finite resolution bounded by the number of nulls, so low-threshold percentage counts would be meaningless
- The goal is a single global DR-quality score rather than per-sample (cell-wise) statistical comparison against a marginal-resampling null

## Inputs

- Tabula Muris FACS bone marrow scRNA-seq count matrix (~4,800 cells), e.g. FACS/<Tissue>-counts.csv from the Tabula Muris Consortium Figshare deposit
- Per-cell EMBEDR p-values from data and marginal-resampled null embeddings for t-SNE and UMAP at default (k/kEff=15) and EMBEDR-optimized neighborhood sizes

## Outputs

- figure5_pvalue_threshold_table.csv — rows: embedding (t-SNE default, t-SNE optimized, UMAP default, UMAP optimized) x p-value threshold; columns: cell counts and percentages of total marrow cells

## How to apply

Load the Tabula Muris FACS bone marrow count matrix (~4,800 cells) with numpy/pandas and run the EMBEDR class with DRA='tsne' (FIt-SNE/openTSNE implementation) and DRA='umap' at the default neighborhood size (k/kEff=15), generating multiple data embeddings (n_data_embed 3-10 recommended) and enough null embeddings via marginal resampling to give empirical p-values adequate resolution. Run an EMBEDR_sweep (sweep_type='perplexity' for t-SNE, 'n_neighbors' for UMAP), inspect it with sweep_boxplot/sweep_lineplot to identify EMBEDR-optimal neighborhood sizes, then call fit_samplewise_optimal to generate per-cell-optimal embeddings. For each of the four embeddings (t-SNE default, t-SNE optimized, UMAP default, UMAP optimized), count cells whose EMBEDR p-value falls below each reported threshold and convert counts to percentages of the ~4,800 cells. Write the counts-and-percentages table to figure5_pvalue_threshold_table.csv; with EMBEDR-optimized hyperparameters, expect t-SNE to embed more than 50% of marrow cells far better than the random null, improving over UMAP at the same neighborhood size and over both methods' defaults.

## Related tools

- **EMBEDR** (Generates per-sample p-values by comparing data embeddings to marginal-resampled null embeddings; the EMBEDR_sweep class identifies optimal perplexity/n_neighbors and fit_samplewise_optimal produces cell-wise optimal embeddings) — https://github.com/ejohnson643/EMBEDR
- **FIt-SNE (via openTSNE)** (Fast FFT-accelerated t-SNE implementation used as the t-SNE DRA inside EMBEDR) — https://github.com/KlugerLab/FIt-SNE
- **UMAP** (Alternative DRA compared against t-SNE at default and EMBEDR-optimized n_neighbors)
- **numpy** (Loads the count matrix (np.loadtxt/CSV parsing) and drives count/percentage computations)
- **scipy** (Required EMBEDR dependency used in the statistical machinery underlying null-distribution p-value estimation)
- **pandas** (Writes the counts-and-percentages threshold table to figure5_pvalue_threshold_table.csv)

## Examples

```
sweepObj = EMBEDR_sweep(project_name='marrow', DRA='tsne', sweep_type='perplexity', n_data_embed=3, n_null_embed=1, n_jobs=-1); sweepObj.fit(X); sweepObj.fit_samplewise_optimal()
```

## Evaluation signals

- figure5_pvalue_threshold_table.csv exists and contains a row for every embedding x p-value-threshold combination (4 embeddings: t-SNE default/optimized, UMAP default/optimized)
- For every embedding, reported percentages lie strictly between 0 and 100
- Cell counts never exceed the total number of Tabula Muris marrow cells (~4,800) and are non-negative integers
- Percentages are monotonically non-increasing as the p-value threshold is tightened within each embedding
- Consistency check against the paper's finding: with EMBEDR-optimized hyperparameters, t-SNE embeds >50% of marrow cells far better than the random null and outperforms UMAP at the same neighborhood size and both defaults

## Limitations

- Calculated p-values have finite resolution because null distributions are computed empirically; the number of null embeddings bounds the lower limit of attainable p-values
- No theoretical description of the tails of the null distributions or principled method for parameterizing the null exists
- EMBEDR's null-generation process has not been extended to non-normalized datasets
- Optimal perplexity values from fit_samplewise_optimal can only be selected from the values actually used in the sweep, and sample-wise optimal fitting requires the sweep to have been run first
- Runtime depends on the speed of current DR implementations used to generate many null embeddings (GPU extensions and quadratic-rate optimization schemes were not included)

## Evidence
