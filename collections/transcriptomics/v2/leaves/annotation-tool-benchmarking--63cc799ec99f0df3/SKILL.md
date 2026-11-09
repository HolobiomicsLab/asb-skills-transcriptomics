---
name: annotation-tool-benchmarking
description: Use when evaluating or comparing cell type annotation methods on single-cell
  or spatial omics data that has a ground-truth annotation, e.g. a MERFISH mouse brain
  section with CCF-mapped labels.
when_to_use_negative:
- The goal is annotation itself on new data, not method comparison — run accordion()
  directly rather than a benchmarking harness.
- The comparison target is non-marker-based reference-mapping methods — the source
  study explicitly did not compare with tools that do not rely on marker genes.
- No high-quality ground-truth annotation exists for the dataset, making ontology-similarity
  scoring impossible.
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Cell Marker Accordion
  role: Primary annotation method under evaluation; also supplies positive markers
    to baseline user-input tools
  repo: https://github.com/TebaldiLab/cellmarkeraccordion
- name: Seurat
  role: 'Preprocessing: SCT-transform normalization, PCA, FindNeighbors/FindClusters
    (Louvain), UMAP for both tested and baseline pipelines'
- name: ScType
  role: Baseline marker-based annotation tool run with default parameters
- name: SCINA
  role: Baseline annotation tool, run with max 100 markers per cell type from the
    Cell Marker Accordion database
- name: clustifyR
  role: Baseline annotation tool run with default parameters using Accordion-derived
    positive markers
- name: scCATCH
  role: Baseline annotation tool run with default parameters
- name: scSorter
  role: Baseline annotation tool run with default parameters
- name: ontologySimilarity
  role: 'Scoring: get_sim_grid computes Cell Ontology tree similarity between predicted
    and ground-truth annotations (v2.7)'
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-025-60900-4
    title: 'Cell Marker Accordion: interpretable single-cell and spatial omics annotation
      in health and disease'
  source_content_hash: sha256:529f99cf6d7bf18015169189e80c7cbf7a392618828db70d822d6b532479c063
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/TebaldiLab/cellmarkeraccordion
  tools:
  - Cell Marker Accordion
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
yes_rate: 1.0
n_claims: 16
---

# annotation-tool-benchmarking

## Summary

A method for benchmarking marker-based single-cell and spatial annotation tools against a shared preprocessed dataset, scoring predicted versus ground-truth cell types using Cell Ontology similarity. In the source study, the Cell Marker Accordion was compared with ScType, SCINA, clustifyR, scCATCH and scSorter on a MERFISH mouse brain dataset.

## When to use

Use when evaluating or comparing cell type annotation methods on single-cell or spatial omics data that has a ground-truth annotation, e.g. a MERFISH mouse brain section with CCF-mapped labels.

## When NOT to use

- The goal is annotation itself on new data, not method comparison — run accordion() directly rather than a benchmarking harness.
- The comparison target is non-marker-based reference-mapping methods — the source study explicitly did not compare with tools that do not rely on marker genes.
- No high-quality ground-truth annotation exists for the dataset, making ontology-similarity scoring impossible.

## Inputs

- MERFISH mouse brain spatial dataset (section 'Zhuang-ABCA-1.080', 37,068 barcodes) with per-cell spatial coordinates
- Ground-truth cell type annotation per cell/cluster
- NIfTI CCF registration images (0.01 mm voxel spacing) for Allen CCF 2020 mapping
- Marker gene database (Cell Marker Accordion database built from CellMarker2.0 and PanglaoDB)
- Cell Ontology tree (release 2024-08-16)

## Outputs

- Per-tool performance comparison table (e.g. merfish_annotation_performance.csv) with average ontology similarity scores
- Spatial plot of winning annotations on tissue coordinates (e.g. merfish_spatial_annotation.png)
- Per-cluster/cell type annotations from each tool

## How to apply

