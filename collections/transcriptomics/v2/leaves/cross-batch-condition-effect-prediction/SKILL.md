---
name: cross-batch-condition-effect-prediction
description: Use when you need to predict gene-expression count matrices for unseen
  condition/batch combinations (e.g., ctrl/healthy vs stim/severe) from multi-batch,
  multi-condition scRNA-seq data, including out-of-sample held-out cell subsets.
when_to_use_negative:
- Each batch is measured under only one condition — there is not enough information
  to fully disentangle batch and condition effects
- Condition types are strongly correlated/correlated with batch effects, since scDisInFact
  assumes condition types are mutually independent and condition effect independent
  of batch effect
- The task is single-task batch effect removal or key gene detection only, where task-specific
  methods may suffice and the multi-condition latent structure is unnecessary
edam_operation: http://edamontology.org/operation_3800
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scDisInFact
  role: 'Primary method: disentangles shared-bio, unshared-bio (condition-specific),
    and batch latent factors; predicts counts for arbitrary condition/batch combinations
    via predict_counts'
  repo: https://github.com/ZhangLabGT/scDisInFact
- name: scGen
  role: Baseline perturbation-prediction method trained on the same scenario splits
    for comparison
- name: scPreGAN
  role: Baseline perturbation-prediction method trained on the same scenario splits
    for comparison
  repo: https://github.com/XiajieWei/scPreGAN-reproducibility
- name: PyTorch
  role: Backend for instantiating and training the scDisInFact conditional variational
    autoencoder on GPU
- name: numpy
  role: Count matrix representation (ncells, ngenes) and computation of MSE/Pearson/R2
    metrics
- name: pandas
  role: meta_cells dataframe holding condition and batch annotations used to subset
    input cells per scenario
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-024-45227-w
    title: 'scDisInFact: disentangled learning for integration and prediction of multi-batch
      multi-condition single-cell RNA-sequencing data'
  source_content_hash: sha256:8884112ba3691106c4c01e6546f032ba4be56aa81f95bed7b869b824fbb0fc81
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
yes_rate: 0.7692307692307693
n_claims: 13
---

# cross-batch-condition-effect-prediction

## Summary

Perturbation prediction across conditions and batches in multi-batch multi-condition scRNA-seq data using scDisInFact's disentangled latent factors, benchmarked against scGen and scPreGAN with cell-type-specific MSE, Pearson correlation, and R2.

## When to use

Use when you need to predict gene-expression count matrices for unseen condition/batch combinations (e.g., ctrl/healthy vs stim/severe) from multi-batch, multi-condition scRNA-seq data, including out-of-sample held-out cell subsets.

## When NOT to use

- Each batch is measured under only one condition — there is not enough information to fully disentangle batch and condition effects
- Condition types are strongly correlated/correlated with batch effects, since scDisInFact assumes condition types are mutually independent and condition effect independent of batch effect
- The task is single-task batch effect removal or key gene detection only, where task-specific methods may suffice and the multi-condition latent structure is unnecessary

## Inputs

- Count matrix (numpy.array of shape (ncells, ngenes))
- meta_cells pandas.DataFrame of shape (ncells, xx) with condition columns (e.g., 'condition 1', 'condition 2') and a 'batch' column matching counts rows
- Held-out cell subsets defining out-of-sample scenarios (e.g., <ctrl,severe> cells)

## Outputs

- Predicted count matrices for target condition/batch combinations from each method
- Cell-type-specific MSE, Pearson correlation, and R2 per scenario and method (e.g., prediction_metrics.csv reproducing Fig 3a,b)

## How to apply

