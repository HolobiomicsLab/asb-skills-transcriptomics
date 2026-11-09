---
name: single-cell-cluster-annotation
description: Use when a preprocessed and clustered Seurat object or count matrix from
  scRNA-seq or imaging-based spatial data (e.g., MERFISH mouse brain) needs cell type
  labels per cluster or per cell, and marker-gene-based annotation with interpretable
  results is desired.
when_to_use_negative:
- 'No marker-gene-independent comparison: the article notes no direct comparison with
  reference-based, non-marker annotation tools, so this skill is not the choice when
  a comprehensive reference atlas mapping is required'
- Tissue not represented in the Accordion database (728 cell types across 212 human
  tissues plus mouse) — check list_tissues() first
- Aberrant/disease cell states require the disease_accordion_marker database, not
  the default healthy accordion_marker database
edam_operation: http://edamontology.org/operation_3565
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0621
tools:
- name: Cell Marker Accordion
  role: R package providing the accordion() function and consistency-weighted marker
    database for automatic annotation
  repo: https://github.com/TebaldiLab/cellmarkeraccordion
- name: Seurat
  role: Preprocessing, normalization, PCA, clustering (FindNeighbors/FindClusters),
    and UMAP visualization prior to annotation
- name: Shiny app
  role: Interactive web application to explore, filter and download the Cell Marker
    Accordion database without programming
  repo: https://rdds.it/CellMarkerAccordion/
- name: ontologySimilarity
  role: get_sim_grid function computes Cell Ontology-tree-based similarity of predicted
    vs ground-truth annotations for benchmarking
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-025-60900-4
    title: 'Cell Marker Accordion: interpretable single-cell and spatial omics annotation
      in health and disease'
  source_content_hash: sha256:1d8e96f3f93338d5ed2a96d8117bb24680cdbc9a7318b59ceb555e52736cda29
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/TebaldiLab/cellmarkeraccordion
  tools:
  - Cell Marker Accordion
  - SCINA
  - ScType
  - Seurat
  - Shiny app
  - clustifyR
  - ontologySimilarity
  - scCATCH
  - scSorter
  edam_operation: http://edamontology.org/operation_3565
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0621
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 14
---

# single-cell-cluster-annotation

## Summary

Automatically annotate cell types in single-cell and spatial omics clusters using consistency-weighted markers from the Cell Marker Accordion database (accordion() function), validated to outperform other marker-based tools (e.g., >13% on MERFISH mouse brain; ~23% average).

## When to use

Use when a preprocessed and clustered Seurat object or count matrix from scRNA-seq or imaging-based spatial data (e.g., MERFISH mouse brain) needs cell type labels per cluster or per cell, and marker-gene-based annotation with interpretable results is desired.

## When NOT to use

- No marker-gene-independent comparison: the article notes no direct comparison with reference-based, non-marker annotation tools, so this skill is not the choice when a comprehensive reference atlas mapping is required
- Tissue not represented in the Accordion database (728 cell types across 212 human tissues plus mouse) — check list_tissues() first
- Aberrant/disease cell states require the disease_accordion_marker database, not the default healthy accordion_marker database

## Inputs

- Seurat object (raw or normalized counts, genes x cells)
- Raw or normalized count matrix with cluster_info mapping cells to clusters
- Preprocessed/clustered scRNA-seq or spatial dataset (e.g., MERFISH mouse brain coronal section)
- Accordion marker database (accordion_marker or disease_accordion_marker)

## Outputs

- Seurat object with accordion_per_cluster annotation column
- Per-cluster winning cell type annotations with top marker genes
- Dot plot of top N cell types per cluster with impact scores and Cell Ontology relationships

## How to apply

Preprocess the data in Seurat (NormalizeData or SCT-transform, PCA, FindNeighbors/FindClusters, UMAP), then load the built-in database with data(accordion_marker) and run accordion() specifying species (Human/Mouse), tissue (check available tissues with list_tissues()), annotation_resolution ('cluster' by default, or 'cell'), and max_n_marker (e.g., 30). The function exploits positive and negative markers weighted by specificity score (SPs) and evidence consistency score (ECs), integrated across source databases via Jaccard similarity, and mapped to the Cell Ontology tree. Input can be a Seurat object, or a raw/normalized count matrix with cluster_info for each cell. For spatial data such as the MERFISH brain section 'Zhuang-ABCA-1.080' (37,068 barcodes), map coordinates to Allen CCF 2020 and annotate clusters with accordion(species='Mouse', tissue='brain'). Benchmark against ScType, SCINA, clustifyR, scCATCH and scSorter by scoring predictions against ground truth via ontology-based similarity (ontologySimilarity get_sim_grid).

## Related tools

- **Cell Marker Accordion** (R package providing the accordion() function and consistency-weighted marker database for automatic annotation) — https://github.com/TebaldiLab/cellmarkeraccordion
- **Seurat** (Preprocessing, normalization, PCA, clustering (FindNeighbors/FindClusters), and UMAP visualization prior to annotation)
- **Shiny app** (Interactive web application to explore, filter and download the Cell Marker Accordion database without programming) — https://rdds.it/CellMarkerAccordion/
- **ontologySimilarity** (get_sim_grid function computes Cell Ontology-tree-based similarity of predicted vs ground-truth annotations for benchmarking)

## Examples

```
data <- accordion(data, assay="RNA", species="Mouse", tissue="brain", annotation_resolution="cluster", max_n_marker=30, include_detailed_annotation_info=TRUE, plot=TRUE)
```

## Evaluation signals

- Predicted labels scored against ground truth via average percentage similarity on the Cell Ontology tree (ontologySimilarity get_sim_grid); Accordion expected to exceed other marker-based tools by ~13% on MERFISH brain spatial data and ~23% on average
- Spatial mapping of winning annotations on tissue coordinates should show high similarity with the original annotation
- accordion() with include_detailed_annotation_info=TRUE returns top marker genes and top cell types per cluster; verify the winning annotation is highlighted and top cell types cluster together on the Cell Ontology tree
- All target tissue/species combinations must be present in list_tissues() output before annotation

## Limitations

- No direct comparison was conducted with annotation tools that do not rely on marker genes; non-marker-based methods require high-quality comprehensive references
- Benchmark improvement claims rest on limited high-quality ground-truth datasets; availability of such datasets constrains benchmark expansion
- Disease-context application (e.g., MDS cohort) derived from a small patient cohort; cancer benchmark limited to two liquid and two solid tumors

## Evidence

- [methods] Automatic annotation via accordion() with cluster resolution and max_n_marker: "Users can annotate clusters or cells exploiting the built-in Cell Marker Accordion database with the accordion() function"
- [discussion] Accordion improved annotation by 23% on average over marker-based tools: "the Cell Marker Accordion improved the identification of cell types compared to other available marker-based annotation tools12,17,19,40,41,64,65, with an average increase of 23%"
- [methods] Validation includes a spatial transcriptomic brain dataset: "we exploited eight single-cell and multi-omics datasets from different healthy tissues along with a spatial transcriptomic brain dataset"
- [readme] README shows concrete accordion() invocation with parameters: "data <- accordion(data, assay ="RNA", species ="Human", tissue="blood", annotation_resolution = "cluster", max_n_marker = 30, include_detailed_annotation_info = TRUE, plot = TRUE)"
- [intro] Marker weighting by specificity and evidence consistency scores: "The Accordion database allows marker genes to be weighted not only by their specificity but also by their evidence consistency score, measuring the agreement of different annotation sources"
