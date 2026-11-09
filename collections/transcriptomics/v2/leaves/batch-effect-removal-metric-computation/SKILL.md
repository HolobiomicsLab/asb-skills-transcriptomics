---
name: batch-effect-removal-metric-computation
description: Use when comparing multi-batch multi-condition scRNA-seq integration
  methods on held-out <condition,batch> matrices and you need ARI/ASW-batch values
  on shared vs condition-specific latent factors to judge batch-removal quality.
when_to_use_negative:
- 'Each batch is measured under only one condition: the paper states there is not
  enough information to fully disentangle batch and condition effect, so metric comparison
  of shared vs unshared factors is unreliable'
- The goal is key-gene detection or perturbation prediction (predict_counts/extract_gene_scores),
  not batch-removal quality assessment
- Input is a single-batch dataset with no batch labels — ASW-batch cannot be computed
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scDisInFact
  role: Source of the shared-bio factor z_cs (mu_c) and unshared-bio factors z_ds
    (mu_d) that the ARI and ASW-batch metrics are computed on
  repo: https://github.com/ZhangLabGT/scDisInFact
- name: scINSIGHT
  role: Baseline that factorizes matrices into common and condition-specific modules
    via non-negative matrix factorization; its common factor supplies the comparison
    ARI/ASW-batch values
  repo: https://github.com/Vivianstats/scINSIGHT
- name: numpy
  role: Concatenation of z_cs and z_ds factor arrays across training count matrices
    before metric computation
- name: pandas
  role: Holds meta_cells with batch/condition columns used as silhouette labels and
    for assembling the metrics CSV
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-024-45227-w
    title: 'scDisInFact: disentangled learning for integration and prediction of multi-batch
      multi-condition single-cell RNA-sequencing data'
  source_content_hash: sha256:e2b67870ebb1d37a8babd9cc03d34cf012873d747643069816d1dbfc93d2dd73
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ZhangLabGT/scDisInFact
  tools:
  - numpy
  - pandas
  - scDisInFact
  - scINSIGHT
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7857142857142857
n_claims: 14
---

# batch-effect-removal-metric-computation

## Summary

Computes disentanglement quality metrics (ARI against ground-truth cell types, ASW-batch silhouette on batch labels) on scDisInFact's shared- and unshared-bio factors and on scINSIGHT's common factor, to quantify how well each method separates cell types while removing batch effect (Fig. 2d of the scDisInFact paper).

## When to use

Use when comparing multi-batch multi-condition scRNA-seq integration methods on held-out <condition,batch> matrices and you need ARI/ASW-batch values on shared vs condition-specific latent factors to judge batch-removal quality.

## When NOT to use

- Each batch is measured under only one condition: the paper states there is not enough information to fully disentangle batch and condition effect, so metric comparison of shared vs unshared factors is unreliable
- The goal is key-gene detection or perturbation prediction (predict_counts/extract_gene_scores), not batch-removal quality assessment
- Input is a single-batch dataset with no batch labels — ASW-batch cannot be computed

## Inputs

- Held-out multi-batch multi-condition count matrices (numpy.array, ncells x ngenes) with meta_cells dataframe containing condition 1, condition 2 and batch columns
- Shared-bio factor matrix z_cs extracted from scDisInFact model.inference (mu_c per training matrix, concatenated)
- Unshared-bio factor matrices z_ds_cond1 and z_ds_cond2 from scDisInFact mu_d
- Ground-truth cell-type labels and batch labels per cell
- scINSIGHT common-module factor matrix trained with the combined condition type (ctrl_severe, stim_healthy, ctrl_healthy, stim_severe)

## Outputs

- batch_removal_metrics_fig2d.csv with columns method, metric (ARI, ASW-batch), value, and rows for scDisInFact shared, scDisInFact unshared, scINSIGHT
- Optionally a grouped bar plot reproducing the Fig. 2d layout

## How to apply