Construct a scdisinfact dataset from the count matrix and meta_cells with create_scdisinfact_dataset, passing condition_key = ['condition 1', 'condition 2'] and batch_key = 'batch'. Instantiate the scDisInFact model with latent dimensions Ks = [8, 2, 2] (one shared-bio factor plus one unshared-bio factor per condition type) on torch.device('cuda:0' if available) and train with model.train_model(nepochs = 100). For each scenario (within-batch and cross-batch prediction of treatment, severity, or both condition types), call model.predict_counts with the input counts, meta_cells, condition_keys, batch_key, the target predict_conds, and predict_batch; use a held-out subset such as <ctrl,severe> cells as the out-of-sample setting. Train scGen and scPreGAN on the same splits as baselines, then compute cell-type-specific MSE, Pearson correlation, and R2 between predicted and gold-standard counts to assemble the comparison (Fig. 3a,b style). scDisInFact is appropriate because it disentangles condition effect from batch effect, unlike scGen and scPreGAN which do not account for batch effects.

## Related tools

- **scDisInFact** (Primary method: disentangles shared-bio, unshared-bio (condition-specific), and batch latent factors; predicts counts for arbitrary condition/batch combinations via predict_counts) — https://github.com/ZhangLabGT/scDisInFact
- **scGen** (Baseline perturbation-prediction method trained on the same scenario splits for comparison)
- **scPreGAN** (Baseline perturbation-prediction method trained on the same scenario splits for comparison) — https://github.com/XiajieWei/scPreGAN-reproducibility
- **PyTorch** (Backend for instantiating and training the scDisInFact conditional variational autoencoder on GPU)
- **numpy** (Count matrix representation (ncells, ngenes) and computation of MSE/Pearson/R2 metrics)
- **pandas** (meta_cells dataframe holding condition and batch annotations used to subset input cells per scenario)

## Examples

```
counts_predict = model.predict_counts(input_counts = counts_input, meta_cells = meta_input, condition_keys = ["condition 1", "condition 2"], batch_key = "batch", predict_conds = ["ctrl", "healthy"], predict_batch = 0)
```

## Evaluation signals

- scDisInFact should achieve the smallest cell-type-specific MSE and highest Pearson correlation and R2 versus scGen and scPreGAN in all six in-sample scenarios (Fig. 3a)
- Superiority should also hold in out-of-sample scenarios using the held-out <ctrl,severe> cell subset (Fig. 3b), including both within-batch and cross-batch predictions
- Training loss curve from model.train_model(nepochs = 100) should decrease without divergence, and model.eval() should run cleanly after training
- Predicted counts from predict_counts should match the target predict_conds/predict_batch specification and have shape (n_input_cells, ngenes)
- Metrics table (prediction_metrics.csv) must contain values for all methods × all six scenarios (treatment, severity, both × within-batch, cross-batch)

## Limitations

- When each batch of cells is measured under only one condition, there is not enough information to fully disentangle the batch and condition effect
- scDisInFact assumes condition types are mutually independent, which does not always hold for real-world datasets
- scDisInFact assumes condition effect is independent of batch effect and batch effects are consistent across conditions
- Condition and batch effects are assumed additive in latent space, limiting modeling of higher-order combinatorial conditions or batch effects
- For highly correlated condition types, users must create a combined condition type (e.g., stim1_stim2, ctrl1_stim2, stim1_ctrl2, ctrl1_ctrl2) and train on it instead

## Evidence

- [intro] Existing methods for this task, such as scGen17 and scPreGAN18, do not account for batch effects between data matrices: "Existing methods for this task, such as scGen17 and scPreGAN18, do not account for batch effects between data matrices"
- [methods] predict_counts workflow: "counts_predict = model.predict_counts(input_counts = counts_input, meta_cells = meta_input, condition_keys = ["condition 1", "condition 2"]"
- [methods] create dataset and train: "data_dict = create_scdisinfact_dataset(counts, meta_cells, condition_key = ["condition 1", "condition 2"], batch_key = "batch")"
- [discussion] single-condition-per-batch limitation: "when each batch of cells is measured under only one condition, there is not enough information to fully disentangle the batch and condition effect."
- [readme] latent dimensions Ks: "# declare latent dimensions, we have two condition types, so there are three element corresponding to 
# shared-bio factor, unshared-bio factor for condition 1, unshared-bio factor for condition 2
Ks"
- [discussion] additivity assumption: "scDisInFact assumes the condition and batch effects are additive to the cells’ original biological identity (shared-bio factors) in the latent space."
