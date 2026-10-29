---
name: asw-silhouette-batch-mixing-scoring
description: Use when comparing integration/disentanglement methods on multi-batch
  multi-condition scRNA-seq latent factors and you have batch labels plus ground-truth
  cell-type labels to score batch mixing vs. cell-type preservation.
when_to_use_negative:
- Each batch is measured under only one condition — per the article, 'when each batch
  of cells is measured under only one condition, there is not enough information to
  fully disentangle the batch and condition effect', so disentanglement scores on
  separate shared/unshared factors are not meaningful.
- The condition types under study are strongly correlated and were not converted to
  a combined condition type (e.g. ctrl_severe) as the article recommends for highly
  correlated condition types.
- The goal is evaluating a single method's clustering rather than a controlled batch-removal
  comparison — ASW-batch here presupposes matched baseline runs (scINSIGHT common
  factor on the same held-out matrices).
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scDisInFact
  role: provides the shared-bio (z_cs) and unshared-bio (z_ds_cond1/z_ds_cond2) latent
    factors whose clustering quality and batch mixing are scored
  repo: https://github.com/ZhangLabGT/scDisInFact
- name: scINSIGHT
  role: NMF baseline whose common/condition-specific modules are scored with the same
    ARI/ASW-batch metrics for the Fig. 2d comparison
  repo: https://github.com/Vivianstats/scINSIGHT
- name: scanpy
  role: KNN-graph/Leiden clustering of cells in the shared-bio factor space before
    computing ARI against ground-truth cell-type labels
- name: numpy
  role: concatenation and numeric handling of the factor matrices (numpy.array of
    shape (ncells, ngenes) inputs)
- name: pandas
  role: holds meta_cells (shape (ncells, xx)) supplying batch ID and condition columns,
    and stores the assembled metric comparison table
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-024-45227-w
    title: 'scDisInFact: disentangled learning for integration and prediction of multi-batch
      multi-condition single-cell RNA-sequencing data'
  source_content_hash: sha256:1f2e0dd88bca173afec61db709a8d399aeb36ad30ea66cdc89aa91cff2282e0d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ZhangLabGT/scDisInFact
  tools:
  - numpy
  - pandas
  - scDisInFact
  - scINSIGHT
  - scanpy
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# ASW-silhouette-batch-mixing-scoring

## Summary

Quantifies batch-effect removal in learned latent factors by clustering cells in factor space and computing ARI against ground-truth cell-type labels plus ASW-batch (silhouette width on batch labels), as used to compare scDisInFact's shared/unshared-bio factors against scINSIGHT's common modules in the simulated disentanglement test (Fig. 2d).

## When to use

Use when comparing integration/disentanglement methods on multi-batch multi-condition scRNA-seq latent factors and you have batch labels plus ground-truth cell-type labels to score batch mixing vs. cell-type preservation.

## When NOT to use

- Each batch is measured under only one condition — per the article, 'when each batch of cells is measured under only one condition, there is not enough information to fully disentangle the batch and condition effect', so disentanglement scores on separate shared/unshared factors are not meaningful.
- The condition types under study are strongly correlated and were not converted to a combined condition type (e.g. ctrl_severe) as the article recommends for highly correlated condition types.
- The goal is evaluating a single method's clustering rather than a controlled batch-removal comparison — ASW-batch here presupposes matched baseline runs (scINSIGHT common factor on the same held-out matrices).

## Inputs

- shared-bio factor matrix z_cs (numpy array, ncells x 8) from scDisInFact model.inference (mu_c) across all training count matrices
- unshared-bio factor matrices z_ds_cond1 and z_ds_cond2 (numpy arrays) from mu_d per condition type
- ground-truth cell-type labels and batch labels from meta_cells (pandas dataframe)
- scINSIGHT common factor matrix trained on the same held-out data configuration

## Outputs

- batch_removal_metrics_fig2d.csv with columns method, metric, value
- ARI values (range [0, 1]) per method on shared/common factors
- ASW-batch values (silhouette rescaled to [0, 1]) per method on shared and unshared factors
- optional grouped bar plot matching Fig 2d layout

## How to apply

