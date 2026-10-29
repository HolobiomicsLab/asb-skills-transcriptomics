---
name: consistency-weighted-marker-scoring
description: Use when a Seurat object or count matrix from human/murine single-cell
  or spatial data needs automated, interpretable cell type, cell cycle, or pathway
  annotation from consistency-weighted markers.
when_to_use_negative:
- The input is already an annotated feature/cell-type table rather than raw or normalized
  single-cell expression data
- A non-marker-based reference-mapping annotation (e.g. reference-based label transfer
  with a high-quality comprehensive reference) is preferred; the article notes it
  did not directly compare against non-marker-based tools
- The tissue or cell type of interest is absent from the Accordion database's 728
  cell types / 212 human tissues collection, and no custom marker table is supplied
  via accordion_custom()
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: Cell Marker Accordion
  role: R package providing accordion(), accordion_cellcycle(), and accordion_custom()
    functions that perform consistency-weighted marker scoring for cell type, cell
    cycle, and pathway annotation
  repo: https://github.com/TebaldiLab/cellmarkeraccordion
- name: R Shiny
  role: Interactive web application to explore, filter, and download the Cell Marker
    Accordion database without programming skills
  repo: https://rdds.it/CellMarkerAccordion/
- name: Seurat
  role: Standard scRNA-seq pre-processing and clustering (NormalizeData, FindVariableFeatures,
    ScaleData, RunPCA, FindNeighbors, FindClusters, RunUMAP) before annotation; results
    are returned on the Seurat object
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-025-60900-4
    title: 'Cell Marker Accordion: interpretable single-cell and spatial omics annotation
      in health and disease'
  source_content_hash: sha256:6a3e5a1c5e856978b9407ae27fb12a60ef450ce9bf3a9e9a03314e0831ce0418
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/TebaldiLab/cellmarkeraccordion
  tools:
  - Cell Marker Accordion
  - R
  - R Shiny
  - Seurat
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# consistency-weighted-marker-scoring

## Summary

Annotate single-cell and spatial omics populations using the Cell Marker Accordion's consistency-weighted marker database, in which each marker carries a specificity score (SPs) and an evidence consistency score (ECs) measuring agreement across annotation sources.

## When to use

Use when a Seurat object or count matrix from human/murine single-cell or spatial data needs automated, interpretable cell type, cell cycle, or pathway annotation from consistency-weighted markers.

## When NOT to use

- The input is already an annotated feature/cell-type table rather than raw or normalized single-cell expression data
- A non-marker-based reference-mapping annotation (e.g. reference-based label transfer with a high-quality comprehensive reference) is preferred; the article notes it did not directly compare against non-marker-based tools
- The tissue or cell type of interest is absent from the Accordion database's 728 cell types / 212 human tissues collection, and no custom marker table is supplied via accordion_custom()

## Inputs

- Seurat object (processed single-cell or spatial data)
- Raw or normalized count matrix with genes on rows and cells on columns
- Optional data.table of cluster assignments (cell, cluster columns) when annotating a count matrix
- Built-in Accordion marker databases (accordion_marker, disease_accordion_marker) and signature tables (e.g. innate immune response signature)

## Outputs

- Seurat object annotated with consistency-weighted cell type labels (e.g. accordion_per_cluster column)
- Per-cluster/per-cell top marker genes ranked by impact score
- Dot plot of top N cell types per cluster, with dot size proportional to impact score and winning annotation highlighted
- Per-cell cell cycle phase assignments (G0, G1, S, G2M)
- Per-cell pathway signature scores and per-condition/per-cell-type top genes (e.g. stored in Seurat misc slot)

## How to apply

