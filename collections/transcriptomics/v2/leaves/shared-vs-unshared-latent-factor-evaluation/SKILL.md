---
name: shared-vs-unshared-latent-factor-evaluation
description: Use when you have shared-bio and unshared-bio factor matrices from a
  disentanglement-test run plus ground-truth cell-type and batch labels, and you must
  quantify cell-type preservation (ARI) and batch removal (ASW-batch) per factor class.
when_to_use_negative:
- Each batch is measured under only one condition — the paper states there is not
  enough information to fully disentangle batch and condition effect in this configuration
- Input is real expression data rather than simulated data with ground-truth cell-type
  labels, so ARI against truth cannot be computed
- The goal is key-gene detection or perturbation prediction rather than factor-level
  batch-removal quality assessment
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: scDisInFact
  role: Trains the conditional variational autoencoder and yields z_cs shared-bio
    factors and z_ds unshared-bio factors via model.inference() for metric computation
  repo: https://github.com/ZhangLabGT/scDisInFact
- name: scINSIGHT
  role: Baseline that factorizes the same held-out matrices into common and condition-specific
    modules; its common factor receives the same ARI/ASW-batch metrics
  repo: https://github.com/Vivianstats/scINSIGHT
- name: numpy
  role: Concatenates per-dataset factor arrays (np.concatenate of z_cs and z_ds) into
    (ncells, K) matrices before clustering
- name: pandas
  role: Stores meta_cells with batch and condition columns and writes the method/metric/value
    comparison CSV
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-024-45227-w
    title: 'scDisInFact: disentangled learning for integration and prediction of multi-batch
      multi-condition single-cell RNA-sequencing data'
  source_content_hash: sha256:a907d70c68ad6331b85e9ebc843620814bd093b4c28af5a16b5ce14822c7ba61
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ZhangLabGT/scDisInFact
  tools:
  - numpy
  - pandas
  - scDisInFact
  - scINSIGHT
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# Shared- vs unshared-bio latent factor evaluation

## Summary

Evaluates how well scDisInFact's disentangled latent factors separate cell types from batch effects by computing ARI and ASW-batch on shared-bio factors and ASW-batch on unshared-bio factors, benchmarked against scINSIGHT's common module on simulated multi-batch multi-condition data (Fig. 2d).

## When to use

Use when you have shared-bio and unshared-bio factor matrices from a disentanglement-test run plus ground-truth cell-type and batch labels, and you must quantify cell-type preservation (ARI) and batch removal (ASW-batch) per factor class.

## When NOT to use

- Each batch is measured under only one condition — the paper states there is not enough information to fully disentangle batch and condition effect in this configuration
- Input is real expression data rather than simulated data with ground-truth cell-type labels, so ARI against truth cannot be computed
- The goal is key-gene detection or perturbation prediction rather than factor-level batch-removal quality assessment

## Inputs

- Shared-bio factor matrix z_cs (numpy array, ncells x K_shared) from the held-out disentanglement-test run
- Unshared-bio factor matrices z_ds_cond1 and z_ds_cond2 (numpy arrays, ncells x K_unshared)
- Ground-truth cell-type labels per cell
- Batch labels per cell (e.g. batch column of meta_cells pandas.dataframe)
- Count matrices and meta_cells for the <ctrl,severe> batch-1 and <stim,healthy> batch-2 held-out configuration
- scINSIGHT common-module factor scores from the same held-out data

## Outputs

- batch_removal_metrics_fig2d.csv with columns method, metric, value and rows for scDisInFact shared, scDisInFact unshared, and scINSIGHT
- Optional grouped bar plot reproducing Fig. 2d layout

## How to apply

First hold out one matrix per condition-batch combination (e.g. the <ctrl,severe> batch-1 and <stim,healthy> batch-2 matrices) so batch and condition effects are not confounded, then run scDisInFact with Ks=[8,2,2] for two condition types and extract z_cs (concatenated shared-bio factors) and z_ds_cond1/z_ds_cond2 (unshared factors). Cluster cells in the shared-bio factor space using KNN-graph/Leiden clustering in scanpy and compute ARI against ground-truth cell-type labels plus ASW-batch (silhouette width on batch labels, rescaled from [-1,1] to [0,1]). Compute ASW-batch on the unshared-bio factor matrices to verify batch effect is also removed from condition-specific factors. Train the scINSIGHT baseline (non-negative matrix factorization into common and condition-specific modules, using the Cartesian-product combined condition type such as ctrl_severe/stim_healthy) on the identical held-out configuration and compute the same metrics on its common factor. Assemble results into batch_removal_metrics_fig2d.csv with columns method, metric, value and compare against the paper's criterion that scDisInFact shows stronger batch mixing on shared factors while preserving cell-type ARI relative to scINSIGHT.

## Related tools

- **scDisInFact** (Trains the conditional variational autoencoder and yields z_cs shared-bio factors and z_ds unshared-bio factors via model.inference() for metric computation) — https://github.com/ZhangLabGT/scDisInFact
- **scINSIGHT** (Baseline that factorizes the same held-out matrices into common and condition-specific modules; its common factor receives the same ARI/ASW-batch metrics) — https://github.com/Vivianstats/scINSIGHT
- **numpy** (Concatenates per-dataset factor arrays (np.concatenate of z_cs and z_ds) into (ncells, K) matrices before clustering)
- **pandas** (Stores meta_cells with batch and condition columns and writes the method/metric/value comparison CSV)

## Examples

```
dict_inf = model.inference(counts=dataset.counts_norm.to(model.device), batch_ids=dataset.batch_id[:,None].to(model.device)); z_c = dict_inf['mu_c']
```

## Evaluation signals

- Every ARI and ASW-batch value lies in [0,1], with silhouette widths rescaled from [-1,1] to [0,1]
- The CSV contains rows for scDisInFact shared, scDisInFact unshared, and scINSIGHT methods
- scDisInFact shared-bio ARI exceeds scINSIGHT common-factor ARI (better cell-type separation while removing batch)
- scDisInFact achieves a higher ASW-batch score than scINSIGHT on the unshared-bio factors
- The held-out configuration is confirmed: <ctrl,severe> batch-1 and <stim,healthy> batch-2 matrices excluded from training

## Limitations

- Insufficient information to disentangle batch and condition effects when each batch is measured under only one condition
- Silhouette rescaling from [-1,1] to [0,1] is a convention; raw values should not be compared across differently scaled runs
- Simulated ground-truth labels are required for ARI; the paper's qualitative acceptance criterion ('stronger batch mixing while preserving ARI') is comparative, not an absolute threshold

## Evidence

- [methods] Extract shared-bio factor and unshared-bio factors. Shared-bio factor is removed of batch and condition effect.: "Extract shared-bio factor and unshared-bio factors. Shared-bio factor is removed of batch and condition effect."
- [intro] scINSIGHT factorizes matrices into common and condition-specific modules via NMF: "scINSIGHT14 factorizes the scRNA-seq matrices into common and condition-specific modules using non-negative matrix factorization"
- [readme] README code concatenating shared-bio factors across training matrices: "shared-bio factor, concatenate across all training matrices
z_cs = np.concatenate(z_cs, axis = 0)"
- [discussion] Limitation when each batch has one condition: "when each batch of cells is measured under only one condition, there is not enough information to fully disentangle the batch and condition effect."
