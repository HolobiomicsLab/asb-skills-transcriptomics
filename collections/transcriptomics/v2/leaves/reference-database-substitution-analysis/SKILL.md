---
name: reference-database-substitution-analysis
description: Use when evaluating whether annotation accuracy of a cell-based method
  (e.g., SingleR) is reference-database dependent, or when testing whether substituting
  a curated tissue-specific reference such as CellMatch improves cluster annotation
  consistency.
when_to_use_negative:
- The goal is to evaluate scCATCH itself rather than reference-database sensitivity
  of an existing annotation tool — use the full scCATCH comparison workflow (findmarkergene/findcelltype)
  instead
- The scRNA-seq dataset has no trusted benchmark or ground-truth labels, so annotation
  consistency cannot be computed
- The species or tissue of interest is not represented in CellMatch (human/mouse references
  across 184 tissue types), since the article notes a limited reference database can
  lead to incorrect annotation
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: SingleR
  role: Cell-based annotation tool rerun under two reference configurations (default
    vs CellMatch) to isolate reference-database effects
- name: scCATCH
  role: Source of the CellMatch reference database ('cellmatch' data object) substituted
    for SingleR's defaults
  repo: https://github.com/ZJUFanLab/scCATCH
- name: Seurat
  role: Used to process the benchmark scRNA-seq datasets and derive clusters fed to
    annotation
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.isci.2020.100882
    title: 'scCATCH: Automatic Annotation on Cell Types of Clusters from Single-Cell
      RNA Sequencing Data'
  source_content_hash: sha256:731c45c66fe03cf00640289c88a2b7baaf44f7b9486cd7eda38b846501dbd8fd
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
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# reference-database-substitution-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A benchmarking skill that swaps the reference database underlying a cell-annotation tool (SingleR's default ImmGen/HPCA/Encode/Blueprint references) with an alternative (scCATCH's CellMatch: 353 cell types, 686 subtypes, 184 tissues, 2,096 human/mouse references) and quantifies the impact on annotation consistency against benchmark labels.

## When to use

Use when evaluating whether annotation accuracy of a cell-based method (e.g., SingleR) is reference-database dependent, or when testing whether substituting a curated tissue-specific reference such as CellMatch improves cluster annotation consistency.

## When NOT to use

- The goal is to evaluate scCATCH itself rather than reference-database sensitivity of an existing annotation tool — use the full scCATCH comparison workflow (findmarkergene/findcelltype) instead
- The scRNA-seq dataset has no trusted benchmark or ground-truth labels, so annotation consistency cannot be computed
- The species or tissue of interest is not represented in CellMatch (human/mouse references across 184 tissue types), since the article notes a limited reference database can lead to incorrect annotation

## Inputs

- Benchmark cell identity scRNA-seq datasets from three tissues (available in the ZJUFanLab/scCATCH_performance_comparison repository)
- SingleR default reference databases (ImmGen/HPCA/Encode/Blueprint)
- CellMatch reference database (cellmatch object: 353 cell types, 686 subtypes, 184 tissue types, 2,096 human and mouse references, bundled in scCATCH)

## Outputs

- Per-cluster cell type assignments from SingleR under default references
- Per-cluster cell type assignments from SingleR under CellMatch references
- Side-by-side table of per-dataset annotation consistency values (default vs CellMatch), cf. Tables S1/S2

## How to apply

Load the cell identity benchmark scRNA-seq datasets from ZJUFanLab/scCATCH_performance_comparison and run SingleR on each with its default reference databases, recording the cell type assigned per cluster. Rerun SingleR on identical inputs but supply the CellMatch reference database (the 'cellmatch' data object bundled in scCATCH, installable via devtools::install_github('ZJUFanLab/scCATCH')) in place of the defaults. For both runs, compute per-dataset annotation consistency as agreement with the benchmark labels, and compile the paired default-vs-CellMatch values side by side. The expected outcome, per Tables S1/S2 of the scCATCH paper, is that CellMatch-based SingleR annotations rise from roughly 0%-10% consistency with default references to 80%-90% consistency, supporting CellMatch's utility as a reference database.

## Related tools

- **SingleR** (Cell-based annotation tool rerun under two reference configurations (default vs CellMatch) to isolate reference-database effects)
- **scCATCH** (Source of the CellMatch reference database ('cellmatch' data object) substituted for SingleR's defaults) — https://github.com/ZJUFanLab/scCATCH
- **Seurat** (Used to process the benchmark scRNA-seq datasets and derive clusters fed to annotation)

## Examples

```
devtools::install_github('ZJUFanLab/scCATCH'); data(cellmatch); singler_res <- SingleR(test = bench_data, ref = cellmatch, labels = cellmatch$celltype)
```

## Evaluation signals

- CellMatch-based SingleR annotations reach 80%-90% consistency with benchmark labels, matching Tables S1/S2 of the Results
- Default-reference SingleR runs reproduce the reported near-0% (0%-10%) consistency on the three benchmark datasets
- Paired default-vs-CellMatch runs use identical datasets, clustering, and preprocessing so differences are attributable to the reference database alone
- The substituted reference is the canonical CellMatch panel (353 cell types, 686 subtypes, 184 tissues, 2,096 human/mouse references), not a custom subset
- Reproducibility check against the scCATCH_performance_comparison repository source code and results

## Limitations

- The performance of annotation majorly depends on the reference database; a limited reference database can lead to incorrect cell type annotation
- Inadequate clustering analysis (too many or too few clusters) introduces errors into cell type annotation independent of the reference used
- CellMatch covers human and mouse references across 184 tissue types; markers from other species require custom extensions

## Evidence

- [readme] The source code and results of performance comparison on the detail of the process among scCATCH, CellAssign, Garnett, SingleR, scMap and CHETAH, and CellMatch database: [evidence span withheld — anchor status: external_doc]
