---
name: single-cell-clustering-based-subsampling
description: Use when a single-cell expression dataset is highly imbalanced and a
  small, balanced subsample of cells must be chosen for manual labeling, with or without
  prior marker gene knowledge.
when_to_use_negative:
- The input is not a processed, log-normalized Seurat object with unique cell IDs
  — LEADER requires this format
- The goal is uncertainty-driven active learning rather than heuristic cluster balancing;
  the article benchmarks those as distinct approaches
- No clustering signal is expected in the data (e.g., very few cell types already
  balanced), making equal-per-cluster sampling unnecessary
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: leader
  role: Implements adaptive_reweighting() for cluster-based and marker-based cell
    selection and get_markers() for reading marker YAML files
  repo: https://github.com/camlab-bioml/leader
- name: Seurat
  role: Clusters all cells (first 30 PCs; tested resolutions 0.4/0.8/1.2, kNN 10/20/30);
    NormalizeData/FindVariableFeatures/ScaleData prepare the object
- name: scuttle
  role: Calculates logcounts and normalizes with size factors prior to cell selection
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-024-45198-y
    title: The impacts of active and self-supervised learning on efficient annotation
      of single-cell expression data
  source_content_hash: sha256:9cca0f9f751d5a6efc80c5965127ea7ca06a94cf29e174759cf91743796f17dd
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/camlab-bioml/leader
  tools:
  - Seurat
  - leader
  - scuttle
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6428571428571429
n_claims: 14
---

# single-cell-clustering-based-subsampling

## Summary

Adaptive reweighting (LEADER) constructs an approximately balanced cell subsample for manual labeling by clustering all cells (Seurat) and sampling an equal number of cells per cluster, or per putative cell type when a marker YAML file is supplied. It is a heuristic to capture rare cell types and reduce cell type imbalance in labeled training sets.

## When to use

Use when a single-cell expression dataset is highly imbalanced and a small, balanced subsample of cells must be chosen for manual labeling, with or without prior marker gene knowledge.

## When NOT to use

- The input is not a processed, log-normalized Seurat object with unique cell IDs — LEADER requires this format
- The goal is uncertainty-driven active learning rather than heuristic cluster balancing; the article benchmarks those as distinct approaches
- No clustering signal is expected in the data (e.g., very few cell types already balanced), making equal-per-cluster sampling unnecessary

## Inputs

- Processed Seurat object with log-normalized expression, scaled variable features, and unique cell IDs in the meta.data rownames
- Optional marker YAML file with 'cell_types' as the first key, positive markers required and negative markers optional per cell type
- Requested subsample size (n_requested_cells)
- Optional size-factor-normalized logcounts computed with the scuttle R package

## Outputs

- CSV of selected cell IDs (adaptive_reweighting_subsample.csv) defining a balanced subsample of the requested size
- Subsample verified to be roughly balanced across clusters or putative cell types

## How to apply

Load a processed Seurat object (log-normalized, variable features identified and scaled, unique cell IDs as rownames of the meta.data slot). Run LEADER with mode='cluster_based' to cluster all cells and sample the same number of cells from each cluster (the article used Seurat on the first 30 PCs, testing resolutions 0.4/0.8/1.2 and kNN 10/20/30). If prior knowledge exists, provide a YAML file of positive and optional negative markers per expected cell type; LEADER scores each cluster by average positive minus negative marker expression, assigns the highest-scoring putative type per cluster, and samples equally per assigned type (mode='marker_based'). Write the selected cell IDs (of the requested subsample size) to adaptive_reweighting_subsample.csv and confirm the subsample is roughly balanced across clusters or putative types.

## Related tools

- **leader** (Implements adaptive_reweighting() for cluster-based and marker-based cell selection and get_markers() for reading marker YAML files) — https://github.com/camlab-bioml/leader
- **Seurat** (Clusters all cells (first 30 PCs; tested resolutions 0.4/0.8/1.2, kNN 10/20/30); NormalizeData/FindVariableFeatures/ScaleData prepare the object)
- **scuttle** (Calculates logcounts and normalizes with size factors prior to cell selection)

## Examples

```
markers <- get_markers(system.file("extdata", "markers.yml", package = "leader")); adaptive_reweighting(seu, markers, n_requested_cells = 40, mode = "marker_based")
```

## Evaluation signals

- Selected cell IDs are unique and their count equals n_requested_cells
- The subsample contains approximately equal numbers of cells per cluster (cluster_based) or per putative cell type (marker_based)
- The seurat object had unique cell IDs as rownames of the meta.data slot so IDs can be exported
- Marker YAML parses via get_markers() and every cluster receives a putative cell type assignment
- Marker-aware scores reflect average positive marker expression minus negative marker expression per cluster and cell type

## Limitations

- Ground truth labels were derived from previous studies rather than performed manual annotation; the manual annotation process was only simulated
- Marker files can be circular, especially for cell line data, as markers were derived from differential expression between cell lines
- Dropout at the single-cell level could complicate annotation if specific cell type markers are not expressed; not addressed in the benchmarking
- The snRNA-Seq marker file was adapted from personal communication with the authors rather than a published resource

## Evidence

- [discussion] we cluster with Seurat13. Next, we sampled cells from the resulting clusters such that the same number of cells is taken from each cluster.: "we cluster with Seurat13. Next, we sampled cells from the resulting clusters such that the same number of cells is taken from each cluster."
- [discussion] the average expression of a set of marker genes provided by the user is calculated for each cell type and cluster individually: "the average expression of a set of marker genes provided by the user is calculated for each cell type and cluster individually"
- [readme] Leader generates subsampled datasets by clustering all cells and selecting the desired number of cells from these clusters.: "Leader generates subsampled datasets by clustering all cells and selecting the desired number of cells from these clusters."
- [readme] In the marker aware version, each cluster is putatively assigned to a cell type using the average expression of marker genes for each expected cell type provided by the user.: "each cluster is putatively assigned to a cell type using the average expression of marker genes for each expected cell type provided by the user."
