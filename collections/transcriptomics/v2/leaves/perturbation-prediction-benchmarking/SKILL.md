---
name: perturbation-prediction-benchmarking
description: Use when evaluating perturbation-prediction accuracy of multi-batch multi-condition
  scRNA-seq methods on held-out conditions or batches, using cell-type-specific MSE,
  Pearson correlation, and R2 metrics.
when_to_use_negative:
- Each batch is measured under only one condition, so batch and condition effects
  cannot be disentangled and the comparison would be uninformative
- The goal is batch effect removal or condition-associated key gene detection rather
  than predicting unseen condition/batch count matrices
- Input is a single-condition, single-batch dataset with no held-out combination to
  predict against a gold standard
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2269
tools:
- name: scDisInFact
  role: 'Primary method: trained on multi-batch multi-condition counts and used to
    predict held-out condition/batch counts via predict_counts'
  repo: https://github.com/ZhangLabGT/scDisInFact
- name: scGen
  role: Baseline perturbation-prediction method trained on the same splits without
    batch-effect modeling
- name: scPreGAN
  role: Baseline perturbation-prediction method trained on the same splits without
    batch-effect modeling
  repo: https://github.com/XiajieWei/scPreGAN-reproducibility
- name: numpy
  role: Holds count matrices (ncells, ngenes) and computes cell-type-specific MSE,
    Pearson, R2 metrics
- name: pandas
  role: Metadata handling (meta_cells) and writing the metric score table to CSV
- name: PyTorch
  role: Backend for scDisInFact training on torch.device('cuda:0' if available)
provenance:
  source_task_ids:
  - task_005
  - task_007
  source_papers:
  - doi: 10.1038/s41467-024-45227-w
    title: 'scDisInFact: disentangled learning for integration and prediction of multi-batch
      multi-condition single-cell RNA-sequencing data'
  source_content_hash: sha256:32fac318f25d93bcecc8c04113ef9ca51ec564345f8a71146fce580a65d2ed70
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
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# perturbation-prediction-benchmarking

## Summary

Benchmark scDisInFact's perturbation prediction against scGen and scPreGAN by generating predicted count matrices for held-out condition/batch combinations and computing cell-type-specific MSE, Pearson correlation, and R2 against gold-standard counts, across within-batch and cross-batch scenarios.

## When to use

Use when evaluating perturbation-prediction accuracy of multi-batch multi-condition scRNA-seq methods on held-out conditions or batches, using cell-type-specific MSE, Pearson correlation, and R2 metrics.

## When NOT to use

- Each batch is measured under only one condition, so batch and condition effects cannot be disentangled and the comparison would be uninformative
- The goal is batch effect removal or condition-associated key gene detection rather than predicting unseen condition/batch count matrices
- Input is a single-condition, single-batch dataset with no held-out combination to predict against a gold standard

## Inputs

- Count matrix (numpy.array, ncells x ngenes) with cells from multiple conditions and batches
- Cell metadata (pandas.dataframe, ncells x xx) with condition and batch columns (e.g., 'condition 1', 'condition 2', 'batch')
- Gold-standard count matrices for the held-out condition/batch combinations
- Held-out test split (e.g., PW034-705 GBM matrix or <ctrl,severe> cell subset) and scenario definitions (within-batch vs cross-batch)

## Outputs

- Predicted count matrices per method per scenario (from model.predict_counts and baselines)
- Cell-type-specific accuracy table (MSE, Pearson correlation, R2) per method per scenario, e.g. prediction_metrics.csv or prediction_scores_gbm.csv

## How to apply

