---
name: paired-scrnaseq-spatial-transcriptomics-benchmarking
description: Use when comparing a spatial subpopulation identification method against
  baselines (Louvain, K-means, Hclust, SpaGCN, STAGATE, BayesSpace, DR-SC, scCoGAPS)
  on simulated data with known ground-truth subcluster labels, using ARI as the metric.
when_to_use_negative:
- The analysis target is real biological ST data without ground-truth cluster labels
  — ARI against truth cannot be computed and this benchmark design does not apply.
- The task is spatial-domain identification of ST data itself rather than spatial
  analysis of scRNA-seq data; the article notes scSpace 'focuses on the spatial analysis
  of scRNA-seq data whereas other methods target the spatial domain identification
  of ST data'.
- Spatially heterogeneous subcluster count exceeds 10 or the simulation design differs
  fundamentally (e.g., no spatial heterogeneity), making the 2-10 subcluster benchmark
  series inapplicable.
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0622
tools:
- name: scSpace
  role: 'Method under test: reconstructs cell pseudo-space by extracting characteristic
    matrices via TCA, projecting cells with an MLP, and space-informed clustering
    on the joint gene-expression/spatial-weight embedding'
  repo: github.com/ZJUFanLab/scSpace
- name: PyTorch
  role: Backend for scSpace's MLP-based projection of single cells into pseudo space
  repo: https://pytorch.org/get-started/locally/
- name: scanpy
  role: Runs Louvain/Leiden community clustering baseline with igraph and leidenalg
    on the simulated datasets
  repo: https://github.com/scverse/scanpy
- name: scikit-learn
  role: Provides K-means, agglomerative hierarchical clustering, and adjusted Rand
    index (ARI) computation
- name: SpaGCN
  role: Spatial-domain identification baseline method
- name: STAGATE
  role: Spatial-domain identification baseline method
- name: BayesSpace
  role: Spatial-domain identification baseline method
- name: DR-SC
  role: Spatial-domain identification baseline method
- name: Splatter
  role: Generates 140 paired scRNA-seq/ST simulations with 5000 genes and 2-10 spatially
    heterogeneous subclusters
- name: pandas
  role: Aggregates 140-per-method ARI scores into a comparison table
- name: matplotlib
  role: Renders boxplots of per-method ARI distributions across simulations (Fig.
    1d-e)
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-023-38121-4
    title: Reconstruction of the cell pseudo-space from single-cell RNA sequencing
      data with scSpace
  source_content_hash: sha256:2495b8e027607ce74f90f6eefc1c61c88b34d50b2c7d1b70c0685bfc1052402e
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
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 24
---

# paired-scrnaseq-spatial-transcriptomics-benchmarking

## Summary

Benchmarking skill for evaluating scSpace against clustering and spatial-domain methods on simulated paired scRNA-seq/spatial transcriptomics (ST) datasets, scored by adjusted Rand index (ARI) against ground-truth spatially heterogeneous subclusters.

## When to use

Use when comparing a spatial subpopulation identification method against baselines (Louvain, K-means, Hclust, SpaGCN, STAGATE, BayesSpace, DR-SC, scCoGAPS) on simulated data with known ground-truth subcluster labels, using ARI as the metric.

## When NOT to use

- The analysis target is real biological ST data without ground-truth cluster labels — ARI against truth cannot be computed and this benchmark design does not apply.
- The task is spatial-domain identification of ST data itself rather than spatial analysis of scRNA-seq data; the article notes scSpace 'focuses on the spatial analysis of scRNA-seq data whereas other methods target the spatial domain identification of ST data'.
- Spatially heterogeneous subcluster count exceeds 10 or the simulation design differs fundamentally (e.g., no spatial heterogeneity), making the 2-10 subcluster benchmark series inapplicable.

## Inputs

- 140 simulated paired scRNA-seq/ST datasets generated with Splatter (5000 genes each, 2-10 spatially heterogeneous subclusters, with ground-truth subcluster labels per cell)
- Ground-truth subcluster label vector per simulated cell
- Predicted cluster assignments from each method (scSpace and baselines)

## Outputs

- Per-simulation ARI scores for each method (140 values per method)
- Aggregated ARI comparison table (pandas)
- Boxplots of ARI distributions per method across the simulation series (Fig. 1d-e equivalent)

