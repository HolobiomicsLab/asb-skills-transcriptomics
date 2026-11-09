---
name: clustering-accuracy-evaluation-ari-purity
description: Use when a produced set of cluster_labels must be scored against reference
  cell-type labels (e.g., PBMC CITE-seq/multiome marker-gene labels) via ARI and purity,
  and when quantifying ARI improvement from posterior-probability label filtering.
when_to_use_negative:
- No reference labels exist for the dataset — purity and ARI are defined only against
  a reference labeling; use internal metrics (e.g., silhouette) instead.
- Only one cluster is present in the data — the Monte Carlo posterior used for the
  PEP < 0.1 filtering cannot accurately estimate uncertainty since every point receives
  one label type.
- Evaluating cluster structure rather than label agreement — ARI/purity against reference
  labels will not capture cluster count or stability issues driven by fire temperature
  c.
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_2229
- http://edamontology.org/topic_0091
tools:
- name: scikit-learn (sklearn)
  role: Provides synthetic Gaussian cluster generation (sigma=0.1) and ARI computation
    against reference labels
  repo: https://github.com/gersteinlab/forest-fire-clustering
- name: numpy
  role: Averages ARI/purity across unseen-cluster-count and Gaussian-noise conditions
  repo: https://github.com/gersteinlab/forest-fire-clustering
- name: Forest Fire Clustering (FFC)
  role: Source of cluster_labels and pval_list posterior significances used for the
    PEP < 0.1 filtering step
  repo: https://github.com/gersteinlab/forest-fire-clustering
- name: scanpy
  role: Preprocesses PBMC RNA counts with the zheng17 recipe and supplies PCA embeddings
    whose clusters are scored
  repo: https://github.com/gersteinlab/forest-fire-clustering
provenance:
  source_task_ids:
  - task_003
  - task_005
  source_papers:
  - doi: 10.1038/s41467-022-31107-8
    title: Forest Fire Clustering for single-cell sequencing combines iterative label
      propagation with parallelized Monte Carlo simulations
  source_content_hash: sha256:f2b644d4bfb8b80dca3575af0dc5e5334121718a618d8d67e21e0613728818e5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gersteinlab/forest-fire-clustering
  tools:
  - Forest Fire Clustering (FFC)
  - Python
  - numba
  - numpy
  - scanpy
  - scikit-learn (sklearn)
  - scipy
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_2229
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# clustering-accuracy-evaluation-ari-purity

## Summary

Evaluates single-cell clustering results against reference cell-type labels using adjusted Rand index (ARI) and purity scores, then re-scores after filtering to high-confidence labels (posterior significance/PEP < 0.1) to quantify label-confidence gains.

## When to use

Use when a produced set of cluster_labels must be scored against reference cell-type labels (e.g., PBMC CITE-seq/multiome marker-gene labels) via ARI and purity, and when quantifying ARI improvement from posterior-probability label filtering.

## When NOT to use

- No reference labels exist for the dataset — purity and ARI are defined only against a reference labeling; use internal metrics (e.g., silhouette) instead.
- Only one cluster is present in the data — the Monte Carlo posterior used for the PEP < 0.1 filtering cannot accurately estimate uncertainty since every point receives one label type.
- Evaluating cluster structure rather than label agreement — ARI/purity against reference labels will not capture cluster count or stability issues driven by fire temperature c.

## Inputs

- cluster_labels per method (numpy arrays from FFC, Louvain, Leiden, PARC)
- Reference cell-type labels (CITE-seq from cell-surface proteins via 10x Genomics marker genes; multiome from marker-gene labels)
- cluster_obj.pval_list (posterior significance values from FFC.validate()/pval())
- Extended out-of-sample labels with reference labels for synthetic streaming tests

## Outputs

- Per method x dataset ARI and purity scores
- Pre- vs post-filtering (PEP < 0.1) ARI comparison
- ari_purity_benchmark.csv summary table
- Average ARI/purity over unseen-cluster-count and noise conditions

