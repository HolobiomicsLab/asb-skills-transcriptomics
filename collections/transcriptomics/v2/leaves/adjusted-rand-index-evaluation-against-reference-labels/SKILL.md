---
name: adjusted-rand-index-evaluation-against-reference-labels
description: Use when benchmarking a single-cell clustering method against reference
  cell-type or tissue labels and you need a chance-corrected agreement score (ARI)
  to compare methods or quantify gains from high-confidence label filtering.
when_to_use_negative:
- When no external reference labels exist for the dataset — ARI requires ground truth;
  use internal metrics (silhouette scores, label entropies) instead
- When comparing partitions with different numbers of cells (unfiltered vs. PEP-filtered
  subsets) without noting that filtered ARI is computed on a restricted subset, not
  a like-for-like comparison
- When only one cluster is present in the data, since Monte Carlo validation cannot
  estimate uncertainty and ARI against a multi-class reference is uninformative
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: scikit-learn (sklearn)
  role: Provides the Adjusted Rand Index implementation for comparing cluster labels
    to reference labels; also used to generate synthetic benchmark shapes
- name: forest-fire-clustering (FFC)
  role: Clustering method under evaluation; its cluster_labels and PEP-filtered labels
    are scored against ground truth
  repo: https://github.com/gersteinlab/forest-fire-clustering
- name: pegasus
  role: State-of-the-art baseline whose 'pg.infer_cell_types' cluster assignments
    are scored with ARI for comparison
- name: scanpy
  role: Preprocesses the MCA matrix with the 'zheng17' recipe (top 5000 HVGs, 30 PCs)
    so all methods are scored on identical inputs
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-022-31107-8
    title: Forest Fire Clustering for single-cell sequencing combines iterative label
      propagation with parallelized Monte Carlo simulations
  source_content_hash: sha256:23bea80d3c56f170bf269ad80e6399cecd19d8c7fbfa3f21f557b45e5834f92f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gersteinlab/forest-fire-clustering
  tools:
  - Python
  - forest-fire-clustering (FFC)
  - numba
  - numpy
  - pegasus
  - scanpy
  - scikit-learn (sklearn)
  - scipy
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# Adjusted Rand Index evaluation against reference labels

## Summary

Quantifies clustering quality by computing the Adjusted Rand Index (ARI) between inferred cluster labels and external ground-truth labels (e.g., tissue-of-origin for the Mouse Cell Atlas), alongside runtime and memory, to benchmark methods like Forest Fire Clustering against state-of-the-art single-cell clustering baselines.

## When to use

Use when benchmarking a single-cell clustering method against reference cell-type or tissue labels and you need a chance-corrected agreement score (ARI) to compare methods or quantify gains from high-confidence label filtering.

## When NOT to use

- When no external reference labels exist for the dataset — ARI requires ground truth; use internal metrics (silhouette scores, label entropies) instead
- When comparing partitions with different numbers of cells (unfiltered vs. PEP-filtered subsets) without noting that filtered ARI is computed on a restricted subset, not a like-for-like comparison
- When only one cluster is present in the data, since Monte Carlo validation cannot estimate uncertainty and ARI against a multi-class reference is uninformative

## Inputs

- Cluster label vector from each clustering method (e.g., FFC cluster_labels, pegasus cell-type calls, scanpy clusters)
- Ground-truth reference label vector (e.g., MCA tissue-of-origin labels)
- PEP-filtered label subset (cells with posterior significance PEP < 0.1 from cluster_obj.pval_list)
- Benchmark results table (e.g., mca_benchmark_results.csv) for recording ARI, runtime, and memory per method

## Outputs

- ARI value per method (unfiltered), e.g., FFC ~0.38 on MCA
- ARI value on the PEP<0.1 high-confidence subset, e.g., FFC ~0.72 on MCA
- Per-method wall-clock runtime and peak memory metrics
- mca_benchmark_results.csv summarizing ARI (unfiltered and filtered), runtime, and memory for FFC and all baselines

## How to apply

Run each clustering method (e.g., FFC via FFC().preprocess(X); cluster_obj.fit(fire_temp=100), plus baselines like pegasus 'pg.infer_cell_types' and scanpy-based clustering) on identically preprocessed data — for MCA, the scanpy 'zheng17' recipe with the top 5000 highly variable genes and first 30 principal components. Compute the Adjusted Rand Index between each method's cluster labels and tissue-of-origin ground-truth labels, and record per-method wall-clock runtime and peak memory. To measure label-confidence gains, call cluster_obj.validate() then cluster_obj.pval(), retain only cells with posterior significance PEP < 0.1 from pval_list, and recompute ARI on the filtered subset. ARI is chance-corrected, so it rewards reproducing the reference partition structure rather than raw label counts; report unfiltered and filtered ARI side-by-side in a results table (e.g., mca_benchmark_results.csv).

## Related tools

- **scikit-learn (sklearn)** (Provides the Adjusted Rand Index implementation for comparing cluster labels to reference labels; also used to generate synthetic benchmark shapes)
- **forest-fire-clustering (FFC)** (Clustering method under evaluation; its cluster_labels and PEP-filtered labels are scored against ground truth) — https://github.com/gersteinlab/forest-fire-clustering
- **pegasus** (State-of-the-art baseline whose 'pg.infer_cell_types' cluster assignments are scored with ARI for comparison)
- **scanpy** (Preprocesses the MCA matrix with the 'zheng17' recipe (top 5000 HVGs, 30 PCs) so all methods are scored on identical inputs)

## Examples

```
from sklearn.metrics import adjusted_rand_score; ari = adjusted_rand_score(tissue_of_origin_labels, cluster_obj.cluster_labels)
```

## Evaluation signals

- ARI values fall in the chance-corrected range (0 ≈ random agreement, 1 = perfect agreement with tissue-of-origin labels)
- FFC's ARI on MCA improves substantially from approximately 0.38 to approximately 0.72 when restricted to PEP < 0.1 high-confidence labels
- FFC's ARI and runtime are competitive with (or outperform) pegasus 'pg.infer_cell_types' and scanpy-based baselines
- All methods are scored on identical preprocessed input (zheng17 recipe, top 5000 highly variable genes, first 30 principal components) to ensure a fair benchmark
- Output table mca_benchmark_results.csv contains unfiltered ARI, PEP<0.1-filtered ARI, runtime, and memory for every method

## Limitations

- PEP-filtered ARI is computed on a smaller, high-confidence cell subset, so the 0.38 → 0.72 gain is not directly comparable to unfiltered baseline ARI
- ARI does not penalize differences in runtime or memory; those must be recorded separately for an efficiency benchmark
- Fire temperature c strongly influences the size and number of discovered clusters, so ARI results are sensitive to the fire_temp=100 choice

## Evidence

- [discussion] ARI via purity/ARI benchmarks: "There are three main ways to evaluate the performance of unsupervised methods. First, purity scores evaluate the cluster labels based on a set of reference labels."
- [discussion] MCA preprocessing parameters: "The gene expression count matrix for MCA was also preprocessed using the “zheng17” recipe in the scanpy package"
- [readme] FFC API for clustering and validation: "cluster_obj.preprocess(X)
cluster_obj.fit(fire_temp=100)
cluster_obj.cluster_labels"
- [readme] Posterior significance values for label filtering: "cluster_obj.pval_list # list of posterior significance values"
- [abstract] Monte Carlo internal validation rationale: "calculates a non-parametric posterior probability that each cell is assigned a cell-type label"