Construct the scdisinfact dataset from the count matrix and metadata with create_scdisinfact_dataset (condition_key = ["condition 1", "condition 2"], batch_key = "batch"), instantiate scdisinfact with latent dimensions Ks = [8, 2, 2] (shared-bio factor plus one unshared-bio factor per condition type), and train with train_model(nepochs = 100) on a CUDA device. For each scenario (within-batch and cross-batch for each condition type or both), call model.predict_counts with input counts and meta_cells filtered to the source condition/batch, specifying predict_conds and predict_batch; use a held-out cell subset (e.g., the <ctrl,severe> cells) for the out-of-sample setting. Train scGen and scPreGAN on the identical training splits without batch-effect modeling and generate predictions for the same scenarios. Compute cell-type-specific MSE, Pearson correlation, and R2 between predicted and gold-standard counts per cell type (e.g., Myeloid, Oligodendrocytes, tumor cells in the GBM data), assemble the results into a metrics CSV, and check that scDisInFact attains the smallest MSE and highest Pearson/R2 in both in-sample and out-of-sample scenarios.

## Related tools

- **scDisInFact** (Primary method: trained on multi-batch multi-condition counts and used to predict held-out condition/batch counts via predict_counts) — https://github.com/ZhangLabGT/scDisInFact
- **scGen** (Baseline perturbation-prediction method trained on the same splits without batch-effect modeling)
- **scPreGAN** (Baseline perturbation-prediction method trained on the same splits without batch-effect modeling) — https://github.com/XiajieWei/scPreGAN-reproducibility
- **numpy** (Holds count matrices (ncells, ngenes) and computes cell-type-specific MSE, Pearson, R2 metrics)
- **pandas** (Metadata handling (meta_cells) and writing the metric score table to CSV)
- **PyTorch** (Backend for scDisInFact training on torch.device('cuda:0' if available))

## Examples

```
from scDisInFact import scdisinfact, create_scdisinfact_dataset; data_dict = create_scdisinfact_dataset(counts, meta_cells, condition_key=["condition 1", "condition 2"], batch_key="batch"); model = scdisinfact(data_dict=data_dict, Ks=[8,2,2], device=torch.device("cuda:0")); losses = model.train_model(nepochs=100)
```

## Evaluation signals

- The metrics CSV contains rows for all cell types (e.g., Myeloid, Oligodendrocytes, tumor cells) under both within-batch and cross-batch scenarios
- scDisInFact achieves the smallest cell-type-specific MSE and highest Pearson correlation and R2 versus scGen and scPreGAN in all in-sample (Fig. 3a) and out-of-sample (Fig. 3b) scenarios
- For the GBM data, scDisInFact's scores reproduce the Fig. 4d ranking, with the improvement over baselines more pronounced when batch effect exists between input and held-out matrices
- The out-of-sample setting uses the held-out <ctrl,severe> cell subset (or held-out PW034-705 matrix) so predictions are never scored on training cells
- Training used the declared configuration: Ks = [8, 2, 2] and train_model(nepochs = 100)

## Limitations

- scDisInFact assumes condition types are mutually independent, which does not always hold for real-world datasets; highly correlated condition types should be combined into one condition type before training
- Condition effect is assumed independent of batch effect and additive in latent space, limiting modeling of higher-order conditions or batch effects
- When each batch is measured under only one condition, there is not enough information to fully disentangle batch and condition effects
- Testing datasets from the manuscript are not included in the repository and are only available upon request; only demo_data is provided

## Evidence

- [results] On the simulated datasets, scDisInFact achieves the smallest cell-type-specific MSE and the highest Pearson correlation and R2: [evidence span withheld — anchor status: not_found]
- [methods] create_scdisinfact_dataset invocation: "data_dict = create_scdisinfact_dataset(counts, meta_cells, condition_key = ["condition 1", "condition 2"], batch_key = "batch")"
- [methods] training call: "losses = model.train_model(nepochs = 100)"
- [methods] predict_counts usage: "counts_predict = model.predict_counts(input_counts = counts_input, meta_cells = meta_input, condition_keys = ["condition 1", "condition 2"], batch_key = "batch", predict_conds = ["ctrl", "healthy"],"
- [intro] baseline gap: "Existing methods for this task, such as scGen17 and scPreGAN18, do not account for batch effects between data matrices"
- [readme] data availability: "The testing datasets in the manuscript are available upon request."
