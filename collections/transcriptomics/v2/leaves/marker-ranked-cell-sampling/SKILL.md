---
name: marker-ranked-cell-sampling
description: Use when constructing an initial 10–40 cell training set for single-cell
  cell-type annotation and a marker gene file (positive/negative markers per expected
  cell type) is available to guide cell selection.
when_to_use_negative:
- No marker file is available for the expected cell types (e.g. snRNA-Seq markers
  must come from author communication or PanglaoDB) — use cluster_based mode instead
- Markers are unreliable or circular, e.g. derived from differential expression between
  the same cell lines being annotated, or missing due to single-cell dropout
- Input is a marker-corruption scenario where markers are replaced by random highly
  expressed genes — accuracy degrades by design
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: leader
  role: Implements marker-aware adaptive reweighting (adaptive_reweighting with mode='marker_based')
    and YAML marker parsing via get_markers()
  repo: https://github.com/camlab-bioml/leader
- name: Seurat
  role: Normalizes data, finds variable features, scales expression, and clusters
    cells prior to marker-based cluster assignment
- name: caret
  role: Trains the logistic regression and random forest classifiers on the first
    20 PCs of scaled logcount data using the marker-sampled training set
- name: scran
  role: findMarkers used to derive cell type markers, e.g. differential expression
    between a cell line and all others
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-024-45198-y
    title: The impacts of active and self-supervised learning on efficient annotation
      of single-cell expression data
  source_content_hash: sha256:02671f9d39051386cada03c7ae0b5e9357e16a745be6c2f875d786f54851ccf3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/camlab-bioml/leader
  tools:
  - Seurat
  - caret
  - leader
  - scran
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# marker-ranked-cell-sampling

## Summary

Marker-aware adaptive reweighting: cluster a single-cell expression dataset, putatively assign each cluster to a known cell type via average marker-gene expression, and sample equal numbers of cells per cluster to build a balanced initial active-learning training set.

## When to use

Use when constructing an initial 10–40 cell training set for single-cell cell-type annotation and a marker gene file (positive/negative markers per expected cell type) is available to guide cell selection.

## When NOT to use

- No marker file is available for the expected cell types (e.g. snRNA-Seq markers must come from author communication or PanglaoDB) — use cluster_based mode instead
- Markers are unreliable or circular, e.g. derived from differential expression between the same cell lines being annotated, or missing due to single-cell dropout
- Input is a marker-corruption scenario where markers are replaced by random highly expressed genes — accuracy degrades by design

## Inputs

- Processed Seurat object with log-normalized expression, scaled variable features, and unique cell IDs in meta.data
- YAML marker file with positive (and optional negative) marker genes per expected cell type
- Integer number of requested cells (e.g. 40, or 20 in the active-learning benchmark)

## Outputs

- Subsampled set of cell IDs forming a balanced training dataset
- Downstream initial training set for active learning classifiers and cell-type probability/entropy predictions

## How to apply

Start from a processed Seurat object with log-normalized counts, variable features identified (e.g. vst, 2000 features), and all genes scaled, with unique cell IDs in the meta.data rownames. Define expected cell types in a YAML marker file listing positive markers (e.g. CD3/CD8 for CD8 T cells) and optional negative markers (e.g. B220, CD4). Run leader::adaptive_reweighting(seu, markers, n_requested_cells=N, mode="marker_based"): cells are clustered, the average expression of each cell type's marker genes is computed per cluster, clusters are putatively assigned to cell types, and N cells are sampled from the assigned clusters. Use the returned cell_id set as the balanced training data for downstream classifiers (e.g. logistic regression / random forest on 20 PCs via caret); the paper shows marker prior knowledge improves annotation accuracy.

## Related tools

- **leader** (Implements marker-aware adaptive reweighting (adaptive_reweighting with mode='marker_based') and YAML marker parsing via get_markers()) — https://github.com/camlab-bioml/leader
- **Seurat** (Normalizes data, finds variable features, scales expression, and clusters cells prior to marker-based cluster assignment)
- **caret** (Trains the logistic regression and random forest classifiers on the first 20 PCs of scaled logcount data using the marker-sampled training set)
- **scran** (findMarkers used to derive cell type markers, e.g. differential expression between a cell line and all others)

## Examples

```
markers <- get_markers(system.file("extdata", "markers.yml", package = "leader")); adaptive_reweighting(seu, markers, n_requested_cells = 40, mode = "marker_based")
```

## Evaluation signals

- Marker file parses via get_markers() and follows the required YAML structure with a first 'cell_types' row and indented cell types
- Returned cell IDs sample the same number of cells per putatively assigned cluster/cell type (e.g. n_requested_cells=40 yields balanced representation)
- Annotation accuracy of classifiers trained on marker-ranked cells exceeds accuracy from random slot filling of equal size
- Seurat object meets preconditions: log-normalized, variable features scaled, and unique cell IDs as rownames of seu@meta.data

## Limitations

- Circularity in cell type marker definition, especially for cell line data where markers derive from differential expression between the same cell lines
- Dropout at the single-cell level can complicate annotation when specific cell type markers are not expressed
- Ground-truth labels were derived from previous studies rather than manual annotation, so marker assignment quality inherits that ambiguity
- Random corruption of 10–100% of markers (replaced by random highly expressed genes) degrades marker-aware selection

## Evidence

- [readme] In the marker aware version, each cluster is putatively assigned to a cell type using the average expression of marker genes for each expected cell type provided by the user.: "In the marker aware version, each cluster is putatively assigned to a cell type using the average expression of marker genes for each expected cell type provided by the user."
- [discussion] we trained a random forest and a logistic regression classifier using the first 20 principal components of scaled and centered logcount expression data as input using the caret R package: "we trained a random forest and a logistic regression classifier using the first 20 principal components of scaled and centered logcount expression data as input using the caret R package"
- [discussion] the average expression of a set of marker genes provided by the user is calculated for each cell type and cluster individually: "the average expression of a set of marker genes provided by the user is calculated for each cell type and cluster individually"
- [abstract] In addition, we demonstrate that having prior knowledge of cell type markers improves annotation accuracy.: "In addition, we demonstrate that having prior knowledge of cell type markers improves annotation accuracy."
