---
name: adjusted-rand-index-clustering-accuracy-computation
description: Use when benchmarking clustering or spatial-domain methods against simulations
  that retain ground-truth cell labels, and you need a chance-corrected agreement
  score (ARI) per method per dataset, e.g. to reproduce the scSpace Fig. 1d-e comparison.
when_to_use_negative:
- No ground-truth reference labels exist for the cells (ARI requires a reference labeling;
  it cannot be computed on purely exploratory clusterings of real tissue data)
- The goal is to assess spatial reconstruction quality (pseudo-space coordinates or
  pairwise cellular spatial associations) rather than agreement between two label
  assignments over the same cells
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: scikit-learn
  role: Provides adjusted_rand_score used to compute ARI between predicted clusters
    and ground-truth subclusters for every simulation
- name: scanpy
  role: Runs the Louvain/Leiden community-clustering baseline (with igraph and leidenalg)
    whose cluster labels feed into the ARI comparison
  repo: https://github.com/scverse/scanpy
- name: scSpace
  role: Benchmarked method producing cluster assignments via space-informed clustering
    on the joint gene-expression/spatial-weight embedding, evaluated by ARI
  repo: https://github.com/ZJUFanLab/scSpace
- name: pandas
  role: Aggregates the 140-per-method ARI scores into a comparison table
- name: matplotlib
  role: Renders boxplots of per-method ARI distributions across the simulation series
    (Fig. 1d-e)
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-023-38121-4
    title: Reconstruction of the cell pseudo-space from single-cell RNA sequencing
      data with scSpace
  source_content_hash: sha256:002c2d4fc2a31cb4fb050ff46ab696e66b0bde4678a25d7f15d6acb63ba840e9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/scverse/scanpy
  tools:
  - BayesSpace
  - DR-SC
  - SpaGCN
  - igraph
  - leidenalg
  - matplotlib
  - numpy
  - pandas
  - scSpace
  - scanpy
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# adjusted-rand-index-clustering-accuracy-computation

## Summary

Computes adjusted Rand index (ARI) between predicted cluster labels and ground-truth subcluster labels to benchmark clustering accuracy, as used to show scSpace outperforms Louvain, K-means, Hclust, SpaGCN, STAGATE, BayesSpace, and DR-SC across 140 simulated paired scRNA-seq/ST datasets.

## When to use

Use when benchmarking clustering or spatial-domain methods against simulations that retain ground-truth cell labels, and you need a chance-corrected agreement score (ARI) per method per dataset, e.g. to reproduce the scSpace Fig. 1d-e comparison.

## When NOT to use

- No ground-truth reference labels exist for the cells (ARI requires a reference labeling; it cannot be computed on purely exploratory clusterings of real tissue data)
- The goal is to assess spatial reconstruction quality (pseudo-space coordinates or pairwise cellular spatial associations) rather than agreement between two label assignments over the same cells

## Inputs

- Ground-truth subcluster label vector for every simulated cell (retained from Splatter simulation)
- Predicted cluster assignment vector from each method being benchmarked (scSpace and baselines)
- 140 paired scRNA-seq/spatial-transcriptomics simulation datasets with 5000 genes and 2–10 spatially heterogeneous subclusters

## Outputs

- Per-method ARI score for each of the 140 simulations (140 values per method)
- Aggregated ARI comparison table (pandas DataFrame, methods × simulations)
- Boxplots of ARI distributions per method across the simulation series reproducing Fig. 1d-e (matplotlib)

## How to apply

First run each candidate method — scSpace (TCA characteristic matrices + MLP pseudo-space projection + space-informed clustering on the joint gene-expression/spatial-weight embedding in PyTorch) and baselines (Louvain/Leiden via scanpy with igraph and leidenalg, K-means via scikit-learn, agglomerative hierarchical clustering, SpaGCN, STAGATE, BayesSpace, DR-SC) — on the same 140 Splatter-simulated paired scRNA-seq/ST datasets spanning 2–10 spatially heterogeneous subclusters of 5000 genes each. For every simulation and method, call scikit-learn's adjusted_rand_score on the predicted cluster assignment versus the retained ground-truth subcluster labels. Validate that all ARI values fall in the valid 0–1 range. Aggregate the 140 per-method ARI scores into a comparison table with pandas, then render boxplots of ARI distributions per method across the subcluster-count series with matplotlib to reproduce Fig. 1d-e and check that scSpace attains equal or higher ARI than all baselines.

## Related tools

- **scikit-learn** (Provides adjusted_rand_score used to compute ARI between predicted clusters and ground-truth subclusters for every simulation)
- **scanpy** (Runs the Louvain/Leiden community-clustering baseline (with igraph and leidenalg) whose cluster labels feed into the ARI comparison) — https://github.com/scverse/scanpy
- **scSpace** (Benchmarked method producing cluster assignments via space-informed clustering on the joint gene-expression/spatial-weight embedding, evaluated by ARI) — https://github.com/ZJUFanLab/scSpace
- **pandas** (Aggregates the 140-per-method ARI scores into a comparison table)
- **matplotlib** (Renders boxplots of per-method ARI distributions across the simulation series (Fig. 1d-e))

## Examples

```
from sklearn.metrics import adjusted_rand_score; ari = adjusted_rand_score(ground_truth_subclusters, predicted_clusters)
```

## Evaluation signals

- All ARI values lie within the valid 0–1 range for all 140 simulations per method
- scSpace attains equal or higher ARI than every baseline (Louvain, K-means, Hclust, SpaGCN, STAGATE, BayesSpace, DR-SC, scCoGAPS) consistently as subcluster count rises from 2 to 10
- Comparison table contains one ARI value per method per simulation (9 methods × 140 simulations)
- Boxplots of ARI distributions per method reproduce the ordering reported in Fig. 1d-e

## Limitations

- ARI only measures label agreement against ground truth from simulations; it does not capture spatial coherence or pseudo-space reconstruction accuracy on real tissues
- Applicability of the underlying spatially variated subpopulation identification is limited to tissues with spatial coherence — scSpace should be applied cautiously to tissues with low spatial coherence (e.g., Patients 5 and 9 of SCC)
- The benchmark conclusion depends on Splatter simulation parameters (5000 genes, 2–10 subclusters); results may not transfer to other simulation regimes or real unlabeled data

## Evidence

- [abstract] Benchmark scSpace against simulated datasets using ARI: "We benchmark scSpace with both simulated and biological datasets, and demonstrate that scSpace can accurately and robustly identify spatially variated cell subpopulations."
- [methods] Joint embedding basis for the cluster labels being scored: "so that gene expression and spatial weight of cells can be embedded jointly for the further cell typing with higher accuracy and precision"
- [readme] Figure code availability: "Code for generating the figures in this study can be found [here](AnalysisPaper)."