Extract the shared-bio factor matrix z_cs (batch- and condition-effect-removed) and the unshared-bio factor matrices z_ds_cond1/z_ds_cond2 from the scDisInFact run by looping over data_dict['datasets'], calling model.inference to obtain mu_c/mu_d, and concatenating z_c across training matrices and each condition-specific z_d separately. Cluster cells in the shared-bio factor space (KNN-graph/Leiden clustering in scanpy) and compute ARI against ground-truth cell-type labels and ASW-batch (silhouette width computed on batch labels, rescaled from [-1, 1] to [0, 1]). Compute ASW-batch on the unshared-bio factor matrices to verify batch effect is also removed from condition-specific factors. Train the scINSIGHT baseline (non-negative matrix factorization into common and condition-specific modules) on the same held-out configuration using the Cartesian-product combined condition type (e.g. ctrl_severe, stim_healthy, ctrl_healthy, stim_severe) and compute identical ARI/ASW-batch metrics on its common factor. Assemble a comparison table with columns method, metric, value and save as batch_removal_metrics_fig2d.csv; acceptance follows the paper's qualitative criterion that scDisInFact shows stronger batch mixing on shared factors while preserving cell-type ARI relative to scINSIGHT.

## Related tools

- **scDisInFact** (provides the shared-bio (z_cs) and unshared-bio (z_ds_cond1/z_ds_cond2) latent factors whose clustering quality and batch mixing are scored) — https://github.com/ZhangLabGT/scDisInFact
- **scINSIGHT** (NMF baseline whose common/condition-specific modules are scored with the same ARI/ASW-batch metrics for the Fig. 2d comparison) — https://github.com/Vivianstats/scINSIGHT
- **scanpy** (KNN-graph/Leiden clustering of cells in the shared-bio factor space before computing ARI against ground-truth cell-type labels)
- **numpy** (concatenation and numeric handling of the factor matrices (numpy.array of shape (ncells, ngenes) inputs))
- **pandas** (holds meta_cells (shape (ncells, xx)) supplying batch ID and condition columns, and stores the assembled metric comparison table)

## Examples

```
dict_inf = model.inference(counts = dataset.counts_norm.to(model.device), batch_ids = dataset.batch_id[:,None].to(model.device), print_stat = True)
```

## Evaluation signals

- Every ARI and ASW-batch value in batch_removal_metrics_fig2d.csv lies in [0, 1] (silhouette range [-1, 1] bounded to [0, 1] after the standard rescaling).
- The CSV contains rows for all three series: scDisInFact shared-bio factors, scDisInFact unshared-bio factors, and scINSIGHT.
- Latent dimensions match Ks = [8, 2, 2]: shared-bio factor plus one unshared-bio factor per condition type (two condition types).
- z_cs is concatenated across all training matrices before scoring; z_ds_cond1 and z_ds_cond2 are concatenated separately per condition.
- Qualitative acceptance: scDisInFact shows stronger batch mixing (higher ASW-batch) on shared factors while preserving cell-type ARI relative to scINSIGHT, and higher ASW-batch on unshared factors.

## Limitations

- When each batch of cells is measured under only one condition, there is not enough information to fully disentangle the batch and condition effect, undermining the factor-specific scores.
- scDisInFact assumes condition and batch effects are additive to the cells' original biological identity in latent space and mutually independent, which does not always hold for real-world datasets.
- Scores reflect the simulated disentanglement-test configuration (with <ctrl,severe> batch-1 and <stim,healthy> batch-2 matrices held out) and the paper's qualitative comparison criterion rather than an absolute threshold.

## Evidence

- [abstract] scDisInFact learns latent factors that disentangle condition effect from batch effect, enabling batch effect removal, condition-associated key gene detection, and perturbation prediction simultaneously.: [evidence span withheld — anchor status: not_found]
- [methods] Extract shared-bio factor and unshared-bio factors: "Extract shared-bio factor and unshared-bio factors. Shared-bio factor is removed of batch and condition effect."
- [readme] latent dimensions Ks correspond to shared and unshared factors per condition type: "shared-bio factor, unshared-bio factor for condition 1, unshared-bio factor for condition 2"
- [discussion] additivity assumption limiting modeling of higher-order effects: "scDisInFact assumes the condition and batch effects are additive to the cells’ original biological identity (shared-bio factors) in the latent space."
- [discussion] combined condition type workaround for correlated conditions: "the users can create a combined condition type (stim1_stim2, ctrl1_stim2, stim1_ctrl2, ctrl1_ctrl2) and train scDisInFact on the combined condition type instead."
