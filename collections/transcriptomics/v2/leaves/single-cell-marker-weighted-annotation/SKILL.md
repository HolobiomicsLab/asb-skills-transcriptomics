---
name: single-cell-marker-weighted-annotation
description: Use when single-cell or spatial RNA-seq data (Seurat object or raw/normalized
  count matrix) must be annotated with interpretable, consistency-weighted markers,
  or when disease-critical cell types such as leukemic stem cells must be scored.
when_to_use_negative:
- No suitable tissue, species, or disease cell type is covered by the built-in Accordion
  database and no user-provided marker signature is available
- The goal is malignant-cell identification via copy-number calling rather than marker-based
  annotation (the article did not directly compare against non-marker-based tools
  like SCEVAN/CopyKAT)
- A high-quality, comprehensive reference dataset and label-transfer approach is preferred
  over marker-based annotation
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Cell Marker Accordion
  role: Core R package providing accordion() and accordion_disease() functions with
    the consistency-weighted marker database for annotation
  repo: https://github.com/TebaldiLab/cellmarkeraccordion
- name: Seurat
  role: Pre-processing (NormalizeData, ScaleData, PCA, clustering, UMAP) and visualization
    (DimPlot, FeaturePlot) of the scRNA-seq input
- name: R Shiny
  role: Interactive web app to explore, filter, and download the Accordion marker
    database without programming
  repo: https://github.com/TebaldiLab/shiny_cellmarkeraccordion
- name: data.table
  role: Formatting cluster_info (cell, cluster) data.table when annotating raw count
    matrices
provenance:
  source_task_ids:
  - task_002
  - task_006
  source_papers:
  - doi: 10.1038/s41467-025-60900-4
    title: 'Cell Marker Accordion: interpretable single-cell and spatial omics annotation
      in health and disease'
  source_content_hash: sha256:d19cbc0ad8777f682ff667f9e582a02b459fc0c3adf7cb1f8b75d2e880c363e2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/TebaldiLab/cellmarkeraccordion
  tools:
  - Cell Marker Accordion
  - R
  - R Shiny
  - Seurat
  - data.table
  - ggplot2
  - ontologyIndex
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# single-cell-marker-weighted-annotation

## Summary

Automatic annotation of single-cell and spatial omics cell populations using the Cell Marker Accordion database, whose marker genes are weighted by a specificity score (SPs) and an evidence consistency score (ECs) aggregated across heterogeneous published marker sources.

## When to use

Use when single-cell or spatial RNA-seq data (Seurat object or raw/normalized count matrix) must be annotated with interpretable, consistency-weighted markers, or when disease-critical cell types such as leukemic stem cells must be scored.

## When NOT to use

- No suitable tissue, species, or disease cell type is covered by the built-in Accordion database and no user-provided marker signature is available
- The goal is malignant-cell identification via copy-number calling rather than marker-based annotation (the article did not directly compare against non-marker-based tools like SCEVAN/CopyKAT)
- A high-quality, comprehensive reference dataset and label-transfer approach is preferred over marker-based annotation

## Inputs

- Seurat object or raw/normalized count matrix (genes on rows, cells on columns)
- Built-in Accordion marker database (accordion_marker or disease_accordion_marker, via data())
- Cluster IDs per cell when annotating a raw count matrix (cluster_info data.table)
- Ontology resources: Cell Ontology (2024-08-16), Uberon (2024-09-03), NCI Thesaurus for disease cell types

## Outputs

- Per-cluster or per-cell cell-type annotations stored in the Seurat object (e.g. accordion_per_cluster metadata column)
- Detailed annotation info: top marker genes by impact score, top competing cell types and their ontology relationships
- Per-cell disease scores (e.g. LHSC_per_cell_score, Neoplastic_monocyte_per_cell_score) for export to CSV
- Dot plot of top N cell types per cluster with impact-score-proportional dot size

## How to apply

