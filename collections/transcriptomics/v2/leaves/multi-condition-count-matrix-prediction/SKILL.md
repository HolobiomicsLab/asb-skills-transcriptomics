---
name: multi-condition-count-matrix-prediction
description: Use when a multi-batch, multi-condition scRNA-seq count matrix plus meta_cells
  must yield predicted counts for held-out or out-of-sample condition combinations,
  with per-cell-type accuracy metrics.
when_to_use_negative:
- Each batch is measured under only one condition, so there is not enough information
  to fully disentangle batch and condition effects
- The task is batch effect removal or condition-associated key gene (CKG) detection
  only — use extract_gene_scores or the integration task instead
- Condition types are strongly correlated in real data; combine them into a single
  combined condition type (e.g. stim1_stim2, ctrl1_ctrl2) before training rather than
  modeling them as independent
edam_operation: http://edamontology.org/operation_3800
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scDisInFact
  role: 'Primary model: conditional variational autoencoder that disentangles shared-bio
    and unshared-bio factors and generates predicted counts via predict_counts'
  repo: https://github.com/ZhangLabGT/scDisInFact
- name: scGen
  role: Baseline perturbation-prediction method trained on the same scenario splits
    for comparison
- name: scPreGAN
  role: Baseline perturbation-prediction method trained on the same scenario splits
    for comparison
  repo: https://github.com/XiajieWei/scPreGAN-reproducibility
- name: numpy
  role: Holds the (ncells, ngenes) count matrix and computes MSE/Pearson/R2 metrics
- name: pandas
  role: meta_cells DataFrame with condition and batch columns; subset input cells
    for prediction scenarios
- name: PyTorch
  role: Training backend; model instantiated on torch.device("cuda:0" if available)
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-024-45227-w
    title: 'scDisInFact: disentangled learning for integration and prediction of multi-batch
      multi-condition single-cell RNA-sequencing data'
  source_content_hash: sha256:a1aa4f68543a6eea6d307da449de489a768693b401861f0ce2b22b7044f1da40
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
  edam_operation: http://edamontology.org/operation_3800
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 15
---

# multi-condition-count-matrix-prediction

## Summary

Predicts cell-type-specific perturbed scRNA-seq count matrices for unseen condition-batch combinations (e.g. <ctrl, healthy, batch 0> from <stim, severe, batch 1> input) using scDisInFact's disentangled shared-bio and unshared-bio latent factors, and benchmarks predictions against scGen and scPreGAN using MSE, Pearson correlation, and R2.

## When to use

Use when a multi-batch, multi-condition scRNA-seq count matrix plus meta_cells must yield predicted counts for held-out or out-of-sample condition combinations, with per-cell-type accuracy metrics.

## When NOT to use

- Each batch is measured under only one condition, so there is not enough information to fully disentangle batch and condition effects
- The task is batch effect removal or condition-associated key gene (CKG) detection only — use extract_gene_scores or the integration task instead
- Condition types are strongly correlated in real data; combine them into a single combined condition type (e.g. stim1_stim2, ctrl1_ctrl2) before training rather than modeling them as independent

## Inputs

- counts: numpy.array of shape (ncells, ngenes) raw scRNA-seq count matrix
- meta_cells: pandas.DataFrame of shape (ncells, xx) with columns for condition 1, condition 2, and batch IDs matching counts rows
- Held-out gold-standard count matrices for target condition-batch combinations

## Outputs

- counts_predict: predicted count matrices for each target condition combination and batch
- prediction_metrics.csv: cell-type-specific MSE, Pearson correlation, and R2 for scDisInFact, scGen, and scPreGAN across all six in-sample and out-of-sample scenarios

## How to apply

