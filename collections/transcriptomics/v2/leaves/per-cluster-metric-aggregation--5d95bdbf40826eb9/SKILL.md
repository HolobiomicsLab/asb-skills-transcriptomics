---
name: per-cluster-metric-aggregation
description: Use when a per-cell metric must be summarized per annotated cluster and
  globally to compare methods, e.g. local velocity inconsistency via mean cosine distance
  to k=30 neighbors across velocity methods.
when_to_use_negative:
- Cluster annotations are missing or unreliable, since per-cluster aggregation would
  inherit annotation errors
- The metric being aggregated is not per-cell (e.g., dataset-level summary statistics),
  making cluster-wise averaging meaningless
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0091
tools:
- name: scKINETICS
  role: Source of the velocity vectors whose local consistency is being aggregated
    and compared against baselines
  repo: http://github.com/dpeerlab/scKINETICS
- name: Jupyter
  role: Notebook environment in which the neighbor graph, cosine distances, and per-cluster
    aggregation are computed
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad267
    title: 'scKINETICS: inference of regulatory velocity with single-cell transcriptomics
      data'
  source_content_hash: sha256:0115335d38ee95db7a1cbb2a29d0789ca11f065b2604dd6d01cf1d0ce20ab4f3
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Jupyter
  - scKINETICS
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# per-cluster-metric-aggregation

## Summary

Aggregates a per-cell quality metric (e.g., cosine distance between a cell's velocity vector and its k=30 neighbors' velocities) within annotated clusters and across the dataset, enabling method-level comparison of velocity consistency for scKINETICS versus baselines like scVelo, VeloVAE, and UniTVelo.

## When to use

Use when a per-cell metric must be summarized per annotated cluster and globally to compare methods, e.g. local velocity inconsistency via mean cosine distance to k=30 neighbors across velocity methods.

## When NOT to use

- Cluster annotations are missing or unreliable, since per-cluster aggregation would inherit annotation errors
- The metric being aggregated is not per-cell (e.g., dataset-level summary statistics), making cluster-wise averaging meaningless

## Inputs

- Velocity matrices for each method (scKINETICS, scVelo, VeloVAE, UniTVelo)
- Per-cell cluster annotations
- Neighbor graph specification (k=30)

## Outputs

- Per-cell mean cosine distance to k=30 neighbors' velocities
- Per-cluster and dataset-wide mean local inconsistency per method
- velocity_consistency_metrics.csv with per-method mean local inconsistency values

## How to apply

First build a k-nearest-neighbor graph with k=30 over the cells (e.g., with scanpy's neighbors computation) on the expression embedding shared by all methods. For each cell, compute the cosine distance between its velocity vector and the velocity vectors of its 30 nearest neighbors using scikit-learn's cosine_distances. Average the per-cell cosine distances, then aggregate within each cluster annotation and across the whole dataset for each velocity method. Write the per-method mean local inconsistency values to a CSV (velocity_consistency_metrics.csv). The metric is meaningful because scKINETICS, which infers velocities jointly with a gene regulatory network under manifold constraints, is expected to produce a more concordant velocity field than methods learned without those priors.

## Related tools

- **scKINETICS** (Source of the velocity vectors whose local consistency is being aggregated and compared against baselines) — http://github.com/dpeerlab/scKINETICS
- **Jupyter** (Notebook environment in which the neighbor graph, cosine distances, and per-cluster aggregation are computed)

## Examples

```
sc.pp.neighbors(adata, n_neighbors=30); dist = cosine_distances(V, V[adata.obs['knn30_idx']]); adata.obs['local_inconsistency'] = dist.mean(1); adata.obs.groupby('cluster')['local_inconsistency'].mean().to_csv('velocity_consistency_metrics.csv')
```

## Evaluation signals

- Computed dataset-wide means reproduce the reported values within numerical tolerance: scKINETICS 0.0029, scVelo 0.2419, VeloVAE 0.0449, UniTVelo 0.0077
- scKINETICS shows the lowest mean cosine distance in every cluster, confirming the most locally consistent velocity field
- Per-cluster means average consistently to the dataset-wide mean when weighted by cluster cell counts
- velocity_consistency_metrics.csv contains one row per method with finite, non-negative cosine distance values

## Limitations

- Comparative averages (0.0029 vs 0.2419/0.0449/0.0077) are specific to this dataset and k=30 neighbor choice; values need not transfer to other datasets
- Aggregation depends on the quality of velocity inference and cluster annotations upstream; noisy clusters can obscure per-cluster differences

## Evidence

- [abstract] local inconsistency metric: [evidence span withheld — anchor status: not_found]
- [other] k=30 neighbor cosine distance: [evidence span withheld — anchor status: not_found]
- [other] cosine_distances computation: [evidence span withheld — anchor status: not_found]
- [readme] manifold-constrained velocity inference: "leveraging biologically-motivated priors from epigenetic data, gene-gene co-expression, and constraints on cells’ future states imposed by the phenotypic manifold"
