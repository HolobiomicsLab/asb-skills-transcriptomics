---
name: single-cell-rna-seq-cell-type-annotation
description: Use when raw or normalized scRNA-seq/spatial count data (Seurat object
  or count matrix) needs automated cell type labels, and marker-based annotation with
  interpretable top marker genes per cluster is required.
when_to_use_negative:
- The annotation target requires non-marker-based reference mapping or direct comparison
  with such tools — the article explicitly did not compare with annotation tools that
  do not rely on marker genes
- No high-quality ground truth is needed and a reference-based method is preferred;
  the article notes non-marker-based methods require comprehensive reference datasets
  and are susceptible to technical variation
- The tissue of interest is absent from the built-in Accordion database and no user-provided
  signature is available (check with list_tissues() first)
edam_operation: http://edamontology.org/operation_3744
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2229
tools:
- name: Cell Marker Accordion
  role: Primary R package performing automatic annotation via consistency-weighted
    markers from its built-in database (accordion(), accordion_cellcycle(), accordion_custom()
    functions)
  repo: https://github.com/TebaldiLab/cellmarkeraccordion
- name: Seurat
  role: Preprocessing, normalization, PCA, Louvain clustering and UMAP of the single-cell
    data before annotation; hosts annotation results in Seurat object metadata/misc
    slots
- name: R Shiny
  role: Interactive web application for exploring, filtering and downloading the Cell
    Marker Accordion database without programming skills
  repo: TebaldiLab/shiny_cellmarkeraccordion
- name: ontologySimilarity
  role: Computes percentage similarity between predicted and ground-truth cell types
    per cluster over the Cell Ontology tree for benchmarking (get_sim_grid, v2.7)
- name: ScType
  role: Marker-based baseline annotation tool run with its in-built marker database
    for comparison in the benchmark
- name: scCATCH
  role: Marker-based baseline annotation tool run with its in-built marker database
    for comparison in the benchmark
- name: SCINA
  role: Baseline annotator using positive markers from the Accordion database as input,
    capped at 100 marker genes per cell type
- name: clustifyR
  role: Baseline annotator using positive markers from the Accordion database as input
- name: scSorter
  role: Baseline annotator using positive markers from the Accordion database as input;
    excluded ('Timeout') if runtime exceeds 3 hours
merged_aliases:
- scrna-seq-cell-type-annotation
merged_alias_records:
- alias: single-cell-rna-seq-cell-type-annotation-benchmarking
  slug: single-cell-rna-seq-cell-type-annotation-benchmarking
  jaccard_score: 0.8571
  method: token-set-jaccard
  decision: auto
provenance:
  source_task_ids:
  - task_007
  - task_003
  source_papers:
  - doi: 10.1038/s41467-025-60900-4
    title: 'Cell Marker Accordion: interpretable single-cell and spatial omics annotation
      in health and disease'
  source_content_hash: sha256:0093095e53e9cc42f5d494bfdd4a7117a415f09d38e449a438a7ad362fe56108
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/TebaldiLab/cellmarkeraccordion
  tools:
  - ACTINN
  - Cell BLAST
  - Cell Marker Accordion
  - CellMarker2.0
  - CellTypist
  - MarkerCount
  - Python
  - R
  - R Shiny
  - R4scSHARP
  - SCANPY
  - SCINA
  - SELINA
  - ScType
  - Seurat
  - SingleCellNet
  - SingleR
  - clustifyR
  - conda
  - mtSC
  - numpy
  - ontologySimilarity
  - pandas
  - pip
  - scCATCH
  - scSHARP
  - scSorter
  - scaLR
  - scikit-learn
  - scipy
  - scmap
  edam_operation: http://edamontology.org/operation_3744
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 16
---

# single-cell-rna-seq-cell-type-annotation

## Summary

Automated cell type annotation of single-cell and spatial omics data using the Cell Marker Accordion, which exploits a curated database of consistency-weighted marker genes (specificity score SPs and evidence consistency score ECs) to annotate clusters or cells with interpretable outputs.

## When to use

Use when raw or normalized scRNA-seq/spatial count data (Seurat object or count matrix) needs automated cell type labels, and marker-based annotation with interpretable top marker genes per cluster is required.

## When NOT to use

- The annotation target requires non-marker-based reference mapping or direct comparison with such tools — the article explicitly did not compare with annotation tools that do not rely on marker genes
- No high-quality ground truth is needed and a reference-based method is preferred; the article notes non-marker-based methods require comprehensive reference datasets and are susceptible to technical variation
- The tissue of interest is absent from the built-in Accordion database and no user-provided signature is available (check with list_tissues() first)

## Inputs

- Seurat object, or raw/normalized count matrix (genes on rows, cells on columns)
- Cluster IDs per cell (cluster_info) when using a raw count matrix
- Species and tissue specification (e.g. 'Human'/'blood', 'Mouse'/'bone marrow')
- Optional: user-provided marker signature table for accordion_custom()
- Pre-processed clusters from Seurat (log-normalization, vst variable features, PCA, Louvain clustering)

## Outputs

- Per-cluster or per-cell cell type annotations (e.g. accordion_per_cluster metadata column)
- Top marker genes that impacted each annotation (stored in Seurat misc slot)
- Dot plot of top N cell types per cluster with impact scores and winning annotation highlighted
- Cell cycle phase assignments (G0, G1, S, G2M) from accordion_cellcycle()
- Per-cell pathway/signature scores and condition-specific top genes from accordion_custom()

