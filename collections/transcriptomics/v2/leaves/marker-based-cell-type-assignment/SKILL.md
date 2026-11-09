---
name: marker-based-cell-type-assignment
description: Use when you have a log-normalized Seurat object of single-cell (snRNA-seq/RNA-seq)
  expression, a YAML marker file, and need a balanced subsample of cell IDs for manual
  or classifier labeling.
when_to_use_negative:
- No prior marker knowledge is available — use mode = 'cluster_based' instead, sampling
  equal cells per Seurat cluster
- The goal is differential expression or cell-type discovery rather than creating
  a balanced labeled training subsample
- Cells lack unique cell IDs in the meta.data slot, since IDs are what LEADER outputs
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: leader
  role: R package implementing adaptive_reweighting with mode='marker_based'; reads
    marker YAML via get_markers()
  repo: https://github.com/camlab-bioml/leader
- name: Seurat
  role: Clustering cells on principal components prior to marker-informed subsampling;
    NormalizeData/FindVariableFeatures/ScaleData preprocessing
- name: scuttle
  role: Computation of normalized logcounts and size factors for the expression matrix
- name: scran
  role: findMarkers used for marker gene definition in the paper's marker files
- name: PanglaoDB
  role: Source of the 20 most sensitive markers used in marker files for the Tabula
    datasets
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-024-45198-y
    title: The impacts of active and self-supervised learning on efficient annotation
      of single-cell expression data
  source_content_hash: sha256:91d6cd4e80e43af32eccc72933d1735560465d6d29b01ab1b722a8389d0dfc09
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/camlab-bioml/leader
  tools:
  - PanglaoDB
  - Seurat
  - leader
  - scran
  - scuttle
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# marker-based-cell-type-assignment

## Summary

Marker-informed adaptive reweighting (LEADER): assign each cluster of single-cell expression data a putative cell type by comparing average positive vs negative marker gene expression, then sample cells equally per putative type to build a balanced labeled training set.

## When to use

Use when you have a log-normalized Seurat object of single-cell (snRNA-seq/RNA-seq) expression, a YAML marker file, and need a balanced subsample of cell IDs for manual or classifier labeling.

## When NOT to use

- No prior marker knowledge is available — use mode = 'cluster_based' instead, sampling equal cells per Seurat cluster
- The goal is differential expression or cell-type discovery rather than creating a balanced labeled training subsample
- Cells lack unique cell IDs in the meta.data slot, since IDs are what LEADER outputs

## Inputs

- Processed Seurat object (log-normalized, variable features identified and scaled) with unique cell IDs as meta.data rownames
- Marker YAML file ('cell_types' top-level key) with positive and optional negative marker genes per expected cell type

## Outputs

- CSV of selected cell IDs (e.g. adaptive_reweighting_subsample.csv) balanced across putative cell types
- Balanced subsampled training set for downstream annotation/classifier training

## How to apply

Normalize the Seurat object and scale variable features, ensuring unique cell IDs are the rownames of the meta.data slot. Cluster the cells (in the paper: Seurat clustering on the first 30 principal components, testing resolutions 0.4/0.8/1.2 and kNN 10/20/30). Load the marker YAML via get_markers(), where each cell type lists positive markers (e.g. CD3, CD8 for CD8 T cells) and optional negative markers (e.g. B220, CD4); compute, per cluster and cell type, the average expression of positive markers minus negative markers and assign each cluster its highest-scoring putative type. Then run adaptive_reweighting(seu, markers, n_requested_cells=N, mode='marker_based') so the requested number of cells is drawn equally from putatively assigned cell types, and write the selected cell_ids to adaptive_reweighting_subsample.csv. Rationale: the marker-aware version of adaptive reweighting was shown to be competitive with existing active-learning selection approaches and prior marker knowledge improved annotation accuracy.

## Related tools

- **leader** (R package implementing adaptive_reweighting with mode='marker_based'; reads marker YAML via get_markers()) — https://github.com/camlab-bioml/leader
- **Seurat** (Clustering cells on principal components prior to marker-informed subsampling; NormalizeData/FindVariableFeatures/ScaleData preprocessing)
- **scuttle** (Computation of normalized logcounts and size factors for the expression matrix)
- **scran** (findMarkers used for marker gene definition in the paper's marker files)
- **PanglaoDB** (Source of the 20 most sensitive markers used in marker files for the Tabula datasets)

## Examples

```
markers <- get_markers(system.file("extdata", "markers.yml", package = "leader")); adaptive_reweighting(seu, markers, n_requested_cells = 40, mode = "marker_based")
```

## Evaluation signals

- Output CSV contains exactly n_requested_cells unique cell IDs drawn from the meta.data rownames
- Subsample is roughly balanced across the putative cell types assigned to clusters (similar counts per type)
- Each cluster's assigned putative cell type is the one with the highest average positive-minus-negative marker score
- Marker YAML parses via get_markers() with the required 'cell_types' first row and positive (required) / negative (optional) labels
- Downstream classifier trained on the marker-informed subsample outperforms a cluster_based-only subsample, as expected from the paper's finding that marker knowledge improves annotation accuracy

## Limitations

- Circularity in marker definition: for cell line data, markers were derived from differential expression between the cell lines being annotated
- Dropout at single-cell level could complicate annotation if specific cell type markers are not expressed; not addressed in the benchmarking
- Ground truth labels were taken from previous studies; the manual annotation process was simulated rather than performed
- The snRNA-Seq (SCP1089 PDAC) marker file was adapted from personal communication with the original authors rather than a published resource

## Evidence

- [abstract] Abstract: "We introduce adaptive reweighting, a heuristic procedure tailored to single-cell data—including a marker-aware version—that shows competitive performance with existing approaches."
- [discussion] Discussion: "the average expression of a set of marker genes provided by the user is calculated for each cell type and cluster individually"
- [discussion] Discussion (marker-aware sampling): [evidence span withheld — anchor status: not_found]
- [readme] README (leader): "In the marker aware version, each cluster is putatively assigned to a cell type using the average expression of marker genes for each expected cell type provided by the user."
- [readme] README (marker YAML structure): "The first row must contain the word `cell_types`. Each indented element is an expected cell type with required positive labels and optional negative labels."
