---
name: unsupervised-clustering-annotation-comparison
description: Use when you have a simulated or labeled scRNA-seq count matrix with
  known cell-type labels and a marker list, and you need to rank clustering-then-annotation
  baselines (Seurat, SC3, PhenoGraph, densityCut, dynamicTreeCut) against marker-based
  or eager-learning classifiers.
when_to_use_negative:
- Input is already a cluster-to-cell-type annotation or a precomputed feature/embedding
  table rather than a raw count matrix with labels
- The goal is running a single method's assignment rather than a head-to-head benchmark
  across unsupervised, eager-learning and marker-learning categories
- No ground-truth cell-type labels (or marker list) exist, so accuracy and per-cell-type
  F1-score cannot be computed
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: AutomaticCellTypeIdentification
  role: R wrapper providing the eagersupervised (e.g. Seurat, scmapcluster) and markersupervised
    (e.g. SCINA) interfaces used to run the baseline methods on the same count matrix
  repo: https://github.com/xiebb123456/AutomaticCellTypeIdentification
- name: R
  role: Execution environment for running the method wrappers and computing accuracy/F1-score
    metrics
- name: CellAssign
  role: Marker-based probabilistic cell-type assignment method being benchmarked against
    the unsupervised and eager-learning baselines
- name: SCINA
  role: Marker-learning baseline run on the same simulated counts with the same marker
    signatures via markersupervised(test, marker, method='SCINA')
- name: scmap-cluster
  role: Eager-learning baseline for cluster-level label transfer included in the eagersupervised
    method set
  repo: https://github.com/hemberg-lab/scmap
- name: Seurat
  role: Unsupervised clustering baseline; the Seurat pipeline is also used to find
    cell type specific markers for annotating clusters
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.csbj.2021.10.027
    title: Automatic cell type identification methods for single-cell RNA sequencing
  source_content_hash: sha256:2ea8ddf8f93973dc2b237d720a4b08187b6d7948cfe0fd85b601f8f55732b104
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
  - R
  - SCINA
  - Seurat
  - scmap-cluster
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# unsupervised-clustering-annotation-comparison

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark unsupervised scRNA-seq cell-type identification (cluster then annotate clusters via markers) against eager-learning and marker-learning methods such as CellAssign and SCINA, scoring accuracy and mean per-cell-type F1-score. This mirrors the evaluation logic of the AutomaticCellTypeIdentification review, which quantitatively compares methods on prediction accuracy, F1-score, unlabeling rate and running time.

## When to use

Use when you have a simulated or labeled scRNA-seq count matrix with known cell-type labels and a marker list, and you need to rank clustering-then-annotation baselines (Seurat, SC3, PhenoGraph, densityCut, dynamicTreeCut) against marker-based or eager-learning classifiers.

## When NOT to use

- Input is already a cluster-to-cell-type annotation or a precomputed feature/embedding table rather than a raw count matrix with labels
- The goal is running a single method's assignment rather than a head-to-head benchmark across unsupervised, eager-learning and marker-learning categories
- No ground-truth cell-type labels (or marker list) exist, so accuracy and per-cell-type F1-score cannot be computed

## Inputs

- Simulated scRNA-seq count matrix (genes x cells) with known cell-type labels
- Cell-type marker list / marker signatures for the simulated cell types
- Manually annotated labels for held-out or simulated cells (count matrix with labels as training/testing input)

## Outputs

- Comparison table (e.g. cellassign_comparison_results.csv) with one row per method and numeric accuracy and mean per-cell-type F1-score columns
- Ranked method list ordering CellAssign against unsupervised, eager-learning and SCINA baselines

## How to apply

Load the simulated scRNA-seq count matrix (genes as rows, cells as columns) together with the known cell-type labels and the cell-type marker list. Run CellAssign using the supplied marker genes to assign cells probabilistically to types, and run SCINA on the same counts with the same marker signatures via the markersupervised interface of the AutomaticCellTypeIdentification R package. Run the unsupervised baselines (Seurat clustering, SC3, PhenoGraph, densityCut, dynamicTreeCut) and annotate clusters by mapping cluster markers to cell types; also run eager-learning baselines such as scmap-cluster and correlation-based prediction through the eagersupervised interface. Compute accuracy as correctly predicted cells divided by total labeled cells (excluding unlabeled cells) and mean per-cell-type F1-score (harmonic mean of precision and recall averaged over cell types) for every method. Rank methods by accuracy and F1-score and write the comparison table (e.g. cellassign_comparison_results.csv). Rationale: the underlying review evaluates thirty-two published methods on accuracy, F1-score, unlabeling rate and running time, so these two metrics plus a full method ranking are the study-consistent evaluation basis.

## Related tools

- **AutomaticCellTypeIdentification** (R wrapper providing the eagersupervised (e.g. Seurat, scmapcluster) and markersupervised (e.g. SCINA) interfaces used to run the baseline methods on the same count matrix) — https://github.com/xiebb123456/AutomaticCellTypeIdentification
- **R** (Execution environment for running the method wrappers and computing accuracy/F1-score metrics)
- **CellAssign** (Marker-based probabilistic cell-type assignment method being benchmarked against the unsupervised and eager-learning baselines)
- **SCINA** (Marker-learning baseline run on the same simulated counts with the same marker signatures via markersupervised(test, marker, method='SCINA'))
- **scmap-cluster** (Eager-learning baseline for cluster-level label transfer included in the eagersupervised method set) — https://github.com/hemberg-lab/scmap
- **Seurat** (Unsupervised clustering baseline; the Seurat pipeline is also used to find cell type specific markers for annotating clusters)

## Examples

```
markersupervised(test, marker, method='SCINA')
```

## Evaluation signals

- The output comparison table contains a row for each evaluated method with numeric accuracy and F1-score columns
- CellAssign ranks above all unsupervised (Seurat, SC3, PhenoGraph, densityCut, dynamicTreeCut), eager-learning (scmap-cluster, correlation-based) and SCINA baselines on both accuracy and F1-score
- Accuracy denominator equals total labeled cells, with unlabeled cells excluded from the calculation
- F1-score is the harmonic mean of precision and recall averaged over cell types, matching the study's metric definition
- The evaluation also tracks the study's other dimensions (unlabeling rate and running time) so no method is favored by unrecorded unlabeling

## Limitations

- Unsupervised methods require a post-hoc cluster-to-cell-type annotation step via marker mapping, which is a known weakness of the clustering-then-annotate paradigm versus direct marker-based assignment
- Methods can leave cells unlabeled, which changes the effective accuracy denominator and must be reported (the review evaluates unlabeling rate explicitly)
- Marker-based and unsupervised comparisons depend on marker list quality; markers are typically sourced from PanglaoDB, CellMarker and CancerSEA

## Evidence

- [readme] markersupervised methods include scTyper, Markercount, SCSA, DigitalCellSorter and SCINA.: [evidence span withheld — anchor status: not_found]
- [readme] The input of training and testing data is count matrix, the row is gene and the column is cell.: [evidence span withheld — anchor status: external_doc]
