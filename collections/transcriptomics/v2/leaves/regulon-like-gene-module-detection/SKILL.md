---
name: regulon-like-gene-module-detection
description: Use when you have a cell-lineage-specific GRN plus identified driver
  regulators (e.g., from CEFCON's driver_regulators() step) and need gene modules
  and per-cell module activity matrices for downstream enrichment or clustering benchmarks.
when_to_use_negative:
- You only have raw scRNA-seq data and no constructed lineage-specific GRN or identified
  driver regulators — the GRN-construction and driver-regulator steps of CEFCON must
  run first.
- You need regulons restricted to transcription factors only and must match the classic
  SCENIC definition rather than CEFCON's extended non-TF-inclusive modules.
- Your trajectory contains multiple fate branches with bifurcations, since CEFCON
  can only handle one cell lineage at a time.
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0602
tools:
- name: CEFCON
  role: Provides the cell-lineage-specific GRN and driver regulators and exposes RGM_activity()
    to detect regulon-like gene modules
  repo: https://github.com/WPZgithub/CEFCON
- name: networkx
  role: Graph operations on the inferred GRN to extract out-degree and in-degree neighborhoods
    of each driver regulator
- name: AUCell
  role: Scores per-cell activity of each RGM using the aertslab AUCell ranking-based
    scheme
  repo: https://github.com/aertslab/AUCell
- name: pyscenic
  role: Environment used for AUCell computation and as the regulon-detection baseline
    for benchmarking RGM quality
- name: scanpy
  role: Handles the scRNA-seq expression matrix used for AUCell activity scoring
provenance:
  source_task_ids:
  - task_004
  - task_008
  source_papers:
  - doi: 10.1038/s41467-023-44103-3
    title: Deciphering driver regulators of cell fate decisions from single-cell transcriptomics
      data with CEFCON
  source_content_hash: sha256:dc8ba022f81d02c76bf267ca15c24724b064ecc012254739834916b6f0638aab
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
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# regulon-like-gene-module-detection

## Summary

Detect regulon-like gene modules (RGMs) involving driver regulators identified by CEFCON, including non-TF genes and both out-degree and in-degree module types, and score their per-cell activity with AUCell. This extends regulon detection beyond SCENIC-style TF-only regulons to capture more biologically meaningful modules in cell-lineage-specific GRNs.

## When to use

Use when you have a cell-lineage-specific GRN plus identified driver regulators (e.g., from CEFCON's driver_regulators() step) and need gene modules and per-cell module activity matrices for downstream enrichment or clustering benchmarks.

## When NOT to use

- You only have raw scRNA-seq data and no constructed lineage-specific GRN or identified driver regulators — the GRN-construction and driver-regulator steps of CEFCON must run first.
- You need regulons restricted to transcription factors only and must match the classic SCENIC definition rather than CEFCON's extended non-TF-inclusive modules.
- Your trajectory contains multiple fate branches with bifurcations, since CEFCON can only handle one cell lineage at a time.

## Inputs

- Cell-lineage-specific GRN as cell_lineage_GRN.csv (edges selected with edge_threshold_avgDegree=8)
- Driver regulators with influence scores as driver_regulators.csv
- Gene differential-expression file with per-gene log fold change
- scRNA-seq expression matrix (rows = cells, columns = genes)

## Outputs

- Regulon-like gene modules written to RGMs.csv (differentially expressed modules with >= 10 genes, out-degree and in-degree types)
- Per-cell RGM AUCell activity matrix written to AUCell_mtx.csv (activity score for every RGM across all cells)

## How to apply

Load the constructed cell-lineage-specific GRN (cell_lineage_GRN.csv, inferred with edge_threshold_avgDegree=8) and the driver regulators with influence scores (driver_regulators.csv) using pandas. Using networkx on the GRN graph, extract for each driver regulator both its out-degree neighborhood (direct targets it regulates) and its in-degree neighborhood (targets co-regulated by shared incoming regulators), yielding regulon-like gene modules that may include non-TF genes. Filter candidate modules against the gene differential-expression log fold-change file and retain only modules passing the differential-expression criterion with at least 10 genes, saving them to RGMs.csv. Compute the per-cell AUCell activity matrix (AUCell_mtx.csv) with the aertslab AUCell scoring scheme; validate downstream by GO_BP/KEGG enrichment precision (fraction of modules significant at BH-adjusted p <= 0.05) and NMI of hierarchical clustering of the AUCell matrix against reference cell-type labels, where CEFCON's RGMs should outperform pySCENIC regulons (Fig. 5b-d).

## Related tools

- **CEFCON** (Provides the cell-lineage-specific GRN and driver regulators and exposes RGM_activity() to detect regulon-like gene modules) — https://github.com/WPZgithub/CEFCON
- **networkx** (Graph operations on the inferred GRN to extract out-degree and in-degree neighborhoods of each driver regulator)
- **AUCell** (Scores per-cell activity of each RGM using the aertslab AUCell ranking-based scheme) — https://github.com/aertslab/AUCell
- **pyscenic** (Environment used for AUCell computation and as the regulon-detection baseline for benchmarking RGM quality)
- **scanpy** (Handles the scRNA-seq expression matrix used for AUCell activity scoring)

## Examples

```
cefcon_results.RGM_activity()  # detect RGMs involving driver regulators and compute AUCell per-cell activity
```

## Evaluation signals

- RGMs.csv contains only modules that pass the differential-expression filter and the >= 10 gene size threshold.
- AUCell_mtx.csv provides an activity score for every RGM across all cells (dimensions = n_RGMs x n_cells).
- GO_BP and KEGG enrichment precision (fraction of modules significant at BH-adjusted p <= 0.05) exceeds the SCENIC baseline on hESC, mESC, and mHSC lineage datasets.
- Hierarchical clustering of the AUCell RGM-activity matrix yields higher normalized mutual information (NMI) against reference cell-type labels than SCENIC regulons, consistent with Fig. 5b-d.
- GRN edges feeding module detection were selected with edge_threshold_avgDegree=8, matching the upstream CEFCON run.

## Limitations

- CEFCON can only handle one cell lineage at a time and cannot process multiple fate trajectories with bifurcations simultaneously.
- Unlike SCENIC regulons, RGMs include non-TF genes, so module composition is not directly comparable to TF-only regulon definitions.
- Module quality depends on the upstream GRN inference (edge_threshold_avgDegree=8) and driver regulator identification; inaccurate GRNs propagate into modules.

## Evidence

- [intro] In addition, CEFCON detects the regulon-like gene modules (RGMs) involving the identified driver regulators.: "In addition, CEFCON detects the regulon-like gene modules (RGMs) involving the identified driver regulators."
- [methods] RGM detection via RGM_activity: "# Identify regulon-like gene modules
cefcon_results.RGM_activity()"
- [methods] Edge threshold parameter: "cefcon_results = cefcon_GRN_model.get_cefcon_results(edge_threshold_avgDegree=8)"
- [readme] AUCell role: "Moreover, CEFCON detects gene regulatory modules (i.e., RGMs) involving the identified driver regulators and measure their activities using [AUCell]"
- [results] Enrichment precision finding: [evidence span withheld — anchor status: not_found]
