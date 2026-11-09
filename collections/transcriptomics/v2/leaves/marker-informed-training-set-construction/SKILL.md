---
name: marker-informed-training-set-construction
description: Use when building a labeled training set for single-cell annotation without
  active learning, but prior knowledge of cell type marker genes (e.g. PanglaoDB markers)
  is available to guide sampling.
when_to_use_negative:
- No prior marker knowledge exists for the expected cell types — use cluster-based
  adaptive reweighting or active learning instead
- The marker file is circular or unreliable, e.g. cell-line markers derived by differential
  expression between the very cell lines being annotated
- The goal is to benchmark annotation tools themselves rather than to construct a
  training set
edam_operation: http://edamontology.org/operation_3800
edam_topics:
- http://edamontology.org/topic_2229
- http://edamontology.org/topic_3308
tools:
- name: leader
  role: R package implementing marker-informed adaptive reweighting; reads markers.yml
    via get_markers() and samples equal cells per putatively assigned cluster via
    adaptive_reweighting(mode='marker_based')
  repo: https://github.com/camlab-bioml/leader
- name: Seurat
  role: Clusters all cells prior to equal sampling from clusters
- name: scran
  role: findMarkers used for differential-expression-based marker derivation between
    cell lines
- name: scuttle
  role: Computes logcounts and size-factor normalization of expression matrices before
    cell selection
- name: PanglaoDB
  role: Source of the 20 most sensitive markers used to build marker files for the
    Tabula datasets
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-024-45198-y
    title: The impacts of active and self-supervised learning on efficient annotation
      of single-cell expression data
  source_content_hash: sha256:2743bce0ab46c46cee2d98e7bcc26162c1a521b2a33d0cf014c1c0bfc516be27
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/camlab-bioml/leader
  tools:
  - HDCytoData
  - PanglaoDB
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
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# marker-informed-training-set-construction

## Summary

Constructs balanced single-cell training sets by clustering all cells (Seurat), assigning each cluster to an expected cell type via user-provided marker genes, and sampling equal numbers of cells per putatively assigned cluster — an adaptive reweighting alternative to active learning.

## When to use

Use when building a labeled training set for single-cell annotation without active learning, but prior knowledge of cell type marker genes (e.g. PanglaoDB markers) is available to guide sampling.

## When NOT to use

- No prior marker knowledge exists for the expected cell types — use cluster-based adaptive reweighting or active learning instead
- The marker file is circular or unreliable, e.g. cell-line markers derived by differential expression between the very cell lines being annotated
- The goal is to benchmark annotation tools themselves rather than to construct a training set

## Inputs

- Processed, log-normalized Seurat object with unique cell IDs as rownames of the meta.data slot
- Marker YAML file with cell_types: entries containing positive and optional negative markers (e.g. PanglaoDB-derived markers.yml)
- Expression matrices normalized with scuttle (logcounts, size factors)
- Ground-truth cell type labels for evaluation
- Marker files for CyTOF data (protein markers)

## Outputs

- Subsampled training sets of cell IDs (e.g. 100/250/500 labeled cells) balanced across putatively assigned cell types
- Balanced accuracy and sensitivity on held-out test cells per dataset x classifier x selection-method combination
- Aggregated median balanced accuracy and sensitivity ranks per selection method (e.g. fig3b_selection_method_ranks.csv)

## How to apply

Start from a log-normalized Seurat object with unique cell IDs in the meta.data rownames and a YAML marker file listing positive (and optional negative) markers per expected cell type (e.g. the 20 most sensitive PanglaoDB markers for Tabula datasets, author-provided markers for snRNA-Seq). Cluster all cells with Seurat, compute the average marker gene expression per cell type and cluster to putatively assign each cluster to a cell type, then sample the same number of cells from each putatively assigned cluster (e.g. 100, 250 or 500 cells total per 50:50 stratified train/test split). Run this via the leader R package with mode='marker_based', reading markers with get_markers(), and compare against cluster-based adaptive reweighting, maximum-entropy/lowest-max-probability active learning, and random selection by training classifiers (SingleR, scmap, SingleCellNet, random forest via caret on 20 PCs, SVM, CyTOF-LDA) and measuring balanced accuracy and sensitivity on held-out test cells.

## Related tools

- **leader** (R package implementing marker-informed adaptive reweighting; reads markers.yml via get_markers() and samples equal cells per putatively assigned cluster via adaptive_reweighting(mode='marker_based')) — https://github.com/camlab-bioml/leader
- **Seurat** (Clusters all cells prior to equal sampling from clusters)
- **scran** (findMarkers used for differential-expression-based marker derivation between cell lines)
- **scuttle** (Computes logcounts and size-factor normalization of expression matrices before cell selection)
- **PanglaoDB** (Source of the 20 most sensitive markers used to build marker files for the Tabula datasets)

## Examples

```
markers <- get_markers(system.file("extdata", "markers.yml", package = "leader")); adaptive_reweighting(seu, markers, n_requested_cells = 40, mode = "marker_based")
```

## Evaluation signals

- Marker YAML parses correctly with get_markers() and the first line contains the word 'cell_types'
- Output cell IDs are unique and drawn from each putatively assigned cluster in equal numbers matching n_requested_cells
- Trained classifiers reach balanced accuracy competitive with active learning across the ten 50:50 stratified train/test splits
- Median sensitivity/balanced-accuracy ranks of marker-informed selection are reproducible across the six datasets and classifiers (Fig. 3B pattern in fig3b_selection_method_ranks.csv)
- Seurat object preconditions hold: log-normalized expression, variable features scaled, unique cell IDs in meta.data

## Limitations

- Marker corruption testing showed accuracy degrades when 10–100% of markers are replaced with random highly expressed genes
- Circularity in marker definition for cell line data, since markers derive from differential expression between the cell lines themselves
- Ground truth labels were taken from previous studies rather than manual annotation, and annotation was simulated using existing labels; investigators were not blinded and experiments were not randomized
- Marker dropout at single-cell level can complicate annotation when specific markers are not expressed

## Evidence

- [readme] In the marker aware version, each cluster is putatively assigned to a cell type using the average expression of marker genes for each expected cell type provided by the user.: "In the marker aware version, each cluster is putatively assigned to a cell type using the average expression of marker genes"
- [discussion] we calculated logcounts and normalized with the size factor using the scuttle R package52: "we calculated logcounts and normalized with the size factor using the scuttle R package"
- [discussion] the marker files for the tabula datasets were created based on the 20 most sensitive markers from the PanglaoDB database45: "the marker files for the tabula datasets were created based on the 20 most sensitive markers from the PanglaoDB database"
- [discussion] the average expression of a set of marker genes provided by the user is calculated for each cell type and cluster individually: "the average expression of a set of marker genes provided by the user is calculated for each cell type and cluster individually"
- [discussion] randomly corrupted 10, 25, 50, 75 and 100% of markers. Each of these randomly selected markers was replaced with a random gene not in the original marker file: "randomly corrupted 10, 25, 50, 75 and 100% of markers"
