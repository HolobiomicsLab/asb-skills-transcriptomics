---
name: batch-mixing-metric-evaluation-asw
description: Use when a multi-patient, multi-condition scRNA-seq dataset (e.g., 6-patient
  GBM, DMSO vs 0.2 uM panobinostat) has been factorized and you must score batch effect
  removal (ASW-batch) and cell-type separation (ARI) across methods.
when_to_use_negative:
- Each batch of cells was measured under only one condition, so batch and condition
  effects cannot be fully disentangled and the unshared-factor scores are uninterpretable
- The dataset has a single batch or no batch labels, making ASW-batch undefined
- Input lacks meta_cells columns mapping each cell to its condition and batch ID,
  which create_scdisinfact_dataset requires
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: scDisInFact
  role: Provides the shared-bio factors (mu_c) and unshared-bio factors (mu_d) whose
    ASW-batch and ARI scores are being evaluated
  repo: github.com/ZhangLabGT/scDisInFact
- name: scINSIGHT
  role: Baseline method supplying common and condition-specific gene modules for the
    same ASW-batch/ARI comparison
  repo: github.com/Vivianstats/scINSIGHT
- name: numpy
  role: Holds count matrices as (ncells, ngenes) arrays and concatenated factor matrices
    z_cs/z_ds
- name: pandas
  role: meta_cells dataframe carrying batch, condition, and cell-annotation labels
    used in ASW-batch and ARI computation
- name: PyTorch
  role: Runs model.inference and model.generative forward passes on GPU to extract
    latent factors
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-024-45227-w
    title: 'scDisInFact: disentangled learning for integration and prediction of multi-batch
      multi-condition single-cell RNA-sequencing data'
  source_content_hash: sha256:006805438b51d3ed07217132a823875c06e48150afe960978f353cfd52c18082
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - PyTorch
  - numpy
  - pandas
  - scDisInFact
  - scINSIGHT
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.46153846153846156
n_claims: 13
---

# batch-mixing-metric-evaluation-asw

## Summary

Quantifies disentanglement quality of latent factors from multi-batch multi-condition scRNA-seq methods by computing ASW-batch (batch mixing) and ARI (cell-type separation) on shared- and unshared-bio factors, benchmarking scDisInFact against scINSIGHT on the GBM panobinostat dataset as in Fig 4a.

## When to use

Use when a multi-patient, multi-condition scRNA-seq dataset (e.g., 6-patient GBM, DMSO vs 0.2 uM panobinostat) has been factorized and you must score batch effect removal (ASW-batch) and cell-type separation (ARI) across methods.

## When NOT to use

- Each batch of cells was measured under only one condition, so batch and condition effects cannot be fully disentangled and the unshared-factor scores are uninterpretable
- The dataset has a single batch or no batch labels, making ASW-batch undefined
- Input lacks meta_cells columns mapping each cell to its condition and batch ID, which create_scdisinfact_dataset requires

## Inputs

- GBM count matrix (numpy array of shape (ncells, ngenes), DMSO vehicle vs 0.2 uM panobinostat)
- meta_cells pandas dataframe with columns for drug-treatment condition, patient-of-origin batch ID, and cell-type annotations
- Trained scDisInFact model (Ks latent dims) and scINSIGHT common/condition-specific module scores as baseline factors

## Outputs

- Concatenated shared-bio factor matrix z_cs and unshared-bio factor matrices z_ds (per condition type)
- gbm_asw_ari_scores.csv containing ASW-batch and ARI entries for both scDisInFact and scINSIGHT on shared and unshared factors

## How to apply

First construct the scdisinfact dataset with create_scdisinfact_dataset, passing condition_key (drug treatment) and batch_key (patient of origin), then instantiate scdisinfact with latent dimensions Ks (one shared-bio factor plus one unshared-bio factor per condition type) and call train_model with nepochs = 100 on GPU. Run model.inference and model.generative over every training count matrix and concatenate mu_c into z_cs (shared-bio factors) and mu_d into z_ds (unshared-bio factors). Train scINSIGHT (R package) on the same GBM matrix to obtain its common and condition-specific modules as the baseline factors. Compute ASW-batch (average silhouette width on batch labels) and ARI (adjusted Rand index against cell annotations) on shared factors of both methods, plus ASW-batch on unshared factors. Aggregate scores into gbm_asw_ari_scores.csv matching the Fig 4a / Supplementary Fig 8b layout; scDisInFact is expected to outperform scINSIGHT on shared-factor ARI and unshared-factor ASW-batch.

## Related tools

- **scDisInFact** (Provides the shared-bio factors (mu_c) and unshared-bio factors (mu_d) whose ASW-batch and ARI scores are being evaluated) — github.com/ZhangLabGT/scDisInFact
- **scINSIGHT** (Baseline method supplying common and condition-specific gene modules for the same ASW-batch/ARI comparison) — github.com/Vivianstats/scINSIGHT
- **numpy** (Holds count matrices as (ncells, ngenes) arrays and concatenated factor matrices z_cs/z_ds)
- **pandas** (meta_cells dataframe carrying batch, condition, and cell-annotation labels used in ASW-batch and ARI computation)
- **PyTorch** (Runs model.inference and model.generative forward passes on GPU to extract latent factors)

## Examples

```
from scDisInFact import scdisinfact, create_scdisinfact_dataset; data_dict = create_scdisinfact_dataset(counts, meta_cells, condition_key = ["condition 1"], batch_key = "batch"); model = scdisinfact(data_dict = data_dict, Ks = [8, 2], device = device); losses = model.train_model(nepochs = 100)
```

## Evaluation signals

- gbm_asw_ari_scores.csv exists and contains ASW-batch and ARI entries for both scDisInFact and scINSIGHT on shared and unshared factors
- scDisInFact shared-factor ARI (cell-type separation) and unshared-factor ASW-batch (batch effect removal) exceed the scINSIGHT baseline, matching the Fig 4a result
- Low ASW-batch on scDisInFact shared factors indicates batch mixing (successful batch effect removal) while ARI against cell annotations stays high
- Shared factors (z_cs) are confirmed free of batch and condition effect via the extraction loop over all training count matrices
- Expert review of the aggregated scores against the Fig 4a / Supplementary Fig 8b layout

## Limitations

- When each batch of cells is measured under only one condition, there is not enough information to fully disentangle the batch and condition effect, undermining unshared-factor metrics
- The GBM testing datasets are not included in the scDisInFact repository (only demo_data) and are available upon request, so the evaluation depends on obtaining them separately
- scDisInFact assumes condition effect is independent of batch effect and batch effects are consistent across conditions; violations would distort ASW-batch comparisons

## Evidence

- [methods] data_dict = create_scdisinfact_dataset(counts, meta_cells, condition_key = ["condition 1", "condition 2"], batch_key = "batch"): "data_dict = create_scdisinfact_dataset(counts, meta_cells, condition_key = ["condition 1", "condition 2"], batch_key = "batch")"
- [methods] train_model nepochs 100: "losses = model.train_model(nepochs = 100)"
- [readme] shared-bio factor extraction: "Extract shared-bio factor and unshared-bio factors. Shared-bio factor is removed of batch and condition effect."
- [discussion] disentanglement limitation: "when each batch of cells is measured under only one condition, there is not enough information to fully disentangle the batch and condition effect."
- [abstract] outperformance claim: "Our results demonstrate that scDisInFact outperforms existing methods that focus on individual tasks"
