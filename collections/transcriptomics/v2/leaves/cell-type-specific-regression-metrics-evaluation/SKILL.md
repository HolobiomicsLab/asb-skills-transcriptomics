---
name: cell-type-specific-regression-metrics-evaluation
description: Use when predicted and true count matrices must be scored per cell type
  for multiple prediction scenarios (within-batch/cross-batch, in-sample/out-of-sample)
  and compared against baselines like scGen and scPreGAN.
when_to_use_negative:
- Input is a matrix of latent embeddings or cluster labels rather than gene count
  predictions — regression metrics on counts are not meaningful
- Only one global aggregate metric is required and cell-type stratification is not
  part of the evaluation design
- The dataset lacks cell-type annotations in meta_cells, making per-cell-type scoring
  impossible
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: scDisInFact
  role: Generates the perturbation-predicted count matrices (predict_counts) whose
    accuracy is being scored
  repo: https://github.com/ZhangLabGT/scDisInFact
- name: scGen
  role: Baseline perturbation-prediction method scored with the same cell-type-specific
    metrics
- name: scPreGAN
  role: Baseline perturbation-prediction method scored with the same cell-type-specific
    metrics
  repo: https://github.com/XiajieWei/scPreGAN-reproducibility
- name: numpy
  role: Holds count matrices as (ncells, ngenes) arrays and computes MSE/correlation/R2
    numerics
- name: pandas
  role: Holds meta_cells annotations used to stratify metrics by cell type and assembles
    the CSV benchmark table
provenance:
  source_task_ids:
  - task_008
  - task_005
  source_papers:
  - doi: 10.1038/s41467-024-45227-w
    title: 'scDisInFact: disentangled learning for integration and prediction of multi-batch
      multi-condition single-cell RNA-sequencing data'
  source_content_hash: sha256:6edc78c55a3bb40cd9e120b6c82165b8a2b218ad0ab86ef7b77e4b0fcfdf8239
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ZhangLabGT/scDisInFact
  tools:
  - PyTorch
  - numpy
  - pandas
  - scDisInFact
  - scGen
  - scPreGAN
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# cell-type-specific-regression-metrics-evaluation

## Summary

Benchmark perturbation-prediction accuracy on scRNA-seq count predictions by computing cell-type-specific MSE, Pearson correlation, and R2 between predicted and gold-standard counts, assembling results across scenarios and methods into a CSV comparison table (e.g., prediction_metrics.csv, perturbation_prediction_scores.csv).

## When to use

Use when predicted and true count matrices must be scored per cell type for multiple prediction scenarios (within-batch/cross-batch, in-sample/out-of-sample) and compared against baselines like scGen and scPreGAN.

## When NOT to use

- Input is a matrix of latent embeddings or cluster labels rather than gene count predictions — regression metrics on counts are not meaningful
- Only one global aggregate metric is required and cell-type stratification is not part of the evaluation design
- The dataset lacks cell-type annotations in meta_cells, making per-cell-type scoring impossible

## Inputs

- Predicted count matrices (numpy.array of shape (ncells, ngenes)) from scDisInFact predict_counts for each scenario
- Gold-standard (true) count matrices for the held-out or target condition cells
- Cell-type annotations from meta_cells (pandas.dataframe) to stratify metrics
- Baseline predicted counts from scGen and scPreGAN on the same splits

## Outputs

- Cell-type-specific MSE, Pearson correlation, and R2 values per method and scenario
- Consolidated benchmark table prediction_metrics.csv / perturbation_prediction_scores.csv

## How to apply

After generating predicted count matrices with scDisInFact's model.predict_counts (and the scGen/scPreGAN baselines) for each of the six within-batch and cross-batch scenarios toward a target condition/batch, split predicted-versus-true counts by cell type annotation and compute three regression metrics with numpy/pandas: mean squared error (MSE), Pearson correlation, and R2. Run the identical evaluation under both the in-sample setting and the out-of-sample setting where the target condition cells (e.g., held-out <ctrl,severe> or <moderate, 40-65>) were excluded from training. Assemble metric values for all scenarios x settings x methods into a single CSV table (prediction_metrics.csv / perturbation_prediction_scores.csv) so per-cell-type columns can be compared against the reported Fig. 3a,b and Fig. 6a,b comparisons. The rationale is that aggregate metrics can mask cell-type-level failures, so every metric must be stratified by cell type.

## Related tools

- **scDisInFact** (Generates the perturbation-predicted count matrices (predict_counts) whose accuracy is being scored) — https://github.com/ZhangLabGT/scDisInFact
- **scGen** (Baseline perturbation-prediction method scored with the same cell-type-specific metrics)
- **scPreGAN** (Baseline perturbation-prediction method scored with the same cell-type-specific metrics) — https://github.com/XiajieWei/scPreGAN-reproducibility
- **numpy** (Holds count matrices as (ncells, ngenes) arrays and computes MSE/correlation/R2 numerics)
- **pandas** (Holds meta_cells annotations used to stratify metrics by cell type and assembles the CSV benchmark table)

## Examples

```
counts_predict = model.predict_counts(input_counts=counts_input, meta_cells=meta_input, condition_keys=["condition 1", "condition 2"], batch_key="batch", predict_conds=["ctrl", "healthy"], predict_batch=0)
```

## Evaluation signals

- The scored CSV contains MSE, R2, and Pearson correlation per cell type for all 6 scenarios in both in-sample and out-of-sample settings for all three methods (scDisInFact, scGen, scPreGAN)
- On the simulated datasets, scDisInFact should show the smallest cell-type-specific MSE and highest Pearson correlation and R2 in all scenarios, reproducing Fig. 3a,b
- On the COVID-19 dataset, scDisInFact should outperform both baselines across all 6 prediction scenarios toward <moderate, 40-65> in batch Lee_2020, reproducing Fig. 6a,b
- Metric values are stratified by cell type rather than reported only as global aggregates

## Limitations

- Scores reflect scDisInFact's assumption that condition and batch effects are additive in latent space, which limits modeling of higher-order effects and may cap achievable accuracy
- If each batch is measured under only one condition, batch and condition effects cannot be fully disentangled, which can degrade prediction quality and thus the metric comparison
- Metrics assume a matching gold-standard count matrix for the target condition exists (out-of-sample) — fully unseen condition-batch combinations without ground truth cannot be scored

## Evidence

- [other] Compute cell-type-specific MSE, R2, and Pearson correlation between predicted and true counts for scDisInFact and both baselines, and write the benchmark table to perturbation_prediction_scores.csv: [evidence span withheld — anchor status: not_found]
- [methods] counts_predict = model.predict_counts(input_counts = counts_input, meta_cells = meta_input, condition_keys = ["condition 1", "condition 2"]: "counts_predict = model.predict_counts(input_counts = counts_input, meta_cells = meta_input, condition_keys = ["condition 1", "condition 2"]"
- [results] scDisInFact achieves the smallest cell-type-specific MSE and the highest Pearson correlation and R2 among scDisInFact, scGen, and scPreGAN in all in-sample (Fig. 3a) and out-of-sample (Fig. 3b) test scenarios: [evidence span withheld — anchor status: not_found]
- [methods] meta_cells (`pandas.dataframe` of the shape `(ncells, xx)`: "meta_cells` (`pandas.dataframe` of the shape `(ncells, xx)`"
