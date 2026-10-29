---
name: cluster-based-adaptive-reweighting-sampling
description: Use when constructing a labeled training set from a single-cell expression
  dataset (Seurat object with log-normalized expression) where rare cell types must
  be captured and active learning is impractical or no iterative labeling budget exists.
when_to_use_negative:
- When an iterative labeling budget exists and active learning (maximum-entropy or
  lowest-max-probability selection of the 10 most uncertain cells) is feasible — active
  learning outperformed adaptive reweighting across most datasets in the benchmark
- When the input Seurat object has not been log-normalized and had variable features
  scaled — leader requires this preprocessing and unique cell IDs in meta.data
- When reliable marker knowledge exists and the goal is strictly maximizing accuracy
  — the article shows prior marker knowledge improves annotation accuracy, so marker-informed
  strategies may be preferable
edam_operation: http://edamontology.org/operation_3800
edam_topics:
- http://edamontology.org/topic_2229
tools:
- name: leader (R package)
  role: Implements adaptive_reweighting() for cluster-based and marker-based cell
    selection from a Seurat object
  repo: https://github.com/camlab-bioml/leader
- name: Seurat
  role: 'Clustering engine: cells are clustered with Seurat, then equal numbers of
    cells are sampled per cluster'
- name: scuttle
  role: Computes logcounts and size-factor normalization to prepare expression matrices
    before cell selection
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-024-45198-y
    title: The impacts of active and self-supervised learning on efficient annotation
      of single-cell expression data
  source_content_hash: sha256:62f2b0ee2147e524bfb287d21d5fa4533a2d47019570756c96e8a2d7a1e9698d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/camlab-bioml/leader
  tools:
  - HDCytoData
  - Seurat
  - SingleCellNet
  - SingleR
  - leader
  - leader (R package)
  - scmap
  - scran
  - scuttle
  edam_operation: http://edamontology.org/operation_3800
  edam_topics:
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# cluster-based-adaptive-reweighting-sampling

## Summary

A heuristic cell-selection method (implemented in the leader R package) that clusters all cells with Seurat and samples equal numbers of cells from each cluster to build balanced single-cell training sets, optionally using marker genes to assign clusters to putative cell types.

## When to use

Use when constructing a labeled training set from a single-cell expression dataset (Seurat object with log-normalized expression) where rare cell types must be captured and active learning is impractical or no iterative labeling budget exists.

## When NOT to use

- When an iterative labeling budget exists and active learning (maximum-entropy or lowest-max-probability selection of the 10 most uncertain cells) is feasible — active learning outperformed adaptive reweighting across most datasets in the benchmark
- When the input Seurat object has not been log-normalized and had variable features scaled — leader requires this preprocessing and unique cell IDs in meta.data
- When reliable marker knowledge exists and the goal is strictly maximizing accuracy — the article shows prior marker knowledge improves annotation accuracy, so marker-informed strategies may be preferable

## Inputs

- Processed Seurat object with log-normalized gene expression, scaled variable features, and unique cell IDs in meta.data rownames
- Marker YAML file (mode="marker_based") listing expected cell types with positive and optional negative markers
- Integer n_requested_cells (e.g. 100, 250, or 500 per split in the benchmark)

## Outputs

- Subsampled set of cell IDs constituting a cluster-balanced (or marker-informed) training dataset
- Training sets usable for classifiers such as SingleR, scmap, SingleCellNet, CyTOF-LDA, random forest (caret), and SVM

## How to apply

Start from a processed Seurat object: normalize with NormalizeData, identify variable features (vst, 2000 features), and scale all genes with ScaleData, ensuring unique cell IDs are the rownames of the meta.data slot. Call leader's adaptive_reweighting with the requested number of cells; in cluster-based mode (mode="cluster_based") the function clusters all cells with Seurat and samples the same number of cells from each cluster, while marker-based mode (mode="marker_based") additionally averages user-provided marker gene expression per cell type and cluster to putatively assign clusters before equal sampling. Markers are supplied as a YAML file with 'cell_types', positive markers, and optional negative markers, loaded via get_markers(). The returned cell IDs constitute the more balanced training set, which can then be used to train annotators such as SingleR, scmap, SingleCellNet, or caret random forest. Rationale: the paper found adaptive reweighting is competitive with active learning in some situations and is tailored to capture rare cell types without iterative labeling.

## Related tools

- **leader (R package)** (Implements adaptive_reweighting() for cluster-based and marker-based cell selection from a Seurat object) — https://github.com/camlab-bioml/leader
- **Seurat** (Clustering engine: cells are clustered with Seurat, then equal numbers of cells are sampled per cluster)
- **scuttle** (Computes logcounts and size-factor normalization to prepare expression matrices before cell selection)

## Examples

```
markers <- get_markers(system.file("extdata", "markers.yml", package = "leader")); adaptive_reweighting(seu, markers, n_requested_cells = 40, mode = "marker_based")
```

## Evaluation signals

- The output cell ID list length equals n_requested_cells (e.g. 40 in the README example; 100/250/500 in the benchmark)
- Clustering step (Seurat) must run on a log-normalized Seurat object with scaled variable features; verify NormalizeData/FindVariableFeatures/ScaleData were applied first
- In marker_based mode, each cluster must receive a putative cell type assignment derived from average marker expression before sampling
- Downstream check: balanced accuracy on held-out test cells (50:50 stratified splits, ten per dataset) should be competitive with random selection and active learning baselines

## Limitations

- Active learning outperformed random selection and adaptive reweighting across most datasets in the benchmark; no single cell selection method is consistently best
- There is circularity in how cell type markers are defined, especially for cell line data, where markers derive from differential expression between cell lines
- Ground truth cell type labels were taken from previous studies rather than manual annotation; the manual annotation process was only simulated using existing annotations
- Dropout at the single-cell level could complicate annotation if specific markers are not expressed — a concern for the marker-aware variant

## Evidence

- [methods] Cluster with Seurat then sample equal numbers of cells per cluster (adaptive reweighting): "we cluster with Seurat13. Next, we sampled cells from the resulting clusters such that the same number of cells is taken from each cluster."
- [readme] LEADER is a method to subset cells aiming to capture rare cell types with the goal of creating more balanced datasets.: "(LEADER) is a method to subset cells aiming to capture rare cell types with the goal of creating more balanced datasets."
- [readme] Marker-based mode assigns clusters via marker expression: "In the marker aware version, each cluster is putatively assigned to a cell type using the average expression of marker genes for each expected cell type provided by the user."
- [abstract] Adaptive reweighting is competitive with existing approaches: "We introduce adaptive reweighting, a heuristic procedure tailored to single-cell data—including a marker-aware version—that shows competitive performance with existing approaches."
- [results] Active learning outperforms adaptive reweighting across most datasets: [evidence span withheld — anchor status: not_found]
- [readme] Preprocessing requirement for the Seurat input: "The input seurat oject must be log normalized and the most variable features must be identified and scaled prior to cell selection with LEADER"
