---
name: annotation-consistency-quantification
description: Use when you need to validate an automatic cluster annotation tool by
  computing the fraction of clusters whose predicted cell types match published manual
  annotations across one or more scRNA-seq datasets.
when_to_use_negative:
- You only have raw unclustered scRNA-seq data with no cluster assignments and no
  reference manual annotations to compare against — annotation consistency cannot
  be computed without a ground truth
- Your tissue or species is not represented in the CellMatch reference database (human/mouse),
  since the article states limited reference databases can lead to incorrect annotation
- The goal is cell-level annotation benchmarking against tools like SingleR, CellAssign,
  Garnett, scMap, or CHETAH rather than cluster-level consistency with published annotations
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scCATCH
  role: 'Automatic annotation toolkit: createscCATCH() builds the object, findmarkergene()
    identifies cluster marker genes, findcelltype() assigns top-scoring cell types
    used for consistency scoring'
  repo: https://github.com/ZJUFanLab/scCATCH
- name: Seurat
  role: Used for clustering the scRNA-seq datasets to obtain per-cluster labels prior
    to scCATCH annotation; a Seurat object can be converted directly via createscCATCH(data
    = Seurat_obj[['RNA']]@data, cluster = as.character(Idents(Seurat_obj)))
- name: scCATCH_performance_comparison
  role: Source code and results of the performance comparison process among scCATCH,
    CellAssign, Garnett, SingleR, scMap and CHETAH, and CellMatch database
  repo: https://raw.githack.com/ZJUFanLab/scCATCH_performance_comparison/master/scCATCH/tutorial.html
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.isci.2020.100882
    title: 'scCATCH: Automatic Annotation on Cell Types of Clusters from Single-Cell
      RNA Sequencing Data'
  source_content_hash: sha256:28176d8a2967d330d563148b679bfa2f2dd1b2f28cfe9c74744d3780eb45cf74
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/ZJUFanLab/scCATCH
  repo_url: https://github.com/ZJUFanLab/scCATCH
  tools:
  - Seurat
  - scCATCH
  - scCATCH_performance_comparison
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# annotation-consistency-quantification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Quantify how many clusters an automatic cell-type annotator (scCATCH) labels consistently with published manual annotations across benchmark scRNA-seq datasets, reporting per-dataset consistency rates and a cross-dataset average (67%-100%, average 83% in the scCATCH iScience 2020 study).

## When to use

Use when you need to validate an automatic cluster annotation tool by computing the fraction of clusters whose predicted cell types match published manual annotations across one or more scRNA-seq datasets.

## When NOT to use

- You only have raw unclustered scRNA-seq data with no cluster assignments and no reference manual annotations to compare against — annotation consistency cannot be computed without a ground truth
- Your tissue or species is not represented in the CellMatch reference database (human/mouse), since the article states limited reference databases can lead to incorrect annotation
- The goal is cell-level annotation benchmarking against tools like SingleR, CellAssign, Garnett, scMap, or CHETAH rather than cluster-level consistency with published annotations

## Inputs

- Six published scRNA-seq datasets (Enge, Wu, Lindsey, Zheng, Zeisel, Heng) referenced by the scCATCH performance comparison materials
- Per-cluster labels from clustering analysis (e.g., Seurat Idents)
- Published manual cell type annotations per cluster (ground truth)
- CellMatch tissue-specific cell taxonomy reference database

## Outputs

- Per-cluster automatic cell type annotations (top-scoring cell type per cluster)
- Per-dataset cluster-consistent rate (fraction of matching clusters)
- Aggregated consistency table (Table 2-equivalent, e.g., consistency_table.csv) with per-dataset rates and cross-dataset average

## How to apply

