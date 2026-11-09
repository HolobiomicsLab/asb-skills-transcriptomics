---
name: ontology-based-cell-type-similarity-scoring
description: Use when benchmarking cell type annotation tools (e.g. Cell Marker Accordion
  vs ScType, SCINA, clustifyR, scCATCH, scSorter) and you need a Cell Ontology-aware
  similarity metric instead of exact label matching.
when_to_use_negative:
- Comparing annotations across cell types that cannot be mapped to the Cell Ontology
  (no shared ontology terms), since similarity would be undefined or misleading
- Evaluating non-marker-based annotation approaches — the article explicitly notes
  no direct comparison was conducted with tools that do not rely on marker genes
- Scoring datasets lacking reliable ground truth, which the discussion identifies
  as constraining benchmark expansion
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: ontologySimilarity
  role: R package whose get_sim_grid function computes the semantic similarity between
    predicted and ground-truth cell types over the Cell Ontology tree (v2.7)
- name: Cell Marker Accordion
  role: Annotation method whose predictions are scored against ground truth using
    the ontology similarity metric (accordion() with max_n_marker=30 and tissue specified)
  repo: https://github.com/TebaldiLab/cellmarkeraccordion
- name: ScType, SCINA, clustifyR, scCATCH, scSorter
  role: Marker-based baseline annotation tools whose outputs are scored with the same
    similarity metric for comparison
provenance:
  source_task_ids:
  - task_003
  - task_004
  - task_001
  source_papers:
  - doi: 10.1038/s41467-025-60900-4
    title: 'Cell Marker Accordion: interpretable single-cell and spatial omics annotation
      in health and disease'
  source_content_hash: sha256:f795c185daa816267782e575bf1c89ce4031086ab443ca79119ec428e64a5b83
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/TebaldiLab/cellmarkeraccordion
  tools:
  - Cell Marker Accordion
  - R
  - SCINA
  - ScType
  - ScType, SCINA, clustifyR, scCATCH, scSorter
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
yes_rate: 0.7272727272727273
n_claims: 11
---

# ontology-based-cell-type-similarity-scoring

## Summary

Scores cell type annotation accuracy by computing the semantic similarity between predicted and ground-truth cell type labels over the Cell Ontology tree (release 2024-08-16) using the ontologySimilarity R package, producing a finite 0-100 percentage similarity per cluster/dataset.

## When to use

Use when benchmarking cell type annotation tools (e.g. Cell Marker Accordion vs ScType, SCINA, clustifyR, scCATCH, scSorter) and you need a Cell Ontology-aware similarity metric instead of exact label matching.

## When NOT to use

- Comparing annotations across cell types that cannot be mapped to the Cell Ontology (no shared ontology terms), since similarity would be undefined or misleading
- Evaluating non-marker-based annotation approaches — the article explicitly notes no direct comparison was conducted with tools that do not rely on marker genes
- Scoring datasets lacking reliable ground truth, which the discussion identifies as constraining benchmark expansion

## Inputs

- Predicted cell type annotations per cluster/cell (from Cell Marker Accordion, ScType, SCINA, clustifyR, scCATCH, scSorter)
- Ground-truth cell type annotations from original dataset labels
- Cell Ontology tree (release 2024-08-16) with cell-type nomenclature mapped to ontology terms
- Seurat object with clusters to annotate (e.g. MERFISH Zhuang-ABCA-1.080 section, 37,068 barcodes)

## Outputs

- Per-tool, per-dataset table of percentage similarity (0-100 scale), e.g. merfish_annotation_performance.csv
- Per-cluster average percentage similarity between predicted and ground-truth cell types
- Spatial plots of winning annotations, e.g. merfish_spatial_annotation.png

## How to apply

First harmonize nomenclature by mapping original and predicted cell type labels to the Cell Ontology (release 2024-08-16) so that shared and distinct types are comparable. For each cluster, compute similarity between the predicted type and the ground-truth type using the get_sim_grid function of the ontologySimilarity R package (v2.7) traversing the Cell Ontology tree, then average the per-cluster similarities to obtain a per-dataset percentage similarity on a 0-100 scale. Assemble a per-tool, per-dataset table of these similarity scores (and runtimes measured with all tools set to 4 threads) to quantify improvement, e.g. the ~23% average gain of the Cell Marker Accordion over marker-based baselines on PBMC FACS-sorted, CITE-seq/AbSeq bone marrow, and immune, pancreas, liver, lung, kidney datasets, and >13% on the MERFISH Zhuang-ABCA-1.080 mouse brain section. Validation requires every table cell to carry a finite similarity score (0-100) and the winning tool to show the highest scores on healthy-tissue datasets.

## Related tools

- **ontologySimilarity** (R package whose get_sim_grid function computes the semantic similarity between predicted and ground-truth cell types over the Cell Ontology tree (v2.7))
- **Cell Marker Accordion** (Annotation method whose predictions are scored against ground truth using the ontology similarity metric (accordion() with max_n_marker=30 and tissue specified)) — https://github.com/TebaldiLab/cellmarkeraccordion
- **ScType, SCINA, clustifyR, scCATCH, scSorter** (Marker-based baseline annotation tools whose outputs are scored with the same similarity metric for comparison)

## Examples

```
library(ontologySimilarity); sim <- get_sim_grid(ontology = cell_ontology, annot1 = predicted_ct, annot2 = ground_truth_ct); mean_similarity <- 100 * mean(diag(sim))
```

## Evaluation signals

- Every per-tool, per-dataset similarity table cell carries a finite score on the 0-100 scale
- Cell Marker Accordion shows the highest annotation performance on healthy-tissue datasets, consistent with the reported ~23% average improvement over marker-based baselines
- MERFISH spatial benchmark reproduces >13% improvement over the other marker-based tools with high spatial similarity to original annotations
- Cell type labels from both prediction and ground truth map cleanly to Cell Ontology (release 2024-08-16) terms before scoring
- Pairwise Jaccard sanity check on the underlying databases yields low overlap (average 0.08, max 0.13 across seven databases; max 0.23 between CellMarker2.0 and PanglaoDB), confirming ontology-based scoring rather than set overlap is needed

## Limitations

- scSorter may need to be excluded ('Timeout') if it exceeds 3 hours of runtime
- Limited availability of high-quality ground-truth datasets constrains benchmark expansion
- The similarity metric penalizes semantically distant labels, so non-standard nomenclature must first be mapped to the Cell Ontology to avoid artificially low scores

## Evidence

- [methods] Compute annotation performance as the average percentage similarity between predicted and ground-truth cell types per cluster using the get_sim_grid function of the ontologySimilarity R package (v2.7).: [evidence span withheld — anchor status: not_found]
- [results] Score every tool by computing the average percentage similarity between predicted cell types and the ground-truth annotation using the Cell Ontology tree with the get_sim_grid function.: [evidence span withheld — anchor status: not_found]
- [methods] mapping them to the Cell Ontology (release 2024-08-16): "mapping them to the Cell Ontology (release 2024-08-16)"
- [discussion] improved the identification of cell types compared to other available marker-based annotation tools with an average increase of 23%: "the Cell Marker Accordion improved the identification of cell types compared to other available marker-based annotation tools12,17,19,40,41,64,65, with an average increase of 23%"
