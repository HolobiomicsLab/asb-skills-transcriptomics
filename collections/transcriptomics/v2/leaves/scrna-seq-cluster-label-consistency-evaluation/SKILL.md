---
name: scrna-seq-cluster-label-consistency-evaluation
description: Use when evaluating whether a cell-annotation tool's accuracy is limited
  by its bundled reference database, or when testing whether substituting an external
  taxonomy reference (e.g., CellMatch) improves cluster-label consistency on benchmark
  datasets.
when_to_use_negative:
- The goal is to evaluate scCATCH's own evidence-based scoring (findmarkergene/findcelltype)
  rather than to ablate the reference database used by a cell-based annotator such
  as SingleR or CellAssign.
- The input dataset has no benchmark/truth labels against which cluster-level annotation
  consistency can be computed.
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0625
tools:
- name: SingleR
  role: Cell-based annotation method rerun on the benchmark datasets with default
    references and with CellMatch substituted as the reference
- name: scCATCH
  role: Source of the CellMatch reference database (cellmatch data) used in place
    of SingleR's default references
  repo: https://github.com/ZJUFanLab/scCATCH
- name: Seurat
  role: Processing pipeline for the scRNA-seq benchmark datasets (e.g., producing
    sc_data and sc_cluster inputs for annotation)
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.isci.2020.100882
    title: 'scCATCH: Automatic Annotation on Cell Types of Clusters from Single-Cell
      RNA Sequencing Data'
  source_content_hash: sha256:b92de463038a8d40bed8cf25103fccb2873b0e14550246c086e5d36b92f3c582
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
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0625
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# scrna-seq-cluster-label-consistency-evaluation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A benchmarking procedure that quantifies how the choice of reference database affects cell type annotation accuracy, by running an annotator such as SingleR on the same benchmark scRNA-seq datasets with its default references and then with the CellMatch reference (353 cell types, 686 subtypes, 184 tissues, 2,096 human/mouse references) and comparing per-cluster agreement.

## When to use

Use when evaluating whether a cell-annotation tool's accuracy is limited by its bundled reference database, or when testing whether substituting an external taxonomy reference (e.g., CellMatch) improves cluster-label consistency on benchmark datasets.

## When NOT to use

- The goal is to evaluate scCATCH's own evidence-based scoring (findmarkergene/findcelltype) rather than to ablate the reference database used by a cell-based annotator such as SingleR or CellAssign.
- The input dataset has no benchmark/truth labels against which cluster-level annotation consistency can be computed.

## Inputs

- Benchmark cell identity scRNA-seq datasets from three different tissues (from the ZJUFanLab/scCATCH_performance_comparison repository)
- CellMatch reference database (cellmatch: 353 cell types, 686 subtypes, 184 tissue types, 2,096 human and mouse references)
- Per-cluster cluster assignments for each benchmark dataset

## Outputs

- Per-dataset annotation consistency values (agreement with benchmark labels) for default-reference runs
- Per-dataset annotation consistency values for CellMatch-reference runs
- Side-by-side comparison table of consistency values (cf. Tables S1/S2)

## How to apply

Load the three cell identity benchmark scRNA-seq datasets used in the scCATCH performance comparison (available via the ZJUFanLab/scCATCH_performance_comparison repository). First run SingleR on each benchmark dataset with its default reference databases and record the cell type assigned to each cluster. Then rerun SingleR on the same datasets supplying the CellMatch reference database in place of the defaults. For each dataset compute annotation consistency as agreement with the benchmark labels, for both the default-reference and CellMatch-reference runs, and compile side-by-side consistency values into a table. Validation: confirm CellMatch-based annotations show the reported improvement from roughly 0%-10% consistency with default references up to 80%-90% consistency with CellMatch, as reported in Tables S1/S2 of the Results.

## Related tools

- **SingleR** (Cell-based annotation method rerun on the benchmark datasets with default references and with CellMatch substituted as the reference)
- **scCATCH** (Source of the CellMatch reference database (cellmatch data) used in place of SingleR's default references) — https://github.com/ZJUFanLab/scCATCH
- **Seurat** (Processing pipeline for the scRNA-seq benchmark datasets (e.g., producing sc_data and sc_cluster inputs for annotation))

## Examples

```
obj <- createscCATCH(data = sc_data, cluster = sc_cluster); obj <- findmarkergene(obj, species = 'Human', marker = cellmatch, tissue = 'tissue_name', cancer = 'normal')
```

## Evaluation signals

- Default-reference SingleR consistency on the three benchmark datasets falls roughly in the 0%-10% agreement range, matching the paper's report.
- CellMatch-reference SingleR consistency rises to 80%-90% agreement on all three benchmark datasets.
- Consistency is computed per dataset as agreement with the benchmark cell identity labels and compiled into a side-by-side table.
- The CellMatch reference used contains the expected scope: 353 cell types, 686 subtypes, 184 tissue types, and 2,096 human and mouse references.

## Limitations

- Annotation performance is reference-database dependent: a limited reference database potentially leads to incorrect annotation of cell types.
- Improvement was demonstrated with CellMatch applied to SingleR and CellAssign; generalization to other annotators is not established here.
- Inadequate clustering (too many or too few clusters) can introduce errors into cell type annotation, affecting consistency measurements.

## Evidence

- [results] Using CellMatch as the underlying reference, SingleR's consistent rate improved from 0% to 80%–90% on the three validation datasets, performing better than with its default reference databases.: [evidence span withheld — anchor status: not_found]