Load a processed Seurat object (or raw/normalized count matrix with genes on rows, cells on columns) and run accordion() with species, tissue (e.g. species="Mouse", tissue="bone marrow"), annotation_resolution="cluster" (or "cell"), max_n_marker=30, and allow_unknown=FALSE to obtain consistency-weighted cell type labels per cluster. The scoring exploits the built-in Accordion database (15,479 marker genes, 728 cell types, 212 human tissues), which weights each marker by its specificity score and an evidence consistency score measuring agreement across source databases such as CellMarker2.0 and PanglaoDB. For pathway-level scoring (e.g. the innate immune response signature in_im_resp_sig), call accordion_custom() with category_column="terms", marker_column="Symbol", annotation_resolution="cell", max_n_marker=NULL, and condition_group_info/celltype_group_info metadata columns to obtain per-cell scores and per-condition top genes. For cell cycle, use accordion_cellcycle() to classify cells into G0, G1, S and G2M phases from the built-in marker collection. Extract annotation results (accordion_per_cluster labels), top marker genes, and per-cell scores from the Seurat object, then compare score distributions between conditions (e.g. WT vs Mettl3 KO, Vehicle vs STM2457).

## Related tools

- **Cell Marker Accordion** (R package providing accordion(), accordion_cellcycle(), and accordion_custom() functions that perform consistency-weighted marker scoring for cell type, cell cycle, and pathway annotation) — https://github.com/TebaldiLab/cellmarkeraccordion
- **R Shiny** (Interactive web application to explore, filter, and download the Cell Marker Accordion database without programming skills) — https://rdds.it/CellMarkerAccordion/
- **Seurat** (Standard scRNA-seq pre-processing and clustering (NormalizeData, FindVariableFeatures, ScaleData, RunPCA, FindNeighbors, FindClusters, RunUMAP) before annotation; results are returned on the Seurat object)

## Examples

```
data <- accordion(data, assay="RNA", species="Mouse", tissue="bone marrow", annotation_resolution="cluster", max_n_marker=30, allow_unknown=FALSE)
```

## Evaluation signals

- Annotated populations reproduce expected biology in the benchmark task: increased stem cell and megakaryocyte fractions, decreased erythroid fractions, increased G0 fraction, and a significant innate immune response score increase in both Mettl3 KO and STM2457 conditions (Fig. 8C,D,G,H,I patterns)
- The function returns consistency-weighted labels per cluster with detailed annotation info: top marker genes by impact score, top cell types, and their relationship on the cell ontology tree when include_detailed_annotation_info=TRUE and plot=TRUE
- Species and tissue parameters resolve to entries in the database (checked via list_tissues(species=...)), and annotation succeeds with allow_unknown=FALSE, i.e. every cluster receives a confident label
- Cell cycle fractions sum to 1 across G0, G1, S, G2M assignments from accordion_cellcycle()
- Per-cell signature scores and the top 5 marker genes per condition and cell type are retrievable from the Seurat misc slot after accordion_custom()

## Limitations

- Non-marker-based annotation methods require high-quality comprehensive reference datasets and are susceptible to technical variations such as differences in experimental platforms or sequencing strategies; the paper did not directly compare Cell Marker Accordion with these approaches
- Marker gene databases are heterogeneous with different marker sets for the same cell type and non-standard nomenclature, which the ECs weighting mitigates but which still depends on the underlying source databases (CellMarker2.0, PanglaoDB, cell-sorting markers)
- Availability of high-quality ground-truth datasets constrains broader benchmarking; the disease benchmark covered only two liquid and two solid tumors

## Evidence

- [methods] the Cell Marker Accordion database describes each marker with a specificity score (SPs) and an evidence consistency score (ECs): "the Cell Marker Accordion database describes each marker with a specificity score (SPs) and an evidence consistency score (ECs)"
- [methods] accordion function usage: "Users can annotate clusters or cells exploiting the built-in Cell Marker Accordion database with the accordion() function"
- [methods] database composition: "Cell Marker Accordion includes a comprehensive collection of 15479 marker genes associated with 728 cell types in 212 human tissues"
- [methods] package capabilities and repo: "The Cell Marker Accordion includes an R package to automatically identify cell type, cell cycle stage and pathway activation in single-cell and spatial omics data"
- [readme] annotation function signature from README: "data <- accordion(data, assay ="RNA", species ="Human", tissue="blood", annotation_resolution = "cluster", max_n_marker = 30, include_detailed_annotation_info = TRUE, plot = TRUE)"
- [discussion] METTL3 case study finding: "The Cell Marker Accordion identified cell cycle changes and activation of immune response pathways in specific hematopoietic cell types, consistent with the formation of aberrant endogenous dsRNAs"
