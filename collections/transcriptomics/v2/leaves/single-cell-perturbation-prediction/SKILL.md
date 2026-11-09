---
name: single-cell-perturbation-prediction
description: Use when a multi-batch multi-condition scRNA-seq dataset (e.g., 3-batch
  COVID-19 atlas with age and disease severity condition types) requires predicting
  counts toward a held-out condition/batch combination, in-sample or out-of-sample.
when_to_use_negative:
- Each batch is measured under only one condition, so there is not enough information
  to fully disentangle batch and condition effects
- Highly correlated condition types without combining them first — the article recommends
  creating a combined condition type (e.g., stim1_stim2, ctrl1_stim2) and training
  on it instead
- Single-batch, single-condition datasets where perturbation prediction across batches
  is not the goal — task-specific methods suffice
edam_operation: http://edamontology.org/operation_3800
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scDisInFact
  role: 'Primary method: conditional VAE that disentangles shared-bio and unshared-bio
    factors and performs cross-batch, multi-condition perturbation prediction via
    predict_counts'
  repo: https://github.com/ZhangLabGT/scDisInFact
- name: scGen
  role: Baseline perturbation prediction method for benchmarking on the same 6 scenarios
    and both settings
- name: scPreGAN
  role: Baseline perturbation prediction method for benchmarking on the same 6 scenarios
    and both settings
  repo: https://github.com/XiajieWei/scPreGAN-reproducibility
- name: numpy
  role: Holds the input count matrix (ncells, ngenes) fed to create_scdisinfact_dataset
- name: pandas
  role: Holds meta_cells dataframe with condition and batch columns matched to counts
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-024-45227-w
    title: 'scDisInFact: disentangled learning for integration and prediction of multi-batch
      multi-condition single-cell RNA-sequencing data'
  source_content_hash: sha256:75c7725843e0efed4510a987e93a1a02beabb51c0ced361d4eea5caa264ec503
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
  edam_operation: http://edamontology.org/operation_3800
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# single-cell-perturbation-prediction

## Summary

Predicts gene expression responses to unseen biological conditions in multi-batch multi-condition scRNA-seq data using scDisInFact, which disentangles shared-bio factors, condition-specific unshared-bio factors, and batch effect, and benchmarks against scGen and scPreGAN.

## When to use

Use when a multi-batch multi-condition scRNA-seq dataset (e.g., 3-batch COVID-19 atlas with age and disease severity condition types) requires predicting counts toward a held-out condition/batch combination, in-sample or out-of-sample.

## When NOT to use

- Each batch is measured under only one condition, so there is not enough information to fully disentangle batch and condition effects
- Highly correlated condition types without combining them first — the article recommends creating a combined condition type (e.g., stim1_stim2, ctrl1_stim2) and training on it instead
- Single-batch, single-condition datasets where perturbation prediction across batches is not the goal — task-specific methods suffice

## Inputs

- Count matrices (numpy.array of shape (ncells, ngenes)), e.g., processed HDF5 studies from the Fred Hutch COVID atlas
- Cell metadata (pandas.DataFrame of shape (ncells, xx)) with columns for each condition type (age, disease severity) and batch ID (study)
- Baseline method configurations for scGen and scPreGAN on the same prediction scenarios

## Outputs

- Predicted count matrices for each (condition combination, batch) prediction scenario
- Benchmark table (perturbation_prediction_scores.csv) with cell-type-specific MSE, R2, and Pearson correlation for all 6 scenarios in both in-sample and out-of-sample settings for scDisInFact, scGen, and scPreGAN

## How to apply

