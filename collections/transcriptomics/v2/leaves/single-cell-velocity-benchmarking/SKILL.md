---
name: single-cell-velocity-benchmarking
description: Use when you have velocity matrices from two or more RNA velocity methods
  plus per-cell cluster annotations and need a quantitative, dataset-wide comparison
  of velocity field quality via local neighbor consistency.
when_to_use_negative:
- You only have a single velocity method's output — there is no baseline to benchmark
  against and no relative comparison can be made.
- You want to evaluate velocity accuracy against known ground-truth trajectories or
  pseudotime rather than internal consistency of the velocity field.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0092
tools:
- name: scKINETICS
  role: Primary velocity method under evaluation; its EM-inferred velocities are compared
    against baselines for local consistency
  repo: http://github.com/dpeerlab/scKINETICS
- name: Jupyter
  role: Notebook environment used to run the benchmarking workflow and the scKINETICS
    demonstrations
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad267
    title: 'scKINETICS: inference of regulatory velocity with single-cell transcriptomics
      data'
  source_content_hash: sha256:0d378aebfad121cabd9ca8a7836c06908fa28c47f1945751026b9632d4644354
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Jupyter
  - scKINETICS
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# single-cell-velocity-benchmarking

## Summary

Benchmarks single-cell RNA velocity methods by quantifying local consistency of inferred velocity vectors: for each cell, the mean cosine distance between its velocity and those of its k=30 nearest neighbors is computed per method (e.g., scKINETICS vs. scVelo, VeloVAE, UniTVelo), with lower values indicating a smoother, more concordant velocity field.

## When to use

Use when you have velocity matrices from two or more RNA velocity methods plus per-cell cluster annotations and need a quantitative, dataset-wide comparison of velocity field quality via local neighbor consistency.

## When NOT to use

- You only have a single velocity method's output — there is no baseline to benchmark against and no relative comparison can be made.
- You want to evaluate velocity accuracy against known ground-truth trajectories or pseudotime rather than internal consistency of the velocity field.

## Inputs

- Per-method inferred velocity matrices (cells x genes) for scKINETICS, scVelo, VeloVAE, and UniTVelo
- Per-cell cluster annotations
- Cell-by-gene expression/neighbor graph inputs for scanpy neighbors computation (k=30)

## Outputs

- velocity_consistency_metrics.csv with per-method mean local inconsistency values (overall and per-cluster)
- Per-cell mean cosine distance to k=30 neighbor velocity vectors for each method

## How to apply

Load the inferred velocity matrices for each method together with per-cell cluster annotations, then build a k-nearest-neighbor graph with k=30 using scanpy's neighbors computation. For each cell, compute scikit-learn cosine_distances between its velocity vector and the velocity vectors of its 30 nearest neighbors, average per-cell distances, and aggregate within each cluster and across the whole dataset per method. Write per-method mean local inconsistency values to velocity_consistency_metrics.csv. The rationale, from the scKINETICS benchmark, is that a well-inferred velocity field should be locally concordant on the phenotypic manifold; scKINETICS achieved 0.0029 mean cosine distance versus scVelo 0.2419, VeloVAE 0.0449, and UniTVelo 0.0077.

## Related tools

- **scKINETICS** (Primary velocity method under evaluation; its EM-inferred velocities are compared against baselines for local consistency) — http://github.com/dpeerlab/scKINETICS
- **Jupyter** (Notebook environment used to run the benchmarking workflow and the scKINETICS demonstrations)

## Examples

```
import scanpy as sc; from sklearn.metrics.pairwise import cosine_distances; sc.pp.neighbors(adata, n_neighbors=30); d = cosine_distances(v, v[nbrs]).mean(1)
```

## Evaluation signals

- Computed mean local inconsistency per method reproduces reported values within numerical tolerance: scKINETICS 0.0029, scVelo 0.2419, VeloVAE 0.0449, UniTVelo 0.0077.
- scKINETICS yields the lowest mean cosine distance across all clusters (most locally consistent velocity field).
- velocity_consistency_metrics.csv contains one row per method with overall and per-cluster mean cosine distances.
- kNN graph used for evaluation is built with k=30 neighbors via scanpy's neighbors computation for every method consistently.
- Cosine distances are bounded in [0, 2]; any value outside this range indicates a computation error.

## Limitations

- Local cosine consistency is an internal consistency metric; it measures smoothness of the velocity field on the manifold, not correctness of inferred directions against ground truth.
- Results depend on the choice of k (k=30 here) and on the neighbor graph construction; different k or graph parameters change the metric values.
- Comparison requires velocity matrices from all methods on the same set of cells and genes with aligned cluster annotations.

## Evidence

- [abstract] scKINETICS successfully extends and improves existing velocity approaches to generate interpretable, mechanistic models of gene regulatory dynamics: "scKINETICS successfully extends and improves existing velocity approaches to generate interpretable, mechanistic models of gene regulatory dynamics"
- [readme] We introduce scKINETICS ... a dynamical model of gene expression change which is fit with the simultaneous learning of per-cell transcriptional velocities and a governing gene regulatory network: "We introduce scKINETICS (Key regulatory Interaction NETwork for Inferring Cell Speed), a dynamical model of gene expression change which is fit with the simultaneous learning"
- [abstract] All python code and an accompanying Jupyter notebook with demonstrations are available: "All python code and an accompanying Jupyter notebook with demonstrations are available"
- [intro] leveraging biologically-motivated priors from epigenetic data, gene-gene co-expression, and constraints on cells’ future states imposed by the phenotypic manifold: "leveraging biologically-motivated priors from epigenetic data, gene-gene co-expression, and constraints on cells’ future states imposed by the phenotypic manifold"
