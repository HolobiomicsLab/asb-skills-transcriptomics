---
name: baseline-method-benchmarking
description: Use when comparing a new multi-batch multi-condition integration method
  against a baseline (e.g., scDisInFact vs scINSIGHT) on identical data, needing quantitative
  metrics like ASW-batch and ARI.
when_to_use_negative:
- Each batch is measured under only one condition — there is insufficient information
  to fully disentangle batch and condition effects, so a disentanglement benchmark
  is not meaningful
- The task is single-task (e.g., only perturbation prediction or only key gene detection)
  where task-specific baselines like scGen or scPreGAN, not factor-disentanglement
  baselines like scINSIGHT, are the appropriate comparators
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scDisInFact
  role: Method under evaluation; learns shared-bio and unshared-bio factors disentangled
    from batch effect on the GBM panobinostat dataset
  repo: https://github.com/ZhangLabGT/scDisInFact
- name: scINSIGHT
  role: Baseline; factorizes the same matrices into common and condition-specific
    modules via non-negative matrix factorization for comparison
  repo: https://github.com/Vivianstats/scINSIGHT
- name: numpy
  role: Concatenate per-batch latent factor arrays (z_cs, z_ds) and compute evaluation
    metrics
- name: pandas
  role: Hold cell metadata with condition and batch columns; write gbm_asw_ari_scores.csv
- name: PyTorch
  role: Training device management (torch.device cuda:0) and no-gradient forward passes
    through scDisInFact encoders/decoders
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-024-45227-w
    title: 'scDisInFact: disentangled learning for integration and prediction of multi-batch
      multi-condition single-cell RNA-sequencing data'
  source_content_hash: sha256:b728db95e92929759bb914506ce2a4641c2154b48c886dd0666f97d8dc54c0ae
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
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8235294117647058
n_claims: 17
---

# baseline-method-benchmarking

## Summary

Benchmark a proposed method (scDisInFact) against a baseline (scINSIGHT) on the same multi-batch multi-condition scRNA-seq dataset, quantifying disentanglement quality with ASW-batch and ARI metrics, as done for the 6-patient GBM DMSO-vs-panobinostat dataset in Fig 4a.

## When to use

Use when comparing a new multi-batch multi-condition integration method against a baseline (e.g., scDisInFact vs scINSIGHT) on identical data, needing quantitative metrics like ASW-batch and ARI.

## When NOT to use

- Each batch is measured under only one condition — there is insufficient information to fully disentangle batch and condition effects, so a disentanglement benchmark is not meaningful
- The task is single-task (e.g., only perturbation prediction or only key gene detection) where task-specific baselines like scGen or scPreGAN, not factor-disentanglement baselines like scINSIGHT, are the appropriate comparators

## Inputs

- GBM panobinostat count matrix (numpy array, ncells x ngenes)
- Cell metadata (pandas dataframe) with condition column (DMSO vs 0.2 uM panobinostat) and batch column (patient of origin, 6 patients)
- scINSIGHT common and condition-specific module factors from the same dataset

## Outputs

- Concatenated shared-bio factor matrix z_cs for both methods
- Concatenated unshared-bio factor matrices z_ds per condition type for both methods
- ASW-batch and ARI scores for both methods on shared and unshared factors, written to gbm_asw_ari_scores.csv

## How to apply

Construct the benchmark dataset once and run both methods on it identically: for scDisInFact, build the dataset with create_scdisinfact_dataset using the condition key (drug treatment: DMSO vs 0.2 uM panobinostat) and batch key (patient of origin), then instantiate scdisinfact with latent dimensions Ks (one shared-bio factor plus one unshared-bio factor per condition type) and train with train_model(nepochs=100) on GPU. Extract factors via model.inference and model.generative over all training count matrices, concatenating mu_c into z_cs (shared-bio factors) and mu_d into z_ds (unshared-bio factors). Train scINSIGHT on the same GBM dataset to obtain common and condition-specific modules as baseline factors. Compute ASW-batch (average silhouette width on batch labels) and ARI (adjusted Rand index against cell annotations) on the shared factors of both methods, plus ASW-batch on the unshared factors, then aggregate scores in the Fig 4a / Supplementary Fig 8b layout. The rationale is that scDisInFact disentangles shared-bio, unshared-bio, and batch factors, while scINSIGHT factorizes into common and condition-specific modules, so identical metrics on both factor sets isolate disentanglement quality.

## Related tools

- **scDisInFact** (Method under evaluation; learns shared-bio and unshared-bio factors disentangled from batch effect on the GBM panobinostat dataset) — https://github.com/ZhangLabGT/scDisInFact
- **scINSIGHT** (Baseline; factorizes the same matrices into common and condition-specific modules via non-negative matrix factorization for comparison) — https://github.com/Vivianstats/scINSIGHT
- **numpy** (Concatenate per-batch latent factor arrays (z_cs, z_ds) and compute evaluation metrics)
- **pandas** (Hold cell metadata with condition and batch columns; write gbm_asw_ari_scores.csv)
- **PyTorch** (Training device management (torch.device cuda:0) and no-gradient forward passes through scDisInFact encoders/decoders)

## Examples

```
from scDisInFact import scdisinfact, create_scdisinfact_dataset; data_dict = create_scdisinfact_dataset(counts, meta_cells, condition_key=["drug_treatment"], batch_key="patient"); model = scdisinfact(data_dict=data_dict, Ks=[8,2], device=torch.device("cuda:0")); losses = model.train_model(nepochs=100)
```

## Evaluation signals

- Output CSV gbm_asw_ari_scores.csv exists and contains ASW-batch and ARI entries for both scDisInFact and scINSIGHT on shared and unshared factors
- scDisInFact shared-factor ASW-batch reflects batch mixing consistent with the reported Fig 4a result (expert review against the figure)
- Both methods are trained on the identical GBM dataset object with the same condition key (drug treatment) and batch key (patient)
- Training losses converge over nepochs=100; model.eval() is called and inference runs over all training count matrices before metric computation
- scDisInFact outperforms scINSIGHT on ARI for shared-bio factors and ASW-batch for unshared-bio factors, matching the paper's finding

## Limitations

- When each batch of cells is measured under only one condition, there is not enough information to fully disentangle batch and condition effect
- scDisInFact assumes condition types are mutually independent, which does not always hold for real-world datasets
- scDisInFact assumes condition effect is independent of batch effect, batch effects are consistent across conditions, and condition/batch effects are additive in latent space, limiting modeling of higher-order effects
- Testing datasets from the manuscript (including the GBM dataset) are not in the repository due to size and are available upon request

## Evidence

- [intro] scINSIGHT factorizes the scRNA-seq matrices into common and condition-specific modules using non-negative matrix factorization: "scINSIGHT14 factorizes the scRNA-seq matrices into common and condition-specific modules using non-negative matrix factorization"
- [methods] create_scdisinfact_dataset with condition and batch keys: "data_dict = create_scdisinfact_dataset(counts, meta_cells, condition_key = ["condition 1", "condition 2"], batch_key = "batch")"
- [methods] training with nepochs=100: "losses = model.train_model(nepochs = 100)"
- [readme] shared-bio factor is batch- and condition-free: "Extract shared-bio factor and unshared-bio factors. Shared-bio factor is removed of batch and condition effect."
- [discussion] limitation on disentanglement when batches are condition-specific: "when each batch of cells is measured under only one condition, there is not enough information to fully disentangle the batch and condition effect."