Construct the scdisinfact dataset with create_scdisinfact_dataset(counts, meta_cells, condition_key, batch_key), using the biological variables (e.g., age and disease severity) as condition keys and study ID as the batch key. Train the conditional variational autoencoder with latent dimensions Ks = [shared-bio dim, one unshared-bio dim per condition type] (e.g., [8, 2, 2] for two condition types) on device cuda:0 when available via model.train_model(nepochs = 100), then call model.eval(). For prediction, subset input cells matching a source condition/batch combination and call model.predict_counts(input_counts, meta_cells, condition_keys, batch_key, predict_conds, predict_batch) to generate counts toward the target condition (e.g., <moderate, 40-65> in batch Lee_2020). Run the same scenarios under an out-of-sample setting by holding out cells of the target condition from training. Benchmark against scGen and scPreGAN by computing cell-type-specific MSE, R2, and Pearson correlation between predicted and true counts.

## Related tools

- **scDisInFact** (Primary method: conditional VAE that disentangles shared-bio and unshared-bio factors and performs cross-batch, multi-condition perturbation prediction via predict_counts) — https://github.com/ZhangLabGT/scDisInFact
- **scGen** (Baseline perturbation prediction method for benchmarking on the same 6 scenarios and both settings)
- **scPreGAN** (Baseline perturbation prediction method for benchmarking on the same 6 scenarios and both settings) — https://github.com/XiajieWei/scPreGAN-reproducibility
- **numpy** (Holds the input count matrix (ncells, ngenes) fed to create_scdisinfact_dataset)
- **pandas** (Holds meta_cells dataframe with condition and batch columns matched to counts)

## Examples

```
counts_predict = model.predict_counts(input_counts = counts_input, meta_cells = meta_input, condition_keys = ["condition 1", "condition 2"], batch_key = "batch", predict_conds = ["ctrl", "healthy"], predict_batch = 0)
```

## Evaluation signals

- Scored table must contain MSE, R2, and Pearson correlation per cell type for all 6 scenarios in both in-sample and out-of-sample settings for all three methods, reproducing Fig 6a,b
- scDisInFact should outperform scGen and scPreGAN across all 6 prediction scenarios, with improvements more pronounced for tasks involving prediction of batch effects
- Training loss curve from model.train_model(nepochs = 100) decreases and model.eval() completes without error
- Predicted counts from predict_counts correspond to the requested predict_conds (e.g., <moderate, 40-65>) and predict_batch (e.g., Lee_2020)
- meta_cells columns for conditions and batch match counts row-wise before create_scdisinfact_dataset

## Limitations

- When each batch is measured under only one condition, there is not enough information to fully disentangle the batch and condition effect
- scDisInFact assumes condition types are mutually independent, which does not always hold for real-world datasets
- scDisInFact assumes the condition effect is independent of the batch effect and batch effects are consistent across conditions
- Condition and batch effects are assumed additive to shared-bio factors in latent space, limiting modeling of higher-order combinatorial conditions or batch effects
- Testing datasets from the manuscript are not included in the repository due to size and are available only upon request

## Evidence

- [results] In both the in-sample and out-of-sample tests on the COVID-19 dataset, scDisInFact outperforms scGen and scPreGAN across all 6 prediction scenarios: [evidence span withheld — anchor status: not_found]
- [methods] data_dict = create_scdisinfact_dataset(counts, meta_cells, condition_key = ["condition 1", "condition 2"], batch_key = "batch"): "data_dict = create_scdisinfact_dataset(counts, meta_cells, condition_key = ["condition 1", "condition 2"], batch_key = "batch")"
- [readme] losses = model.train_model(nepochs = 100): "model = scdisinfact(data_dict = data_dict, Ks = Ks, device = device)
losses = model.train_model(nepochs = 100)
_ = model.eval()"
- [methods] predict_counts(input_counts, meta_cells, condition_keys, batch_key, predict_conds, predict_batch): "counts_predict = model.predict_counts(input_counts = counts_input, meta_cells = meta_input, condition_keys = ["condition 1", "condition 2"]"
- [discussion] Limitation: insufficient information to disentangle batch and condition effects when each batch is measured under only one condition: "when each batch of cells is measured under only one condition, there is not enough information to fully disentangle the batch and condition effect."
