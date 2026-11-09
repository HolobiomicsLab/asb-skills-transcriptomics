---
name: multi-algorithm-single-cell-clustering
description: Use when the input is a preprocessed n × d scRNA-seq expression matrix
  (genes as rows, cells as columns) with a known number of cell types k, and no single
  clustering algorithm can be trusted to suit the dataset.
when_to_use_negative:
- The number of cell types k is unknown and cannot be estimated, since base clusterers
  and wMetaC are run with the known k
- The task is transcript-level quantification, differential expression, or trajectory
  inference rather than cell clustering — scEFSC handles clustering only downstream
  of feature selection
- The dataset has no reliable gene filter available and contains mostly low-level
  expressed genes; genes detected in less than 2% of cells must first be removed or
  noise will propagate into all base clusterers
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: scEFSC
  role: wrapper pipeline that performs the ensemble feature selection, base clustering,
    and wMetaC weighted meta-clustering
  repo: https://github.com/Conan-Bian/scEFSC
- name: SC3
  role: base clustering algorithm (v1.18.0) run on feature-selected subsets
  repo: https://github.com/hemberg-lab/SC3
- name: CIDR
  role: base clustering algorithm (v0.1.5) run on feature-selected subsets
  repo: https://github.com/VCCRI/CIDR
- name: Monocle
  role: base clustering algorithm (v2.18.0) run on feature-selected subsets
  repo: https://github.com/cole-trapnell-lab/monocle-release
- name: pcaReduce
  role: base clustering algorithm (v1.0) run on feature-selected subsets
  repo: https://github.com/JustinaZ/pcaReduce
- name: Rphenograph
  role: base clustering algorithm (v0.99.1) run on feature-selected subsets
  repo: https://github.com/JinmiaoChenLab/Rphenograph
- name: Seurat
  role: base clustering algorithm (v4.0.0, reduction dimension 10) run on feature-selected
    subsets
- name: SHARP
  role: base clustering algorithm (v1.1.0) run on feature-selected subsets
  repo: https://github.com/shibiaowan/SHARP
- name: SINCERA
  role: base clustering algorithm (v0.99.0, group-average linkage) run on feature-selected
    subsets
  repo: https://github.com/xu-lab/SINCERA
- name: RaceID
  role: base clustering algorithm (v0.2.2, maximum 20 clusters) run on feature-selected
    subsets
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.csbj.2022.04.023
    title: 'scEFSC: Accurate single-cell RNA-seq data analysis via ensemble consensus
      clustering based on multiple feature selections'
  source_content_hash: sha256:496de61e98aeea8859f167139bd28ba6b7af40e284acfe6a25fbb53619b9b5bf
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
  - ACTIONet
  - CIDR
  - Monocle
  - Monocle3
  - ParallelPCA
  - RaceID
  - Randomly
  - Rphenograph
  - SC3
  - SHARP
  - SINCERA
  - Scanpy
  - Seurat
  - ZINB-WaVE
  - pcaReduce
  - scDHA
  - scEFSC
  - scLENS
  - scTransform
  - scVI
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
merged_aliases:
- graph-based-single-cell-clustering
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# multi-algorithm-single-cell-clustering

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Cluster an scRNA-seq expression matrix with nine base scRNA-seq clustering algorithms (SC3, CIDR, monocle, pcaReduce, Rphenograph, Seurat, SHARP, SINCERA, RaceID) and combine the resulting labels via a wMetaC weighted meta-clustering step, as done in scEFSC to avoid choosing a single best-performing algorithm per dataset.

## When to use

Use when the input is a preprocessed n × d scRNA-seq expression matrix (genes as rows, cells as columns) with a known number of cell types k, and no single clustering algorithm can be trusted to suit the dataset.

## When NOT to use

- The number of cell types k is unknown and cannot be estimated, since base clusterers and wMetaC are run with the known k
- The task is transcript-level quantification, differential expression, or trajectory inference rather than cell clustering — scEFSC handles clustering only downstream of feature selection
- The dataset has no reliable gene filter available and contains mostly low-level expressed genes; genes detected in less than 2% of cells must first be removed or noise will propagate into all base clusterers

## Inputs

- n × d scRNA-seq expression matrix (rows = genes, columns = cells) in count/TPM/CPM-like form
- Known number of cell types k for the dataset
- Published cell-type reference labels for validation (NMI/ARI)

## Outputs