Load the count matrix (numpy.array, shape (ncells, ngenes)) and meta_cells (pandas.DataFrame) and build the dataset with create_scdisinfact_dataset(condition_key=["condition 1", "condition 2"], batch_key="batch"). Instantiate scdisinfact with latent dimensions Ks=[8, 2, 2] (shared-bio factor plus one unshared-bio factor per condition type) on torch.device("cuda:0" if available) and train with train_model(nepochs=100), then call model.eval(). For each scenario (within-batch and cross-batch for each condition type and both jointly), call model.predict_counts with condition_keys, batch_key, target predict_conds, and predict_batch; use held-out subsets such as <ctrl, severe> cells as the out-of-sample test. Train scGen and scPreGAN on the same splits to produce baseline predicted count matrices, then compute cell-type-specific MSE, Pearson correlation, and R2 against gold-standard counts with numpy/pandas and assemble results into prediction_metrics.csv (reproducing Fig. 3a,b).

## Related tools

- **scDisInFact** (Primary model: conditional variational autoencoder that disentangles shared-bio and unshared-bio factors and generates predicted counts via predict_counts) — https://github.com/ZhangLabGT/scDisInFact
- **scGen** (Baseline perturbation-prediction method trained on the same scenario splits for comparison)
- **scPreGAN** (Baseline perturbation-prediction method trained on the same scenario splits for comparison) — https://github.com/XiajieWei/scPreGAN-reproducibility
- **numpy** (Holds the (ncells, ngenes) count matrix and computes MSE/Pearson/R2 metrics)
- **pandas** (meta_cells DataFrame with condition and batch columns; subset input cells for prediction scenarios)
- **PyTorch** (Training backend; model instantiated on torch.device("cuda:0" if available))

## Examples

```
data_dict = create_scdisinfact_dataset(counts, meta_cells, condition_key=["condition 1", "condition 2"], batch_key="batch"); model = scdisinfact(data_dict=data_dict, Ks=[8,2,2], device=torch.device("cuda:0")); model.train_model(nepochs=100); counts_predict = model.predict_counts(input_counts=counts_input, meta_cells=meta_input, condition_keys=["condition 1", "condition 2"], batch_key="batch", predict_conds=["ctrl","healthy"], predict_batch=0)
```

## Evaluation signals

- scDisInFact should achieve the smallest cell-type-specific MSE and highest Pearson correlation and R2 vs scGen and scPreGAN in all in-sample (Fig. 3a) and out-of-sample (Fig. 3b) scenarios
- predicted counts shape must match the target condition-batch cell subset and gene dimension of the input counts
- training loss from train_model(nepochs=100) should converge before model.eval() and prediction
- out-of-sample predictions must use held-out cell subsets (e.g. <ctrl, severe>) never seen during training
- metrics for all 6 scenarios (within-batch/cross-batch × treatment/severity/both) × 3 methods should populate prediction_metrics.csv without missing values

## Limitations

- scDisInFact assumes condition types are mutually independent, which does not always hold for real-world datasets
- Assumes condition effect is independent of batch effect and batch effects are consistent across conditions
- Assumes condition and batch effects are additive to shared-bio factors in latent space, limiting modeling of higher-order effects
- Insufficient information to disentangle batch and condition effects when each batch is measured under only one condition
- Testing datasets from the manuscript are not included in the repo due to size and are available upon request; only demo_data is provided

## Evidence

- [methods] counts_predict = model.predict_counts(input_counts = counts_input, meta_cells = meta_input, condition_keys = ["condition 1", "condition 2"]: "counts_predict = model.predict_counts(input_counts = counts_input, meta_cells = meta_input, condition_keys = ["condition 1", "condition 2"]"
- [methods] losses = model.train_model(nepochs = 100): "losses = model.train_model(nepochs = 100)"
- [results] On the simulated datasets, scDisInFact achieves the smallest cell-type-specific MSE and the highest Pearson correlation and R2 among scDisInFact, scGen, and scPreGAN: [evidence span withheld — anchor status: not_found]
- [discussion] when each batch of cells is measured under only one condition, there is not enough information to fully disentangle the batch and condition effect: "when each batch of cells is measured under only one condition, there is not enough information to fully disentangle the batch and condition effect."
- [readme] Given the input count matrix `counts` (`numpy.array` of the shape `(ncells, ngenes)`) and meta data `meta_cells` (`pandas.dataframe`: "Given the input count matrix `counts` (`numpy.array` of the shape `(ncells, ngenes)`) and meta data `meta_cells` (`pandas.dataframe` of the shape `(ncells, xx)`)"
