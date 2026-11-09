---
name: velocity-vector-nearest-neighbor-analysis
description: Use when comparing velocity inference methods (e.g. scKINETICS, scVelo,
  VeloVAE, UniTVelo) on the same single-cell dataset and you need a quantitative,
  cluster-resolved measure of velocity-field smoothness to show one method's vectors
  are more locally concordant.
when_to_use_negative:
- You have only one velocity method and no baselines to compare — the metric is a
  relative benchmarking measure, not an absolute quality certificate.
- The goal is gene regulatory network inference or regulator identification (scKINETICS's
  GRN/EM output) rather than velocity-field consistency assessment.
- Cluster annotations are unavailable or unreliable, since cluster-level aggregation
  is a core part of the reported analysis.
edam_operation: http://edamontology.org/operation_2424
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scKINETICS
  role: Source of the velocity matrices under evaluation; its velocities are benchmarked
    against baselines using local cosine distance to k=30 neighbors
  repo: http://github.com/dpeerlab/scKINETICS
- name: Jupyter
  role: Notebook environment in which the kNN graph construction, cosine distance
    computation, and aggregation are run
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad267
    title: 'scKINETICS: inference of regulatory velocity with single-cell transcriptomics
      data'
  source_content_hash: sha256:19101fb044f9609705d543cbb9146d0f3babb6e3a63611f27c7fbdfbca1e5f49
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Jupyter
  - scKINETICS
  edam_operation: http://edamontology.org/operation_2424
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# velocity-vector-nearest-neighbor-analysis

## Summary

Quantifies local consistency of inferred single-cell RNA velocity vectors by measuring mean cosine distance between each cell's velocity and those of its k=30 nearest neighbors, per method and per cluster. This benchmarking metric exposed scKINETICS's locally concordant velocity field relative to scVelo, VeloVAE, and UniTVelo.

## When to use

Use when comparing velocity inference methods (e.g. scKINETICS, scVelo, VeloVAE, UniTVelo) on the same single-cell dataset and you need a quantitative, cluster-resolved measure of velocity-field smoothness to show one method's vectors are more locally concordant.

## When NOT to use

- You have only one velocity method and no baselines to compare — the metric is a relative benchmarking measure, not an absolute quality certificate.
- The goal is gene regulatory network inference or regulator identification (scKINETICS's GRN/EM output) rather than velocity-field consistency assessment.
- Cluster annotations are unavailable or unreliable, since cluster-level aggregation is a core part of the reported analysis.

## Inputs

- Inferred per-cell velocity matrix for each method being compared (scKINETICS, scVelo, VeloVAE, UniTVelo)
- Per-cell cluster annotations matching the velocity matrices
- Processed single-cell expression data (e.g. AnnData object) from which the neighbor graph is computed

## Outputs

- Per-method mean local inconsistency values (velocity_consistency_metrics.csv), aggregated per cluster and dataset-wide
- Cluster-level comparison showing which method yields the most locally consistent velocity field

## How to apply

Load the inferred velocity matrices for each method (scKINETICS plus baselines) together with per-cell cluster annotations. Build a k-nearest-neighbor graph with k=30 over the cells using scanpy's neighbors computation. For each cell, compute the cosine distance between its velocity vector and the velocity vectors of its 30 nearest neighbors with scikit-learn's cosine_distances, then average per-cell distances and aggregate within each cluster and across the whole dataset per method. Write the per-method mean local inconsistency values to velocity_consistency_metrics.csv. The rationale is that a biologically coherent velocity field should assign similar directions to neighboring cells on the phenotypic manifold, so lower mean cosine distance indicates a more consistent, trustworthy velocity estimate.

## Related tools

- **scKINETICS** (Source of the velocity matrices under evaluation; its velocities are benchmarked against baselines using local cosine distance to k=30 neighbors) — http://github.com/dpeerlab/scKINETICS
- **Jupyter** (Notebook environment in which the kNN graph construction, cosine distance computation, and aggregation are run)

## Examples

```
import scanpy as sc; from sklearn.metrics.pairwise import cosine_distances; sc.pp.neighbors(adata, n_neighbors=30); d = cosine_distances(v, v[adata.obsp['connectivities'].indices]).mean()
```

## Evaluation signals

- Computed averages reproduce the reported values within numerical tolerance: scKINETICS 0.0029, scVelo 0.2419, VeloVAE 0.0449, UniTVelo 0.0077.
- scKINETICS yields the lowest mean cosine distance (0.0029), i.e. the most locally consistent velocity field among the compared methods.
- The k-nearest-neighbor graph is built with exactly k=30 neighbors via scanpy's neighbors computation.
- Output file velocity_consistency_metrics.csv contains per-method and per-cluster aggregated mean cosine distances for all four methods.

## Limitations

- Reported baselines show large spread (scVelo 0.2419 vs UniTVelo 0.0077), so the metric is sensitive to the competing methods' characteristics; results are specific to the benchmarked dataset (acute pancreatitis) and cluster definitions.
- The metric evaluates local concordance only — it does not by itself validate biological correctness of directions or the inferred gene regulatory network.

## Evidence

- [results] reporting scKINETICS velocity concordance finding: [evidence span withheld — anchor status: not_found]
- [abstract] dynamical model description: "a dynamical model of gene expression change which is fit with the simultaneous learning of per-cell transcriptional velocities and a governing gene regulatory network"
- [intro] phenotypic manifold constraint rationale: "constraints on cells’ future states imposed by the phenotypic manifold"
- [readme] tool availability: "All python code and an accompanying Jupyter notebook with demonstrations are available"
