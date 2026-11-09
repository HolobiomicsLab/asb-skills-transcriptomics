---
name: sweep-summary-visualization
description: Use when an EMBEDR_sweep over perplexity/kEff values has been fitted
  and you need to visualize per-sample p-value distributions across scales or summarize
  p-value trajectories by annotation groups (e.g. Tabula Muris cell types).
when_to_use_negative:
- When no hyperparameter sweep has been run — these visualizations require an EMBEDR_sweep
  object fitted across multiple perplexity/kEff values, not a single EMBEDR fit
- When evaluating a single DR embedding's global quality rather than characterizing
  how embedding quality varies across neighborhood scales or samples
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: EMBEDR
  role: Provides the EMBEDR_sweep class whose sweep_boxplot and sweep_lineplot methods
    render sweep summary figures
  repo: github.com/ejohnson643/EMBEDR
- name: Python
  role: Runtime language for computing per-cell-type median/90th-percentile summaries
    with numpy/scipy and saving CSV/PNG outputs
- name: openTSNE
  role: t-SNE implementation (custom variation) underlying the tsne DRA whose perplexity
    sweep is being visualized
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.patter.2022.100443
    title: 'EMBEDR: Distinguishing signal from noise in single-cell omics data'
  source_content_hash: sha256:8e39719e81a71211edb6c735267ac2abbc35ac0a5c4ba6a7d15bfaaa5efcec6b
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
  - numpy
  - openTSNE
  - scikit-learn
  - scipy
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# sweep-summary-visualization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Visualize EMBEDR hyperparameter (perplexity/kEff) sweep results with sweep_boxplot and sweep_lineplot to expose the p-value spectrum across scales, and export per-group (e.g. per-cell-type) median/90th-percentile p-value summary tables for reproduction figures.

## When to use

Use when an EMBEDR_sweep over perplexity/kEff values has been fitted and you need to visualize per-sample p-value distributions across scales or summarize p-value trajectories by annotation groups (e.g. Tabula Muris cell types).

## When NOT to use

- When no hyperparameter sweep has been run — these visualizations require an EMBEDR_sweep object fitted across multiple perplexity/kEff values, not a single EMBEDR fit
- When evaluating a single DR embedding's global quality rather than characterizing how embedding quality varies across neighborhood scales or samples

## Inputs

- Fitted EMBEDR_sweep object with per-sample EMBEDR p-values at each sweep value
- Per-cell p-value trajectories grouped by cell-type annotation (e.g. macrophages, granulocytes, B cells)
- Sweep parameter values (perplexity/kEff list, e.g. [25, 100, 250] or spanning ~150-1200)

## Outputs

- Sweep boxplot figure of EMBEDR p-value distributions per sweep value
- Sweep lineplot figure of p-value statistics across the sweep
- Per-cell-type median and 90th-percentile p-value table (figure6_celltype_summary.csv)
- Reproduction figure (figure6_celltype_summary.png)

## How to apply

After running an EMBEDR_sweep (e.g. DRA='tsne' with perplexity/kEff values spanning ~150 to ~1200 on the ~4,800-cell Tabula Muris bone marrow data, n_data_embed >= 3 and n_null_embed >= 1), call the sweep object's sweep_boxplot and sweep_lineplot methods to render the full p-value spectrum across sweep values. To reproduce Figure 6-style cell-type-specific spectra, group per-cell p-value trajectories by the cell-type annotation (from czbiohub tabula-muris data_ingest metadata) and compute the median and 90th-percentile p-value per cell type at every kEff. Save the summary table (e.g. figure6_celltype_summary.csv) and the corresponding figure (e.g. figure6_celltype_summary.png). Caching of kNN graphs and affinity matrices in the project directory means re-visualization after changing n_data_embed does not recompute existing embeddings.

## Related tools

- **EMBEDR** (Provides the EMBEDR_sweep class whose sweep_boxplot and sweep_lineplot methods render sweep summary figures) — github.com/ejohnson643/EMBEDR
- **Python** (Runtime language for computing per-cell-type median/90th-percentile summaries with numpy/scipy and saving CSV/PNG outputs)
- **openTSNE** (t-SNE implementation (custom variation) underlying the tsne DRA whose perplexity sweep is being visualized)

## Examples

```
sweepObj = EMBEDR_sweep(project_name='fig6', project_dir='./projects/', DRA='tsne', n_data_embed=3, n_null_embed=1, sweep_type='perplexity', sweep_values=[150, 300, 600, 1200]); sweepObj.fit(X); sweepObj.sweep_boxplot(); sweepObj.sweep_lineplot()
```

## Evaluation signals

- All reported p-value statistics (medians and 90th percentiles) lie within [0, 1]
- The summary figure reproduces the qualitative Figure 6 pattern: macrophages best embedded near kEff ~150 while granulocytes and B cells prefer kEff ~1200
- Every sweep value in the sweep_values list appears in the boxplot/lineplot with both data and null embedding p-values represented
- The per-cell-type summary CSV contains one row per (cell type, kEff) pair with both median and 90th-percentile columns populated

## Limitations

- EMBEDR p-values have finite resolution because null distributions are computed empirically; the number of null embeddings bounds the lower limit of attainable p values
- The sample-wise optimal perplexity is selected only from the values used in the sweep
- Timings for hyperparameter sweeps (figures like Figure 4) are only reported in supplementary Table S2, not in the main text

## Evidence
