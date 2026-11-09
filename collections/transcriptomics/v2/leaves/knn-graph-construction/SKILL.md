---
name: knn-graph-construction
description: Use when comparing velocity inference methods (e.g., scKINETICS, scVelo,
  VeloVAE, UniTVelo) on local smoothness of their velocity fields, or when a kNN neighborhood
  structure over cells is needed for neighborhood-aggregated metrics.
when_to_use_negative:
- The task is to infer velocities from spliced/unspliced counts rather than to evaluate
  already-inferred velocity fields.
- The comparison target is a global trajectory metric rather than neighborhood-level
  velocity concordance among cells.
edam_operation: http://edamontology.org/operation_3898
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scKINETICS
  role: Source of the inferred velocity vectors whose local consistency is evaluated
    on the kNN graph
  repo: https://github.com/dpeerlab/scKINETICS
- name: Jupyter
  role: Notebook environment in which the scanpy neighbor graph and cosine-distance
    consistency computations are run
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad267
    title: 'scKINETICS: inference of regulatory velocity with single-cell transcriptomics
      data'
  source_content_hash: sha256:63904d82a6f2bf7d286885c9fcf28bd1f775a3b0526cf5ff15e7b5dc96762f1f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/dpeerlab/scKINETICS
  tools:
  - Jupyter
  - scKINETICS
  edam_operation: http://edamontology.org/operation_3898
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 3
---

# knn-graph-construction

## Summary

Construct a k-nearest-neighbor graph (k=30) over single-cell transcriptomic embeddings with scanpy, then score each cell's velocity local inconsistency as the mean cosine distance between its velocity vector and those of its 30 nearest neighbors.

## When to use

Use when comparing velocity inference methods (e.g., scKINETICS, scVelo, VeloVAE, UniTVelo) on local smoothness of their velocity fields, or when a kNN neighborhood structure over cells is needed for neighborhood-aggregated metrics.

## When NOT to use

- The task is to infer velocities from spliced/unspliced counts rather than to evaluate already-inferred velocity fields.
- The comparison target is a global trajectory metric rather than neighborhood-level velocity concordance among cells.

## Inputs

- Per-cell velocity matrices for each method (scKINETICS, scVelo, VeloVAE, UniTVelo)
- Per-cell cluster annotations
- Scanpy AnnData object with cell embeddings for neighbor computation

## Outputs

- velocity_consistency_metrics.csv with per-method (and per-cluster) mean local inconsistency values
- k-nearest-neighbor graph (k=30) over cells

## How to apply

Load the per-cell velocity matrices for each method together with per-cell cluster annotations. Build the kNN graph with scanpy's neighbors computation using k=30 over the cells. For each cell, compute cosine distances between its velocity vector and the velocity vectors of its 30 nearest neighbors using scikit-learn's cosine_distances, average the per-cell distances, and aggregate within each cluster and across the dataset per method. Write the per-method mean local inconsistency values to velocity_consistency_metrics.csv and compare against reported benchmarks (scKINETICS 0.0029, scVelo 0.2419, VeloVAE 0.0449, UniTVelo 0.0077) within numerical tolerance.

## Related tools

- **scKINETICS** (Source of the inferred velocity vectors whose local consistency is evaluated on the kNN graph) — https://github.com/dpeerlab/scKINETICS
- **Jupyter** (Notebook environment in which the scanpy neighbor graph and cosine-distance consistency computations are run)

## Examples

```
import scanpy as sc; from sklearn.metrics.pairwise import cosine_distances; sc.pp.neighbors(adata, n_neighbors=30); v = adata.obsm['velocity']; d = cosine_distances(v, v[adata.obsp['connectivities'].nonzero()[1]])
```

## Evaluation signals

- Reproduced mean cosine distances match reported values: scKINETICS 0.0029, scVelo 0.2419, VeloVAE 0.0449, UniTVelo 0.0077, within numerical tolerance.
- velocity_consistency_metrics.csv exists and contains one mean local inconsistency value per method.
- kNN graph uses exactly k=30 neighbors per cell as computed by scanpy's neighbors computation.
- Per-cluster aggregates are consistent with the dataset-wide mean for each method.

## Limitations

- Local inconsistency depends on the k parameter; values are only comparable when all methods are evaluated on the same k=30 graph.
- Cosine distance to neighbors conflates velocity field smoothness with neighborhood geometry, so results should be interpreted relative to the shared embedding.

## Evidence

- [other] Do scKINETICS-derived velocity vectors show lower local inconsistency among neighboring cells (mean cosine distance to k=30 neighbors) than competing velocity methods: [evidence span withheld — anchor status: not_found]
- [other] Build a k-nearest-neighbor graph with k=30 over the cells using scanpy's neighbors computation.: [evidence span withheld — anchor status: not_found]
- [other] the computed averages should reproduce the reported values (scKINETICS 0.0029, scVelo 0.2419, VeloVAE 0.0449, UniTVelo 0.0077) within numerical tolerance: [evidence span withheld — anchor status: not_found]
- [abstract] scKINETICS successfully extends and improves existing velocity approaches: "scKINETICS successfully extends and improves existing velocity approaches to generate interpretable, mechanistic models of gene regulatory dynamics"