## How to apply

Generate 140 paired scRNA-seq/ST simulation datasets with Splatter, each with 5000 genes and spatially heterogeneous subcluster counts varying from 2 to 10, retaining ground-truth subcluster labels for every simulated cell. Run scSpace on each simulation: extract characteristic matrices of the ST and scRNA-seq data via transfer component analysis (TCA), project cells into the pseudo space with an MLP model (PyTorch), and perform space-informed clustering on the joint gene-expression/spatial-weight embedding. Run the same baselines per simulation: Louvain/Leiden community clustering via scanpy with igraph and leidenalg, K-means and agglomerative hierarchical clustering via scikit-learn, and the spatial-domain methods SpaGCN, STAGATE, BayesSpace, and DR-SC. Compute adjusted Rand index (ARI) between each method's predicted clusters and the ground-truth subclusters per simulation using scikit-learn, aggregate into a comparison table with pandas, and render boxplots of ARI distributions per method with matplotlib to reproduce Fig. 1d-e.

## Related tools

- **scSpace** (Method under test: reconstructs cell pseudo-space by extracting characteristic matrices via TCA, projecting cells with an MLP, and space-informed clustering on the joint gene-expression/spatial-weight embedding) — github.com/ZJUFanLab/scSpace
- **PyTorch** (Backend for scSpace's MLP-based projection of single cells into pseudo space) — https://pytorch.org/get-started/locally/
- **scanpy** (Runs Louvain/Leiden community clustering baseline with igraph and leidenalg on the simulated datasets) — https://github.com/scverse/scanpy
- **scikit-learn** (Provides K-means, agglomerative hierarchical clustering, and adjusted Rand index (ARI) computation)
- **SpaGCN** (Spatial-domain identification baseline method)
- **STAGATE** (Spatial-domain identification baseline method)
- **BayesSpace** (Spatial-domain identification baseline method)
- **DR-SC** (Spatial-domain identification baseline method)
- **Splatter** (Generates 140 paired scRNA-seq/ST simulations with 5000 genes and 2-10 spatially heterogeneous subclusters)
- **pandas** (Aggregates 140-per-method ARI scores into a comparison table)
- **matplotlib** (Renders boxplots of per-method ARI distributions across simulations (Fig. 1d-e))

## Examples

```
python run_benchmark.py --method scSpace --splatter_n_datasets 140 --n_genes 5000 --subclusters 2-10 --metric ari
```

## Evaluation signals

- ARI values fall within the valid 0-1 range for all 140 simulations per method
- scSpace attains equal or higher ARI than all baselines (Louvain, K-means, Hclust, SpaGCN, STAGATE, BayesSpace, DR-SC, scCoGAPS) at each subcluster count from 2 to 10
- scSpace achieves the highest ARI consistently as the number of simulated spatially heterogeneous subclusters increases
- Each method produces exactly 140 ARI values, one per simulation, with no missing or failed runs
- Boxplots of ARI distributions per method visually reproduce the pattern reported in Fig. 1d-e

## Limitations

- scSpace should be applied cautiously to tissues with low spatial coherence, such as Patients 5 and 9 of SCC evaluated in the paper
- The choice of spatial reference directly influences the spatial reconstruction; an appropriateness assessment or pre-processing of the reference should be performed first
- Spatially heterogeneous cell subpopulations are difficult to identify by traditional single-cell analysis pipelines due to transcriptome similarity, motivating the space-informed approach used here

## Evidence

- [methods] Generation of simulated paired datasets: [evidence span withheld — anchor status: not_found]
- [methods] Run scSpace pipeline: [evidence span withheld — anchor status: not_found]
- [methods] Run baseline methods: [evidence span withheld — anchor status: not_found]
- [methods] Compute ARI: [evidence span withheld — anchor status: not_found]
- [results] Finding statement: [evidence span withheld — anchor status: not_found]
- [discussion] Scope caveat from discussion: "scSpace focuses on the spatial analysis of scRNA-seq data whereas other methods target the spatial domain identification of ST data"
- [abstract] scSpace abstract summary: "We benchmark scSpace with both simulated and biological datasets, and demonstrate that scSpace can accurately and robustly identify spatially variated cell subpopulations."
