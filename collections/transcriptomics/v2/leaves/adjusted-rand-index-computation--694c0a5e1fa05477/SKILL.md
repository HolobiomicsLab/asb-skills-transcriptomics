---
name: adjusted-rand-index-computation
description: Use when comparing clustering methods on single-cell data where a ground-truth
  label vector exists (e.g., Splatter simulations with known cell types), and you
  need a chance-corrected agreement metric like ARI (often alongside purity) for downstream
  paired tests.
when_to_use_negative:
- No reference/ground-truth labels are available — use internal validation instead
  (e.g., FFC's Monte Carlo posterior validation, cluster_obj.validate(), silhouette
  scores)
- Comparing clusterings with unequal cell sets or unmatched cells — adjusted_rand_score
  requires label vectors aligned over the same cells
- The data is continuous trajectory data where cells are ordered along a path rather
  than partitioned into discrete types — ARI against discrete labels is not meaningful
  for such data
edam_operation: http://edamontology.org/operation_0564
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0091
tools:
- name: scikit-learn (sklearn)
  role: Provides sklearn.metrics.adjusted_rand_score used to compute the ARI between
    predicted and ground-truth cluster labels
  repo: https://github.com/gersteinlab/forest-fire-clustering
- name: Splatter
  role: R package used to simulate discrete cell-type scRNA-seq datasets that supply
    the ground-truth labels against which ARI is computed
- name: Python
  role: Language in which ARI computation and Wilcoxon signed-rank testing are executed
    alongside the FFC pipeline
  repo: https://github.com/gersteinlab/forest-fire-clustering
- name: R
  role: Used to run the SC3 and Seurat baseline clusterings whose labels are evaluated
    by ARI
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-022-31107-8
    title: Forest Fire Clustering for single-cell sequencing combines iterative label
      propagation with parallelized Monte Carlo simulations
  source_content_hash: sha256:99c2612ca726ed59a55ccd7343eab851b4c53dbb244e7dd3d9c039ba9d32aeb9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gersteinlab/forest-fire-clustering
  tools:
  - Python
  - R
  - Splatter
  - scikit-learn (sklearn)
  edam_operation: http://edamontology.org/operation_0564
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# adjusted-rand-index-computation

## Summary

Computes the adjusted Rand index (ARI) to quantify agreement between predicted cluster labels (e.g., Forest Fire Clustering, SC3, Seurat) and ground-truth cell-type labels on scRNA-seq benchmarks such as Splatter-simulated discrete cell-type datasets, enabling paired statistical comparison of clustering methods.

## When to use

Use when comparing clustering methods on single-cell data where a ground-truth label vector exists (e.g., Splatter simulations with known cell types), and you need a chance-corrected agreement metric like ARI (often alongside purity) for downstream paired tests.

## When NOT to use

