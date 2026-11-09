---
name: pca-clustering-validation
description: Use when you have clustered simulated (or ground-truthed) feature profiles
  with k-means/k-medoids and need an unsupervised confirmation that cluster assignments
  reflect the known simulation design before computing incorrect-assignment rates.
when_to_use_negative:
- The data are real experimental samples with no simulated ground-truth design — PCA
  alignment with 'simulation settings' is then not meaningful
- Raw, untransformed count matrices — the log(x+1) transformation (and per-cell-type
  aggregation or CPM > 1 filtering) is a prerequisite in this workflow
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3308
tools:
- name: MOSim
  role: Generates the simulated bulk (mosim) or single-cell (sc_mosim) multi-omics
    count data whose clustered profiles are validated by PCA
  repo: github.com/ConesaLab/MOSim
- name: R
  role: Implementation environment for log transformation, k-means/k-medoids clustering,
    and PCA on simulated profiles
- name: Seurat
  role: Provides the pbmc.multiome 10x Genomics seed dataset (SeuratData package)
    used as input for sc_mosim single-cell simulation
provenance:
  source_task_ids:
  - task_003
  - task_001
  source_papers:
  - doi: 10.1093/bib/bbaf110
    title: 'MOSim: bulk and single-cell multilayer regulatory network simulator'
  source_content_hash: sha256:7c927d7e6d590893e4e4915058ee5956416b673a181933de5a51206e6bb73ef0
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/ConesaLab/MOSim
  tools:
  - MOSim
  - R
  - Seurat
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# pca-clustering-validation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A validation technique from the MOSim simulator paper: after clustering simulated multi-omics feature profiles (k-means or k-medoids), run PCA on the log(x+1)-transformed profiles to confirm that the cluster structure aligns with the simulation settings (e.g., simulated temporal patterns or cell-type/DE-group labels).

## When to use

Use when you have clustered simulated (or ground-truthed) feature profiles with k-means/k-medoids and need an unsupervised confirmation that cluster assignments reflect the known simulation design before computing incorrect-assignment rates.

## When NOT to use

- The data are real experimental samples with no simulated ground-truth design — PCA alignment with 'simulation settings' is then not meaningful
- Raw, untransformed count matrices — the log(x+1) transformation (and per-cell-type aggregation or CPM > 1 filtering) is a prerequisite in this workflow

## Inputs

- log(x+1)-transformed simulated count matrix (bulk RNA-seq/ATAC-seq gene profiles or per-cell-type aggregated scRNA-seq/scATAC-seq profiles)
- k-means or k-medoids cluster assignments for the simulated feature profiles
- Ground-truth simulation labels (temporal patterns, DE groups, cell types)

## Outputs

- PCA projection/ordination of simulated feature profiles colored or labeled by cluster and ground-truth simulation setting
- Qualitative confirmation that clustering aligns with simulation settings, complementing the computed incorrect cluster-assignment rate

## How to apply

First transform the simulated count matrix — aggregate per-cell-type averages for single-cell data or filter lowly expressed genes (CPM > 1) for bulk — and apply a log(x+1) transformation. After clustering (k-means with k selected by maximizing the Silhouette coefficient and minimizing intra-cluster variability, or k-medoids on Spearman's distance 1 − Spearman's correlation), project the transformed profiles with PCA and visually/numerically check that samples or features separate along principal components in a way consistent with the simulated groups, time points, or cell-type/DE-group labels. PCA is the confirmatory step: the clustering metric (e.g., incorrect cluster-assignment rate below 0.5% for bulk temporal profiles, ~3.3% for single-cell across-cell-type profiles) is the quantitative criterion, while PCA confirms the clustering aligns with the simulation settings.

## Related tools

- **MOSim** (Generates the simulated bulk (mosim) or single-cell (sc_mosim) multi-omics count data whose clustered profiles are validated by PCA) — github.com/ConesaLab/MOSim
- **R** (Implementation environment for log transformation, k-means/k-medoids clustering, and PCA on simulated profiles)
- **Seurat** (Provides the pbmc.multiome 10x Genomics seed dataset (SeuratData package) used as input for sc_mosim single-cell simulation)

## Examples

```
pca <- prcomp(t(log(x + 1))); plot(pca$x[,1], pca$x[,2], col = clusters, pch = 19)
```

## Evaluation signals

- PCA ordination shows separation of feature profiles consistent with the simulated groups, time points, or cell-type/DE-group labels
- Incorrect cluster-assignment rate computed against ground-truth labels is below 0.5% for bulk temporal profiles (k-means, k=7 on STATegra-seeded data)
- Incorrect cluster-assignment rate is ~3.3% for single-cell across-cell-type profiles (k-medoids, k=10 on pbmcMultiome-seeded data)
- Cluster count matches the optimal k selected by maximizing the Silhouette coefficient and minimizing intra-cluster variability
- Cluster structure was computed on log(x+1)-transformed (and, for single-cell, per-cell-type-aggregated) data, not raw counts

## Limitations

- PCA confirmation is qualitative; the quantitative criterion is the incorrect cluster-assignment rate against simulated ground-truth labels
- Cluster structure depends on correct preprocessing: aggregation to average count per cell type for single-cell, CPM > 1 filtering for bulk, and log(x+1) transformation
- Applicable only to simulated data where simulation settings (co-expression patterns, DE groups, cell types) provide ground truth for alignment checks

## Evidence

- [methods] log transformation log(x+1): [evidence span withheld — anchor status: not_found]
