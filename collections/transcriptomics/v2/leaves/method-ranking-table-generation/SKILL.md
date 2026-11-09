---
name: method-ranking-table-generation
description: 'Use when a paper''s claim of one method outperforming baselines must
  be reproduced: multiple cell-type identification methods are run on identical data
  and ranked by accuracy and F1-score in a comparison table.'
when_to_use_negative:
- Input is a single method's predictions with no competing baselines — a comparison
  table requires all methods run on the same labeled data
- Data lacks ground-truth cell-type labels (e.g., unlabeled real tissue), since accuracy
  and F1-score against reference labels cannot be computed
- The goal is evaluating methods on running time or unlabeling rate alone rather than
  the accuracy/F1-score ranking this skill produces
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: AutomaticCellTypeIdentification
  role: R wrapper providing a uniform eagersupervised/lazysupervised/markersupervised
    interface to run the competing methods on the same train/test inputs
  repo: https://github.com/xiebb123456/AutomaticCellTypeIdentification
- name: R
  role: Implementation environment for running wrappers and computing accuracy/F1-score
    metrics and the ranking table
- name: CellAssign
  role: Marker-based method whose probabilistic assignments are benchmarked against
    the baselines in the comparison table
- name: SCINA
  role: Marker-supervised baseline run on the same simulated counts and marker signatures;
    invoked via markersupervised(test, marker, method='SCINA')
- name: Seurat
  role: Unsupervised clustering baseline; its pipeline also finds cell-type specific
    markers used for cluster annotation
- name: scmap-cluster
  role: Eager-learning baseline (available as scmapcluster in the wrapper) run on
    the same simulated data
  repo: https://github.com/hemberg-lab/scmap
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.csbj.2021.10.027
    title: Automatic cell type identification methods for single-cell RNA sequencing
  source_content_hash: sha256:f6ef4fff855e8f22d618236c0e8b7a902e2e84e76957ffbdaa3bb49546d14473
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
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# method-ranking-table-generation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark-generation skill that runs competing cell-type identification methods on the same input data, computes accuracy and mean per-cell-type F1-score for each, and writes a ranked comparison table (e.g., cellassign_comparison_results.csv) used to establish superiority claims such as CellAssign outperforming unsupervised, eager-learning, and SCINA baselines.

## When to use

Use when a paper's claim of one method outperforming baselines must be reproduced: multiple cell-type identification methods are run on identical data and ranked by accuracy and F1-score in a comparison table.

## When NOT to use

- Input is a single method's predictions with no competing baselines — a comparison table requires all methods run on the same labeled data
- Data lacks ground-truth cell-type labels (e.g., unlabeled real tissue), since accuracy and F1-score against reference labels cannot be computed
- The goal is evaluating methods on running time or unlabeling rate alone rather than the accuracy/F1-score ranking this skill produces

## Inputs

- Simulated scRNA-seq count matrix with known cell-type labels (genes x cells)
- Cell-type marker list/gene sets from the CellAssign study simulation setup
- Predicted cell-type labels from each competing method (unsupervised cluster annotations, eager-learning predictions, SCINA assignments)

## Outputs

- Ranked comparison table (e.g., cellassign_comparison_results.csv) with one row per method and numeric accuracy and F1-score columns
- Per-method accuracy (correctly predicted cells / total labeled cells, excluding unlabeled)
- Mean per-cell-type F1-score (harmonic mean of precision and recall averaged over cell types)

## How to apply

Run every method under comparison on the identical input data (e.g., simulated scRNA-seq count matrices with known labels and the marker list), using a consistent interface such as the AutomaticCellTypeIdentification R package's eagersupervised, lazysupervised, and markersupervised wrappers for methods like Seurat, scmapcluster, CELLBLAST, and SCINA. Compute accuracy as correctly predicted cells divided by total labeled cells, excluding unlabeled cells, and compute mean per-cell-type F1-score as the harmonic mean of precision and recall averaged over cell types — the same metrics the review uses to evaluate its thirty-two published methods. Rank methods by these numeric columns, verify each evaluated method appears as a row with numeric accuracy and F1-score, and check the resulting ordering is consistent with the published relative ordering (e.g., CellAssign above Seurat, SC3, PhenoGraph, densityCut, dynamicTreeCut, scmap-cluster, correlation-based prediction, and SCINA). Write the ranked table to a CSV such as cellassign_comparison_results.csv.

## Related tools

- **AutomaticCellTypeIdentification** (R wrapper providing a uniform eagersupervised/lazysupervised/markersupervised interface to run the competing methods on the same train/test inputs) — https://github.com/xiebb123456/AutomaticCellTypeIdentification
- **R** (Implementation environment for running wrappers and computing accuracy/F1-score metrics and the ranking table)
- **CellAssign** (Marker-based method whose probabilistic assignments are benchmarked against the baselines in the comparison table)
- **SCINA** (Marker-supervised baseline run on the same simulated counts and marker signatures; invoked via markersupervised(test, marker, method='SCINA'))
- **Seurat** (Unsupervised clustering baseline; its pipeline also finds cell-type specific markers used for cluster annotation)
- **scmap-cluster** (Eager-learning baseline (available as scmapcluster in the wrapper) run on the same simulated data) — https://github.com/hemberg-lab/scmap

## Examples

```
eagersupervised(train,test,label_train,method='Seurat'); markersupervised(test,marker,method='SCINA')
```

## Evaluation signals

- Output CSV contains a row for every evaluated method (CellAssign, Seurat, SC3, PhenoGraph, densityCut, dynamicTreeCut, scmap-cluster, correlation-based, SCINA) with numeric accuracy and F1-score columns
- Accuracy invariant: correctly predicted cells divided by total labeled cells, with cells lacking label information excluded from the denominator
- F1-score per method is the harmonic mean of precision and recall averaged over cell types, not a micro-averaged value
- Ranking check: CellAssign's accuracy and F1-score exceed those of all unsupervised, eager-learning, and SCINA baselines, consistent with the study's reported ordering
- All methods were run on the identical count matrix and marker list — row counts and labeled-cell totals match across methods' metric computations

## Limitations

- Rankings are only valid for the specific simulated datasets and marker lists used; the review notes method advantages and disadvantages vary with the information available
- Unlabeled cells (cells a method declines to assign) are excluded from accuracy, so methods with high unlabeling rates are not penalized in this metric
- Unsupervised baselines require an additional cluster-to-type annotation step via markers, which introduces an extra source of variability absent from direct marker-based methods

## Evidence

- [readme] markersupervised methods include scTyper, Markercount, SCSA, DigitalCellSorter and SCINA.: [evidence span withheld — anchor status: not_found]
- [readme] eagersupervised methods include ACTINN, CaSTLe, CHETAH, clustifyr, Garnett, Markercount, MARS, scClassifR, scHPL, SciBet, scID, scLearn, scmapcluster, scPred, scVI, Seurat, SingleCellNet and SingleR.: [evidence span withheld — anchor status: not_found]
