---
name: purity-score-evaluation
description: Use when comparing cluster assignments from an unsupervised single-cell
  method against known ground-truth cell-type labels (e.g., Splatter-simulated discrete
  types) and you need a label-accuracy metric such as purity to rank methods.
when_to_use_negative:
- The dataset has no trusted reference labels — purity and ARI are supervised metrics
  and cannot be computed without ground truth (use internal metrics like silhouette
  scores instead)
- You only have a single dataset rather than paired per-dataset results across multiple
  simulations — a Wilcoxon signed-rank comparison of methods is not applicable
- Clusters are expected to sub-divide a continuous trajectory rather than match discrete
  reference labels — purity against discrete types will not capture transition states
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: scikit-learn (sklearn)
  role: Provides sklearn.metrics.adjusted_rand_score for computing ARI on paired per-dataset
    label vectors during benchmark evaluation
- name: Splatter
  role: Generates the simulated scRNA-seq datasets with known ground-truth labels
    against which purity is computed
- name: Python
  role: Used for metric computation (adjusted_rand_score) and writing benchmark_comparison_results.csv
- name: R
  role: Runs SC3 and Seurat baselines on the same datasets so purity/ARI can be compared
    across methods
- name: forest-fire-clustering
  role: Method under evaluation; produces cluster_labels from FFC.preprocess and FFC.fit
    that are scored against reference labels
  repo: gersteinlab/forest-fire-clustering
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-022-31107-8
    title: Forest Fire Clustering for single-cell sequencing combines iterative label
      propagation with parallelized Monte Carlo simulations
  source_content_hash: sha256:5f7046724dbe3c94084244c1694a5f26f065cfd0fdd42ffaebd9e071772e920e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gersteinlab/forest-fire-clustering
  tools:
  - Python
  - R
  - Splatter
  - forest-fire-clustering
  - scikit-learn (sklearn)
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# purity-score-evaluation

## Summary

Benchmarking skill for scoring unsupervised single-cell clustering outputs (e.g., Forest Fire Clustering vs SC3/Seurat) against known reference labels using cluster purity, alongside adjusted Rand index, as described for the Splatter-simulated scRNA-seq benchmarks in the Forest Fire Clustering paper.

## When to use

Use when comparing cluster assignments from an unsupervised single-cell method against known ground-truth cell-type labels (e.g., Splatter-simulated discrete types) and you need a label-accuracy metric such as purity to rank methods.

## When NOT to use

- The dataset has no trusted reference labels — purity and ARI are supervised metrics and cannot be computed without ground truth (use internal metrics like silhouette scores instead)
- You only have a single dataset rather than paired per-dataset results across multiple simulations — a Wilcoxon signed-rank comparison of methods is not applicable
- Clusters are expected to sub-divide a continuous trajectory rather than match discrete reference labels — purity against discrete types will not capture transition states

## Inputs

- Ground-truth cell-type label vector per dataset (e.g., Splatter-simulated discrete cell-type labels)
- Predicted cluster label vector from each clustering method (Forest Fire Clustering, SC3, Seurat)
- Paired per-dataset metric vectors for statistical testing (ARI or purity, one value per simulated dataset)

## Outputs

- Per-dataset purity score per method (range 0–1)
- Per-dataset adjusted Rand index per method (sklearn.metrics.adjusted_rand_score)
- Wilcoxon signed-rank test statistics and P-values for FFC vs SC3 and FFC vs Seurat
- benchmark_comparison_results.csv containing ARI, purity, significance statistics, and runtime

## How to apply

For each simulated or labeled dataset, compare the cluster label of every cell to the reference ground-truth label and compute purity — the fraction of cells assigned to clusters whose dominant reference label matches — plus the adjusted Rand index via sklearn.metrics.adjusted_rand_score. Run this pairwise for every method under test (Forest Fire Clustering with FFC.preprocess + FFC.fit(fire_temp=100), SC3 and Seurat at default recommended settings) on the same inputs, e.g., the 20 Splatter datasets with 5 discrete cell types. Treat per-dataset purity and ARI values as paired samples across datasets and test method differences with a one-sided Wilcoxon signed-rank test (the paper reports P = 9.5 × 10−7 for FFC vs SC3 on ARI). Persist per-dataset ARI, purity, significance statistics, and runtime to a results table such as benchmark_comparison_results.csv.

## Related tools

- **scikit-learn (sklearn)** (Provides sklearn.metrics.adjusted_rand_score for computing ARI on paired per-dataset label vectors during benchmark evaluation)
- **Splatter** (Generates the simulated scRNA-seq datasets with known ground-truth labels against which purity is computed)
- **Python** (Used for metric computation (adjusted_rand_score) and writing benchmark_comparison_results.csv)
- **R** (Runs SC3 and Seurat baselines on the same datasets so purity/ARI can be compared across methods)
- **forest-fire-clustering** (Method under evaluation; produces cluster_labels from FFC.preprocess and FFC.fit that are scored against reference labels) — gersteinlab/forest-fire-clustering

## Examples

```
from sklearn.metrics import adjusted_rand_score; ari = adjusted_rand_score(ground_truth_labels, cluster_labels)
```

## Evaluation signals

- All purity scores fall in [0, 1], with values approaching 1.0 when each cluster's dominant reference label accounts for nearly all of its cells
- ARI values computed via sklearn.metrics.adjusted_rand_score are near 1 for near-perfect label recovery and near 0 (or negative) for random assignments
- Per-dataset ARI/purity values are paired across methods (same 40 simulated datasets) so a one-sided Wilcoxon signed-rank test is valid; expected result direction: FFC > SC3 in both ARI and purity
- Output CSV (benchmark_comparison_results.csv) contains one row per dataset per method with ARI, purity, and runtime columns fully populated
- Recomputing purity from the saved cluster labels and ground-truth labels reproduces the recorded values exactly

## Limitations

- Purity requires reference labels, so it only applies where ground truth exists (simulations or experimentally curated labels like the marker-gene-defined PBMC/CITE-seq types); purely exploratory clusterings cannot be scored this way
- Purity does not penalize over-clustering — splitting clusters can inflate purity without reflecting better biology; use ARI and silhouette scores alongside it
- Monte Carlo trials of the method under test vary with cluster count, so per-dataset runtimes and stability can differ between datasets and must be recorded per run

## Evidence

- [discussion] Purity as one of three standard unsupervised evaluation metrics: "There are three main ways to evaluate the performance of unsupervised methods. First, purity scores evaluate the cluster labels based on a set of reference labels."
- [results] Splatter benchmark workflow with ARI and Wilcoxon testing: [evidence span withheld — anchor status: not_found]
- [results] Benchmark outcome supporting correct application: [evidence span withheld — anchor status: not_found]
- [readme] Generating the cluster labels that purity is computed on: "cluster_obj.preprocess(X)
cluster_obj.fit(fire_temp=100)
cluster_obj.cluster_labels"
