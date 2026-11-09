---
name: marker-based-cell-type-annotation
description: Use when annotating a scRNA-seq count matrix from a marker gene list
  — top 5 or 15 genes per cell type derived with Seurat, or canonical markers from
  PanglaoDB/CellMarker/CancerSEA — and no labeled training expression matrix is used.
when_to_use_negative:
- Only a labeled training expression matrix plus testing matrix is available — the
  eagersupervised/lazysupervised wrappers (e.g. Seurat, SingleR, CELLBLAST) consume
  train/test expression data, whereas markersupervised consumes marker lists instead
  of expression matrices.
- No marker list exists and none can be derived (no labels for Seurat FindAllMarkers,
  no canonical/resource markers) — marker-learning wrappers cannot train from expression
  alone.
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3170
tools:
- name: AutomaticCellTypeIdentification
  role: R package providing the markersupervised wrapper that routes a named method
    (scTyper, Markercount, SCSA, DigitalCellSorter, SCINA) with (test, marker, method)
    arguments
  repo: https://github.com/xiebb123456/AutomaticCellTypeIdentification
- name: Seurat
  role: 'Derives cell-type-specific markers: FindAllMarkers run with manually annotated
    labels as ''active.ident'' supplies the top 5/15 genes per type'
- name: SCSA
  role: Example marker-learning method dispatched by the markersupervised wrapper
    using differentially expressed markers from the training dataset
- name: SCINA
  role: Marker-learning baseline run on the same simulated counts with the same marker
    signatures and compared on accuracy and F1-score
- name: MarkerCount
  role: Marker-based cell-type identifier (MarkerCount() function) usable from R via
    reticulate as an alternative annotation path
  repo: https://github.com/combio-dku/MarkerCount
provenance:
  source_task_ids:
  - task_007
  - task_006
  source_papers:
  - doi: 10.1016/j.csbj.2021.10.027
    title: Automatic cell type identification methods for single-cell RNA sequencing
  source_content_hash: sha256:17a2c8f8efd4fb2a17d79a3154c362f85fc935e1f44bf6dfc0d77f2fd83b0ae6
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/xiebb123456/AutomaticCellTypeIdentification
  repo_url: https://github.com/xiebb123456/AutomaticCellTypeIdentification
  tools:
  - AutomaticCellTypeIdentification
  - CellAssign
  - CellMarker
  - EasyCellType
  - MarkerCount
  - R
  - SCINA
  - SCSA
  - Seurat
  - clusterProfiler
  - scmap-cluster
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# marker-based-cell-type-annotation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Assigns cell types to scRNA-seq cells from per-cell-type marker gene lists rather than labeled training expression matrices, using the markersupervised wrapper (scTyper, Markercount, SCSA, DigitalCellSorter, SCINA) of the AutomaticCellTypeIdentification R package or MarkerCount's marker-based mode.

## When to use

Use when annotating a scRNA-seq count matrix from a marker gene list — top 5 or 15 genes per cell type derived with Seurat, or canonical markers from PanglaoDB/CellMarker/CancerSEA — and no labeled training expression matrix is used.

## When NOT to use

- Only a labeled training expression matrix plus testing matrix is available — the eagersupervised/lazysupervised wrappers (e.g. Seurat, SingleR, CELLBLAST) consume train/test expression data, whereas markersupervised consumes marker lists instead of expression matrices.
- No marker list exists and none can be derived (no labels for Seurat FindAllMarkers, no canonical/resource markers) — marker-learning wrappers cannot train from expression alone.

## Inputs

- Testing count matrix (rows = genes, columns = cells)
- Per-cell-type marker gene list (top 5 or 15 genes per type from Seurat FindAllMarkers, or canonical markers from PanglaoDB/CellMarker/CancerSEA)
- Named marker-learning method (scTyper, Markercount, SCSA, DigitalCellSorter, SCINA)
- For MarkerCount: marker matrix as .csv and sample scRNA-seq data as .h5ad

## Outputs

- Per-cell predicted cell-type label vector
- Performance metrics (prediction accuracy, F1-score) if annotated labels are available

## How to apply

Obtain markers either from marker resource centers (PanglaoDB, CellMarker, CancerSEA) or by running the Seurat pipeline with manually annotated labels set as 'active.ident' and selecting genes via FindAllMarkers; take the top 5 or 15 genes per type, or use canonical markers. Ensure the testing data is a count matrix oriented with rows as genes and columns as cells. Call the markersupervised wrapper — which consumes marker lists instead of expression matrices — passing only the test matrix, the marker list, and the method name; this routes to marker-learning methods such as scTyper, Markercount, SCSA, DigitalCellSorter and SCINA. For Python-based methods (e.g. SCSA), load the required conda environment at the beginning in R. Alternatively use MarkerCount's marker-based mode via its MarkerCount() function, callable from R through reticulate. Judge success by the wrapper returning a per-cell prediction vector without argument errors and by accuracy and F1-score versus annotated labels.

## Related tools

- **AutomaticCellTypeIdentification** (R package providing the markersupervised wrapper that routes a named method (scTyper, Markercount, SCSA, DigitalCellSorter, SCINA) with (test, marker, method) arguments) — https://github.com/xiebb123456/AutomaticCellTypeIdentification
- **Seurat** (Derives cell-type-specific markers: FindAllMarkers run with manually annotated labels as 'active.ident' supplies the top 5/15 genes per type)
- **SCSA** (Example marker-learning method dispatched by the markersupervised wrapper using differentially expressed markers from the training dataset)
- **SCINA** (Marker-learning baseline run on the same simulated counts with the same marker signatures and compared on accuracy and F1-score)
- **MarkerCount** (Marker-based cell-type identifier (MarkerCount() function) usable from R via reticulate as an alternative annotation path) — https://github.com/combio-dku/MarkerCount

## Examples

```
markersupervised(test,marker,method='SCSA')
```

## Evaluation signals

- The marker-supervised wrapper resolves a named method (e.g. SCSA) and returns a prediction vector without argument errors.
- Input count matrix is oriented rows = genes, columns = cells before dispatch.
- Marker list format matches the wrapper: per-cell-type genes (top 5 or 15 from FindAllMarkers with labels as 'active.ident', or canonical markers).
- Accuracy (correctly predicted cells / total labeled cells, excluding unlabeled) and mean per-cell-type F1-score are computed and compared against unsupervised and eager-learning baselines.
- Python-based methods (e.g. SCSA) run only after the required conda environment is loaded in R.

## Limitations

- Python-based marker methods require the matching conda environment to be loaded at the beginning in R; failing to load it breaks dispatch.
- The markersupervised wrapper does not accept training expression data — it consumes marker lists only, so annotation quality depends on the supplied markers (top 5 vs 15 genes or canonical markers).
- On the CellAssign simulation study, the marker-learning method SCINA was outperformed on accuracy and F1-score by CellAssign and by unsupervised and eager-learning baselines, so marker-based annotation is not always the best-performing choice.

## Evidence

- [readme] ```markersupervised``` methods include scTyper, Markercount, SCSA, DigitalCellSorter and SCINA.: [evidence span withheld — anchor status: external_doc]
- [readme] MarkerCount works both in reference and marker-based mode, where the latter utilizes only the existing lists of markers, while the former required pre-annotated dataset to train the model.: [evidence span withheld — anchor status: external_doc]