## How to apply

Preprocess the data with a standard Seurat pipeline (NormalizeData, FindVariableFeatures with vst/2000 features, ScaleData, PCA, FindNeighbors, FindClusters, RunUMAP). Run list_tissues() to check tissue availability in the built-in Accordion database (15479 marker genes, 728 cell types, 212 human tissues), then call accordion() with species, tissue, annotation_resolution='cluster' (or 'cell'), max_n_marker=30, and include_detailed_annotation_info=TRUE to obtain consistency-weighted labels plus top marker genes and top cell types per cluster. For custom signatures (e.g. innate immune response), use accordion_custom() with category_column='terms' and marker_column='Symbol'; for cell cycle, use accordion_cellcycle(). Decide between cluster and cell resolution based on whether per-cluster or per-cell scores are needed, and choose allow_unknown=FALSE when all populations must receive a label. Store annotations in the Seurat object (e.g. accordion_per_cluster) and validate against ground truth via ontology-aware similarity (ontologySimilarity get_sim_grid over the Cell Ontology tree, 0-100 scale).

## Related tools

- **Cell Marker Accordion** (Primary R package performing automatic annotation via consistency-weighted markers from its built-in database (accordion(), accordion_cellcycle(), accordion_custom() functions)) — https://github.com/TebaldiLab/cellmarkeraccordion
- **Seurat** (Preprocessing, normalization, PCA, Louvain clustering and UMAP of the single-cell data before annotation; hosts annotation results in Seurat object metadata/misc slots)
- **R Shiny** (Interactive web application for exploring, filtering and downloading the Cell Marker Accordion database without programming skills) — TebaldiLab/shiny_cellmarkeraccordion
- **ontologySimilarity** (Computes percentage similarity between predicted and ground-truth cell types per cluster over the Cell Ontology tree for benchmarking (get_sim_grid, v2.7))
- **ScType** (Marker-based baseline annotation tool run with its in-built marker database for comparison in the benchmark)
- **scCATCH** (Marker-based baseline annotation tool run with its in-built marker database for comparison in the benchmark)
- **SCINA** (Baseline annotator using positive markers from the Accordion database as input, capped at 100 marker genes per cell type)
- **clustifyR** (Baseline annotator using positive markers from the Accordion database as input)
- **scSorter** (Baseline annotator using positive markers from the Accordion database as input; excluded ('Timeout') if runtime exceeds 3 hours)

## Examples

```
data <- accordion(data, assay="RNA", species="Human", tissue="blood", annotation_resolution="cluster", max_n_marker=30, include_detailed_annotation_info=TRUE, plot=TRUE)
```

## Evaluation signals

- Annotation performance computed as average percentage similarity between predicted and ground-truth cell types per cluster via ontologySimilarity get_sim_grid over the Cell Ontology tree (0-100 scale), with every table cell carrying a finite score
- Cell Marker Accordion should achieve the highest annotation performance on healthy-tissue datasets, consistent with the reported average 23% improvement over ScType, SCINA, clustifyR, scCATCH and scSorter
- The output Seurat object must contain the annotation column (e.g. accordion_per_cluster) covering every cluster, with top marker genes stored in the misc slot when include_detailed_annotation_info=TRUE
- In application tasks (e.g. Mettl3 KO / STM2457 bone marrow), annotated populations and condition comparisons should reproduce reported patterns: significant innate immune response score increase, higher G0 fraction, and expected lineage shifts (increased stem cell and megakaryocyte fractions, decreased erythroid fractions)
- Timing comparison on the same machine with all tools configured to use 4 threads should complete without timeout for Accordion

## Limitations

- Marker databases are heterogeneous with non-standard nomenclature; results depend on the built-in Accordion database harmonization (CellMarker2.0, PanglaoDB, cell-sorting markers mapped to Cell Ontology 2024-08-16)
- The cancer benchmark covered only two liquid and two solid tumors; benchmark expansion is planned
- No direct comparison was conducted with annotation tools that do not rely on marker genes (e.g. CNV-based SCEVAN, CopyKAT)
- Availability of high-quality ground-truth datasets constrains benchmark expansion
- All multi-allelic sites are ignored in the related VarTrix variant-extraction workflow

## Evidence

- [methods] Users can annotate clusters or cells exploiting the built-in Cell Marker Accordion database with the accordion() function: "Users can annotate clusters or cells exploiting the built-in Cell Marker Accordion database with the accordion() function"
- [methods] database composition: "Cell Marker Accordion includes a comprehensive collection of 15479 marker genes associated with 728 cell types in 212 human tissues"
- [discussion] benchmark result: "the Cell Marker Accordion improved the identification of cell types compared to other available marker-based annotation tools12,17,19,40,41,64,65, with an average increase of 23%"
- [intro] database weighting rationale: "The Accordion database allows marker genes to be weighted not only by their specificity but also by their evidence consistency score, measuring the agreement of different annotation sources"
- [readme] README usage: "data <- accordion(data, assay ="RNA", species ="Human", tissue="blood", annotation_resolution = "cluster", max_n_marker = 30, include_detailed_annotation_info = TRUE, plot = TRUE)"
- [readme] README preprocessing: [evidence span withheld — anchor status: not_found]