For each published scRNA-seq dataset (e.g., Enge, Wu, Lindsey, Zheng, Zeisel, Heng in the scCATCH study), cluster the data (e.g., with Seurat) and build an scCATCH object with createscCATCH(data = sc_data, cluster = sc_cluster). Identify cluster marker genes with findmarkergene(obj, species, marker, tissue, cancer), using the paper's criteria: log10 fold change >= 0.25, p < 0.05 (Wilcoxon test), and expression in >= 25% of cells. Annotate each cluster with findcelltype(obj), which assigns the top-scoring (top 1) cell type based on validated marker genes matched against the CellMatch database (353 cell types, 686 subtypes, 184 tissue types, 2,096 human/mouse references). Then compare each automatic annotation against the dataset's published manual annotation and compute the cluster-consistent rate per dataset. Aggregate per-dataset rates into a table (analogous to Table 2); correctness is judged by each rate falling within the reported 67%-100% range and the average matching approximately 83%.

## Related tools

- **scCATCH** (Automatic annotation toolkit: createscCATCH() builds the object, findmarkergene() identifies cluster marker genes, findcelltype() assigns top-scoring cell types used for consistency scoring) — https://github.com/ZJUFanLab/scCATCH
- **Seurat** (Used for clustering the scRNA-seq datasets to obtain per-cluster labels prior to scCATCH annotation; a Seurat object can be converted directly via createscCATCH(data = Seurat_obj[['RNA']]@data, cluster = as.character(Idents(Seurat_obj))))
- **scCATCH_performance_comparison** (Source code and results of the performance comparison process among scCATCH, CellAssign, Garnett, SingleR, scMap and CHETAH, and CellMatch database) — https://raw.githack.com/ZJUFanLab/scCATCH_performance_comparison/master/scCATCH/tutorial.html

## Examples

```
obj <- createscCATCH(data = sc_data, cluster = sc_cluster); obj <- findmarkergene(obj, species = 'Human', marker = cellmatch, tissue = 'Pancreas', cancer = 'pancreatic cancer'); obj <- findcelltype(obj)
```

## Evaluation signals

- Each dataset's cluster-consistent rate falls within the reported 67%-100% range
- The six-dataset average consistency rate equals approximately 83% (matching Table 2)
- Every cluster receives exactly one top-scoring (top 1) cell type from findcelltype(obj)
- Marker genes used for annotation satisfy the thresholds log10 fold change >= 0.25, p < 0.05, and >= 25% cell expression
- Output table contains all six datasets (three recorded in CellMatch, three unrecorded) with both internal and external groups summarized

## Limitations

- Performance of scCATCH majorly depends on the reference database; a limited CellMatch database potentially led to incorrect annotation of cell types
- Inadequate clustering analysis introduces errors into annotation, as too many or too few clusters are both problematic for labeling
- Consistency with published manual annotations reflects agreement, not necessarily biological correctness, and effects of multiple clustering algorithms were not yet evaluated

## Evidence

- [abstract] scCATCH accurately annotated 67%-100% (average, 83%) clusters in six published scRNA-seq datasets originating from various tissues.: "scCATCH accurately annotated 67%-100% (average, 83%) clusters in six published scRNA-seq datasets originating from various tissues."
- [intro] genes significantly upregulated in only one cluster (log10 fold change ≥0.25, p < 0.05) and expressed in more than a quarter of cells (≥25%) would be considered marker genes: "genes significantly upregulated in only one cluster (log10 fold change ≥0.25, p < 0.05) and expressed in more than a quarter of cells (≥25%) would be considered marker genes"
- [intro] For each cluster, cell types were scored on the basis of validated marker genes and their supporting literature, and the cell type with the highest score (top 1) was determined for the cluster.: "For each cluster, cell types were scored on the basis of validated marker genes and their supporting literature, and the cell type with the highest score (top 1)"
- [discussion] The performance of scCATCH majorly depends on the reference database. The limited reference database potentially led to incorrect annotation of cell types via scCATCH.: "The performance of scCATCH majorly depends on the reference database. The limited reference database potentially led to incorrect annotation of cell types via scCATCH."
- [readme] obj <- createscCATCH(data = Seurat_obj[['RNA']]@data, cluster = as.character(Idents(Seurat_obj))): "obj <- createscCATCH(data = Seurat_obj[['RNA']]@data, cluster = as.character(Idents(Seurat_obj)))"