## How to apply

Compare each method's cluster_labels (from FFC.fit(fire_temp=100), Louvain, Leiden, or PARC on the first 30 principal components, or top 10,000 highly variable ATAC peaks for multiome) against reference labels derived from 10x Genomics marker genes, computing adjusted Rand index and purity with scikit-learn and numpy. Assemble a method x dataset table of ARI/purity values (e.g., ari_purity_benchmark.csv). To measure the benefit of internal validation, retrieve cluster_obj.pval_list from the FFC object, retain only cells with posterior significance (PEP) < 0.1, and recompute ARI on the filtered cells; in the PBMC benchmark this filtering improved ARI by more than 20% relative to state-of-the-art clustering methods. For synthetic streaming benchmarks (four Gaussian clusters, sigma=0.1), average ARI and purity over unseen-cluster-count and Gaussian-noise conditions and accept the run when both exceed 0.90.

## Related tools

- **scikit-learn (sklearn)** (Provides synthetic Gaussian cluster generation (sigma=0.1) and ARI computation against reference labels) — https://github.com/gersteinlab/forest-fire-clustering
- **numpy** (Averages ARI/purity across unseen-cluster-count and Gaussian-noise conditions) — https://github.com/gersteinlab/forest-fire-clustering
- **Forest Fire Clustering (FFC)** (Source of cluster_labels and pval_list posterior significances used for the PEP < 0.1 filtering step) — https://github.com/gersteinlab/forest-fire-clustering
- **scanpy** (Preprocesses PBMC RNA counts with the zheng17 recipe and supplies PCA embeddings whose clusters are scored) — https://github.com/gersteinlab/forest-fire-clustering

## Examples

```
from forest_fire_clustering.forest_fire_clustering import FFC; cluster_obj = FFC(); cluster_obj.preprocess(X); cluster_obj.fit(fire_temp=100); cluster_obj.validate(); keep = [p < 0.1 for p in cluster_obj.pval_list]; ari = adjusted_rand_score(ref_labels[keep], cluster_obj.cluster_labels[keep])
```

## Evaluation signals

- ARI recomputed after keeping cells with pval_list < 0.1 exceeds the pre-filter ARI by more than 20% on PBMC datasets.
- Benchmark table contains an ARI and purity value for every method (FFC, Louvain, Leiden, PARC) x dataset (CITE-seq, multiome) combination.
- Online out-of-sample benchmark: average ARI and average purity both exceed 0.90 across increasing unseen-cluster counts and Gaussian noise.
- Reference labels are consistent with the source annotation route (cell-surface proteins via 10x Genomics marker genes for CITE-seq; marker-gene labels for multiome).
- Output CSV (ari_purity_benchmark.csv) parses with consistent schema: method, dataset, ARI, purity, and pre/post-filter ARI columns.

## Limitations

- ARI and purity depend on the quality of reference labels; PBMC references derive from cell-surface proteins or marker genes, which can themselves be imperfect.
- Filtering to PEP < 0.1 discards low-confidence cells, so filtered ARI is computed on a cell subset and is not directly comparable to full-dataset ARI without noting the subset.
- Evaluation inherits FFC limitations: the curse of dimensionality degrades clustering when distances become more uniform, and the fire temperature c strongly influences the size and number of discovered clusters.

## Evidence

- [discussion] purity scores evaluate the cluster labels based on a set of reference labels: "There are three main ways to evaluate the performance of unsupervised methods. First, purity scores evaluate the cluster labels based on a set of reference labels."
- [readme] posterior significance values list: "cluster_obj.pval_list # list of posterior significance values"
- [discussion] ARI gain from PEP filtering: [evidence span withheld — anchor status: not_found]
- [discussion] reference labels from 10x marker genes: "For CITE-seq, cell-surface proteins were used to label PBMC cell types using marker genes available from 10× Genomics."
- [discussion] fire temperature sensitivity: "The fire temperature c plays an important role in clustering and strongly influences the size and number of the clusters discovered"
