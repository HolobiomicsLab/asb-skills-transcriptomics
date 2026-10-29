---
name: lineage-tracing-descendant-count-correlation
description: Use when scRNA-seq time-series data with lineage-tracing annotations
  is available and per-cell proliferation rates must be estimated from expression
  alone and validated against empirical clonal growth (descendant counts).
when_to_use_negative:
- The dataset lacks lineage-tracing annotations, so no empirical descendant counts
  exist to validate the expression-derived proliferation estimates against
- The goal is to use ground-truth proliferation rates directly (e.g. the weinreb-interpolate.sh
  task trains 'on only lineage tracing data with ground truth proliferation rates'),
  rather than estimating them from gene expression
- Data has not been pre-processed into a PRESCIENT data_pt torch object (normalized
  expression, time-point labels, cell type annotations)
edam_operation: http://edamontology.org/operation_3438
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: PRESCIENT
  role: Provides the data_pt torch object (data, tps, genes, pca, w) produced by `prescient
    process_data` that supplies the normalized day-2 expression and gene names used
    for KEGG scoring
  repo: https://github.com/gifford-lab/prescient
- name: PyTorch
  role: Used to load the data_pt torch object and extract expression, timepoints,
    and genes
- name: scikit-learn
  role: Builds the k-NN graph in PCA space for smoothing and fits the LogisticRegression
    mapping smoothed birth/death scores to fate probability; data_pt['pca'] is a sklearn.decomposition.PCA
    object
- name: gifford-lab/prescient-analysis
  role: Contains the preprocessing, analysis, and visualization notebooks for the
    Weinreb et al. 2020 hematopoiesis lineage tracing workflow
  repo: https://github.com/gifford-lab/prescient-analysis
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-021-23518-w
    title: Generative modeling of single-cell time series with PRESCIENT enables prediction
      of cell trajectories with interventions
  source_content_hash: sha256:6dc6e2e2b955fda7d631460e9111685158471c6ad3b2394748cdd53d8b29aebe
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gifford-lab/prescient
  tools:
  - PRESCIENT
  - PyTorch
  - gifford-lab/prescient-analysis
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3438
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# lineage-tracing-descendant-count-correlation

## Summary

Estimates per-cell proliferation (descendant) rates from gene expression alone using KEGG cell-cycle (birth) and apoptosis (death) gene-set z-scores, smoothed over a k-NN PCA graph, and validates them against empirical descendant counts from Weinreb lineage tracing data by Pearson correlation.

## When to use

Use when scRNA-seq time-series data with lineage-tracing annotations is available and per-cell proliferation rates must be estimated from expression alone and validated against empirical clonal growth (descendant counts).

## When NOT to use

- The dataset lacks lineage-tracing annotations, so no empirical descendant counts exist to validate the expression-derived proliferation estimates against
- The goal is to use ground-truth proliferation rates directly (e.g. the weinreb-interpolate.sh task trains 'on only lineage tracing data with ground truth proliferation rates'), rather than estimating them from gene expression
- Data has not been pre-processed into a PRESCIENT data_pt torch object (normalized expression, time-point labels, cell type annotations)

## Inputs

- PRESCIENT torch object data_pt (serialized dictionary from `prescient process_data`) with 'data' (normalized expression), 'tps' (timepoint labels), 'genes' (gene names), and 'pca' (sklearn PCA object)
- KEGG_CELL_CYCLE and KEGG_APOPTOSIS gene set annotations
- Weinreb et al. mouse hematopoiesis lineage tracing annotations with per-cell descendant counts

## Outputs

- Per-cell smoothed birth/death proliferation scores
- Pre-computed growth weights analogous to data_pt['w'] from the fitted logistic regression
- Pearson correlation statistic (r = 0.207, p << 1e-45) between gene-expression-derived proliferation estimates and empirical descendant counts (Fig. 2b)

## How to apply