- No reference/ground-truth labels are available — use internal validation instead (e.g., FFC's Monte Carlo posterior validation, cluster_obj.validate(), silhouette scores)
- Comparing clusterings with unequal cell sets or unmatched cells — adjusted_rand_score requires label vectors aligned over the same cells
- The data is continuous trajectory data where cells are ordered along a path rather than partitioned into discrete types — ARI against discrete labels is not meaningful for such data

## Inputs

- Predicted cluster label vector for each cell (from FFC, SC3, Seurat, or another clustering method)
- Ground-truth cell-type label vector (e.g., Splatter-simulated discrete cell-type annotations)
- Multiple simulated or experimental scRNA-seq datasets with known labels (e.g., 20 Splatter datasets with 5 cell types)

## Outputs

- Per-dataset ARI value for each clustering method
- Paired per-dataset ARI vectors usable for one-sided Wilcoxon signed-rank tests
- Summary table rows (e.g., ARI, purity, significance statistics, runtime) written to benchmark_comparison_results.csv

## How to apply

After clustering each dataset (e.g., FFC via FFC.preprocess followed by FFC.fit with fire_temp=100, and baselines SC3/Seurat with default recommended settings), collect the predicted label vector and the known ground-truth labels for every cell. Compute ARI per dataset using sklearn.metrics.adjusted_rand_score, and compute cluster purity for every method against the known Splatter ground-truth labels to complement ARI. Because ARI is computed per dataset over multiple simulated replicates (e.g., 20 discrete datasets with 5 cell types), the paired per-dataset ARI values can be fed into a one-sided Wilcoxon signed-rank test to assess whether one method significantly outperforms another (in the source benchmark, FFC vs SC3 gave P = 9.5 × 10−7). Record results alongside purity and runtime in an output table such as benchmark_comparison_results.csv. ARI is chance-corrected, so 0 indicates label agreement expected at random and 1 indicates perfect recovery of the ground-truth partition.

## Related tools

- **scikit-learn (sklearn)** (Provides sklearn.metrics.adjusted_rand_score used to compute the ARI between predicted and ground-truth cluster labels) — https://github.com/gersteinlab/forest-fire-clustering
- **Splatter** (R package used to simulate discrete cell-type scRNA-seq datasets that supply the ground-truth labels against which ARI is computed)
- **Python** (Language in which ARI computation and Wilcoxon signed-rank testing are executed alongside the FFC pipeline) — https://github.com/gersteinlab/forest-fire-clustering
- **R** (Used to run the SC3 and Seurat baseline clusterings whose labels are evaluated by ARI)

## Examples

```
from sklearn.metrics import adjusted_rand_score; ari = adjusted_rand_score(true_labels, cluster_obj.cluster_labels)
```

## Evaluation signals

- ARI is chance-corrected: values should fall roughly in [0, 1], with 0 indicating random-level agreement and 1 perfect recovery of the Splatter ground-truth partition
- Per-dataset ARI vectors for the compared methods must be paired by dataset (same 20 simulated datasets) before running the one-sided Wilcoxon signed-rank test
- ARI should be reported alongside cluster purity; the two metrics should rank methods consistently if labels and clusters are sane (in the source benchmark FFC beat SC3 on both)
- Results table (benchmark_comparison_results.csv) must contain ARI, purity, significance statistics, and runtime per method per dataset
- Significance output should match the published benchmark magnitude (Wilcoxon signed-rank P = 9.5 × 10−7 for FFC vs SC3) when reproducing the same simulation setup

## Limitations

- ARI against discrete ground-truth labels is only appropriate for discrete cell-type simulations; the same benchmark's 20 continuous trajectory datasets with 4 differentiation paths require complementary metrics (e.g., label entropies along developmental trajectories) rather than ARI alone
- ARI complements but does not replace purity or silhouette scores; the article notes three main evaluation modes — purity scores, ARI, and silhouette scores
- ARI evaluates the point-estimate clustering only and ignores the label confidence/posterior probability information that FFC provides via Monte Carlo validation

## Evidence

- [other] Compute the adjusted Rand index (sklearn.metrics.adjusted_rand_score) and cluster purity for every method against the known Splatter ground-truth labels.: [evidence span withheld — anchor status: not_found]
- [other] Run one-sided Wilcoxon signed-rank tests on the paired per-dataset ARI values of FFC versus SC3 and FFC versus Seurat to assess significance.: [evidence span withheld — anchor status: not_found]
- [discussion] There are three main ways to evaluate the performance of unsupervised methods. First, purity scores evaluate the cluster labels based on a set of reference labels.: "There are three main ways to evaluate the performance of unsupervised methods. First, purity scores evaluate the cluster labels based on a set of reference labels."
- [discussion] Simulated scRNA-seq data were generated using the R package Splatter.: "Simulated scRNA-seq data were generated using the R package Splatter."
- [abstract] our method outperforms state-of-the-art clustering approaches on diverse benchmarks of simulated and experimental data: "our method outperforms state-of-the-art clustering approaches on diverse benchmarks of simulated and experimental data"
