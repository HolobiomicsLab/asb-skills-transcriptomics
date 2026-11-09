---
name: benchmark-performance-table-assembly
description: Use when comparing cell-type annotation tools across benchmark single-cell
  datasets with reliable ground truth, to produce a comparable performance/runtime
  table.
when_to_use_negative:
- When no datasets with reliable ground-truth annotations are available — the paper
  notes limited availability of high-quality ground-truth datasets constrains benchmark
  expansion
- When comparing against non-marker-based annotation tools — the study explicitly
  did not conduct a direct comparison with such approaches
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0089
tools:
- name: Cell Marker Accordion
  role: Method under evaluation; also supplies positive markers for baseline tools
  repo: https://github.com/TebaldiLab/cellmarkeraccordion
- name: ScType
  role: Marker-based baseline annotation tool run with in-built database and defaults
- name: SCINA
  role: Baseline run with Accordion positive markers, max 100 marker genes per cell
    type
- name: clustifyR
  role: Baseline run with Accordion positive markers
- name: scCATCH
  role: Baseline run with in-built marker database and defaults
- name: scSorter
  role: Baseline run with Accordion positive markers; excluded as 'Timeout' if >3
    h
- name: ontologySimilarity
  role: Provides get_sim_grid to compute Cell Ontology-based percentage similarity
    per cluster
- name: Seurat
  role: Standard preprocessing pipeline (log-normalization, variable features, PCA,
    Louvain clustering) and ground-truth handling
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-025-60900-4
    title: 'Cell Marker Accordion: interpretable single-cell and spatial omics annotation
      in health and disease'
  source_content_hash: sha256:f2723e4493d4103637a9ad515a3667c29ea87072d3aad9a22c8fb19d8ff33179
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/TebaldiLab/cellmarkeraccordion
  tools:
  - Cell Marker Accordion
  - R
  - SCINA
  - ScType
  - Seurat
  - clustifyR
  - ontologySimilarity
  - scCATCH
  - scSorter
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0089
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# benchmark-performance-table-assembly

## Summary

Assembles a per-tool, per-dataset benchmark table of annotation performance (Cell Ontology-based percentage similarity, 0–100) and runtime for the Cell Marker Accordion versus ScType, SCINA, clustifyR, scCATCH and scSorter, quantifying the ~23% average improvement.

## When to use

Use when comparing cell-type annotation tools across benchmark single-cell datasets with reliable ground truth, to produce a comparable performance/runtime table.

## When NOT to use

- When no datasets with reliable ground-truth annotations are available — the paper notes limited availability of high-quality ground-truth datasets constrains benchmark expansion
- When comparing against non-marker-based annotation tools — the study explicitly did not conduct a direct comparison with such approaches

## Inputs

- Benchmark scRNA-seq/CITE-seq datasets with ground-truth cell-type annotations (PBMC FACS-sorted, bone marrow AbSeq, E-HCAD-4, pancreas, liver, lung, kidney)
- Seurat-processed objects with Louvain clusters
- Cell Ontology (release 2024-08-16) tree
- Positive marker lists from the Cell Marker Accordion database
- Per-run execution times (4 threads per tool)

## Outputs

- Per-tool, per-dataset table of percentage similarity scores (0–100) between predicted and ground-truth cell types
- Per-tool, per-dataset runtime table
- Average improvement estimate of Cell Marker Accordion over marker-based baselines (~23%)

## How to apply

Preprocess each benchmark dataset (PBMC FACS-sorted, CITE-seq/AbSeq bone marrow, E-HCAD-4, pancreas, liver, lung, kidney) with the standard Seurat pipeline and treat original annotations as ground truth, mapping nomenclature to the Cell Ontology. Run Cell Marker Accordion with max_n_marker=30 and tissue specified; run ScType and scCATCH with in-built databases at defaults, and SCINA, clustifyR and scSorter with positive markers from the Accordion database (SCINA capped at 100 markers/cell type; exclude scSorter as 'Timeout' if >3 h). For each cluster compute the average percentage similarity between predicted and ground-truth cell types with get_sim_grid from ontologySimilarity (v2.7) over the Cell Ontology tree, on a 0–100 scale. Record runtimes on the same machine with all tools configured for 4 threads. Assemble a per-tool, per-dataset table of similarity and runtime and compute the Accordion's average improvement over the marker-based baselines.

## Related tools

- **Cell Marker Accordion** (Method under evaluation; also supplies positive markers for baseline tools) — https://github.com/TebaldiLab/cellmarkeraccordion
- **ScType** (Marker-based baseline annotation tool run with in-built database and defaults)
- **SCINA** (Baseline run with Accordion positive markers, max 100 marker genes per cell type)
- **clustifyR** (Baseline run with Accordion positive markers)
- **scCATCH** (Baseline run with in-built marker database and defaults)
- **scSorter** (Baseline run with Accordion positive markers; excluded as 'Timeout' if >3 h)
- **ontologySimilarity** (Provides get_sim_grid to compute Cell Ontology-based percentage similarity per cluster)
- **Seurat** (Standard preprocessing pipeline (log-normalization, variable features, PCA, Louvain clustering) and ground-truth handling)

## Examples

```
library(ontologySimilarity); sim <- get_sim_grid(ont = cellos_tree, grid1 = predicted_labels, grid2 = ground_truth_labels)
```

## Evaluation signals

- Every table cell carries a finite similarity score on the 0–100 scale from get_sim_grid over the Cell Ontology tree
- Cell Marker Accordion attains the highest annotation performance on the healthy-tissue datasets
- Aggregated improvement over the other marker-based tools is approximately 23%, matching the reported result
- All tools were timed on the same machine with 4 threads, so runtime cells are directly comparable
- Ground-truth nomenclature was mapped to the Cell Ontology before scoring, so similarity scores are ontology-consistent

## Limitations

- scSorter may exceed the 3-hour limit and is recorded as 'Timeout' rather than scored
- Benchmark limited to datasets with reliable ground truth; high-quality ground-truth datasets are scarce
- No direct comparison with non-marker-based annotation approaches was conducted in this study
- Percentage similarity over the Cell Ontology tree is a proxy metric, not direct label accuracy

## Evidence

- [discussion] the Cell Marker Accordion improved the identification of cell types compared to other available marker-based annotation tools12,17,19,40,41,64,65, with an average increase of 23%: "the Cell Marker Accordion improved the identification of cell types compared to other available marker-based annotation tools12,17,19,40,41,64,65, with an average increase of 23%"
- [methods] we exploited eight single-cell and multi-omics datasets from different healthy tissues along with a spatial transcriptomic brain dataset: "we exploited eight single-cell and multi-omics datasets from different healthy tissues along with a spatial transcriptomic brain dataset"
- [methods] Single-cell datasets were analyzed employing a standard pre-processing pipeline for single-cell RNA-seq data, using Seurat: "Single-cell datasets were analyzed employing a standard pre-processing pipeline for single-cell RNA-seq data, using Seurat"
- [intro] The Accordion database allows marker genes to be weighted not only by their specificity but also by their evidence consistency score: "The Accordion database allows marker genes to be weighted not only by their specificity but also by their evidence consistency score, measuring the agreement of different annotation sources"
- [readme] data <- accordion(data, assay ="RNA", species ="Human", tissue="blood", annotation_resolution = "cluster", max_n_marker = 30: "data <- accordion(data, assay ="RNA", species ="Human", tissue="blood", annotation_resolution = "cluster", max_n_marker = 30, include_detailed_annotation_info = TRUE, plot = TRUE)"