Load the day-2 (timepoint 2) cells from the PRESCIENT torch object data_pt in PyTorch, extracting normalized expression (data_pt['data']), timepoint assignments (data_pt['tps']), and gene names (data_pt['genes']). Compute per-cell mean z-scores of the KEGG_CELL_CYCLE gene set (birth component) and the KEGG_APOPTOSIS gene set (death component) over the normalized expression matrix. Smooth the combined birth/death score across each cell's nearest neighbors in PCA space using a k-NN graph built with scikit-learn to reduce single-cell dropout noise. Fit a logistic regression (sklearn.linear_model.LogisticRegression) mapping the smoothed birth/death scores to fate probability, yielding pre-computed growth weights analogous to data_pt['w']. Compute empirical descendant counts per day-2 cell from the Weinreb lineage tracing annotations and correlate fitted scores with counts via scipy.stats.pearsonr.

## Related tools

- **PRESCIENT** (Provides the data_pt torch object (data, tps, genes, pca, w) produced by `prescient process_data` that supplies the normalized day-2 expression and gene names used for KEGG scoring) — https://github.com/gifford-lab/prescient
- **PyTorch** (Used to load the data_pt torch object and extract expression, timepoints, and genes)
- **scikit-learn** (Builds the k-NN graph in PCA space for smoothing and fits the LogisticRegression mapping smoothed birth/death scores to fate probability; data_pt['pca'] is a sklearn.decomposition.PCA object)
- **gifford-lab/prescient-analysis** (Contains the preprocessing, analysis, and visualization notebooks for the Weinreb et al. 2020 hematopoiesis lineage tracing workflow) — https://github.com/gifford-lab/prescient-analysis

## Examples

```
from scipy.stats import pearsonr; r, p = pearsonr(smoothed_growth_scores, empirical_descendant_counts)  # expect r ≈ 0.207, p << 1e-45
```

## Evaluation signals

- Pearson correlation between smoothed birth/death scores and empirical descendant counts reproduces r = 0.207 with p << 1e-45 (weak but significant), matching Fig. 2b
- Extracted cells all have timepoint assignment equal to day 2 (timepoint 2) in data_pt['tps']
- Mean z-scores are computed only over genes present in the KEGG_CELL_CYCLE and KEGG_APOPTOSIS gene sets and matched against data_pt['genes']
- Growth weights from the fitted logistic regression are dimensionally and index-wise comparable to the pre-computed data_pt['w']
- Smoothing neighborhood is built on PCA-space coordinates from the same sklearn PCA stored in data_pt['pca']

## Limitations

- The correlation is weak (r = 0.207): proliferation estimates from KEGG gene-set expression only partially agree with empirical rates, and the article notes 'The model also improves when incorporating growth rates, and would likely benefit from better estimates of proliferation.'
- Information is lost about individual genes when transforming data into PCA space, and lowly-expressed genes important to cell fate decisions may be dropped altogether in the scRNA-seq data
- Association of certain genes with specific cell fates does not necessarily imply causality

## Evidence

- [results] Proliferation estimates derived from gene expression (KEGG cell cycle/apoptosis gene sets) correlate weakly but significantly (r = 0.207, p << 1e-45): [evidence span withheld — anchor status: not_found]
- [methods] Load day-2 cells from data_pt and compute smoothed birth/death scores: "The `prescient process_data` command will generate a torch pt file `data_pt` (serialized dictionary)"
- [methods] For training, all that is needed is normalized gene expression, time-point labels, and cell type annotations.: "For training, all that is needed is normalized gene expression, time-point labels, and cell type annotations."
- [discussion] The model also improves when incorporating growth rates, and would likely benefit from better estimates of proliferation.: "The model also improves when incorporating growth rates, and would likely benefit from better estimates of proliferation."
- [readme] prescient-analysis notebooks operate on Weinreb et al. 2020 lineage tracing data: [evidence span withheld — anchor status: external_doc]