- Final per-cell cluster labels (e.g. saved to scefsc_cluster_labels.csv)
- Individual clustering label sets from nine base algorithms per feature subset
- Clustering similarity matrix with colocation weights w(i,j)=s(i,j)(1−s(i,j))

## How to apply

First reduce the gene set so base clusterers see informative features only: log2-normalize, remove genes detected in less than 2% of cells, pre-select 5000 genes with a non-negative kernel autoencoder, then derive four feature subsets using Low Variance, Laplacian Score, SPEC, and MCFS. Run the nine base clusterers (SC3 v1.18.0, CIDR v0.1.5, monocle v2.18.0, pcaReduce v1.0, Rphenograph v0.99.1, Seurat v4.0.0 with reduction dimension 10, SHARP v1.1.0, SINCERA v0.99.0 with group-average linkage, RaceID v0.2.2 with maximum 20 clusters) on each feature subset using the known number of cell types as k. Compute pairwise Adjusted Rand Index between all clustering labels and discard the labels whose similarity vector has the lowest variance (least diversity). Combine the retained labels with wMetaC: build the clustering similarity matrix, convert it to colocation weights w(i,j)=s(i,j)(1−s(i,j)), compute weighted inter-cluster similarities (δ=0.01), cluster with hierarchical clustering using ward.D, and assign each cell by majority voting. Rationale: since each algorithm has strengths and weaknesses on particular datasets, ensembling multiple clusterers on multiple feature selections outperformed ten single-cell clustering algorithms on eleven of fourteen benchmark datasets by NMI.

## Related tools

- **scEFSC** (wrapper pipeline that performs the ensemble feature selection, base clustering, and wMetaC weighted meta-clustering) — https://github.com/Conan-Bian/scEFSC
- **SC3** (base clustering algorithm (v1.18.0) run on feature-selected subsets) — https://github.com/hemberg-lab/SC3
- **CIDR** (base clustering algorithm (v0.1.5) run on feature-selected subsets) — https://github.com/VCCRI/CIDR
- **Monocle** (base clustering algorithm (v2.18.0) run on feature-selected subsets) — https://github.com/cole-trapnell-lab/monocle-release
- **pcaReduce** (base clustering algorithm (v1.0) run on feature-selected subsets) — https://github.com/JustinaZ/pcaReduce
- **Rphenograph** (base clustering algorithm (v0.99.1) run on feature-selected subsets) — https://github.com/JinmiaoChenLab/Rphenograph
- **Seurat** (base clustering algorithm (v4.0.0, reduction dimension 10) run on feature-selected subsets)
- **SHARP** (base clustering algorithm (v1.1.0) run on feature-selected subsets) — https://github.com/shibiaowan/SHARP
- **SINCERA** (base clustering algorithm (v0.99.0, group-average linkage) run on feature-selected subsets) — https://github.com/xu-lab/SINCERA
- **RaceID** (base clustering algorithm (v0.2.2, maximum 20 clusters) run on feature-selected subsets)

## Examples

```
Rscript run_scefsc.R --input expression_matrix.csv --k 14 --output scefsc_cluster_labels.csv
```

## Evaluation signals

- Pairwise ARI between individual clustering labels is computed and labels with lowest-variance similarity vectors are removed — the retained set should show diverse yet non-degenerate agreement
- Final ensemble labels compared to published cell-type labels give NMI/ARI values; scEFSC achieved the best NMI on 11 of 14 benchmark datasets, so large NMI drops indicate misapplication
- Every cell receives exactly one cluster label after wMetaC majority voting — no missing or duplicated assignments
- Downstream differential gene expression, GO enrichment and KEGG analysis of the clusters should recover biologically interpretable marker structure
- Output CSV (e.g. scefsc_cluster_labels.csv) has one label per input cell and cluster count equal to the supplied k

## Limitations

- The number of cell types must be known in advance; k is supplied to all base clusterers and the meta-clustering step
- Base tools are version-sensitive (e.g. Seurat 4.0.0, SC3 1.18.0, CIDR 0.1.5, monocle 2.18.0, Rphenograph 0.99.1, SHARP 1.1.0, SINCERA 0.99.0, RaceID 0.2.2) and several must be installed from GitHub
- Some public datasets contain cells ambiguously labeled 'abandoned'; these must be removed before evaluation or reference comparisons are biased
- If gene expression values exceed 100, a log2 transformation is needed to rescale before clustering

## Evidence

- [intro] nine base clusterers with parameters: [evidence span withheld — anchor status: not_found]
- [intro] wMetaC details: [evidence span withheld — anchor status: not_found]
