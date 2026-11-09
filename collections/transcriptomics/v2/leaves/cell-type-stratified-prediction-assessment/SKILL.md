---
name: cell-type-stratified-prediction-assessment
description: Use when predicting held-out condition/batch scRNA-seq matrices (e.g.,
  PW034-705) and you must judge prediction accuracy separately per cell type under
  both condition-only and condition-plus-batch scenarios.
when_to_use_negative:
- Input is a single-batch dataset where each batch is measured under only one condition
  — the paper states there is not enough information to fully disentangle batch and
  condition effects, so cross-batch scoring is ill-posed
- The goal is perturbation prediction on data with no batch variation between input
  and held-out matrices; scGen/scPreGAN already handle the condition-only task and
  scDisInFact's main advantage disappears
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scDisInFact
  role: 'Primary predictor: trained on the training matrices with batch-effect modeling
    and queried with predict_counts for within-batch and cross-batch scenarios'
  repo: https://github.com/ZhangLabGT/scDisInFact
- name: scGen
  role: Baseline perturbation-prediction method trained on the same matrices without
    batch-effect modeling
- name: scPreGAN
  role: Baseline perturbation-prediction method trained on the same matrices without
    batch-effect modeling
  repo: https://github.com/XiajieWei/scPreGAN-reproducibility
- name: numpy
  role: Holds the count matrices in (ncells, ngenes) array form and supports the MSE/Pearson/R2
    computations
- name: pandas
  role: Holds meta_cells with condition and batch columns and writes the score table
    to prediction_scores_gbm.csv
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-024-45227-w
    title: 'scDisInFact: disentangled learning for integration and prediction of multi-batch
      multi-condition single-cell RNA-sequencing data'
  source_content_hash: sha256:d96becdab62ed910c43e4b966686c38e7fb2a7395df8ecfef66f70ebdce8577e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ZhangLabGT/scDisInFact
  tools:
  - numpy
  - pandas
  - scDisInFact
  - scGen
  - scPreGAN
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# Cell-type-stratified perturbation prediction assessment

## Summary

Scores perturbation-prediction outputs per cell type (e.g., Myeloid, Oligodendrocytes, tumor cells) using cell-type-specific MSE, Pearson correlation, and R2, comparing scDisInFact against scGen and scPreGAN under within-batch and cross-batch scenarios on GBM scRNA-seq data.

## When to use

Use when predicting held-out condition/batch scRNA-seq matrices (e.g., PW034-705) and you must judge prediction accuracy separately per cell type under both condition-only and condition-plus-batch scenarios.

## When NOT to use

- Input is a single-batch dataset where each batch is measured under only one condition — the paper states there is not enough information to fully disentangle batch and condition effects, so cross-batch scoring is ill-posed
- The goal is perturbation prediction on data with no batch variation between input and held-out matrices; scGen/scPreGAN already handle the condition-only task and scDisInFact's main advantage disappears

## Inputs

- GBM scRNA-seq count matrices (numpy.array, shape (ncells, ngenes)) with PW034-705 held out
- Cell metadata dataframe (pandas.dataframe, shape (ncells, xx)) with condition and batch key columns
- Predicted count matrices from scDisInFact, scGen, and scPreGAN for the held-out condition/batch combination

## Outputs

- prediction_scores_gbm.csv with cell-type-specific MSE, Pearson, and R2 rows for Myeloid, Oligodendrocytes, and tumor cells under within-batch and cross-batch scenarios
- Relative ranking of scDisInFact vs scGen and scPreGAN per scenario, comparable to Fig 4d

## How to apply

Split the GBM count matrices so the PW034-705 matrix is held out as the test set and the remaining matrices form the training set, then build the training dataset with create_scdisinfact_dataset (counts as ncells x ngenes array, metadata with condition and batch key columns). Train scDisInFact with model.train_model(nepochs = 100) on GPU, and train scGen and scPreGAN baselines on the same matrices without batch-effect modeling. Generate predictions with model.predict_counts twice: once for the within-batch scenario (condition effect only) and once for the cross-batch scenario (condition plus batch effect). For each method, compute cell-type-specific MSE, Pearson, and R2 separately for Myeloid, Oligodendrocytes, and tumor cells, and write the table to prediction_scores_gbm.csv; scDisInFact's advantage is expected to be more pronounced in the cross-batch setting.

## Related tools

- **scDisInFact** (Primary predictor: trained on the training matrices with batch-effect modeling and queried with predict_counts for within-batch and cross-batch scenarios) — https://github.com/ZhangLabGT/scDisInFact
- **scGen** (Baseline perturbation-prediction method trained on the same matrices without batch-effect modeling)
- **scPreGAN** (Baseline perturbation-prediction method trained on the same matrices without batch-effect modeling) — https://github.com/XiajieWei/scPreGAN-reproducibility
- **numpy** (Holds the count matrices in (ncells, ngenes) array form and supports the MSE/Pearson/R2 computations)
- **pandas** (Holds meta_cells with condition and batch columns and writes the score table to prediction_scores_gbm.csv)

## Examples

```
data_dict = create_scdisinfact_dataset(counts, meta_cells, condition_key=["condition 1", "condition 2"], batch_key="batch"); model.train_model(nepochs = 100); counts_predict = model.predict_counts(input_counts = counts_input, meta_cells = meta_input, condition_keys = ["condition 1", "condition 2"], batch_key = "batch", predict_conds = ["ctrl", "healthy"], predict_batch = 0)
```

## Evaluation signals

- prediction_scores_gbm.csv contains rows for all three cell types (Myeloid, Oligodendrocytes, tumor cells) under both within-batch and cross-batch scenarios
- scDisInFact's scores reproduce the relative ranking of Fig 4d: it outperforms scGen and scPreGAN in the cross-batch setting
- The gap between scDisInFact and the baselines is larger in the cross-batch scenario than in the within-batch scenario
- Predictions were generated twice via predict_counts: once with condition effect only and once with condition plus batch effect
- Training and baselines consumed the identical training matrices (all matrices except held-out PW034-705)

## Limitations

- When each batch is measured under only one condition, there is not enough information to fully disentangle batch and condition effects, undermining cross-batch assessment
- scDisInFact assumes condition types are mutually independent, condition effect is independent of batch effect, and batch effects are consistent across conditions — violations bias the comparison
- Condition and batch effects are assumed additive in latent space, limiting modeling of higher-order perturbation or batch effects

## Evidence

- [intro] Existing methods for this task, such as scGen17 and scPreGAN18, do not account for batch effects between data matrices: "Existing methods for this task, such as scGen17 and scPreGAN18, do not account for batch effects between data matrices"
- [methods] Predictions of held-out condition/batch via predict_counts: "counts_predict = model.predict_counts(input_counts = counts_input, meta_cells = meta_input, condition_keys = ["condition 1", "condition 2"]"
- [methods] Model training with 100 epochs: "losses = model.train_model(nepochs = 100)"
- [methods] Dataset creation from counts and metadata: "data_dict = create_scdisinfact_dataset(counts, meta_cells, condition_key = ["condition 1", "condition 2"], batch_key = "batch")"
- [readme] README: counts format and metadata alignment requirement: "meta_cells must include columns corresponding to the batch ID and condition of each cells, and the cells in meta_cells should match counts"
