---
name: disentanglement-test-holdout-configuration
description: Use when benchmarking whether a multi-batch multi-condition scRNA-seq
  model separates cell types while removing batch effect in shared factors, and removes
  batch effect from condition-specific (unshared-bio) factors.
when_to_use_negative:
- Each batch is measured under only one condition — the paper states there is not
  enough information to fully disentangle batch and condition effects in that setting
- The task is perturbation prediction of held-out matrices (use predict_counts) rather
  than scoring factor disentanglement quality
- Input is real data with correlated condition types where the assumption of mutually
  independent condition types does not hold
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scDisInFact
  role: Conditional variational autoencoder trained on the held-out configuration;
    its encoders yield mu_c (shared-bio) and mu_d (unshared-bio) latent factors to
    score
  repo: https://github.com/ZhangLabGT/scDisInFact
- name: scINSIGHT
  role: Baseline that factorizes scRNA-seq matrices into common and condition-specific
    modules via non-negative matrix factorization; its common factor is scored with
    the same ARI/ASW-batch metrics
  repo: https://github.com/Vivianstats/scINSIGHT
- name: numpy
  role: Holds count matrices of shape (ncells, ngenes) and concatenates z_cs/z_ds
    factor arrays across training matrices
- name: pandas
  role: meta_cells dataframe of shape (ncells, xx) supplying condition and batch keys
    used to define the held-out matrices and grouping for metrics
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-024-45227-w
    title: 'scDisInFact: disentangled learning for integration and prediction of multi-batch
      multi-condition single-cell RNA-sequencing data'
  source_content_hash: sha256:1c98e74f9c48cf3c31946bc7854c24ba46c92d5c5edd5fcceaa432b9cde3da80
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ZhangLabGT/scDisInFact
  tools:
  - numpy
  - pandas
  - scDisInFact
  - scINSIGHT
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 14
---

# disentanglement-test-holdout-configuration

## Summary

A benchmark configuration for evaluating disentanglement of shared-bio vs unshared-bio latent factors: hold out the <ctrl,severe> batch-1 and <stim,healthy> batch-2 matrices, train scDisInFact and the scINSIGHT NMF baseline, and score cell-type separation (ARI) and batch removal (ASW-batch) as in Fig. 2d.

## When to use

Use when benchmarking whether a multi-batch multi-condition scRNA-seq model separates cell types while removing batch effect in shared factors, and removes batch effect from condition-specific (unshared-bio) factors.

## When NOT to use

- Each batch is measured under only one condition — the paper states there is not enough information to fully disentangle batch and condition effects in that setting
- The task is perturbation prediction of held-out matrices (use predict_counts) rather than scoring factor disentanglement quality
- Input is real data with correlated condition types where the assumption of mutually independent condition types does not hold

## Inputs

- Simulated multi-batch multi-condition scRNA-seq count matrices (numpy arrays of shape (ncells, ngenes)) with meta_cells dataframe holding 'condition 1', 'condition 2', and 'batch' columns
- Ground-truth cell-type labels for ARI computation
- Batch labels for ASW-batch silhouette computation

## Outputs

- z_cs shared-bio factor matrix (ncells x K_shared) batch- and condition-corrected
- z_ds_cond1 / z_ds_cond2 unshared-bio factor matrices
- scINSIGHT common and condition-specific module scores on the same held-out configuration
- batch_removal_metrics_fig2d.csv with columns method, metric, value (ARI and ASW-batch in [0,1])

## How to apply

