---
name: scinsight-cartesian-combined-condition-baseline
description: Use when benchmarking multi-batch multi-condition scRNA-seq disentanglement
  against scINSIGHT, i.e. when a common/condition-specific NMF decomposition must
  be trained on the same held-out simulated configuration as scDisInFact and scored
  with ARI and ASW-batch.
when_to_use_negative:
- When each batch is measured under only one condition — the paper states there is
  not enough information to fully disentangle batch and condition effect, so a combined-condition
  baseline comparison is uninformative
- When the goal is perturbation prediction or condition-associated key gene (CKG)
  detection rather than disentanglement benchmarking — scINSIGHT does not provide
  predict_counts or gene_score equivalents
- When the input is already a latent factor/feature table that does not match the
  held-out simulated configuration, since ARI/ASW-batch must be computed on the exact
  same batch-condition subsets as scDisInFact
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scINSIGHT
  role: NMF baseline factorizing matrices into common and condition-specific modules
    on the Cartesian combined condition type; its common factor is scored with ARI/ASW-batch
  repo: https://github.com/Vivianstats/scINSIGHT
- name: scDisInFact
  role: Method under benchmark; its shared-bio (z_cs) and unshared-bio (z_ds) factors
    are clustered and scored with the same ARI/ASW-batch metrics
  repo: https://github.com/ZhangLabGT/scDisInFact
- name: numpy
  role: Holds the (ncells, ngenes) count arrays and concatenated z_cs / z_ds factor
    matrices used for clustering and metric computation
- name: pandas
  role: Carries meta_cells with batch and condition columns and assembles the method/metric/value
    comparison table into CSV
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-024-45227-w
    title: 'scDisInFact: disentangled learning for integration and prediction of multi-batch
      multi-condition single-cell RNA-sequencing data'
  source_content_hash: sha256:517aaf10432faf2b613601b73f9b552f9e4da1c0bbe7c34daae1477137245ab9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Vivianstats/scINSIGHT
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

# scINSIGHT Cartesian combined-condition baseline

## Summary

Trains the scINSIGHT non-negative matrix factorization baseline on a Cartesian-product combined condition type (e.g. ctrl_severe, stim_healthy, ctrl_healthy, stim_severe) of the simulated disentanglement test, then computes ARI and ASW-batch on its common factor to benchmark scDisInFact's shared- and unshared-bio factor separation (Fig. 2d).

## When to use

Use when benchmarking multi-batch multi-condition scRNA-seq disentanglement against scINSIGHT, i.e. when a common/condition-specific NMF decomposition must be trained on the same held-out simulated configuration as scDisInFact and scored with ARI and ASW-batch.

## When NOT to use

- When each batch is measured under only one condition — the paper states there is not enough information to fully disentangle batch and condition effect, so a combined-condition baseline comparison is uninformative
- When the goal is perturbation prediction or condition-associated key gene (CKG) detection rather than disentanglement benchmarking — scINSIGHT does not provide predict_counts or gene_score equivalents
- When the input is already a latent factor/feature table that does not match the held-out simulated configuration, since ARI/ASW-batch must be computed on the exact same batch-condition subsets as scDisInFact

## Inputs

- Simulated scRNA-seq count matrices for the disentanglement test config: <ctrl,severe> batch-1 and <stim,healthy> batch-2 matrices held out
- Ground-truth cell-type labels per cell
- Batch labels per cell
- scDisInFact shared-bio factor matrix z_cs and unshared-bio factor matrices z_ds_cond1/z_ds_cond2 (numpy arrays of shape (ncells, ndims))

## Outputs

- batch_removal_metrics_fig2d.csv with columns method, metric, value and rows for scDisInFact shared, scDisInFact unshared, and scINSIGHT
- Optionally a grouped bar plot reproducing the Fig. 2d layout

## How to apply

