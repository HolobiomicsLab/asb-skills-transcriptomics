---
name: disentangled-latent-factor-extraction
description: Use when a multi-batch, multi-condition scRNA-seq count matrix (e.g.,
  6-patient GBM, DMSO vs panobinostat) must be decomposed into shared vs condition-specific
  latent factors for batch-effect removal or cell-type/batch-mixing evaluation.
when_to_use_negative:
- Each batch is measured under only one condition — there is not enough information
  to fully disentangle batch and condition effect
- The goal is a single-task analysis only (e.g., pure batch integration without conditions),
  where a dedicated batch-removal method suffices
- Data lacks multiple biological conditions, so unshared-bio factors are undefined
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: scDisInFact
  role: Conditional VAE that disentangles shared-bio and unshared-bio latent factors
    and removes batch effect; used to produce z_cs and z_ds via inference/generative
    passes
  repo: https://github.com/ZhangLabGT/scDisInFact
- name: scINSIGHT
  role: Baseline that factorizes scRNA-seq matrices into common and condition-specific
    modules for ASW-batch/ARI comparison
  repo: https://github.com/Vivianstats/scINSIGHT
- name: PyTorch
  role: Backend for model training on GPU (torch.device cuda:0) and torch.no_grad
    inference passes
- name: numpy
  role: Array handling and np.concatenate of shared/unshared factor matrices across
    training count matrices
- name: pandas
  role: meta_cells dataframe carrying batch ID and condition columns aligned to the
    count matrix
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-024-45227-w
    title: 'scDisInFact: disentangled learning for integration and prediction of multi-batch
      multi-condition single-cell RNA-sequencing data'
  source_content_hash: sha256:69a6ff79caefc3e93016283fc7500798cb1ef950bd39ae9071e7a4bd4c78293f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ZhangLabGT/scDisInFact
  tools:
  - PyTorch
  - numpy
  - pandas
  - scDisInFact
  - scINSIGHT
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# disentangled-latent-factor-extraction

## Summary

Extracts disentangled shared-bio factors (condition- and batch-irrelevant) and unshared-bio factors (condition-related) from multi-batch multi-condition scRNA-seq data using scDisInFact's conditional variational autoencoder, and benchmarks them against scINSIGHT's common/condition-specific modules with ASW-batch and ARI.

## When to use

Use when a multi-batch, multi-condition scRNA-seq count matrix (e.g., 6-patient GBM, DMSO vs panobinostat) must be decomposed into shared vs condition-specific latent factors for batch-effect removal or cell-type/batch-mixing evaluation.

## When NOT to use

- Each batch is measured under only one condition — there is not enough information to fully disentangle batch and condition effect
- The goal is a single-task analysis only (e.g., pure batch integration without conditions), where a dedicated batch-removal method suffices
- Data lacks multiple biological conditions, so unshared-bio factors are undefined

## Inputs

- scRNA-seq count matrix (numpy.array, shape (ncells, ngenes)) per batch
- cell metadata (pandas.dataframe, shape (ncells, xx)) with batch ID and condition columns matching counts
- condition_keys (e.g., drug treatment: DMSO vs 0.2 uM panobinostat) and batch_key (patient of origin)
- Latent dimension list Ks (shared + per-condition-type unshared dimensions)

## Outputs

- Shared-bio factor matrix z_cs (mu_c concatenated across training matrices)
- Unshared-bio factor matrices z_ds per condition type (e.g., z_ds_cond1, z_ds_cond2)
- Trained scdisinfact model losses from train_model
- Benchmark table of ASW-batch and ARI scores for scDisInFact vs scINSIGHT (e.g., gbm_asw_ari_scores.csv)

## How to apply