Preprocess all candidate tools on the same object: in Seurat (v5.0.3) normalize (SCT-transform), run PCA, cluster with FindNeighbors/FindClusters (Louvain), and visualize with UMAP; for the spatial scenario, select the coronal section 'Zhuang-ABCA-1.080' (37,068 barcodes) and map spatial coordinates to the Allen CCF 2020 via NIfTI images with 0.01 mm voxel spacing. Run the Cell Marker Accordion via accordion(species='Mouse', tissue='brain', max_n_marker=30, annotation_resolution='cluster') and the baselines (ScType, SCINA, clustifyR, scCATCH, scSorter) with default parameters, supplying positive markers from the Cell Marker Accordion database to user-input tools (max 100 markers per cell type for SCINA). Score each tool by computing the average percentage similarity between predicted and ground-truth cell types over the Cell Ontology tree using the get_sim_grid function of the ontologySimilarity R package (v2.7), which tolerates synonymous but non-identical labels. Rationale: ontology-aware similarity avoids penalizing correct annotations that differ only in granularity, and a shared preprocessing/database setup isolates the annotation algorithm as the variable under test.

## Related tools

- **Cell Marker Accordion** (Primary annotation method under evaluation; also supplies positive markers to baseline user-input tools) — https://github.com/TebaldiLab/cellmarkeraccordion
- **Seurat** (Preprocessing: SCT-transform normalization, PCA, FindNeighbors/FindClusters (Louvain), UMAP for both tested and baseline pipelines)
- **ScType** (Baseline marker-based annotation tool run with default parameters)
- **SCINA** (Baseline annotation tool, run with max 100 markers per cell type from the Cell Marker Accordion database)
- **clustifyR** (Baseline annotation tool run with default parameters using Accordion-derived positive markers)
- **scCATCH** (Baseline annotation tool run with default parameters)
- **scSorter** (Baseline annotation tool run with default parameters)
- **ontologySimilarity** (Scoring: get_sim_grid computes Cell Ontology tree similarity between predicted and ground-truth annotations (v2.7))

## Examples

```
data <- accordion(data, assay="RNA", species="Mouse", tissue="brain", annotation_resolution="cluster", max_n_marker=30, plot=TRUE)
```

## Evaluation signals

- Each tool is scored by average percentage similarity between predicted and ground-truth annotations over the Cell Ontology tree via ontologySimilarity::get_sim_grid; verify scores fall in [0, 100].
- Reproduced benchmark should show the Cell Marker Accordion improving annotation performance by over 13% in the spatial (MERFISH mouse brain) scenario relative to ScType, SCINA, clustifyR, scCATCH and scSorter (average 23% increase across all scenarios reported in the paper).
- Spatial mapping of winning annotations should show high similarity with the original (ground-truth) annotation when plotted on tissue coordinates.
- All tools must be run on identical preprocessed inputs (same Seurat object, same section 'Zhuang-ABCA-1.080', same marker source) so that score differences are attributable to the annotation algorithm.
- User-input baseline tools receive markers from the Cell Marker Accordion database, capped at 100 markers per cell type for SCINA; check this cap is enforced.

## Limitations

- Benchmark did not include a direct comparison with annotation tools that do not rely on marker genes, which instead require high-quality comprehensive reference datasets.
- Scores depend on the availability of high-quality ground-truth datasets, which the authors note as a constraint on benchmark expansion.
- Results are specific to the tested scenario (one MERFISH mouse brain section); no DOI/title metadata or supplementary dataset contents were available to contextualize generalization.

## Evidence

- [methods] MERFISH preprocessing: "The MERFISH mouse brain dataset was analyzed employing a standard pre-processing pipeline for imaging-based spatial dataset, using Seurat"
- [methods] Validation datasets: "we exploited eight single-cell and multi-omics datasets from different healthy tissues along with a spatial transcriptomic brain dataset"
- [discussion] Overall improvement finding: "the Cell Marker Accordion improved the identification of cell types compared to other available marker-based annotation tools12,17,19,40,41,64,65, with an average increase of 23%"
- [discussion] No marker-free comparison: "For this reason, a direct comparison between these approaches and the Cell Marker Accordion was not conducted in this study."
- [readme] Annotation function usage: "data <- accordion(data, assay ="RNA", species ="Human", tissue="blood", annotation_resolution = "cluster", max_n_marker = 30, include_detailed_annotation_info = TRUE, plot = TRUE)"
