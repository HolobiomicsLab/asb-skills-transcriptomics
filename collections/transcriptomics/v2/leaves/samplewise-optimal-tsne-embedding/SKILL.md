---
name: samplewise-optimal-tsne-embedding
description: Use when an EMBEDR perplexity sweep (e.g. sweep_values=[25, 100, 250])
  has been fit to single-cell data (e.g. Tabula Muris marrow scRNA-seq) and you need
  a per-cell adaptive embedding for downstream clustering or visualization.
when_to_use_negative:
- No EMBEDR perplexity sweep has been fit yet — fit_samplewise_optimal can only be
  carried out after a sweep has been run
- You need a single globally optimal hyperparameter rather than per-cell scales —
  use the sweep-level optimal perplexity instead
- Input data is not normalized — EMBEDR's null-generation process has not been extended
  to non-normalized datasets
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3577
- http://edamontology.org/topic_3170
tools:
- name: EMBEDR
  role: Python package whose EMBEDR_sweep class performs the perplexity sweep, selects
    per-cell optimal perplexities via fit_samplewise_optimal, and renders the sample-wise
    optimal embedding via plot_embedding
  repo: https://github.com/ejohnson643/EMBEDR
- name: FIt-SNE
  role: Fast FFT-accelerated t-SNE implementation used as the tSNE DRA backend for
    generating embeddings
  repo: https://github.com/KlugerLab/FIt-SNE
- name: scikit-learn
  role: DBSCAN clustering of the cell-wise optimal embedding for validation against
    cell-type annotations (eps set to the 1.5th percentile of pairwise distances)
- name: numpy
  role: Array handling for loading count matrices (e.g. np.loadtxt of MNIST-style
    data) and computing pairwise distances
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.patter.2022.100443
    title: 'EMBEDR: Distinguishing signal from noise in single-cell omics data'
  source_content_hash: sha256:d699944dda7ac6ef1addf636d78684eb73956201292b00235eb4d305d529e427
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
  - scikit-learn
  - scipy
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3577
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 15
---

# samplewise-optimal-tsne-embedding

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Generates a novel 'cell-wise optimal' t-SNE embedding in which each cell is embedded at the perplexity from an EMBEDR sweep that minimizes its EMBEDR p-value, concentrating each sample at its most statistically significant neighborhood scale.

## When to use

Use when an EMBEDR perplexity sweep (e.g. sweep_values=[25, 100, 250]) has been fit to single-cell data (e.g. Tabula Muris marrow scRNA-seq) and you need a per-cell adaptive embedding for downstream clustering or visualization.

## When NOT to use

- No EMBEDR perplexity sweep has been fit yet — fit_samplewise_optimal can only be carried out after a sweep has been run
- You need a single globally optimal hyperparameter rather than per-cell scales — use the sweep-level optimal perplexity instead
- Input data is not normalized — EMBEDR's null-generation process has not been extended to non-normalized datasets

## Inputs

- Fitted EMBEDR_sweep object with per-cell EMBEDR p-values across the perplexity sweep (e.g. sweep_values=[25, 100, 250])
- High-dimensional single-cell count matrix (e.g. ~4,800 Tabula Muris bone marrow cells)
- Optional: per-cell cell-type annotations (e.g. Tabula Muris) for downstream validation

## Outputs

- Cell-wise (sample-wise) optimal t-SNE embedding of the data
- Per-cell optimal perplexity (kEff) assignments restricted to sweep values
- Rendered figure (e.g. samplewise_optimal_embedding.png) via EMBEDR_sweep.plot_embedding

## How to apply

After running an EMBEDR_sweep over perplexity values, call the EMBEDR_sweep.fit_samplewise_optimal method, which selects, for each cell, the perplexity (kEff) that minimizes that cell's EMBEDR p-value — restricted to the sweep values used. The method builds the per-cell adaptive affinity matrix and generates the cell-wise optimal t-SNE embedding via the EMBEDR package's tSNE DRA. Render it with EMBEDR_sweep.plot_embedding (e.g. saving samplewise_optimal_embedding.png); note this can only be done after a sweep has been run, and the optimal perplexity is only selected from sweep values. Validate by clustering the embedding (e.g. DBSCAN with eps at the 1.5th percentile of pairwise cell-cell distances) and checking concordance with expert annotations, as in the seven-cluster Tabula Muris marrow analysis.

## Related tools

- **EMBEDR** (Python package whose EMBEDR_sweep class performs the perplexity sweep, selects per-cell optimal perplexities via fit_samplewise_optimal, and renders the sample-wise optimal embedding via plot_embedding) — https://github.com/ejohnson643/EMBEDR
- **FIt-SNE** (Fast FFT-accelerated t-SNE implementation used as the tSNE DRA backend for generating embeddings) — https://github.com/KlugerLab/FIt-SNE
- **scikit-learn** (DBSCAN clustering of the cell-wise optimal embedding for validation against cell-type annotations (eps set to the 1.5th percentile of pairwise distances))
- **numpy** (Array handling for loading count matrices (e.g. np.loadtxt of MNIST-style data) and computing pairwise distances)

## Examples

```
sweepObj = EMBEDR_sweep(project_name='EMBEDR_Sweep_Example', DRA='tsne', sweep_type='perplexity', sweep_values=[25, 100, 250]); sweepObj.fit(X); sweepObj.fit_samplewise_optimal(); sweepObj.plot_embedding()
```

## Evaluation signals

- Optimal perplexity values for each cell are drawn only from the sweep values used (e.g. 25, 100, 250)
- The rendered figure (samplewise_optimal_embedding.png) comes from plot_embedding called after fit_samplewise_optimal, showing the sample-wise optimal embedding
- Downstream DBSCAN clustering (eps = 1.5th percentile of pairwise distances) recovers clusters concordant with annotated cell types, e.g. seven DBSCAN clusters matching the seven Tabula Muris marrow cell types
- Per-cell EMBEDR p-values at the chosen perplexity are lower (more significant) than at other sweep values for that cell

## Limitations

- The sample-wise optimal embedding can only be generated after a perplexity sweep has been run, and optimal perplexity values are restricted to the sweep values
- EMBEDR p-values have finite resolution since null distributions are computed empirically; the number of nulls bounds the lower limit of p-values
- Runtime depends on the speed of current DR algorithm implementations used to generate many null embeddings
- EMBEDR's null-generation process has not been extended to non-normalized datasets

## Evidence