Install the R package from TebaldiLab/cellmarkeraccordion via devtools::install_github and load the built-in accordion_marker (healthy) or disease_accordion_marker database along with the cell_onto (Cell Ontology 2024-08-16) and uberon_onto (Uberon 2024-09-03) resources. Accept as input either a Seurat object or a raw/normalized count matrix (genes x cells); optionally pre-process with Seurat (NormalizeData, ScaleData, RunPCA, FindNeighbors, FindClusters, RunUMAP) as done for the PBMC 10X example. Run accordion() with species, tissue (explore via list_tissues()), annotation_resolution ('cluster' by default or 'cell'), and max_n_marker (e.g. 30); weights are computed as SPs = 1/(number of cell types gene G marks), ECs = number of sources defining the marker, regularized by log10-transformed ECs and rescaled SPs, with negative-marker evidence subtracted from positive and polarity set by the sign. For disease-critical cells call accordion_disease() with parameters such as NCIT_celltypes='Leukemic Hematopoietic Stem Cell', log2FC_threshold=1, and annotation_resolution='cell'. The function returns the winning cell type per cluster or cell plus, via include_detailed_annotation_info=TRUE, the top impacting marker genes and top competing cell types shown on the cell ontology tree.

## Related tools

- **Cell Marker Accordion** (Core R package providing accordion() and accordion_disease() functions with the consistency-weighted marker database for annotation) — https://github.com/TebaldiLab/cellmarkeraccordion
- **Seurat** (Pre-processing (NormalizeData, ScaleData, PCA, clustering, UMAP) and visualization (DimPlot, FeaturePlot) of the scRNA-seq input)
- **R Shiny** (Interactive web app to explore, filter, and download the Accordion marker database without programming) — https://github.com/TebaldiLab/shiny_cellmarkeraccordion
- **data.table** (Formatting cluster_info (cell, cluster) data.table when annotating raw count matrices)

## Examples

```
data <- accordion(data, assay="RNA", species="Human", tissue="blood", annotation_resolution="cluster", max_n_marker=30, include_detailed_annotation_info=TRUE, plot=TRUE)
```

## Evaluation signals

- SPs values lie in the valid range [0,1] since SPs = 1/(number of cell types the gene marks)
- Final marker weights fall within the reported database range (ECs span 1-23; maximum weight 5.6 in the current database)
- Every input cell receives exactly one winning cell-type annotation (per-cell resolution) or every cluster receives one label (per-cluster resolution)
- Annotation matches expected biology on validated datasets (e.g. PBMC blood cell types; AML LHSC scores enriched in patient cells relative to healthy controls as in Fig. 5C-H)
- list_tissues(species='Human') confirms the requested tissue exists in the database before running accordion()

## Limitations

- Annotation accuracy depends on database coverage: the healthy database holds 15,479 marker genes for 728 cell types across 212 human tissues, so unrepresented tissues/states cannot be annotated
- The disease benchmark covered only two liquid and two solid tumors; expansion is planned
- No direct comparison was conducted with annotation tools that do not rely on marker genes
- High-quality ground-truth datasets remain limited, constraining further benchmarking

## Evidence

- [methods] SPs/ECs definition: "the Cell Marker Accordion database describes each marker with a specificity score (SPs) and an evidence consistency score (ECs)"
- [abstract] automatic annotation goal: "We present the Cell Marker Accordion, a user-friendly platform providing automatic annotation and unmatched biological interpretation of single-cell populations, based on consistency weighted markers."
- [readme] accordion() usage: "data <- accordion(data, assay ="RNA", species ="Human", tissue="blood", annotation_resolution = "cluster", max_n_marker = 30"
- [readme] interpretability of results: "reporting for each group of cells the top marker genes which mostly impacted the annotation, together with the top cell types and their relationship based on the cell ontology tree"
- [discussion] performance claim: "the Cell Marker Accordion improved the identification of cell types compared to other available marker-based annotation tools"
