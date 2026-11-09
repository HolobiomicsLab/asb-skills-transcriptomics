---
name: ensemble-consensus-clustering
description: Use when clustering scRNA-seq expression matrices where the number of
  cell types is known and no single algorithm is consistently , and a consensus label
  per cell is needed with NMI/ARI benchmarking against published labels.
when_to_use_negative:
- The number of cell types is unknown — most base algorithms require k set to the
  known number of cell types
- The input is already a reduced feature embedding or cell metadata table rather than
  a raw gene-by-cell count matrix
- You need a single lightweight algorithm for exploratory clustering on one dataset;
  running nine base clusterers plus meta-clustering is designed for benchmark-grade
  consensus, not quick sketches
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scEFSC
  role: The ensemble feature-selection-based consensus clustering pipeline itself
    (autoencoder pre-selection, four feature selections, nine clusterers, wMetaC combination)
  repo: https://github.com/Conan-Bian/scEFSC
- name: SC3
  role: Base clustering algorithm (v1.18.0) run on each feature-selected subset
  repo: github.com/hemberg-lab/SC3
- name: CIDR
  role: Base clustering algorithm (v0.1.5) run on each feature-selected subset
  repo: github.com/VCCRI/CIDR
- name: Monocle
  role: Base clustering algorithm (v2.18.0) run on each feature-selected subset
  repo: github.com/cole-trapnell-lab/monocle-release
- name: pcaReduce
  role: Base clustering algorithm (v1.0) run on each feature-selected subset
  repo: github.com/JustinaZ/pcaReduce
- name: Rphenograph
  role: Base clustering algorithm (v0.99.1) run on each feature-selected subset
  repo: github.com/JinmiaoChenLab/Rphenograph
- name: Seurat
  role: Base clustering algorithm (v4.0.0, 10 reduction dimensions) run on each feature-selected
    subset
- name: SHARP
  role: Base clustering algorithm (v1.1.0) run on each feature-selected subset
  repo: github.com/shibiaowan/SHARP
- name: SINCERA
  role: Base clustering algorithm (v0.99.0, group-average linkage) run on each feature-selected
    subset
  repo: github.com/xu-lab/SINCERA
- name: RaceID
  role: Base clustering algorithm (v0.2.2, max 20 clusters) run on each feature-selected
    subset
- name: wMetaC
  role: Weighted meta-clustering used to combine retained base labels with colocation
    weights w(i,j)=s(i,j)(1-s(i,j)), delta=0.01, ward.D hierarchical clustering and
    final voting
provenance:
  source_task_ids:
  - task_002
  - task_004
  - task_001
  - task_003
  - task_005
  source_papers:
  - doi: 10.1016/j.csbj.2022.04.023
    title: 'scEFSC: Accurate single-cell RNA-seq data analysis via ensemble consensus
      clustering based on multiple feature selections'
  source_content_hash: sha256:f5e32a0c7d655e1e319c0f1aca78885d06b3f0299d49074538887daab8675f3c
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/Conan-Bian/scEFSC
  repo_url: https://github.com/Conan-Bian/scEFSC
  tools:
  - CIDR
  - Monocle
  - PhenoGraph
  - RaceID
  - Rphenograph
  - SAME-clustering
  - SC3
  - SHARP
  - SINCERA
  - Seurat
  - pcaReduce
  - scDHA
  - scEFSC
  - wMetaC
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 20
---

# ensemble-consensus-clustering

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Ensemble consensus clustering for scRNA-seq (as implemented in scEFSC): apply four unsupervised feature selections (Low Variance, Laplacian Score, SPEC, MCFS), cluster each feature subset with nine scRNA-seq algorithms, then merge all labels with wMetaC weighted meta-clustering to obtain a final robust cell-type assignment.

## When to use

Use when clustering scRNA-seq expression matrices where the number of cell types is known and no single algorithm is consistently best, and a consensus label per cell is needed with NMI/ARI benchmarking against published labels.

## When NOT to use

- The number of cell types is unknown — most base algorithms require k set to the known number of cell types
- The input is already a reduced feature embedding or cell metadata table rather than a raw gene-by-cell count matrix
- You need a single lightweight algorithm for exploratory clustering on one dataset; running nine base clusterers plus meta-clustering is designed for benchmark-grade consensus, not quick sketches

## Inputs

- scRNA-seq expression matrix (genes x cells) from GEO/ArrayExpress/dbGaP (e.g. GSE36552, GSE81608, E-MTAB-5061, phs000833.v3.p1)
- Previously published cell-type labels for ARI/NMI evaluation
- Number of cell types k per dataset