First remove the <ctrl,severe> batch-1 and <stim,healthy> batch-2 count matrices from the simulated dataset so the held-out configuration tests generalization. Train scDisInFact by building the dataset with create_scdisinfact_dataset (condition_keys = ['condition 1','condition 2'], batch_key = 'batch'), declaring Ks = [8, 2, 2] for shared-bio plus two unshared-bio dimensions, and running model.train_model(nepochs=100). Run one forward pass with model.inference/generative per training matrix and concatenate mu_c into z_cs (shared-bio) and mu_d into z_ds_cond1/z_ds_cond2 (unshared-bio), as in demo.ipynb. Cluster cells in the z_cs space with KNN-graph/Leiden clustering in scanpy, compute ARI against ground-truth cell-type labels and ASW-batch (silhouette on batch labels, rescaled from [-1,1] to [0,1]), and compute ASW-batch on z_ds_cond1/z_ds_cond2. Train scINSIGHT (non-negative matrix factorization into common and condition-specific modules) on the same held-out configuration using the Cartesian-product combined condition type (e.g. ctrl_severe, stim_healthy) and compute the same metrics on its common factor. Assemble rows for scDisInFact shared, scDisInFact unshared, and scINSIGHT into batch_removal_metrics_fig2d.csv and judge success by scDisInFact showing stronger batch mixing on shared factors while preserving cell-type ARI relative to scINSIGHT.

## Related tools

- **scDisInFact** (Conditional variational autoencoder trained on the held-out configuration; its encoders yield mu_c (shared-bio) and mu_d (unshared-bio) latent factors to score) — https://github.com/ZhangLabGT/scDisInFact
- **scINSIGHT** (Baseline that factorizes scRNA-seq matrices into common and condition-specific modules via non-negative matrix factorization; its common factor is scored with the same ARI/ASW-batch metrics) — https://github.com/Vivianstats/scINSIGHT
- **numpy** (Holds count matrices of shape (ncells, ngenes) and concatenates z_cs/z_ds factor arrays across training matrices)
- **pandas** (meta_cells dataframe of shape (ncells, xx) supplying condition and batch keys used to define the held-out matrices and grouping for metrics)

## Examples

```
from scDisInFact import scdisinfact, create_scdisinfact_dataset; data_dict = create_scdisinfact_dataset(counts, meta_cells, condition_key=["condition 1", "condition 2"], batch_key="batch"); model = scdisinfact(data_dict=data_dict, Ks=[8,2,2], device=device); losses = model.train_model(nepochs=100)
```

## Evaluation signals

- batch_removal_metrics_fig2d.csv exists with rows for scDisInFact shared, scDisInFact unshared, and scINSIGHT, with columns method, metric, value
- Every ARI and ASW-batch value lies in [0, 1] (silhouette range [-1, 1] rescaled to [0, 1])
- scDisInFact attains higher ARI than scINSIGHT on the shared-bio factor space (cell-type separation while removing batch effect), per Fig. 2d
- scDisInFact attains a higher ASW-batch score than scINSIGHT on the unshared-bio factors (batch effect removed from condition-specific factors), per Fig. 2d
- The held-out <ctrl,severe> batch-1 and <stim,healthy> batch-2 matrices are excluded from training in both methods

## Limitations

- When each batch of cells is measured under only one condition, there is not enough information to fully disentangle batch and condition effects
- scDisInFact assumes condition types are mutually independent, that condition effect is independent of batch effect with batch effects consistent across conditions, and that effects are additive in latent space — limiting modeling of higher-order conditions or batch effects
- The paper's qualitative acceptance criterion (scDisInFact shows stronger batch mixing on shared factors while preserving cell-type ARI) must be checked rather than a fixed numeric threshold

## Evidence

- [readme] Extract shared-bio factor and unshared-bio factors. Shared-bio factor is removed of batch and condition effect.: "Extract shared-bio factor and unshared-bio factors. Shared-bio factor is removed of batch and condition effect."
- [intro] scINSIGHT factorization description: "scINSIGHT14 factorizes the scRNA-seq matrices into common and condition-specific modules using non-negative matrix factorization"
- [discussion] Combined condition type for correlated conditions: "the users can create a combined condition type (stim1_stim2, ctrl1_stim2, stim1_ctrl2, ctrl1_ctrl2) and train scDisInFact on the combined condition type instead."
- [abstract] Latent factor disentanglement claim: "scDisInFact learns latent factors that disentangle condition effect from batch effect, enabling it to simultaneously perform three tasks: batch effect removal, condition-associated key gene"
- [methods] Dataset creation call for training inputs: "data_dict = create_scdisinfact_dataset(counts, meta_cells, condition_key = ["condition 1", "condition 2"], batch_key = "batch")"
