---
name: spatial-domain-identification-baseline-evaluation
description: Use when you must quantify whether scSpace's space-informed clustering
  outperforms existing clustering or spatial-domain identification methods on data
  where ground-truth subcluster labels (e.g., Splatter simulations) are available.
when_to_use_negative:
- The goal is spatial-domain identification of ST data itself — the article stresses
  scSpace 'focuses on the spatial analysis of scRNA-seq data whereas other methods
  target the spatial domain identification of ST data', so baselines like SpaGCN/BayesSpace/DR-SC
  are not directly comparable in that setting
- No ground-truth subcluster labels exist (e.g., purely biological datasets), making
  ARI-based benchmarking against reference labels impossible
- The tissue has low spatial coherence (e.g., SCC Patients 5 and 9 in the paper),
  where scSpace and its benchmark interpretation should be applied cautiously
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_2229
- http://edamontology.org/topic_3308
tools:
- name: scSpace
  role: 'Method under test: TCA-based pseudo-space reconstruction and space-informed
    clustering of scRNA-seq data'
  repo: github.com/ZJUFanLab/scSpace
- name: PyTorch
  role: Implementation backend for scSpace's MLP-based projection of single cells
    into pseudo space
  repo: https://pytorch.org/get-started/locally/
- name: scanpy
  role: Louvain/Leiden community clustering baseline run with igraph and leidenalg
  repo: https://github.com/scverse/scanpy
- name: scikit-learn
  role: K-means and agglomerative hierarchical clustering baselines plus adjusted_rand_score
    for ARI computation
- name: SpaGCN
  role: Spatial-domain identification baseline
- name: BayesSpace
  role: Spatial-domain identification baseline
- name: DR-SC
  role: Spatial-domain identification baseline
- name: STAGATE
  role: Spatial-domain identification baseline
- name: pandas
  role: Aggregation of per-simulation ARI scores into comparison tables
- name: matplotlib
  role: Boxplot rendering of per-method ARI distributions across simulations
- name: Splatter
  role: Generation of paired scRNA-seq/ST simulation datasets with ground-truth subcluster
    labels
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-023-38121-4
    title: Reconstruction of the cell pseudo-space from single-cell RNA sequencing
      data with scSpace
  source_content_hash: sha256:b83217f39dbee9c120d92bc681e34824dd564f9668c2ffd1e614fcf581e595ed
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BayesSpace
  - DR-SC
  - PyTorch
  - STAGATE
  - SpaGCN
  - Splatter
  - igraph
  - leidenalg
  - matplotlib
  - numpy
  - pandas
  - scSpace
  - scanpy
  - scikit-learn
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_2229
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 26
---

# spatial-domain-identification-baseline-evaluation

## Summary

Benchmark scSpace against clustering baselines (Louvain/Leiden, K-means, hierarchical clustering) and spatial-domain methods (SpaGCN, STAGATE, BayesSpace, DR-SC) using adjusted Rand index (ARI) on simulated paired scRNA-seq/ST datasets with ground-truth subcluster labels.

## When to use

Use when you must quantify whether scSpace's space-informed clustering outperforms existing clustering or spatial-domain identification methods on data where ground-truth subcluster labels (e.g., Splatter simulations) are available.

## When NOT to use

- The goal is spatial-domain identification of ST data itself — the article stresses scSpace 'focuses on the spatial analysis of scRNA-seq data whereas other methods target the spatial domain identification of ST data', so baselines like SpaGCN/BayesSpace/DR-SC are not directly comparable in that setting
- No ground-truth subcluster labels exist (e.g., purely biological datasets), making ARI-based benchmarking against reference labels impossible
- The tissue has low spatial coherence (e.g., SCC Patients 5 and 9 in the paper), where scSpace and its benchmark interpretation should be applied cautiously

## Inputs

- Paired simulated scRNA-seq/spatial transcriptomics datasets with per-cell ground-truth subcluster labels (Splatter, e.g., 5000 genes, 2-10 spatially heterogeneous subclusters)
- Predicted cluster labels from each method (scSpace and all baselines)
- Method configuration per baseline (scanpy Leiden resolution, K-means k, number of clusters for hierarchical clustering and spatial-domain methods)

## Outputs

- Per-simulation, per-method adjusted Rand index (ARI) scores across all simulations
- Aggregated comparison table of ARI scores per method (pandas DataFrame)
- Boxplots of ARI distributions per method across the simulation series (Fig. 1d-e style)

## How to apply

