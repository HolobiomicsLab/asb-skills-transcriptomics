---
name: cosine-distance-local-inconsistency-metric
description: Use when benchmarking RNA velocity methods on the same cell set and you
  need a quantitative, cluster-resolved measure of how well a method's velocity vectors
  agree with neighbors in the phenotypic manifold.
when_to_use_negative:
- You lack velocity vectors for multiple methods over the same cells — the metric
  is a comparative benchmark, not a stand-alone quality score with an absolute threshold.
- You want trajectory/pseudotime or downstream fate analysis rather than a velocity-field
  smoothness comparison across methods.
- Cell annotations or the underlying shared cell set are inconsistent across methods,
  which would confound per-cluster aggregation.
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3524
tools:
- name: scKINETICS
  role: Source of the reference velocity vectors being scored; its k=30 local cosine-distance
    average (0.0029) is compared against competing methods
  repo: github.com/dpeerlab/scKINETICS
- name: Jupyter
  role: Environment in which the metric computation and comparison against baselines
    are carried out
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad267
    title: 'scKINETICS: inference of regulatory velocity with single-cell transcriptomics
      data'
  source_content_hash: sha256:be27de71ca6026e36f7ed58e4ad3bcb0521bb830b629111a66a366abb9901dab
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Jupyter
  - scKINETICS
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3524
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# cosine-distance-local-inconsistency-metric

## Summary

Quantifies local consistency of single-cell RNA velocity fields by computing, for each cell, the mean cosine distance between its velocity vector and the velocity vectors of its k=30 nearest neighbors, then averaging per cluster and globally. Used in the scKINETICS benchmark to show scKINETICS produces the most locally concordant velocities versus scVelo, VeloVAE, and UniTVelo.

## When to use

Use when benchmarking RNA velocity methods on the same cell set and you need a quantitative, cluster-resolved measure of how well a method's velocity vectors agree with neighbors in the phenotypic manifold.

## When NOT to use

- You lack velocity vectors for multiple methods over the same cells — the metric is a comparative benchmark, not a stand-alone quality score with an absolute threshold.
- You want trajectory/pseudotime or downstream fate analysis rather than a velocity-field smoothness comparison across methods.
- Cell annotations or the underlying shared cell set are inconsistent across methods, which would confound per-cluster aggregation.

## Inputs

- per-method inferred velocity matrices (cells x genes) for scKINETICS, scVelo, VeloVAE, and UniTVelo over a shared cell set
- per-cell cluster annotations matching the velocity matrices
- scanpy AnnData object or expression representation used to compute the k=30 nearest-neighbor graph

## Outputs

- per-method mean local inconsistency values aggregated within each cluster and across the dataset
- per-cell mean cosine distance to the 30 nearest neighbors' velocity vectors
- velocity_consistency_metrics.csv containing the per-method summary statistics

## How to apply

Load the per-method velocity matrices (scKINETICS plus baselines such as scVelo, VeloVAE, UniTVelo) together with per-cell cluster annotations over the same cells. Build a k-nearest-neighbor graph with k=30 over the cells using scanpy's neighbors computation, so the neighborhood reflects the phenotypic manifold rather than the velocity field being scored. For each cell, compute scikit-learn cosine_distances between its velocity vector and the velocity vectors of its 30 nearest neighbors, average per cell, then aggregate within each cluster and across the whole dataset per method. Write per-method mean local inconsistency values (e.g. to velocity_consistency_metrics.csv) and compare methods; lower mean cosine distance indicates a more locally concordant velocity field. In the scKINETICS benchmark this reproduced average cosine distances of 0.0029 (scKINETICS), 0.0077 (UniTVelo), 0.0449 (VeloVAE), and 0.2419 (scVelo).

## Related tools

- **scKINETICS** (Source of the reference velocity vectors being scored; its k=30 local cosine-distance average (0.0029) is compared against competing methods) — github.com/dpeerlab/scKINETICS
- **Jupyter** (Environment in which the metric computation and comparison against baselines are carried out)

## Examples

```
import scanpy as sc; from sklearn.metrics.pairwise import cosine_distances; sc.pp.neighbors(adata, n_neighbors=30); D = cosine_distances(V, V[adata.uns['neighbors']['indices']])
```

## Evaluation signals

- The computed dataset-wide mean cosine distances reproduce the reported values within numerical tolerance: scKINETICS 0.0029, scVelo 0.2419, VeloVAE 0.0449, UniTVelo 0.0077.
- scKINETICS ranks lowest (most locally concordant) in every cluster, matching the reported finding that competing algorithms generated more locally inconsistent velocity vectors in all clusters.
- Per-cell cosine-distance arrays have the same cell ordering as the velocity matrices, and the k=30 neighbor graph is identical across methods being compared.
- Output CSV contains one mean local inconsistency value per method and per-cluster aggregates, with all methods evaluated on the same cells.

## Limitations

- The metric measures only local (neighborhood-level) velocity concordance on the phenotypic manifold; it does not by itself validate biological correctness of directions or GRN inference.
- Reported reference values (0.0029–0.2419) are tied to the specific dataset and the k=30 choice; other datasets or neighborhood sizes will yield different magnitudes.
- No changelog found for the scKINETICS repository, so API changes in the velocity output format may not be traceable.

## Evidence

- [results] average cosine distances of 0.2419 (scVelo), 0.0449 (VeloVAE), and 0.0077 (UniTVelo), versus 0.0029 for scKINETICS: [evidence span withheld — anchor status: not_found]
- [results] competing algorithms generated more locally inconsistent velocity vectors in all clusters: [evidence span withheld — anchor status: not_found]
- [abstract] simultaneous learning of per-cell transcriptional velocities and a governing gene regulatory network: "a dynamical model of gene expression change which is fit with the simultaneous learning of per-cell transcriptional velocities and a governing gene regulatory network"
- [readme] demo covers the package's main functionalities including input generation and visualization: [evidence span withheld — anchor status: external_doc]
