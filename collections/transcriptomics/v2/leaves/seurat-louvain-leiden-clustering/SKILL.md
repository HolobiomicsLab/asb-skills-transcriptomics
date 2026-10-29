---
name: seurat-louvain-leiden-clustering
description: Use when benchmarking a new scRNA-seq clustering framework (e.g., DeepCCI)
  against baselines, and you need Seurat Louvain/Leiden results on the same labeled
  datasets to compute comparative ARI/NMI scores.
when_to_use_negative:
- When the goal is CCI (cell-cell interaction) inference rather than clustering benchmarking
  — the article's DeepCCI workflow separates the cluster model from the interaction
  model
- When the scRNA-seq data lacks ground-truth cell-type labels, since ARI against reference
  labels is the scoring criterion
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Seurat
  role: R package providing the baseline Louvain and Leiden single-cell clustering
    used for ARI/NMI comparison in the benchmark
- name: igraph
  role: Graph-clustering dependency installed alongside Seurat for community detection
    in R
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad596
    title: 'DeepCCI: a deep learning framework for identifying cell–cell interactions
      from single-cell RNA sequencing data'
  source_content_hash: sha256:8b3365fa06133d88c6cae834991324e0040136326a854d4140870ce9746b80ea
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DeepCCI
  - Python
  - R
  - Seurat
  - conda
  - git
  - igraph
  - pip
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5625
n_claims: 16
---

# seurat-louvain-leiden-clustering

## Summary

A baseline clustering method for scRNA-seq data that runs Seurat in R using both Louvain and Leiden community detection, used in the DeepCCI benchmark as one of 13 comparison methods scored by Adjusted Rand Index (ARI) against ground-truth cell-type labels.

## When to use

Use when benchmarking a new scRNA-seq clustering framework (e.g., DeepCCI) against state-of-the-art baselines, and you need Seurat Louvain/Leiden results on the same labeled datasets to compute comparative ARI/NMI scores.

## When NOT to use

- When the goal is CCI (cell-cell interaction) inference rather than clustering benchmarking — the article's DeepCCI workflow separates the cluster model from the interaction model
- When the scRNA-seq data lacks ground-truth cell-type labels, since ARI against reference labels is the scoring criterion

## Inputs

- Public scRNA-seq datasets with ground-truth cell-type labels (CSV or h5 format, e.g., Yan GSE36552, Quake_10x_Limb_Muscle GSE109774)
- R environment with Seurat and igraph installed

## Outputs

- Per-cell cluster assignments from Louvain and Leiden runs
- Per-dataset ARI scores (mean over 10 repeats) for the benchmark table

## How to apply

Install Seurat and igraph in an R environment (the DeepCCI methods section instructs installing Seurat and igraph, plus devtools for GitHub-hosted dependencies). Run Seurat-based clustering in R using both the Louvain and Leiden algorithms on each of the same 12 public scRNA-seq datasets used for the primary model (e.g., Yan/GSE36552 in CSV format, Quake_10x_Limb_Muscle/GSE109774 in h5 format), repeating each run 10 times to control seed variance. Score every clustering result against ground-truth labels with the Adjusted Rand Index, aggregate the mean ARI per method per dataset, and insert the values into the benchmark table (Supplementary Data S4) alongside scTAG, scGNN, scDCC, DEC, K-means, Spectral clustering and other baselines. The comparison target is DeepCCI's reported best ARI on all 12 datasets, including ARI 0.9848 on Quake_10x_Limb_Muscle (GSE109774).

## Related tools

- **Seurat** (R package providing the baseline Louvain and Leiden single-cell clustering used for ARI/NMI comparison in the benchmark)
- **igraph** (Graph-clustering dependency installed alongside Seurat for community detection in R)

## Examples

```
Rscript -e 'install.packages("Seurat"); install.packages("igraph")'
```

## Evaluation signals

- Clustering results must be reproducible across 10 repeats per dataset (seed-variance control)
- ARI computed against ground-truth cell-type labels for every run; mean ARI must be reported per method per dataset
- Results should slot into the benchmark table matching Supplementary Data S4
- DeepCCI should achieve the highest ARI among all 14 methods on all 12 datasets (e.g., 0.9848 on Quake_10x_Limb_Muscle GSE109774) if the baseline is applied correctly
- Seurat R installation must succeed with igraph, matching the paper's dependency list

## Limitations

- The article reports current statistical/clustering methods miss key latent information in scRNA-seq data with high sparsity and heterogeneity, motivating the DeepCCI GCN approach
- Preprocessing each dataset takes about 5-10 minutes and cluster model runs about 25 minutes (with pretraining), so the full 14-method, 12-dataset, 10-repeat benchmark is computationally substantial

## Evidence

- [methods] Seurat listed as a baseline: "install.packages(‘Seurat’)"
- [methods] igraph dependency in R: "install.packages("igraph")"
- [readme] Clustering output description from README: "The clustering results of scRNA-seq data will be output."
- [abstract] Motivation for GCN-based clustering over statistical methods: "current computational methods based on statistical strategies ignore some key latent information contained in scRNA-seq data with high sparsity and heterogeneity"
