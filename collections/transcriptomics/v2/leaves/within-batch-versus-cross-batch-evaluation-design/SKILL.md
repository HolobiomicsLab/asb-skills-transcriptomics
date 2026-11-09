---
name: within-batch-versus-cross-batch-evaluation-design
description: Use when benchmarking perturbation-prediction methods on multi-batch
  scRNA-seq data where the held-out matrix may differ in batch from the input, and
  you must separate condition-only accuracy from condition+batch accuracy.
when_to_use_negative:
- Each batch is measured under only one condition, so batch and condition effects
  cannot be disentangled and the cross-batch scenario is not meaningful
- All input and held-out matrices come from the same batch, in which case only the
  within-batch scenario applies and a batch-aware model offers no advantage
- The goal is batch effect removal or key gene detection rather than perturbation
  prediction accuracy benchmarking
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scDisInFact
  role: Batch-aware predictor whose predict_counts supports both condition-only and
    condition+batch predictions, enabling the two-scenario evaluation
  repo: github.com/ZhangLabGT/scDisInFact
- name: scGen
  role: Batch-agnostic perturbation-prediction baseline evaluated under both scenarios
- name: scPreGAN
  role: Batch-agnostic perturbation-prediction baseline evaluated under both scenarios
  repo: github.com/XiajieWei/scPreGAN-reproducibility
- name: numpy
  role: Holds the (ncells, ngenes) count arrays fed to create_scdisinfact_dataset
- name: pandas
  role: Meta_cells dataframe carrying condition and batch columns and assembling the
    score table
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-024-45227-w
    title: 'scDisInFact: disentangled learning for integration and prediction of multi-batch
      multi-condition single-cell RNA-sequencing data'
  source_content_hash: sha256:30d744cc8cd1490a726cc4375acb078850d403016febef1fe471e7875c7086cd
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - numpy
  - pandas
  - scDisInFact
  - scGen
  - scPreGAN
  edam_operation: http://edamontology.org/operation_2428
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# within-batch-versus-cross-batch-evaluation-design

## Summary

An evaluation design for perturbation prediction that scores each method twice: once for a within-batch scenario (condition effect only) and once for a cross-batch scenario (condition plus batch effect), to expose methods that fail to account for batch effects between input and held-out matrices.

## When to use

Use when benchmarking perturbation-prediction methods on multi-batch scRNA-seq data where the held-out matrix may differ in batch from the input, and you must separate condition-only accuracy from condition+batch accuracy.

## When NOT to use

- Each batch is measured under only one condition, so batch and condition effects cannot be disentangled and the cross-batch scenario is not meaningful
- All input and held-out matrices come from the same batch, in which case only the within-batch scenario applies and a batch-aware model offers no advantage
- The goal is batch effect removal or key gene detection rather than perturbation prediction accuracy benchmarking

## Inputs

- scRNA-seq count matrices (numpy.array, ncells x ngenes) split into training matrices and one held-out matrix (e.g., GBM PW034-705)
- Cell metadata pandas.dataframe with condition-key and batch-key columns matching counts row-wise
- Cell-type labels for per-type scoring (e.g., Myeloid, Oligodendrocytes, tumor cells)

## Outputs

- Prediction score table (e.g., prediction_scores_gbm.csv) with per-cell-type MSE, Pearson, and R2 rows under both within-batch and cross-batch scenarios
- Predicted count matrices from model.predict_counts for each scenario

## How to apply

Hold out one batch-condition matrix (e.g., the PW034-705 GBM matrix) as the test set and train all methods on the remaining matrices so batch effects exist only in the test setting. Train a batch-aware method (scDisInFact) alongside batch-agnostic baselines (scGen, scPreGAN) on identical training matrices. Generate predictions twice per method with model.predict_counts: once supplying only the target condition (within-batch scenario) and once supplying both the target condition and target batch (cross-batch scenario). Score each scenario separately per cell type (e.g., Myeloid, Oligodendrocytes, tumor cells) using cell-type-specific MSE, Pearson correlation, and R2, and write results to a table such as prediction_scores_gbm.csv. The rationale is that scGen and scPreGAN 'do not account for batch effects between data matrices', so their performance degrades disproportionately in the cross-batch setting, which the paired scenario design makes visible.

## Related tools

- **scDisInFact** (Batch-aware predictor whose predict_counts supports both condition-only and condition+batch predictions, enabling the two-scenario evaluation) — github.com/ZhangLabGT/scDisInFact
- **scGen** (Batch-agnostic perturbation-prediction baseline evaluated under both scenarios)
- **scPreGAN** (Batch-agnostic perturbation-prediction baseline evaluated under both scenarios) — github.com/XiajieWei/scPreGAN-reproducibility
- **numpy** (Holds the (ncells, ngenes) count arrays fed to create_scdisinfact_dataset)
- **pandas** (Meta_cells dataframe carrying condition and batch columns and assembling the score table)

## Examples

```
counts_predict = model.predict_counts(input_counts=counts_input, meta_cells=meta_input, condition_keys=["condition 1", "condition 2"], batch_key="batch", predict_conds=["ctrl", "healthy"], predict_batch=0)
```

## Evaluation signals

- Score table contains rows for all cell types (e.g., Myeloid, Oligodendrocytes, tumor cells) under both the within-batch and cross-batch scenarios
- scDisInFact outperforms scGen and scPreGAN in the cross-batch setting, reproducing the ranking reported in Fig 4d
- The improvement of the batch-aware method over baselines is more pronounced in the cross-batch scenario than in the within-batch scenario
- Cross-batch scores for batch-agnostic baselines are worse than their within-batch scores, confirming batch effects were actually present between input and held-out matrices
- Training used the same training matrices for all methods (held-out matrix excluded) so scenario differences are attributable to batch handling, not data leakage

## Limitations

- When each batch of cells is measured under only one condition, there is not enough information to fully disentangle batch and condition effects, invalidating the cross-batch scenario
- scDisInFact assumes condition effect is independent of batch effect and batch effects are consistent across conditions, so cross-batch prediction degrades when this additivity fails
- Condition and batch effects are assumed additive in latent space, limiting modeling of higher-order perturbation or batch effects

## Evidence

- [intro] Cross-batch scenario exposes batch-agnostic baselines: "Existing methods for this task, such as scGen17 and scPreGAN18, do not account for batch effects between data matrices"
- [methods] Two-scenario prediction calls: "counts_predict = model.predict_counts(input_counts = counts_input, meta_cells = meta_input, condition_keys = ["condition 1", "condition 2"]"
- [methods] Dataset construction prerequisite: "data_dict = create_scdisinfact_dataset(counts, meta_cells, condition_key = ["condition 1", "condition 2"], batch_key = "batch")"
- [abstract] Disentanglement enables both scenarios: "scDisInFact learns latent factors that disentangle condition effect from batch effect, enabling it to simultaneously perform three tasks: batch effect removal, condition-associated key gene"
- [discussion] One-condition-per-batch limitation: "when each batch of cells is measured under only one condition, there is not enough information to fully disentangle the batch and condition effect."
