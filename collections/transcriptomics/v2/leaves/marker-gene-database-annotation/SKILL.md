---
name: marker-gene-database-annotation
description: Use when annotating clusters or cells in scRNA-seq or spatial (e.g.,
  MERFISH) datasets from human or murine tissues, especially when marker-based annotation
  consistency and interpretability are needed.
when_to_use_negative:
- When a non-marker-based approach is required for the comparison — the paper notes
  no direct comparison was conducted with annotation tools that do not rely on marker
  genes
- When the tissue of interest is absent from the Accordion database — check list_tissues()
  first, as coverage is limited to the 212 human (and murine) tissues in the built-in
  database
- When reliable cell-type calls require comprehensive reference datasets for non-marker
  methods rather than marker evidence
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Cell Marker Accordion
  role: 'Core annotation engine: assigns cell types using consistency-weighted positive
    and negative markers from the built-in healthy/disease Accordion database'
  repo: https://github.com/TebaldiLab/cellmarkeraccordion
- name: R Shiny app
  role: Interactive browser to explore, filter and download the Cell Marker Accordion
    database without programming skills
  repo: https://github.com/TebaldiLab/shiny_cellmarkeraccordion
- name: Seurat
  role: Standard preprocessing (log-normalization, variable-feature selection, PCA,
    Louvain clustering) producing the clustered object fed to accordion()
- name: ontologySimilarity
  role: get_sim_grid computes percentage similarity (0-100) between predicted and
    ground-truth cell types over the Cell Ontology tree for benchmarking
provenance:
  source_task_ids:
  - task_003
  - task_001
  source_papers:
  - doi: 10.1038/s41467-025-60900-4
    title: 'Cell Marker Accordion: interpretable single-cell and spatial omics annotation
      in health and disease'
  source_content_hash: sha256:b4cd1dcaaa0f40c27e3da7a05b22f5293c8c3eda5c05434293ff0bfa654cfe04
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/TebaldiLab/cellmarkeraccordion
  tools:
  - Cell Marker Accordion
  - R
  - R Shiny app
  - SCINA
  - ScType
  - Seurat
  - clustifyR
  - ontologySimilarity
  - scCATCH
  - scSorter
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# marker-gene-database-annotation

## Summary

Annotate single-cell and spatial omics cell populations using consistency-weighted marker gene databases (Cell Marker Accordion), which integrate and harmonize multiple marker sources (e.g., CellMarker2.0, PanglaoDB) with specificity and evidence-consistency scores.

## When to use

Use when annotating clusters or cells in scRNA-seq or spatial (e.g., MERFISH) datasets from human or murine tissues, especially when marker-based annotation consistency and interpretability are needed.

## When NOT to use

- When a non-marker-based approach is required for the comparison — the paper notes no direct comparison was conducted with annotation tools that do not rely on marker genes
- When the tissue of interest is absent from the Accordion database — check list_tissues() first, as coverage is limited to the 212 human (and murine) tissues in the built-in database
- When reliable cell-type calls require comprehensive reference datasets for non-marker methods rather than marker evidence

## Inputs

- Seurat object or raw/normalized count matrix (genes on rows, cells on columns)
- Species (Human or Mouse) and tissue specification
- Optional cluster-to-cell mapping table (cell, cluster) for count-matrix input
- User-provided marker signatures (optional, in place of the built-in database)

## Outputs

- Seurat object with accordion_per_cluster cell-type annotations (or list of annotation results)
- Per-cluster top marker genes ranked by annotation impact score
- Dot plot of top N candidate cell types per cluster with winning annotation highlighted
- Cell Ontology-based relationship of top predicted cell types

## How to apply

Standardize input as a Seurat object or raw/normalized count matrix (genes x cells) preprocessed with the standard Seurat pipeline (NormalizeData, FindVariableFeatures vst nfeatures=2000, ScaleData, RunPCA, FindNeighbors, FindClusters). Load the healthy or disease Accordion database (accordion_marker / disease_accordion_marker), which harmonizes 15,479 marker genes across 728 cell types in 212 human tissues mapped to the Cell Ontology, then run accordion() specifying species and tissue (explore via list_tissues()) with max_n_marker=30 and annotation_resolution='cluster'. The function exploits positive and negative markers weighted by a specificity score (SPs) and an evidence consistency score (ECs) to assign cell types, optionally returning top marker genes and top cell types positioned on the Cell Ontology tree via include_detailed_annotation_info and plot. Judge correctness by verifying every annotated cluster receives a finite cell-type call, that top-impacting markers are biologically plausible for the tissue, and, per the paper, that average percentage similarity to ground-truth labels (ontologySimilarity get_sim_grid over the Cell Ontology, 0-100 scale) is improved (~23% on average) over ScType, SCINA, clustifyR, scCATCH and scSorter.

## Related tools

- **Cell Marker Accordion** (Core annotation engine: assigns cell types using consistency-weighted positive and negative markers from the built-in healthy/disease Accordion database) — https://github.com/TebaldiLab/cellmarkeraccordion
- **R Shiny app** (Interactive browser to explore, filter and download the Cell Marker Accordion database without programming skills) — https://github.com/TebaldiLab/shiny_cellmarkeraccordion
- **Seurat** (Standard preprocessing (log-normalization, variable-feature selection, PCA, Louvain clustering) producing the clustered object fed to accordion())
- **ontologySimilarity** (get_sim_grid computes percentage similarity (0-100) between predicted and ground-truth cell types over the Cell Ontology tree for benchmarking)

## Examples

```
data <- accordion(data, assay="RNA", species="Human", tissue="blood", annotation_resolution="cluster", max_n_marker=30, include_detailed_annotation_info=TRUE, plot=TRUE)
```

## Evaluation signals

- Every annotated cluster/cell receives a finite cell-type call with a valid Cell Ontology-mapped label
- Benchmark percentage similarity (ontologySimilarity get_sim_grid, 0-100 scale) against ground-truth annotations exceeds marker-based baselines, consistent with the reported ~23% average improvement
- Returned top markers per cluster (max_n_marker=30) are biologically consistent with the annotated cell type and tissue
- The winning annotation in the dot plot corresponds to the maximum impact-score cell type for each cluster
- Jaccard checks of the underlying database show expected low cross-database marker overlap (average ~0.08, max 0.13 across seven databases), confirming the harmonization step is active

## Limitations

- Benchmark coverage limited to two liquid and two solid tumors; cancer benchmark expansion planned
- No direct comparison conducted with annotation tools that do not rely on marker genes
- Limited availability of high-quality ground-truth datasets constrains benchmark expansion
- Available marker databases are heterogeneous with low consistency (average pairwise Jaccard ~0.08), so annotation quality depends on the harmonized database's tissue/cell-type coverage

## Evidence

- [readme] automatic annotation and biological interpretation of single-cell populations using consistency-weighted markers: "This tool provides automated annotation and biological interpretation of single-cell populations using consistency-weighted markers."
- [methods] database composition: "Cell Marker Accordion includes a comprehensive collection of 15479 marker genes associated with 728 cell types in 212 human tissues"
- [discussion] benchmark result: "the Cell Marker Accordion improved the identification of cell types compared to other available marker-based annotation tools12,17,19,40,41,64,65, with an average increase of 23%"
- [other] benchmark scoring method: [evidence span withheld — anchor status: not_found]
- [other] workflow steps: [evidence span withheld — anchor status: not_found]
