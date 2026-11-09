---
name: cell-type-stratified-pvalue-aggregation
description: Use when a per-sample EMBEDR p-value sweep across kEff/perplexity exists
  for an annotated single-cell dataset and you need to determine whether cell types
  are embedded at different neighborhood scales.
when_to_use_negative:
- The dataset has no cell-type annotations to stratify by, so only a global sweep_boxplot/sweep_lineplot
  summary is possible
- You need a single global optimal perplexity for the whole dataset rather than cell-wise
  or population-wise scale comparisons (use the EMBEDR_sweep global summary or fit_samplewise_optimal
  instead)
- The input is a non-normalized dataset — EMBEDR's null-generation process has not
  been extended to non-normalized datasets per the discussion
tools:
- name: EMBEDR
  role: Generates per-sample p-values for each data embedding by comparison against
    marginal-resampling null embeddings; the EMBEDR_sweep class manages the kEff/perplexity
    sweep and its sweep_boxplot/sweep_lineplot functions visualize the stratified
    results
  repo: https://github.com/ejohnson643/EMBEDR
- name: openTSNE
  role: Provides the t-SNE DRA whose custom variation produces the embeddings evaluated
    at each sweep value
- name: numpy
  role: Numeric backend for computing per-cell-type median and 90th-percentile p-value
    summaries
- name: scipy
  role: Required dependency of the EMBEDR package used for statistical computations
- name: scikit-learn
  role: Required dependency of the EMBEDR package (neighborhood/kNN computations underlying
    the p-value statistic)
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.patter.2022.100443
    title: 'EMBEDR: Distinguishing signal from noise in single-cell omics data'
  source_content_hash: sha256:094bb00e8b9e873f1068941c4d3362aa34c478587b2e5957fb213da1f26991c7
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
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# cell-type-stratified-pvalue-aggregation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Stratifies per-sample EMBEDR p-values from a perplexity/kEff sweep by cell-type annotation (e.g., Tabula Muris marrow macrophages, granulocytes, B cells) and computes median and 90th-percentile trajectories per type to reveal that different cell populations have distinct native embedding scales.

## When to use

Use when a per-sample EMBEDR p-value sweep across kEff/perplexity exists for an annotated single-cell dataset and you need to determine whether cell types are best embedded at different neighborhood scales.

## When NOT to use

- The dataset has no cell-type annotations to stratify by, so only a global sweep_boxplot/sweep_lineplot summary is possible
- You need a single global optimal perplexity for the whole dataset rather than cell-wise or population-wise scale comparisons (use the EMBEDR_sweep global summary or fit_samplewise_optimal instead)
- The input is a non-normalized dataset — EMBEDR's null-generation process has not been extended to non-normalized datasets per the discussion

## Inputs

- Tabula Muris bone marrow scRNA-seq count matrix (~4,800 cells) from the Figshare deposit
- Cell-type annotations from the czbiohub tabula-muris data_ingest metadata
- Per-sample EMBEDR p-value trajectories from an EMBEDR_sweep over kEff/perplexity values (DRA='tsne', n_data_embed >= 3, n_null_embed >= 1)

## Outputs

- Per-cell-type median and 90th-percentile p-value table at every kEff value (figure6_celltype_summary.csv)
- Sweep summary figure reproducing the per-cell-type p-value spectra (figure6_celltype_summary.png)
- Identified optimal embedding scale per cell type (e.g., macrophages ~kEff 150; granulocytes and B cells ~kEff 1200)

## How to apply

Run an EMBEDR_sweep (wrapping the EMBEDR class) over kEff/perplexity values spanning small neighborhoods (~150) to large ones (~1200) with DRA='tsne', n_data_embed of at least 3 (3–10 recommended) and n_null_embed >= 1, relying on the project-directory caching of kNN graphs and affinity matrices. At each sweep value, per-sample EMBEDR p-values come from comparing neighborhood correspondence of data embeddings against marginal-resampling null embeddings. Group the per-cell p-value trajectories by the cell-type annotation from the czbiohub tabula-muris data_ingest metadata, then compute the median and 90th-percentile p-value per cell type at every kEff. Visualize with the sweep_boxplot and sweep_lineplot functions and export the summary table. Rationale: EMBEDR's per-sample p-values enable 'cell-wise optimal' scale selection, so stratification by cell type exposes population-specific characteristic scales rather than a single global optimum.

## Related tools

- **EMBEDR** (Generates per-sample p-values for each data embedding by comparison against marginal-resampling null embeddings; the EMBEDR_sweep class manages the kEff/perplexity sweep and its sweep_boxplot/sweep_lineplot functions visualize the stratified results) — https://github.com/ejohnson643/EMBEDR
- **openTSNE** (Provides the t-SNE DRA whose custom variation produces the embeddings evaluated at each sweep value)
- **numpy** (Numeric backend for computing per-cell-type median and 90th-percentile p-value summaries)
- **scipy** (Required dependency of the EMBEDR package used for statistical computations)
- **scikit-learn** (Required dependency of the EMBEDR package (neighborhood/kNN computations underlying the p-value statistic))

## Examples

```
sweepObj = EMBEDR_sweep(project_dir='./projects/', DRA='tsne', n_data_embed=3, n_null_embed=1, sweep_type='perplexity', sweep_values=[150, 300, 600, 900, 1200]); sweepObj.fit(X); sweepObj.sweep_lineplot()
```

## Evaluation signals

- All per-cell-type median and 90th-percentile p-value statistics lie within [0, 1]
- The summary figure qualitatively reproduces the Figure 6 pattern: macrophages best embedded near kEff ~ 150, granulocytes and B cells near kEff ~ 1200
- The exported figure6_celltype_summary.csv contains one median and one 90th-percentile trajectory per cell type across every kEff value in the sweep
- Sweep was run with the minimum recommended settings (n_data_embed >= 3, n_null_embed >= 1); cached kNN graphs and affinity matrices were reused rather than recomputed

## Limitations

- EMBEDR p-values have finite empirical resolution: the number of null embeddings bounds the lower limit on attainable p-values, and no theoretical description of the null-distribution tails exists
- Runtime of a sweep depends on the speed of current DR implementations used to generate many null embeddings; GPU extensions and quadratic-rate optimization schemes were not included in this work
- The sample-wise optimal perplexity can only be selected from the sweep values actually used, so the sweep grid must include the neighborhood scales of interest (e.g., ~150 to ~1200)

## Evidence

- [abstract] We show that the scale of a sample's neighborhood can thus be determined and used to generate a novel "cell-wise optimal" embedding.: "We show that the scale of a sample's neighborhood can thus be determined and used to generate a novel "cell-wise optimal" embedding."
- [readme] This class wraps around the EMBEDR class to manage this parameter sweep.: "This is can be performed in this package using the `EMBEDR_sweep` class.  This class wraps around the `EMBEDR` class to manage this parameter sweep."
- [readme] We can also visualize the entire sweep using the sweep_boxplot and sweep_lineplot functions: "We can also visualize the entire sweep using the `sweep_boxplot` and `sweep_lineplot` functions"
- [readme] find characteristic scales and neighborhood sizes for different samples: "determine optimal values for `perplexity` (or `n_neighbors` in UMAP), find characteristic scales and neighborhood sizes for different samples"
- [discussion] there is a finite resolution to the calculated p values since the null distributions are calculated empirically.: "there is a finite resolution to the calculated p values since the null distributions are calculated empirically."
