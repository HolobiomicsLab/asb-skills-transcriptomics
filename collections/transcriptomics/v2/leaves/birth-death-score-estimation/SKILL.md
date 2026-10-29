---
name: birth-death-score-estimation
description: Use when you need proliferation-rate estimates for single-cell time-series
  modeling but lack direct lineage-traced descendant counts — e.g. supplying growth
  weights (data_pt['w']) to PRESCIENT from expression data only.
when_to_use_negative:
- Empirical lineage-traced descendant counts are already available (e.g. the weinreb-interpolate
  task uses ground truth proliferation rates directly rather than expression-derived
  estimates).
- The dataset's lowly-expressed genes critical to the cell-cycle or apoptosis gene
  sets were dropped during pre-processing — scoring on dropout-heavy expression will
  be unreliable.
- The task requires causal attribution of individual genes to fate outcomes rather
  than an aggregate proliferation score.
edam_operation: http://edamontology.org/operation_0334
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0602
tools:
- name: PRESCIENT
  role: Provides the data_pt torch object whose normalized expression, gene names,
    and growth-weight slot (data_pt['w']) the birth–death scores feed into
  repo: https://github.com/gifford-lab/prescient
- name: PyTorch
  role: Used to load the data_pt torch object (data, timepoints, genes)
- name: scikit-learn
  role: Builds the k-NN graph in PCA space for smoothing and fits the LogisticRegression
    mapping birth/death scores to fate probability (data_pt['pca'] is a sklearn PCA
    object)
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-021-23518-w
    title: Generative modeling of single-cell time series with PRESCIENT enables prediction
      of cell trajectories with interventions
  source_content_hash: sha256:6d3e7898fceff9ebbbda005e911814731e5f55fa667ce50fe4f92efc45f62e69
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gifford-lab/prescient
  tools:
  - PRESCIENT
  - PyTorch
  - scikit-learn
  edam_operation: http://edamontology.org/operation_0334
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# birth-death-score-estimation

## Summary

Estimates per-cell proliferation (birth–death) rates from normalized scRNA-seq expression alone by scoring KEGG cell-cycle and apoptosis gene sets, smoothing across a k-NN graph in PCA space, and fitting a logistic model to produce growth weights for PRESCIENT training.

## When to use

Use when you need proliferation-rate estimates for single-cell time-series modeling but lack direct lineage-traced descendant counts — e.g. supplying growth weights (data_pt['w']) to PRESCIENT from expression data only.

## When NOT to use

- Empirical lineage-traced descendant counts are already available (e.g. the weinreb-interpolate task uses ground truth proliferation rates directly rather than expression-derived estimates).
- The dataset's lowly-expressed genes critical to the cell-cycle or apoptosis gene sets were dropped during pre-processing — scoring on dropout-heavy expression will be unreliable.
- The task requires causal attribution of individual genes to fate outcomes rather than an aggregate proliferation score.

## Inputs

- PRESCIENT torch object data_pt (serialized .pt dictionary) containing normalized expression (data_pt['data']), timepoints (data_pt['tps']), gene names (data_pt['genes']), and PCA fit (data_pt['pca'])
- KEGG_CELL_CYCLE gene set (birth component)
- KEGG_APOPTOSIS gene set (death component)
- Empirical descendant counts per cell from Weinreb et al. lineage tracing annotations (for validation)

## Outputs

- Per-cell smoothed birth/death (proliferation) scores
- Pre-computed growth weights analogous to data_pt['w'] for PRESCIENT training
- Pearson correlation statistic between expression-derived scores and empirical clonal growth rates (expected r = 0.207, p << 1e-45)

## How to apply

Load the PRESCIENT torch object data_pt (created by `prescient process_data`) and extract normalized expression (data_pt['data']), timepoint labels (data_pt['tps']), and gene names (data_pt['genes']). Compute per-cell mean z-scores of the KEGG_CELL_CYCLE gene set (birth component) and KEGG_APOPTOSIS gene set (death component) over the normalized expression matrix, then smooth the combined score across each cell's nearest neighbors in PCA space using a k-NN graph built with scikit-learn to reduce single-cell dropout noise. Fit a scikit-learn LogisticRegression mapping the smoothed birth/death scores to a fate probability, yielding pre-computed growth weights analogous to data_pt['w']. Validate by correlating the fitted scores against empirical per-cell descendant counts from lineage tracing (Weinreb et al. day-2 cells) with scipy.stats.pearsonr; the reported agreement is weak but highly significant (r = 0.207, p << 1e-45, Fig. 2b).

## Related tools

- **PRESCIENT** (Provides the data_pt torch object whose normalized expression, gene names, and growth-weight slot (data_pt['w']) the birth–death scores feed into) — https://github.com/gifford-lab/prescient
- **PyTorch** (Used to load the data_pt torch object (data, timepoints, genes))
- **scikit-learn** (Builds the k-NN graph in PCA space for smoothing and fits the LogisticRegression mapping birth/death scores to fate probability (data_pt['pca'] is a sklearn PCA object))

## Examples

```
data_pt = torch.load('data_pt.pt'); X, genes = data_pt['data'], data_pt['genes']; birth = zscore(X[:, genes_in_kegg_cell_cycle].mean(1)); death = zscore(X[:, genes_in_kegg_apoptosis].mean(1)); r, p = pearsonr(birth - death, empirical_descendant_counts)
```

## Evaluation signals

- Pearson correlation between expression-derived birth/death scores and empirical day-2 Weinreb descendant counts reproduces r = 0.207 with p << 1e-45 (Fig. 2b).
- Fitted growth weights have the same structure as data_pt['w'] and can be substituted in PRESCIENT training without schema errors.
- Smoothed scores are consistent across each cell's k nearest neighbors in PCA space, indicating dropout noise was reduced.
- Model trained with these estimated proliferation rates should improve fate-bias prediction relative to a no-growth model, per the paper's finding that 'The model also improves when incorporating growth rates.'

## Limitations

- The correlation with empirical clonal growth rates is weak (r = 0.207); the paper notes 'The model also improves when incorporating growth rates, and would likely benefit from better estimates of proliferation.'
- Information about individual genes is lost in PCA space, and lowly-expressed genes important to cell fate decisions may be dropped altogether in scRNA-seq data, degrading gene-set scoring.
- Association of certain genes with specific cell fates does not necessarily imply causality.
- The downstream model requires the final time point of the dataset to be at steady-state.

## Evidence

- [results] kegg scoring validated against lineage tracing: [evidence span withheld — anchor status: not_found]
- [methods] torch object contents: "The `prescient process_data` command will generate a torch pt file `data_pt` (serialized dictionary) that contains all the necessary information for downstream training, simulations, and"
- [methods] pca object provenance: "data_pt["pca"]: sklearn.decomposition.PCA object fit to normalized expression and used to produce PCs."
- [discussion] growth rate sensitivity: "The model also improves when incorporating growth rates, and would likely benefit from better estimates of proliferation."
- [intro] weinreb validation dataset: "We validate PRESCIENT on a newly published lineage tracing dataset"
- [readme] ground-truth growth weights available for weinreb: [evidence span withheld — anchor status: external_doc]
