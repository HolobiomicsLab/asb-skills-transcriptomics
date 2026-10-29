---
name: differential-expression-module-filtering
description: Use when you already have a cell-lineage-specific GRN plus driver regulators
  with influence scores and need gene modules tied to those regulators, then per-cell
  activity scores via AUCell.
when_to_use_negative:
- No driver regulators or cell-lineage-specific GRN have been produced yet (CEFCON's
  driver_regulators() and GRN construction steps must run first)
- The trajectory contains multiple lineages with bifurcations to process simultaneously
  — CEFCON handles only one cell lineage at a time
- The question requires extrinsic factors (cell–cell communication, spatial location)
  or epigenetic regulation — these are outside CEFCON's intracellular, RNA-seq scope
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: CEFCON
  role: 'Source framework: provides the cell-lineage-specific GRN, driver regulators
    with influence scores, and the RGM_activity() step that detects regulon-like gene
    modules and their activity'
  repo: https://github.com/WPZgithub/CEFCON
- name: networkx
  role: Graph library used to extract out-degree and in-degree neighborhoods of each
    driver regulator from the inferred GRN
- name: scanpy
  role: Single-cell environment through which the expression matrix is handled and
    AUCell activity is computed
  repo: https://scanpy.readthedocs.io/en/stable/installation.html
- name: pyscenic
  role: Environment for running AUCell to measure per-cell RGM activity
  repo: https://pyscenic.readthedocs.io/en/latest/installation.html
- name: AUCell
  role: Scores the state of the RGM gene-sets in the single-cell RNA-seq data, yielding
    the per-cell activity matrix
  repo: https://github.com/aertslab/AUCell
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-023-44103-3
    title: Deciphering driver regulators of cell fate decisions from single-cell transcriptomics
      data with CEFCON
  source_content_hash: sha256:3478fd27b973a762c8ae8387ea8908fdbe5948d48e8048ccbad5575644ce465e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/WPZgithub/CEFCON
  tools:
  - AUCell
  - CEFCON
  - Python
  - networkx
  - pyscenic
  - scanpy
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# differential-expression-module-filtering

## Summary

Identifies regulon-like gene modules (RGMs) for CEFCON's driver regulators — both out-degree and in-degree types from the cell-lineage-specific GRN — filters them by a differential-expression criterion with a minimum size of 10 genes, and scores per-cell module activity with AUCell.

## When to use

Use when you already have a cell-lineage-specific GRN plus driver regulators with influence scores and need gene modules tied to those regulators, then per-cell activity scores via AUCell.

## When NOT to use

- No driver regulators or cell-lineage-specific GRN have been produced yet (CEFCON's driver_regulators() and GRN construction steps must run first)
- The trajectory contains multiple lineages with bifurcations to process simultaneously — CEFCON handles only one cell lineage at a time
- The question requires extrinsic factors (cell–cell communication, spatial location) or epigenetic regulation — these are outside CEFCON's intracellular, RNA-seq scope

## Inputs

- cell_lineage_GRN.csv — CEFCON-inferred cell-lineage-specific GRN edge list
- driver_regulators.csv — identified driver regulators with influence scores
- gene differential expression file (log fold change per gene)
- scRNA-seq expression matrix (rows = cells, columns = genes), e.g. from an AnnData (.h5ad) object

## Outputs

- RGMs.csv — retained regulon-like gene modules (differential-expression-filtered, ≥10 genes)
- AUCell_mtx.csv — per-cell activity matrix (one activity score per RGM per cell)

## How to apply

Load the constructed cell-lineage-specific GRN (cell_lineage_GRN.csv) and the driver regulators with their influence scores (driver_regulators.csv) using pandas. With networkx on the inferred GRN graph, extract for each driver regulator its out-degree neighborhood (direct targets it regulates) and its in-degree neighborhood (targets co-regulated by shared incoming regulators), since CEFCON detects modules of both out-degree and in-degree types including non-TF genes. Filter candidate modules against the gene differential-expression file (log fold change per gene) and retain only modules passing the differential-expression criterion with at least 10 genes; save them to RGMs.csv. Finally compute per-cell module activity with AUCell (pyscenic/scanpy environment) on the scRNA-seq expression matrix (rows = cells, columns = genes) and save the activity matrix to AUCell_mtx.csv. In the CEFCON Python API this step corresponds to cefcon_results.RGM_activity().

## Related tools

- **CEFCON** (Source framework: provides the cell-lineage-specific GRN, driver regulators with influence scores, and the RGM_activity() step that detects regulon-like gene modules and their activity) — https://github.com/WPZgithub/CEFCON
- **networkx** (Graph library used to extract out-degree and in-degree neighborhoods of each driver regulator from the inferred GRN)
- **scanpy** (Single-cell environment through which the expression matrix is handled and AUCell activity is computed) — https://scanpy.readthedocs.io/en/stable/installation.html
- **pyscenic** (Environment for running AUCell to measure per-cell RGM activity) — https://pyscenic.readthedocs.io/en/latest/installation.html
- **AUCell** (Scores the state of the RGM gene-sets in the single-cell RNA-seq data, yielding the per-cell activity matrix) — https://github.com/aertslab/AUCell

## Examples

```
cefcon_results.RGM_activity()  # identifies RGMs and computes AUCell per-cell activity in the CEFCON pipeline
```

## Evaluation signals

- RGMs.csv contains only modules that pass the differential-expression filter (log fold-change criterion) and the ≥10-gene size threshold
- Every retained module is traceable to a driver regulator's out-degree or in-degree neighborhood in the inferred GRN graph
- AUCell_mtx.csv provides an activity score for every RGM across all cells (complete cells × modules matrix)
- Module membership includes non-TF target genes, consistent with CEFCON's regulon-like (not TF-only) module definition
- Re-running the pipeline with the same GRN and driver regulators reproduces the same module memberships and activity scores

## Limitations

- Only modules satisfying the differential-expression criterion with at least 10 genes are retained; smaller or non-differentially expressed regulator neighborhoods are dropped
- Inherits CEFCON limitations: one cell lineage at a time, intracellular RNA-seq factors only (no cell–cell communication, spatial information, or epigenetics)
- Activity scores depend on AUCell gene-set ranking, so expression-matrix preprocessing (e.g. highly variable gene selection, as in the 1,000-HVG hESC example) affects results

## Evidence

- [intro] In addition, CEFCON detects the regulon-like gene modules (RGMs) involving the identified driver regulators.: "In addition, CEFCON detects the regulon-like gene modules (RGMs) involving the identified driver regulators."
- [methods] # Identify regulon-like gene modules
cefcon_results.RGM_activity(): "# Identify regulon-like gene modules
cefcon_results.RGM_activity()"
- [readme] CEFCON detects gene regulatory modules (i.e., RGMs) involving the identified driver regulators and measure their activities using AUCell: "Moreover, CEFCON detects gene regulatory modules (i.e., RGMs) involving the identified driver regulators and measure their activities using [AUCell]"
- [readme] AUCell is an R-package to analyze the state of gene-sets in single-cell RNA-seq data (i.e. identify cells with active gene signatures).: [evidence span withheld — anchor status: external_doc]
