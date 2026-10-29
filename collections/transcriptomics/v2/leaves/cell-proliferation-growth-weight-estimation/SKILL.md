---
name: cell-proliferation-growth-weight-estimation
description: Use when training PRESCIENT on longitudinal scRNA-seq (e.g., Weinreb
  hematopoiesis days 2/4/6) and fate-bias or interpolation predictions must account
  for cell proliferation via growth weights.
when_to_use_negative:
- The dataset lacks any proliferation information and no lineage-tracing ground truth
  or cell-cycle-based estimates can be computed (the model then relies on unweighted
  population dynamics)
- The task does not require fate-bias or temporal population prediction, e.g., purely
  exploratory clustering of a single time point
- The final time point of the dataset is not at steady-state, violating the model's
  assumption
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: PRESCIENT
  role: Consumes the growth weights during potential training so simulated trajectories
    account for cell proliferation
  repo: https://github.com/gifford-lab/prescient
- name: prescient
  role: '`prescient process_data` writes the pre-computed growth weights into the
    data_pt torch file used downstream'
  repo: https://github.com/gifford-lab/prescient
- name: gifford-lab/prescient-analysis
  role: Provides weinreb-interpolate.sh (ground truth proliferation rates) and weinreb-fate.sh
    (estimated proliferation rates) example training scripts
  repo: https://github.com/gifford-lab/prescient-analysis
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-021-23518-w
    title: Generative modeling of single-cell time series with PRESCIENT enables prediction
      of cell trajectories with interventions
  source_content_hash: sha256:53bd5e042251eef57520512ae9540b40e0f74405afd81443898543eacc5001ae
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gifford-lab/prescient
  tools:
  - PRESCIENT
  - PyTorch
  - gifford-lab/prescient-analysis
  - prescient
  - scanpy
  - scikit-learn
  - wot
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# cell-proliferation-growth-weight-estimation

## Summary

Compute per-cell proliferation/growth weights for PRESCIENT training on single-cell time-series data (e.g., the Weinreb et al. mouse hematopoiesis lineage tracing dataset), so the learned differentiation potential accounts for asymmetric cell proliferation.

## When to use

Use when training PRESCIENT on longitudinal scRNA-seq (e.g., Weinreb hematopoiesis days 2/4/6) and fate-bias or interpolation predictions must account for cell proliferation via growth weights.

## When NOT to use

- The dataset lacks any proliferation information and no lineage-tracing ground truth or cell-cycle-based estimates can be computed (the model then relies on unweighted population dynamics)
- The task does not require fate-bias or temporal population prediction, e.g., purely exploratory clustering of a single time point
- The final time point of the dataset is not at steady-state, violating the model's assumption

## Inputs

- Normalized gene expression matrix with time-point labels and cell type annotations (.csv, .tsv, .txt, or .h5ad)
- Per-cell growth/proliferation rates — either ground-truth clonal rates from the Weinreb lineage tracing dataset or estimated proliferation rates

## Outputs

- data_pt torch object (serialized dictionary) containing normalized expression, timepoint labels, cell types, gene features, PCA/UMAP embeddings, and pre-computed growth weights

## How to apply

Start from pre-processed normalized expression with time-point labels and cell type annotations (Seurat or scanpy output in .csv/.tsv/.txt/.h5ad). Obtain proliferation rates either as ground-truth clonal growth rates from lineage-traced data (used in weinreb-interpolate.sh for the day-4 interpolation task) or as estimated proliferation rates (used in weinreb-fate.sh and veres-fate.sh). Attach the growth weights to each cell so the `prescient process_data` command can embed them in the serialized data_pt torch object alongside normalized expression, timepoint labels, cell types, PCA/UMAP embeddings. During potential training the growth weights bias the model toward proliferating progenitor states; the paper reports fate prediction improves when accounting for proliferation. Decision point: use ground-truth growth when available (interpolation task) and estimated growth otherwise (fate task), since better proliferation estimates are expected to further improve the model.

## Related tools

- **PRESCIENT** (Consumes the growth weights during potential training so simulated trajectories account for cell proliferation) — https://github.com/gifford-lab/prescient
- **prescient** (`prescient process_data` writes the pre-computed growth weights into the data_pt torch file used downstream) — https://github.com/gifford-lab/prescient
- **gifford-lab/prescient-analysis** (Provides weinreb-interpolate.sh (ground truth proliferation rates) and weinreb-fate.sh (estimated proliferation rates) example training scripts) — https://github.com/gifford-lab/prescient-analysis

## Examples

```
bash /scripts/weinreb-interpolate.sh
```

## Evaluation signals

- data_pt torch object loads and contains pre-computed growth weights aligned to cells/timepoint labels
- On the Weinreb hematopoiesis task, fate biases of progenitor cells are predicted (proliferation-aware model outperformed the best-performing existing method)
- For the day-4 held-out interpolation task, the simulated day-4 population's Wasserstein distance to true day-4 cells improves versus WOT interpolation baseline
- Training distances for each of the 5 seeds against the day-6 steady-state target are recorded in wasserstein_distances.csv
- Ground-truth (lineage-traced) growth rates are used for the interpolation task; estimated rates only for the fate task, per the prescient-analysis scripts

## Limitations

- Model performance depends on growth rates and would benefit from better estimates of proliferation
- The model requires the final time point of the dataset to be at steady-state
- Information is lost about individual genes when transforming data into PCA space, and lowly-expressed genes important to cell fate decisions may be dropped altogether

## Evidence

- [discussion] The model also improves when incorporating growth rates, and would likely benefit from better estimates of proliferation.: "The model also improves when incorporating growth rates, and would likely benefit from better estimates of proliferation."
- [abstract] PRESCIENT is able to predict the fate biases of progenitor cells in hematopoiesis when accounting for cell proliferation, improving upon the best-performing existing method.: "PRESCIENT is able to predict the fate biases of progenitor cells in hematopoiesis when accounting for cell proliferation, improving upon the best-performing existing method."
- [readme] weinreb-interpolate.sh trains 5 seeds of 2 layer 400 unit models on only lineage tracing data with ground truth proliferation rates for the interpolation task: [evidence span withheld — anchor status: external_doc]
- [readme] weinreb-fate.sh trains 5 seeds of 2 layer 400 unit models on all data with estimated proliferation rates for clonal fate bias prediction: [evidence span withheld — anchor status: external_doc]
- [methods] The `prescient process_data` command will generate a torch pt file `data_pt` (serialized dictionary): "The `prescient process_data` command will generate a torch pt file `data_pt` (serialized dictionary)"
