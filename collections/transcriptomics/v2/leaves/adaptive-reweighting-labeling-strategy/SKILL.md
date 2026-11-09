---
name: adaptive-reweighting-labeling-strategy
description: Use when a small labeled training set must be constructed from an imbalanced
  single-cell expression matrix (e.g., snRNA-Seq PDAC SCP1089) and rare cell types
  should be captured for annotation.
when_to_use_negative:
- When ground-truth labels already exist and a truly stratified balanced split is
  possible — the paper simulated manual annotation and relied on existing 'ground
  truth' labels
- When no processed Seurat object with log-normalized, scaled variable features is
  available — the input must be normalized and scaled before selection
- When marker knowledge is unreliable — the paper showed marker files can be corrupted
  and circular (e.g., cell-line markers derived by differential expression between
  the same cell lines)
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: leader
  role: R package implementing the adaptive_reweighting cell-selection heuristic (cluster_based
    and marker_based modes) and get_markers() for reading the marker YAML
  repo: https://github.com/camlab-bioml/leader
- name: Seurat
  role: Clusters all cells (first 30 PCs; resolutions 0.4/0.8/1.2, kNN 10/20/30) whose
    clusters are then subsampled equally
- name: scuttle
  role: Computes logcounts and size-factor normalization of the expression matrix
    prior to selection
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-024-45198-y
    title: The impacts of active and self-supervised learning on efficient annotation
      of single-cell expression data
  source_content_hash: sha256:29661c44b09969c21e8f448eff7fcf3f2aa2d3835d8acc1409aeaabf2270169d
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
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# adaptive-reweighting-labeling-strategy

## Summary

LEADER adaptive reweighting is a heuristic cell-selection strategy for single-cell data that builds a more balanced labeled subsample by clustering all cells and sampling equal numbers per cluster, optionally assigning putative cell types to clusters from user-supplied marker genes before sampling.

## When to use

Use when a small labeled training set must be constructed from an imbalanced single-cell expression matrix (e.g., snRNA-Seq PDAC SCP1089) and rare cell types should be captured for annotation.

## When NOT to use

- When ground-truth labels already exist and a truly stratified balanced split is possible — the paper simulated manual annotation and relied on existing 'ground truth' labels
- When no processed Seurat object with log-normalized, scaled variable features is available — the input must be normalized and scaled before selection
- When marker knowledge is unreliable — the paper showed marker files can be corrupted and circular (e.g., cell-line markers derived by differential expression between the same cell lines)

## Inputs

- Processed, log-normalized Seurat object with unique cell IDs in meta.data rownames
- Number of requested subsample cells (n_requested_cells)
- Marker YAML file listing positive and optional negative markers per expected cell type (marker_based mode)
- Clustering settings (resolution 0.4/0.8/1.2, kNN 10/20/30; first 30 PCs)

## Outputs

- adaptive_reweighting_subsample.csv with selected cell IDs of the requested subsample size
- A roughly balanced subsample across clusters (cluster_based) or putative cell types (marker_based)

## How to apply

Start from a processed Seurat object: normalize logcounts (e.g., with scuttle size factors or Seurat NormalizeData), find variable features (vst, 2000 features), scale all genes, and ensure unique cell IDs as rownames of meta.data. Cluster cells with Seurat on the first 30 principal components, testing resolutions 0.4/0.8/1.2 and kNN settings 10/20/30 as implemented in the leader package. In cluster_based mode, run adaptive_reweighting with mode="cluster_based", sampling the same number of cells from each cluster. For the marker-aware variant, supply a YAML file of positive and optional negative markers per expected cell type (first line must be 'cell_types'), compute for each cluster and cell type the average expression of positive markers minus negative markers, assign each cluster its highest-scoring putative type, and sample equally per assigned type with mode="marker_based". The selected cell IDs are written to adaptive_reweighting_subsample.csv for use as the balanced training set.

## Related tools

- **leader** (R package implementing the adaptive_reweighting cell-selection heuristic (cluster_based and marker_based modes) and get_markers() for reading the marker YAML) — https://github.com/camlab-bioml/leader
- **Seurat** (Clusters all cells (first 30 PCs; resolutions 0.4/0.8/1.2, kNN 10/20/30) whose clusters are then subsampled equally)
- **scuttle** (Computes logcounts and size-factor normalization of the expression matrix prior to selection)

## Examples

```
markers <- get_markers(system.file("extdata", "markers.yml", package = "leader")); adaptive_reweighting(seu, markers, n_requested_cells = 40, mode = "marker_based")
```

## Evaluation signals

- Output file adaptive_reweighting_subsample.csv exists and contains cell IDs equal to n_requested_cells
- Subsampled cell counts are approximately equal across clusters (cluster_based) or across putatively assigned cell types (marker_based)
- Input Seurat object passed sanity checks: log-normalized expression, variable features identified and scaled, unique cell IDs in meta.data rownames
- Marker-based run: every cluster received a putative cell type via max(average positive marker expression − negative marker expression); each expected type in the YAML is matched to at least one cluster for a balanced subsample

## Limitations

- Manual annotation was simulated using existing cell type annotations, not performed; 'ground truth' labels carry inherent ambiguity in single-cell biology
- Circularity in marker definition, especially for cell line data where markers derive from differential expression between the same cell lines
- Dropout at the single-cell level could complicate annotation when specific markers are not expressed; not addressed in the benchmarking
- A marker file for the snRNA-Seq dataset was adapted from personal communication with authors rather than a published resource

## Evidence

- [readme] cluster_based and marker informed modes: "Leader generates subsampled datasets by clustering all cells and selecting the desired number of cells from these clusters."
- [readme] marker-aware assignment: "each cluster is putatively assigned to a cell type using the average expression of marker genes for each expected cell type provided by the user."
- [discussion] Seurat clustering then equal sampling: "To accomplish this, we cluster with Seurat13. Next, we sampled cells from the resulting clusters such that the same number of cells is taken from each cluster."
- [discussion] marker-aware scoring: "the average expression of a set of marker genes provided by the user is calculated for each cell type and cluster individually"
- [readme] preprocessing requirement: "The input seurat oject must be log normalized and the most variable features must be identified and scaled prior to cell selection with LEADER"
- [abstract] competitive performance finding: "We introduce adaptive reweighting, a heuristic procedure tailored to single-cell data—including a marker-aware version—that shows competitive performance with existing approaches."
