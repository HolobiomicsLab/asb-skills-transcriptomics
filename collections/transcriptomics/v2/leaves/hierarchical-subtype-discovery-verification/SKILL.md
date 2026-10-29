---
name: hierarchical-subtype-discovery-verification
description: Use when unsupervised transcriptomic data (bulk or single-cell RNA-seq)
  has an unknown number of clusters and the goal is to discover and verify novel,
  biologically meaningful subtypes without manual k selection.
when_to_use_negative:
- The number of clusters is already known and only a single fixed-k partition is needed
  — RCC is designed for datasets with an unknown number of clusters and hierarchical
  subdivision
- Manual, annotation-driven cluster identification via known sample labels and visualization
  is acceptable — RCC targets novel subtype discovery without requiring computational
  expertise or known annotations
- Large integrated datasets with millions of cells — the paper notes computational
  time is a limitation and that analysis of millions-of-cell integrated datasets is
  still 'the next step', not yet demonstrated
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: RCC (Recursive Consensus Clustering)
  role: Primary R package implementing recursive consensus clustering for novel subtype
    discovery from bulk and single-cell transcriptomic data
  repo: https://github.com/MSCTR/RecursiveConsensusClustering
- name: ConsensusClusterPlus
  role: Base algorithm recursively applied by RCC for dividing the data with k-means
    consensus clustering
- name: clusterAnnotation
  role: RCC function used to annotate resulting clusters against known sample attributes
- name: RCC_clus
  role: RCC entry point that recursively determines the optimal number of clusters
    from an expression matrix and config file
- name: scanorama
  role: Optional companion for integration of multiple datasets before RCC clustering
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41598-020-67016-3
    title: Recursive Consensus Clustering for novel subtype discovery from transcriptome
      data
  source_content_hash: sha256:48aed3c56812a7a2282b7672b1ad78ffc0d26b4c85b5b402c0f8d98181af1d7b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MSCTR/RecursiveConsensusClustering
  tools:
  - ConsensusClusterPlus
  - R
  - RCC (Recursive Consensus Clustering)
  - RCC_clus
  - clusterAnnotation
  - scanorama
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# hierarchical-subtype-discovery-verification

## Summary

Reproduces and verifies novel hierarchical subtype structures in transcriptomic datasets using the RCC (Recursive Consensus Clustering) R package, which recursively applies ConsensusClusterPlus with k-means to determine optimal cluster numbers and subdivides clusters level by level.

## When to use

Use when unsupervised transcriptomic data (bulk or single-cell RNA-seq) has an unknown number of clusters and the goal is to discover and verify novel, biologically meaningful subtypes without manual k selection.

## When NOT to use

- The number of clusters is already known and only a single fixed-k partition is needed — RCC is designed for datasets with an unknown number of clusters and hierarchical subdivision
- Manual, annotation-driven cluster identification via known sample labels and visualization is acceptable — RCC targets novel subtype discovery without requiring computational expertise or known annotations
- Large integrated datasets with millions of cells — the paper notes computational time is a limitation and that analysis of millions-of-cell integrated datasets is still 'the next step', not yet demonstrated

## Inputs

- Gene expression matrix (bulk or single-cell RNA-seq, e.g., Biase GSE57249, Pollen SRP041736, Darmanis GSE67835/GSE84465 from GEO/SRA)
- Sample annotation / metadata file mapping sample IDs to cell-type or sample attributes
- RCC config_file specifying clustering parameters

## Outputs

- Per-level RCC cluster assignments for each sample/cell
- Cluster annotation output tables/plots (via clusterAnnotation)
- Combined cluster assignment table (e.g., rcc_cluster_assignments.csv with sample ID, dataset, attribute, RCC cluster per level)
- Intuitive output plots of hierarchical subdivision structure

## How to apply

Install the RCC package from github.com/MSCTR/RecursiveConsensusClustering (with dependencies clv, fields, matrixStats, data.table, cluster, clue, circlize, gdata, scran, ComplexHeatmap, and pforeach). For each dataset, load the expression matrix and run the entry point RCC_clus(x, config_file), which recursively applies ConsensusClusterPlus (consensus clustering with k-means) to determine the optimal number of clusters without manual selection. Annotate clusters against known sample attributes using clusterAnnotation(x, attribute, cluster, OutputDir, file, clus, outputDir) or clusterAttr(x, infoFile, attribute, minSample, category, algorithm, outputDir). Verify the recursive level structure by cross-tabulating RCC clusters per level against known sample attributes: e.g., in the Biase single-cell dataset (GSE57249) expect ICM vs. trophectoderm separation at level 2 in all runs, three BJ fibroblast clusters, and three-way splits of oligodendrocytes, endothelial and fetal quiescent cells in the Darmanis data (qualitative match to Figures 6–8).

## Related tools

- **RCC (Recursive Consensus Clustering)** (Primary R package implementing recursive consensus clustering for novel subtype discovery from bulk and single-cell transcriptomic data) — https://github.com/MSCTR/RecursiveConsensusClustering
- **ConsensusClusterPlus** (Base algorithm recursively applied by RCC for dividing the data with k-means consensus clustering)
- **clusterAnnotation** (RCC function used to annotate resulting clusters against known sample attributes)
- **RCC_clus** (RCC entry point that recursively determines the optimal number of clusters from an expression matrix and config file)
- **scanorama** (Optional companion for integration of multiple datasets before RCC clustering)

## Examples

```
devtools::install_github("MSCTR/RecursiveConsensusClustering"); clus <- RCC_clus(x, config_file)
```

## Evaluation signals

- On the Biase dataset (GSE57249), RCC subdivides mouse blastocyst cells into ICM and trophectoderm clusters at level 2 across 1000 runs, where tSNE + k-means and SC3 fail to find these subclusters
- BJ fibroblast data resolves into three cell-line clusters (Fig 7)
- Oligodendrocyte, endothelial and fetal quiescent cells each subdivide into three clusters in the Darmanis data (Fig 8)
- Cross-tabulation of RCC clusters per level with known sample attributes qualitatively matches Figures 6–8
- No manual k selection is performed — the optimal number of clusters is determined by RCC's recursive consensus clustering with k-means

## Limitations

- Computational time is a limitation of the algorithm; it depends on the underlying base algorithm (consensus clustering with k-means)
- Analysis of large integrated datasets with millions of cells is still the next step, not yet demonstrated
- Performance is tied to the base algorithm (consensus clustering with k-means)

## Evidence

- [intro] RCC uses ConsensusClusterPlus7 (CCP) as the base algorithm for dividing the data.: "RCC uses ConsensusClusterPlus7 (CCP) as the base algorithm for dividing the data."
- [abstract] RCC is an unsupervised clustering algorithm for novel subtype discovery from both bulk and single-cell datasets.: "We have developed Recursive Consensus Clustering (RCC), an unsupervised clustering algorithm for novel subtype discovery from both bulk and single-cell datasets."
- [methods] RCC_clus(x, config_file): "RCC_clus(x, config_file)"
- [methods] clusterAnnotation(x, attribute, cluster, OutputDir, file, clus, outputDir): "clusterAnnotation(x, attribute, cluster, OutputDir, file, clus, outputDir)"
- [discussion] One of the limitations of our algorithm is computational time.: "One of the limitations of our algorithm is computational time. This time is dependent on the underlying base algorithm which currently is consensus clustering with k-means."
- [readme] README install instructions for the RCC package: "devtools::install_github("MSCTR/RecursiveConsensusClustering")"