Generate paired scRNA-seq/ST simulations with Splatter (e.g., 5000 genes per dataset, spatially heterogeneous subcluster counts from 2 to 10, 140 datasets total), retaining per-cell ground-truth subcluster labels. Run scSpace using transfer component analysis (TCA) to extract characteristic matrices of the ST and scRNA-seq data, project cells into the pseudo space with an MLP model (PyTorch), and perform space-informed clustering on the joint gene-expression/spatial-weight embedding. Run baselines on the same simulations: Louvain/Leiden via scanpy with igraph and leidenalg, K-means via scikit-learn, agglomerative hierarchical clustering, and the spatial-domain methods SpaGCN, STAGATE, BayesSpace, and DR-SC. Compute ARI between each method's predicted clusters and ground truth with scikit-learn, aggregate per-method scores across all simulations with pandas, and render boxplots of ARI distributions per method with matplotlib to reproduce the paper's Fig. 1d-e comparison.

## Related tools

- **scSpace** (Method under test: TCA-based pseudo-space reconstruction and space-informed clustering of scRNA-seq data) — github.com/ZJUFanLab/scSpace
- **PyTorch** (Implementation backend for scSpace's MLP-based projection of single cells into pseudo space) — https://pytorch.org/get-started/locally/
- **scanpy** (Louvain/Leiden community clustering baseline run with igraph and leidenalg) — https://github.com/scverse/scanpy
- **scikit-learn** (K-means and agglomerative hierarchical clustering baselines plus adjusted_rand_score for ARI computation)
- **SpaGCN** (Spatial-domain identification baseline)
- **BayesSpace** (Spatial-domain identification baseline)
- **DR-SC** (Spatial-domain identification baseline)
- **STAGATE** (Spatial-domain identification baseline)
- **pandas** (Aggregation of per-simulation ARI scores into comparison tables)
- **matplotlib** (Boxplot rendering of per-method ARI distributions across simulations)
- **Splatter** (Generation of paired scRNA-seq/ST simulation datasets with ground-truth subcluster labels)

## Examples

```
from sklearn.metrics import adjusted_rand_score; ari = adjusted_rand_score(ground_truth_labels, scSpace_predicted_labels)
```

## Evaluation signals

- All ARI values fall within the valid 0-1 range for every simulation and every method
- scSpace attains equal or higher ARI than every baseline (Louvain, K-means, Hclust, SpaGCN, STAGATE, BayesSpace, DR-SC, scCoGAPS) consistently as the number of subclusters increases from 2 to 10
- Each method produces exactly one predicted-cluster assignment per simulated cell, with cluster counts matching the simulation's configured subcluster number
- ARI is computed against the Splatter ground-truth subcluster labels, not against any method's own labels
- Per-method ARI distributions aggregate to 140 scores (one per simulated dataset) in the comparison table

## Limitations

- scSpace should be applied cautiously to tissues with low spatial coherence (e.g., SCC Patients 5 and 9), so benchmark conclusions may not transfer to such tissues
- The choice of spatial reference directly influences the spatial reconstruction, so benchmark results depend on the reference ST data used
- Intrinsic differences in spatial homogeneity between tissues may influence scSpace accuracy, complicating extrapolation from simulated benchmarks to real tissues
- Spatial-domain baselines (SpaGCN, BayesSpace, DR-SC, STAGATE) are designed for spatial-domain identification of ST data, a fundamentally different task from scSpace's spatial analysis of scRNA-seq data

## Evidence

- [results] scSpace achieved the highest ARI consistently as the number of subclusters increased, outperforming all baselines: [evidence span withheld — anchor status: not_found]
- [abstract] scSpace can accurately and robustly identify spatially variated cell subpopulations: "We benchmark scSpace with both simulated and biological datasets, and demonstrate that scSpace can accurately and robustly identify spatially variated cell subpopulations."
- [methods] Using transfer component analysis (TCA), scSpace could extract the characteristic matrixes: "Using [transfer component analysis (TCA)](https://ieeexplore.ieee.org/stamp/stamp.jsp?tp=&arnumber=5640675&tag=1), scSpace could extract the characteristic matrixes of spatial transcriptomics and"
- [discussion] scSpace focuses on the spatial analysis of scRNA-seq data whereas other methods target the spatial domain identification of ST data: "scSpace focuses on the spatial analysis of scRNA-seq data whereas other methods target the spatial domain identification of ST data"
- [discussion] Since the choice of spatial reference would directly influence the spatial reconstruction: "Since the choice of spatial reference would directly influence the spatial reconstruction"