Run scINSIGHT (R/Bioconductor, installable from CRAN) on the simulated matrices held out from the disentanglement test — batch 1 minus <ctrl,severe> and batch 2 minus <stim,healthy> — encoding the two condition types as a Cartesian-product combined condition (ctrl_severe, stim_healthy, ctrl_healthy, stim_severe), mirroring scDisInFact's guidance to train on a combined condition type for correlated conditions. scINSIGHT factorizes the expression matrices into common and condition-specific modules via non-negative matrix factorization. Cluster cells in scINSIGHT's common factor space (KNN-graph/Leiden in scanpy) and compute ARI against ground-truth cell-type labels plus ASW-batch (silhouette width on batch labels, rescaled to [0,1]). For scDisInFact, cluster cells in the shared-bio factor matrix z_cs with the same procedure and compute ASW-batch on the unshared-bio factor matrices z_ds_cond1/z_ds_cond2. Assemble all metric rows into batch_removal_metrics_fig2d.csv with columns method, metric, value; the acceptance criterion is scDisInFact showing stronger batch mixing on shared factors while preserving cell-type ARI relative to scINSIGHT.

## Related tools

- **scINSIGHT** (NMF baseline factorizing matrices into common and condition-specific modules on the Cartesian combined condition type; its common factor is scored with ARI/ASW-batch) — https://github.com/Vivianstats/scINSIGHT
- **scDisInFact** (Method under benchmark; its shared-bio (z_cs) and unshared-bio (z_ds) factors are clustered and scored with the same ARI/ASW-batch metrics) — https://github.com/ZhangLabGT/scDisInFact
- **numpy** (Holds the (ncells, ngenes) count arrays and concatenated z_cs / z_ds factor matrices used for clustering and metric computation)
- **pandas** (Carries meta_cells with batch and condition columns and assembles the method/metric/value comparison table into CSV)

## Examples

```
install.packages("scINSIGHT"); library(scINSIGHT); res <- scINSIGHT(expr = sim_counts, conditions = c("ctrl_severe","stim_healthy","ctrl_healthy","stim_severe"), K = 5)
```

## Evaluation signals

- batch_removal_metrics_fig2d.csv exists and contains rows for all three entries: scDisInFact shared, scDisInFact unshared, and scINSIGHT
- Every ARI and ASW-batch value lies in [0, 1] (silhouette range [-1, 1] rescaled to [0, 1] via the standard transformation)
- scDisInFact's shared-bio factors show higher ASW-batch (stronger batch mixing) than scINSIGHT's common factor while achieving higher or comparable cell-type ARI
- scDisInFact's unshared-bio factors achieve a higher ASW-batch score than scINSIGHT, confirming batch effect removal in condition-specific factors
- scINSIGHT was trained on the same held-out configuration (batch 1 minus <ctrl,severe>, batch 2 minus <stim,healthy>) using the Cartesian-product combined condition type, not the raw two-condition formulation

## Limitations

- When each batch of cells is measured under only one condition, there is not enough information to fully disentangle the batch and condition effect
- scINSIGHT provides common and condition-specific modules but no direct analogue of scDisInFact's per-condition-type unshared-bio factor extraction or combined-condition joint model, limiting metric symmetry in the comparison
- Batch effect removal quality depends on the held-out configuration; results reflect the simulated disentanglement test setup of the manuscript and may not transfer directly to real datasets

## Evidence

- [intro] scINSIGHT factorizes the scRNA-seq matrices into common and condition-specific modules using non-negative matrix factorization: "scINSIGHT14 factorizes the scRNA-seq matrices into common and condition-specific modules using non-negative matrix factorization"
- [discussion] Combined condition type guidance for correlated condition types: "the users can create a combined condition type (stim1_stim2, ctrl1_stim2, stim1_ctrl2, ctrl1_ctrl2) and train scDisInFact on the combined condition type instead."
- [readme] scINSIGHT jointly analyzes multi-sample heterogeneous data: [evidence span withheld — anchor status: external_doc]
- [readme] scINSIGHT installation from CRAN: "You can install `scINSIGHT` from
[CRAN](https://cran.r-project.org/web/packages/scINSIGHT/index.html)"
- [discussion] Single-condition-per-batch disentanglement limitation: "when each batch of cells is measured under only one condition, there is not enough information to fully disentangle the batch and condition effect."
