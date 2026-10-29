---
name: cellmatch-reference-database-querying
description: Use when annotating cell types for identified scRNA-seq clusters (human
  or mouse) by matching cluster marker genes against a literature-derived, tissue-specific
  cell taxonomy database, or when boosting SingleR/CellAssign accuracy with a curated
  reference.
when_to_use_negative:
- The sample's tissue is not represented among CellMatch's 184 tissue types and no
  custom cellmatch can be supplied, since a limited reference database potentially
  leads to incorrect annotation
- The goal is per-cell annotation without prior clustering, which is the cell-based
  strategy (SingleR, CellAssign, Garnett, scMap, CHETAH) rather than scCATCH's cluster-based
  approach
- Validation against geneinfo via rev_gene is unavailable and no species/tissue selection
  can be justified for the dataset at hand
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scCATCH
  role: Provides findmarkergene and findcelltype, which query CellMatch to score and
    assign cell types per cluster; CellMatch is bundled as its underlying data
  repo: https://github.com/ZJUFanLab/scCATCH
- name: SingleR
  role: Cell-based annotator to which CellMatch can be supplied as a reference to
    improve annotation accuracy
  repo: https://github.com/ZJUFanLab/scCATCH_performance_comparison
- name: CellAssign
  role: Cell-based annotator to which CellMatch can be applied, resulting in significant
    improvement in accuracies of cell type annotations
  repo: https://github.com/ZJUFanLab/scCATCH_performance_comparison
- name: rev_gene
  role: scCATCH helper to reverse-validate marker genes against geneinfo when building
    or customizing the reference
  repo: https://github.com/ZJUFanLab/scCATCH
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.isci.2020.100882
    title: 'scCATCH: Automatic Annotation on Cell Types of Clusters from Single-Cell
      RNA Sequencing Data'
  source_content_hash: sha256:78e333e1d2db4a8dd22f1950bfe1550bd667314f500cc6fbf19cbf6983fcf136
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
  - CHETAH
  - CellAssign
  - Garnett
  - Matrix
  - SingleR
  - devtools
  - rev_gene
  - scCATCH
  - scMap
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 14
---

# cellmatch-reference-database-querying

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Query the CellMatch tissue-specific cell taxonomy reference database (353 cell types, 686 subtypes, 184 tissue types, 2,096 human and mouse references) to annotate scRNA-seq clusters via scCATCH's findmarkergene/findcelltype scoring, and optionally supply it as a reference to SingleR or CellAssign.

## When to use

Use when annotating cell types for identified scRNA-seq clusters (human or mouse) by matching cluster marker genes against a literature-derived, tissue-specific cell taxonomy database, or when boosting SingleR/CellAssign accuracy with a curated reference.

## When NOT to use

- The sample's tissue is not represented among CellMatch's 184 tissue types and no custom cellmatch can be supplied, since a limited reference database potentially leads to incorrect annotation
- The goal is per-cell annotation without prior clustering, which is the cell-based strategy (SingleR, CellAssign, Garnett, scMap, CHETAH) rather than scCATCH's cluster-based approach
- Validation against geneinfo via rev_gene is unavailable and no species/tissue selection can be justified for the dataset at hand

## Inputs

- Processed scRNA-seq expression matrix (e.g. loaded with the Matrix package in R) for datasets such as Chen, Xin, or Gierahn
- Character vector of cluster assignments for the same cells
- Species, tissue (and optionally cancer) selections matching the biological sample
- Optional custom cellmatch table (user-supplied marker reference)

## Outputs

- Per-cluster cell type annotations (top-1 scored type per cluster from findcelltype)
- Cluster marker gene sets meeting log10FC >= 0.25, Wilcoxon p < 0.05, >= 25% expression criteria
- Consistent-rate values per cluster and per dataset for benchmark comparison (e.g. table1_consistency_rates.csv)

## How to apply

Build the scCATCH object with createscCATCH(data, cluster), then run findmarkergene(obj, species, marker, tissue, cancer) to restrict candidate markers to the tissue (and optionally cancer) context, followed by findcelltype(obj) to score cell types per cluster against CellMatch and assign the top-1 type. Marker genes are genes upregulated in only one cluster with log10 fold change >= 0.25, Wilcoxon p < 0.05, expressed in >= 25% of cells. Selection of species, tissue, and cancer arguments is the key decision point because annotation performance depends on the reference database; an inadequate or off-tissue subset can yield incorrect labels. As a cross-check, CellMatch can also be applied to SingleR and CellAssign, which reportedly improved their annotation accuracies. On the Chen, Xin, and Gierahn benchmarks, judge success by the per-cluster/per-dataset consistent rate (fraction of clusters whose automatic annotation agrees with manual labels) versus Seurat and the cell-based methods.

## Related tools

- **scCATCH** (Provides findmarkergene and findcelltype, which query CellMatch to score and assign cell types per cluster; CellMatch is bundled as its underlying data) — https://github.com/ZJUFanLab/scCATCH
- **SingleR** (Cell-based annotator to which CellMatch can be supplied as a reference to improve annotation accuracy) — https://github.com/ZJUFanLab/scCATCH_performance_comparison
- **CellAssign** (Cell-based annotator to which CellMatch can be applied, resulting in significant improvement in accuracies of cell type annotations) — https://github.com/ZJUFanLab/scCATCH_performance_comparison
- **rev_gene** (scCATCH helper to reverse-validate marker genes against geneinfo when building or customizing the reference) — https://github.com/ZJUFanLab/scCATCH

## Examples

```
obj <- createscCATCH(data = sc_data, cluster = sc_cluster); obj <- findmarkergene(obj, species, marker, tissue, cancer); obj <- findcelltype(obj)
```

## Evaluation signals

- The loaded cellmatch database matches the published scope: 353 cell types, 686 subtypes, 184 tissue types, and 2,096 human and mouse references
- On the Chen, Xin, and Gierahn benchmarks, scCATCH's per-cluster consistent rate falls in the reported 67%-100% range (average 83%)
- Gierahn consistent rates for weak baselines (e.g. Garnett and CHETAH at 0%) are not reproduced by scCATCH, which accurately annotated the clusters
- Applying CellMatch to SingleR and CellAssign yields a measurable improvement in their annotation accuracies, confirming the database is being used correctly
- findcelltype returns exactly one top-1 cell type per cluster with no empty annotations for clusters that have qualifying marker genes

## Limitations

- The performance of scCATCH majorly depends on the reference database; a limited CellMatch database potentially led to incorrect annotation of cell types
- CellMatch covers human and mouse references (2,096 entries); markers from other species require the custom-species workflow
- Inadequate clustering (too many or few cells per cluster) can introduce errors into the annotation regardless of reference quality

## Evidence
