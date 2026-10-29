---
name: annotation-accuracy-quantification
description: Use when you need to measure whether a cell type annotation method (e.g.,
  SingleR) is limited by its default reference databases, by re-running it with an
  alternative reference such as CellMatch and comparing consistency with benchmark
  labels.
when_to_use_negative:
- When the goal is simply to annotate a new dataset once rather than to quantify annotation
  accuracy across reference-database choices
- When no trusted benchmark labels exist for the datasets, so annotation consistency
  (agreement) cannot be computed
- When evaluating scCATCH itself rather than cell-based annotation methods such as
  SingleR, since scCATCH's marker-gene-based workflow does not accept SingleR-style
  reference panels
tools:
- name: SingleR
  role: Cell-based annotation method benchmarked under two reference-database conditions
    (default ImmGen/HPCA/Encode/Blueprint vs CellMatch)
- name: scCATCH
  role: Source of the CellMatch reference database (cellmatch) substituted for SingleR's
    default references
  repo: https://github.com/ZJUFanLab/scCATCH
- name: Seurat
  role: Data processing pipeline used to prepare scRNA-seq datasets and clusters before
    annotation in the benchmark comparison
- name: scCATCH_performance_comparison
  role: Repository providing the benchmark datasets, source code, and results for
    comparing scCATCH, CellAssign, Garnett, SingleR, scMap and CHETAH with the CellMatch
    database
  repo: https://github.com/ZJUFanLab/scCATCH_performance_comparison
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.isci.2020.100882
    title: 'scCATCH: Automatic Annotation on Cell Types of Clusters from Single-Cell
      RNA Sequencing Data'
  source_content_hash: sha256:216324e2f70c07245d8d9cb21d202c4f5a8a902a0c092b2289951d6ce05f42d4
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
  - SingleR
  - scCATCH
  - scCATCH_performance_comparison
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# annotation-accuracy-quantification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A benchmarking skill that quantifies how much a cell type annotation method's accuracy depends on its reference database, by re-running the same annotator (e.g., SingleR) on identical benchmark scRNA-seq datasets while swapping the underlying reference and computing annotation consistency against trusted labels.

## When to use

Use when you need to measure whether a cell type annotation method (e.g., SingleR) is limited by its default reference databases, by re-running it with an alternative reference such as CellMatch and comparing consistency with benchmark labels.

## When NOT to use

- When the goal is simply to annotate a new dataset once rather than to quantify annotation accuracy across reference-database choices
- When no trusted benchmark labels exist for the datasets, so annotation consistency (agreement) cannot be computed
- When evaluating scCATCH itself rather than cell-based annotation methods such as SingleR, since scCATCH's marker-gene-based workflow does not accept SingleR-style reference panels

## Inputs

- Three cell identity benchmark scRNA-seq datasets from ZJUFanLab/scCATCH_performance_comparison (covering three different tissues)
- SingleR default reference databases (ImmGen/HPCA/Encode/Blueprint)
- CellMatch reference database (cellmatch: 353 cell types, 686 subtypes, 184 tissues, 2,096 human/mouse references, bundled in scCATCH)
- Benchmark cell type labels per cluster/dataset

## Outputs

- Per-dataset annotation consistency values for default-reference and CellMatch-reference SingleR runs
- Side-by-side comparison table of consistency values (e.g., Tables S1/S2 style) showing improvement from ~0%-10% to 80%-90%

## How to apply

Load the three cell identity benchmark scRNA-seq datasets used in the scCATCH performance comparison, available via the ZJUFanLab/scCATCH_performance_comparison repository. Run SingleR on each benchmark dataset with its default reference databases (ImmGen/HPCA/Encode/Blueprint) and record the cell type assigned to each cluster; then rerun SingleR on the same datasets supplying the CellMatch database in place of the defaults — CellMatch bundles 353 cell types, 686 subtypes, 184 tissue types, and 2,096 human/mouse references. For each run, compute per-dataset annotation consistency as agreement between assigned and benchmark labels, and compile a side-by-side table of default-reference versus CellMatch-reference consistency. Validate success by confirming the reported pattern: default references yield roughly 0%-10% consistency while CellMatch lifts SingleR to 80%-90% consistency across the three datasets.

## Related tools

- **SingleR** (Cell-based annotation method benchmarked under two reference-database conditions (default ImmGen/HPCA/Encode/Blueprint vs CellMatch))
- **scCATCH** (Source of the CellMatch reference database (cellmatch) substituted for SingleR's default references) — https://github.com/ZJUFanLab/scCATCH
- **Seurat** (Data processing pipeline used to prepare scRNA-seq datasets and clusters before annotation in the benchmark comparison)
- **scCATCH_performance_comparison** (Repository providing the benchmark datasets, source code, and results for comparing scCATCH, CellAssign, Garnett, SingleR, scMap and CHETAH with the CellMatch database) — https://github.com/ZJUFanLab/scCATCH_performance_comparison

## Evaluation signals

- Default-reference SingleR consistency on the three benchmark datasets falls in the ~0%-10% range reported in the paper's Results tables
- CellMatch-referenced SingleR consistency reaches 80%-90% on each of the three benchmark datasets
- Per-dataset consistency is computed as agreement with benchmark labels for both conditions, enabling a like-for-like side-by-side table
- The same three benchmark datasets and cluster assignments are used for both runs, so only the reference database differs between conditions
- The CellMatch reference resolves to the cellmatch panel of 353 cell types / 686 subtypes / 184 tissues / 2,096 human and mouse references

## Limitations

- Annotation performance is reference-database-dependent: a limited reference database can lead to incorrect cell type annotation, so results may not transfer to tissues poorly covered by CellMatch
- Inadequate clustering analysis (too many or too few cells per cluster) can introduce errors into cell type annotation independent of the reference database
- The reported 80%-90% improvement applies to the three benchmark datasets used in the scCATCH comparison; other datasets or species may behave differently

## Evidence

- [methods] CellMatch reference scale: "CellMatch includes a panel of 353 cell types and related 686 subtypes associated with 184 tissue types, and 2,096 references of human and mouse."
- [discussion] CellMatch improves SingleR accuracy: "Application of CellMatch to SingleR and CellAssign resulted in significant improvement in accuracies of cell type annotations"
- [readme] Comparison repository scope: [evidence span withheld — anchor status: external_doc]
- [discussion] Reference-dependence caveat: "The performance of scCATCH majorly depends on the reference database. The limited reference database potentially led to incorrect annotation of cell types"