Run scDisInFact on the held-out configuration (e.g. <ctrl,severe> batch-1 and <stim,healthy> batch-2 removed), then extract the shared-bio factor z_cs by looping over data_dict['datasets'], calling model.inference to get dict_inf['mu_c'] and concatenating across all training matrices; likewise collect z_ds_cond1/z_ds_cond2 from dict_inf['mu_d']. Cluster cells in the z_cs space with KNN-graph/Leiden in scanpy and compute adjusted_rand_score against ground-truth cell-type labels plus ASW-batch (silhouette on batch labels, rescaled from [-1,1] to [0,1]) with sklearn. Train the scINSIGHT baseline (non-negative matrix factorization into common and condition-specific modules) on the same held-out data using the Cartesian-product combined condition type (ctrl_severe, stim_healthy, ctrl_healthy, stim_severe) as the discussion recommends for correlated condition types, and compute the same ARI/ASW-batch metrics on its common factor. Assemble rows into a method/metric/value table saved as batch_removal_metrics_fig2d.csv; accept when scDisInFact shows higher ARI and ASW-batch on shared factors and higher ASW-batch on unshared factors than scINSIGHT (Fig. 2d).

## Related tools

- **scDisInFact** (Source of the shared-bio factor z_cs (mu_c) and unshared-bio factors z_ds (mu_d) that the ARI and ASW-batch metrics are computed on) — https://github.com/ZhangLabGT/scDisInFact
- **scINSIGHT** (Baseline that factorizes matrices into common and condition-specific modules via non-negative matrix factorization; its common factor supplies the comparison ARI/ASW-batch values) — https://github.com/Vivianstats/scINSIGHT
- **numpy** (Concatenation of z_cs and z_ds factor arrays across training count matrices before metric computation)
- **pandas** (Holds meta_cells with batch/condition columns used as silhouette labels and for assembling the metrics CSV)

## Examples

```
dict_inf = model.inference(counts=dataset.counts_norm.to(model.device), batch_ids=dataset.batch_id[:,None].to(model.device)); z_c = dict_inf['mu_c']
```

## Evaluation signals

- batch_removal_metrics_fig2d.csv exists and contains rows for scDisInFact shared, scDisInFact unshared, and scINSIGHT
- Every ARI and ASW-batch value lies in [0, 1] (silhouette range [-1, 1] rescaled to [0, 1])
- scDisInFact shared-bio factor ARI exceeds scINSIGHT's, indicating stronger cell-type separation under batch correction
- scDisInFact ASW-batch is higher than scINSIGHT's on both shared and unshared factors, indicating stronger batch mixing (Fig. 2d criterion)
- z_cs and z_ds arrays concatenate across all training matrices with matching ncells dimensions as meta_cells

## Limitations

- scDisInFact cannot fully disentangle batch and condition effect when each batch is measured under only one condition, weakening the interpretation of shared vs unshared factor metrics in that design
- The method assumes condition types are mutually independent, that condition effect is independent of batch effect, and that effects are additive in latent space, which constrains which held-out configurations are meaningful
- Findings are qualitative relative to scINSIGHT (stronger batch mixing with preserved ARI), not a fixed numeric threshold

## Evidence

- [abstract] scDisInFact learns latent factors that disentangle condition effect from batch effect, enabling it to simultaneously perform three tasks: batch effect removal: "scDisInFact learns latent factors that disentangle condition effect from batch effect, enabling it to simultaneously perform three tasks: batch effect removal"
- [methods] Extract shared-bio factor and unshared-bio factors: "Extract shared-bio factor and unshared-bio factors. Shared-bio factor is removed of batch and condition effect."
- [discussion] Combined condition type recommendation: "the users can create a combined condition type (stim1_stim2, ctrl1_stim2, stim1_ctrl2, ctrl1_ctrl2) and train scDisInFact on the combined condition type instead."
- [intro] scINSIGHT factorization baseline: "scINSIGHT14 factorizes the scRNA-seq matrices into common and condition-specific modules using non-negative matrix factorization"
- [readme] Demo runtime reference: "Please check [demo.ipynb](https://github.com/ZhangLabGT/scDisInFact/blob/main/demo.ipynb) and detailed function description for the usage of scDisInFact"