## Outputs

- Final per-cell cluster labels (scefsc_cluster_labels.csv)
- ARI and NMI values per method-dataset pair (e.g. benchmark_ari_nmi_table.csv)
- Feature-selection ablation ARI/NMI table

## How to apply

Preprocess the n x d expression matrix with a log2 (base 2) transformation (rescale if values exceed 100), remove genes detected in less than 2% of cells and drop cells labeled 'abandoned'. Pre-select 5000 genes with a single-layer non-negative kernel autoencoder, retaining genes with large encoder-weight variances. Generate four feature subsets via Low Variance, Laplacian Score, SPEC, and MCFS, then cluster each subset with nine algorithms — SC3 v1.18.0, CIDR v0.1.5, monocle v2.18.0, pcaReduce v1.0, Rphenograph v0.99.1 (reduction dimension 10), Seurat v4.0.0, SHARP v1.1.0, SINCERA v0.99.0 (group-average linkage), RaceID v0.2.2 (max 20 clusters) — setting k to the number of cell types. Compute pairwise ARI between all labels and remove labels whose similarity vector has the lowest variance (least diversity), then combine the remaining labels with wMetaC weighted meta-clustering: convert similarities to colocation weights w(i,j)=s(i,j)(1-s(i,j)), compute weighted inter-cluster similarities with delta=0.01, cluster with ward.D hierarchical clustering, and assign cells by majority voting.

## Related tools

- **scEFSC** (The ensemble feature-selection-based consensus clustering pipeline itself (autoencoder pre-selection, four feature selections, nine clusterers, wMetaC combination)) — https://github.com/Conan-Bian/scEFSC
- **SC3** (Base clustering algorithm (v1.18.0) run on each feature-selected subset) — github.com/hemberg-lab/SC3
- **CIDR** (Base clustering algorithm (v0.1.5) run on each feature-selected subset) — github.com/VCCRI/CIDR
- **Monocle** (Base clustering algorithm (v2.18.0) run on each feature-selected subset) — github.com/cole-trapnell-lab/monocle-release
- **pcaReduce** (Base clustering algorithm (v1.0) run on each feature-selected subset) — github.com/JustinaZ/pcaReduce
- **Rphenograph** (Base clustering algorithm (v0.99.1) run on each feature-selected subset) — github.com/JinmiaoChenLab/Rphenograph
- **Seurat** (Base clustering algorithm (v4.0.0, 10 reduction dimensions) run on each feature-selected subset)
- **SHARP** (Base clustering algorithm (v1.1.0) run on each feature-selected subset) — github.com/shibiaowan/SHARP
- **SINCERA** (Base clustering algorithm (v0.99.0, group-average linkage) run on each feature-selected subset) — github.com/xu-lab/SINCERA
- **RaceID** (Base clustering algorithm (v0.2.2, max 20 clusters) run on each feature-selected subset)
- **wMetaC** (Weighted meta-clustering used to combine retained base labels with colocation weights w(i,j)=s(i,j)(1-s(i,j)), delta=0.01, ward.D hierarchical clustering and final voting)

## Examples

```
library(SHARP); res = SHARP(scExp, rN.seed=rN)  # one base clusterer; full ensemble: Rscript run_scEFSC.R --input expr_matrix.csv --k <n_cell_types> --output scefsc_cluster_labels.csv
```

## Evaluation signals

- Final labels achieve non-degenerate ARI/NMI agreement with the dataset's published cell-type labels; scEFSC should attain the best NMI on 11 of 14 benchmark datasets among the ten compared algorithms
- The benchmark table contains ARI and NMI entries for all ten methods (scEFSC plus nine baselines) across all fourteen datasets
- Feature-selection ablation shows full scEFSC equal or better than any single-feature-selection variant (Laplacian Score, Low Variance, SPEC, MCFS alone) on most datasets
- Diversity filtering actually removes labels: pairwise ARI similarity-vector variance identifies and drops the least diverse base label sets before wMetaC
- wMetaC output has exactly k clusters matching the known number of cell types after ward.D clustering and voting

## Limitations

- Requires the true number of cell types as k for most base clustering algorithms
- Ensemble is computationally heavy: nine clustering algorithms across four feature subsets per dataset
- Biological interpretability of final clusters requires downstream differential gene expression, GO enrichment and KEGG analysis to be run separately
- Performance depends on preprocessing choices (log2 rescale threshold of 100, 2% detection cutoff, 5000-gene autoencoder pre-selection); results were validated on fourteen published scRNA-seq datasets and may not transfer to untested data types

## Evidence