Construct the scdisinfact dataset with create_scdisinfact_dataset, passing the per-batch count matrices, meta_cells, condition_key (e.g., drug treatment) and batch_key (patient of origin). Instantiate scdisinfact with latent dimensions Ks — one shared-bio factor dimension plus one unshared-bio factor dimension per condition type (e.g., Ks = [8, 2, 2] for two condition types) — on a CUDA device and call train_model(nepochs = 100). For each training count matrix, run model.inference to get mu_c (shared-bio factor) and mu_d (unshared-bio factors), then model.generative, concatenating mu_c across datasets into z_cs and the per-condition mu_d entries into z_ds. For benchmarking, run scINSIGHT on the same data to obtain common and condition-specific modules as baseline factors, then compute ASW-batch (average silhouette width on batch labels) and ARI (adjusted Rand index against cell annotations) on shared factors and ASW-batch on unshared factors, aggregating scores as in Fig 4a / Supplementary Fig 8b.

## Related tools

- **scDisInFact** (Conditional VAE that disentangles shared-bio and unshared-bio latent factors and removes batch effect; used to produce z_cs and z_ds via inference/generative passes) — https://github.com/ZhangLabGT/scDisInFact
- **scINSIGHT** (Baseline that factorizes scRNA-seq matrices into common and condition-specific modules for ASW-batch/ARI comparison) — https://github.com/Vivianstats/scINSIGHT
- **PyTorch** (Backend for model training on GPU (torch.device cuda:0) and torch.no_grad inference passes)
- **numpy** (Array handling and np.concatenate of shared/unshared factor matrices across training count matrices)
- **pandas** (meta_cells dataframe carrying batch ID and condition columns aligned to the count matrix)

## Examples

```
from scDisInFact import scdisinfact, create_scdisinfact_dataset; data_dict = create_scdisinfact_dataset(counts, meta_cells, condition_key=["condition 1", "condition 2"], batch_key="batch"); model = scdisinfact(data_dict=data_dict, Ks=[8,2,2], device=torch.device("cuda:0")); losses = model.train_model(nepochs=100)
```

## Evaluation signals

- Output CSV (e.g., gbm_asw_ari_scores.csv) exists and contains ASW-batch and ARI entries for both scDisInFact and scINSIGHT on shared and unshared factors
- scDisInFact ARI on shared-bio factors reflects cell-type (annotation) separation; per the source finding it largely outperforms scINSIGHT on the GBM dataset
- scDisInFact shared-factor ASW-batch reflects batch mixing consistent with the reported Fig 4a result (expert review against the figure)
- Concatenated factor dimensions match total cell count across all training count matrices (z_cs rows = sum of ncells per dataset)
- Training losses from train_model(nepochs = 100) decrease monotonically and model.eval() returns without error

## Limitations

- When each batch of cells is measured under only one condition, there is not enough information to fully disentangle the batch and condition effect
- scDisInFact assumes condition types are mutually independent, which does not always hold for real-world datasets; highly correlated condition types should be combined into one condition type
- Assumes condition effect is independent of batch effect, batch effects are consistent across conditions, and effects are additive in latent space, limiting modeling of higher-order conditions or batch effects
- Future work is needed to better accommodate correlation between condition effects and batch effects and improve perturbation effect modeling capacity

## Evidence

- [abstract] scDisInFact learns latent factors that disentangle condition effect from batch effect, enabling it to simultaneously perform three tasks: batch effect removal, condition-associated key gene: "scDisInFact learns latent factors that disentangle condition effect from batch effect, enabling it to simultaneously perform three tasks: batch effect removal, condition-associated key gene"
- [methods] data_dict = create_scdisinfact_dataset(counts, meta_cells, condition_key = ["condition 1", "condition 2"], batch_key = "batch"): "data_dict = create_scdisinfact_dataset(counts, meta_cells, condition_key = ["condition 1", "condition 2"], batch_key = "batch")"
- [methods] losses = model.train_model(nepochs = 100): "losses = model.train_model(nepochs = 100)"
- [readme] Extract shared-bio factor and unshared-bio factors. Shared-bio factor is removed of batch and condition effect.: "Extract shared-bio factor and unshared-bio factors. Shared-bio factor is removed of batch and condition effect."
- [discussion] when each batch of cells is measured under only one condition, there is not enough information to fully disentangle the batch and condition effect.: "when each batch of cells is measured under only one condition, there is not enough information to fully disentangle the batch and condition effect."
