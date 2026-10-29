---
name: cross-batch-prediction-scenario-benchmarking
description: Use when evaluating whether a model can predict single-cell expression
  toward an unseen condition-batch combination (e.g., <moderate, 40-65> in batch Lee_2020)
  and comparing it against baselines across multiple prediction scenarios.
when_to_use_negative:
- Each batch in the dataset was measured under only one condition, so there is insufficient
  information to fully disentangle batch and condition effects
- The condition types are strongly correlated and cannot be assumed mutually independent
  (consider a combined condition type instead)
- The goal is batch effect removal or key gene detection only, without perturbation
  prediction toward a held-out condition
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scDisInFact
  role: 'Primary model: builds the multi-batch multi-condition dataset, trains the
    conditional VAE, and generates predicted counts for each condition-batch prediction
    scenario'
  repo: github.com/ZhangLabGT/scDisInFact
- name: scGen
  role: Baseline perturbation prediction method compared across the same 6 scenarios
    and both settings
- name: scPreGAN
  role: Baseline perturbation prediction method compared across the same 6 scenarios
    and both settings
  repo: github.com/XiajieWei/scPreGAN-reproducibility
- name: numpy
  role: Provides the count matrix format (numpy.array of shape (ncells, ngenes)) used
    as model input
- name: pandas
  role: Provides the meta_cells dataframe with batch and condition columns consumed
    by create_scdisinfact_dataset and predict_counts
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-024-45227-w
    title: 'scDisInFact: disentangled learning for integration and prediction of multi-batch
      multi-condition single-cell RNA-sequencing data'
  source_content_hash: sha256:fd33a7f4b1cd3ab8b5c55e305618523682f039c5419b2b0bb3f2848430c9eeef
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
yes_rate: 0.7272727272727273
n_claims: 11
---

# cross-batch-prediction-scenario-benchmarking

## Summary

Benchmarks perturbation prediction toward a held-out condition-batch combination across within-batch and cross-batch scenarios, comparing a multi-condition model (scDisInFact) against baselines (scGen, scPreGAN) using cell-type-specific MSE, R2, and Pearson correlation under both in-sample and out-of-sample settings.

## When to use

Use when evaluating whether a model can predict single-cell expression toward an unseen condition-batch combination (e.g., <moderate, 40-65> in batch Lee_2020) and comparing it against baselines across multiple prediction scenarios.

## When NOT to use

- Each batch in the dataset was measured under only one condition, so there is insufficient information to fully disentangle batch and condition effects
- The condition types are strongly correlated and cannot be assumed mutually independent (consider a combined condition type instead)
- The goal is batch effect removal or key gene detection only, without perturbation prediction toward a held-out condition

## Inputs

- Processed HDF5 scRNA-seq count matrices (numpy.array of shape (ncells, ngenes)) from multiple batches
- Cell metadata (pandas dataframe of shape (ncells, xx)) with columns for batch ID and condition types (age, disease severity)
- Held-out target condition-batch combination (e.g., condition <moderate, 40-65>, batch Lee_2020)

## Outputs

- Benchmark table (e.g., perturbation_prediction_scores.csv) with MSE, R2, and Pearson correlation per cell type for all 6 scenarios in both in-sample and out-of-sample settings for all methods
- Predicted count matrices for each prediction scenario from model.predict_counts

## How to apply

Construct the scdisinfact dataset from processed HDF5 count matrices and meta_cells using create_scdisinfact_dataset with age and disease severity as the two condition keys and study ID as the batch key. Train the scdisinfact model on GPU with latent dimensions Ks set for the shared-bio factor plus one unshared-bio factor per condition type (e.g., Ks = [8, 2, 2]) via model.train_model(nepochs = 100). Run the 6 within-batch and cross-batch prediction scenarios toward the target condition/batch using model.predict_counts with predict_conds and predict_batch, first in the in-sample setting and then out-of-sample by holding out cells with the target condition from training. Run scGen and scPreGAN baselines on the same scenarios and settings, then compute cell-type-specific MSE, R2, and Pearson correlation between predicted and true counts and write the benchmark table (e.g., perturbation_prediction_scores.csv). The rationale is that scDisInFact models batch and condition effects jointly, so improvements should be most pronounced for cross-batch prediction scenarios.

## Related tools

- **scDisInFact** (Primary model: builds the multi-batch multi-condition dataset, trains the conditional VAE, and generates predicted counts for each condition-batch prediction scenario) — github.com/ZhangLabGT/scDisInFact
- **scGen** (Baseline perturbation prediction method compared across the same 6 scenarios and both settings)
- **scPreGAN** (Baseline perturbation prediction method compared across the same 6 scenarios and both settings) — github.com/XiajieWei/scPreGAN-reproducibility
- **numpy** (Provides the count matrix format (numpy.array of shape (ncells, ngenes)) used as model input)
- **pandas** (Provides the meta_cells dataframe with batch and condition columns consumed by create_scdisinfact_dataset and predict_counts)

## Examples

```
counts_predict = model.predict_counts(input_counts=counts_input, meta_cells=meta_input, condition_keys=["condition 1", "condition 2"], batch_key="batch", predict_conds=["ctrl", "healthy"], predict_batch=0)
```

## Evaluation signals

- The scored table must contain MSE, R2, and Pearson correlation per cell type for all 6 scenarios in both in-sample and out-of-sample settings for all three methods
- The benchmark should reproduce the comparison reported in Fig 6a,b of the article
- Improvement of scDisInFact over scGen and scPreGAN should be more pronounced for tasks involving prediction of batch effects
- Training losses from model.train_model should decrease over the 100 epochs, indicating convergence on the held-out-excluded data
- Predicted counts must match the target condition and batch (predict_conds, predict_batch) specified in each scenario

## Limitations

- Insufficient information to disentangle batch and condition effects when each batch is measured under only one condition
- Assumes condition types are mutually independent, which does not always hold for real-world datasets
- Assumes condition effect is independent of batch effect and batch effects are consistent across conditions
- Condition and batch effects are assumed additive in latent space, limiting modeling of higher-order conditions or batch effects

## Evidence

- [abstract] scDisInFact learns latent factors that disentangle condition effect from batch effect, enabling it to simultaneously perform three tasks: "scDisInFact learns latent factors that disentangle condition effect from batch effect, enabling it to simultaneously perform three tasks"
- [intro] Existing methods do not account for batch effects: "Existing methods for this task, such as scGen17 and scPreGAN18, do not account for batch effects between data matrices"
- [methods] create_scdisinfact_dataset call: "data_dict = create_scdisinfact_dataset(counts, meta_cells, condition_key = ["condition 1", "condition 2"], batch_key = "batch")"
- [methods] predict_counts call: "counts_predict = model.predict_counts(input_counts = counts_input, meta_cells = meta_input, condition_keys = ["condition 1", "condition 2"]"
- [readme] Training setup: "losses = model.train_model(nepochs = 100)"
