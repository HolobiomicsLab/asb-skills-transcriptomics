---
name: driver-regulator-module-filtering
description: Use when you have a CEFCON-inferred cell-lineage-specific GRN plus identified
  driver regulators with influence scores and need their gene modules (out-degree
  and in-degree types) with per-cell AUCell activity.
when_to_use_negative:
- You need modules for a single driver regulator only — CEFCON detects RGMs involving
  all identified driver regulators; restrict results downstream instead.
- Your trajectory contains multiple fate bifurcations to process simultaneously —
  CEFCON can only handle one cell lineage at a time.
- You only need the GRN or driver regulators, not downstream gene modules and their
  activity.
edam_operation: http://edamontology.org/operation_2436
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0602
tools:
- name: CEFCON
  role: Framework whose results object provides the RGM identification and activity
    measurement (RGM_activity) step applied to the inferred GRN and driver regulators
  repo: https://github.com/WPZgithub/CEFCON
- name: networkx
  role: Graph library used to extract out-degree and in-degree neighborhoods of driver
    regulators from the inferred GRN graph
- name: scanpy
  role: scRNA-seq analysis environment (with pyscenic) used to load the expression
    matrix and run per-cell AUCell module activity scoring
- name: AUCell
  role: R-package that measures per-cell activity of the identified regulon-like gene
    modules on the scRNA-seq data
  repo: https://github.com/aertslab/AUCell
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-023-44103-3
    title: Deciphering driver regulators of cell fate decisions from single-cell transcriptomics
      data with CEFCON
  source_content_hash: sha256:34d1e6b5b3bd9cc2ac6b3bb7544ca498d12a512063624e60210628a80ae5bcca
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
  edam_operation: http://edamontology.org/operation_2436
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# driver-regulator-module-filtering

## Summary

Identifies regulon-like gene modules (RGMs) associated with CEFCON's driver regulators from the cell-lineage-specific GRN, retaining only modules passing a differential-expression filter and a minimum gene-size threshold, then quantifies per-cell module activity with AUCell. This is the final stage of CEFCON's deciphering of driver regulators of cell fate decisions from scRNA-seq data.

## When to use

Use when you have a CEFCON-inferred cell-lineage-specific GRN plus identified driver regulators with influence scores and need their gene modules (out-degree and in-degree types) with per-cell AUCell activity.

## When NOT to use

- You need modules for a single driver regulator only — CEFCON detects RGMs involving all identified driver regulators; restrict results downstream instead.
- Your trajectory contains multiple fate bifurcations to process simultaneously — CEFCON can only handle one cell lineage at a time.
- You only need the GRN or driver regulators, not downstream gene modules and their activity.

## Inputs

- Cell-lineage-specific GRN inferred by CEFCON (cell_lineage_GRN.csv)
- Identified driver regulators with influence scores (driver_regulators.csv)
- Gene differential expression file with log fold change per gene
- scRNA-seq expression matrix (rows = cells, columns = genes)

## Outputs

- Regulon-like gene modules (RGMs) for out-degree and in-degree driver regulator types (RGMs.csv)
- Per-cell RGM activity matrix from AUCell (AUCell_mtx.csv)

## How to apply

Load the constructed cell-lineage-specific GRN (cell_lineage_GRN.csv) and the driver regulators with their influence scores (driver_regulators.csv) using pandas. With networkx on the inferred GRN graph, extract for each driver regulator its out-degree neighborhood (direct targets it regulates) and its in-degree neighborhood (targets co-regulated by shared incoming regulators); CEFCON detects RGMs for all driver regulators, including non-TF genes. Filter candidate modules against the gene differential expression file (log fold change per gene) and retain only modules passing the differential-expression criterion with at least 10 genes, saving them to RGMs.csv. Then compute per-cell RGM activity with AUCell (via the pyscenic/scanpy environment) on the scRNA-seq expression matrix (rows = cells, columns = genes) and save the resulting activity matrix to AUCell_mtx.csv. In CEFCON's Python API this is exposed as the RGM_activity step of the results object.

## Related tools

- **CEFCON** (Framework whose results object provides the RGM identification and activity measurement (RGM_activity) step applied to the inferred GRN and driver regulators) — https://github.com/WPZgithub/CEFCON
- **networkx** (Graph library used to extract out-degree and in-degree neighborhoods of driver regulators from the inferred GRN graph)
- **scanpy** (scRNA-seq analysis environment (with pyscenic) used to load the expression matrix and run per-cell AUCell module activity scoring)
- **AUCell** (R-package that measures per-cell activity of the identified regulon-like gene modules on the scRNA-seq data) — https://github.com/aertslab/AUCell

## Examples

```
cefcon_results.driver_regulators(); cefcon_results.RGM_activity()
```

## Evaluation signals

- RGMs.csv contains only modules that pass the differential-expression filter (log fold change criterion) applied to the gene DE file
- Every retained module in RGMs.csv has at least 10 genes (size threshold)
- Both out-degree-type and in-degree-type RGMs are present, corresponding to the regulatory roles of the driver regulators
- AUCell_mtx.csv provides an activity score for every RGM across all cells (full coverage, rows = cells, columns = modules)

## Limitations

- CEFCON can only handle one cell lineage at a time and cannot process multiple fate trajectories with bifurcations simultaneously
- CEFCON only considers intracellular factors; extrinsic factors such as cell–cell communication and spatial location are not modeled
- Epigenetic information controlling cell fate decisions is not incorporated

## Evidence

- [methods] RGM detection with out-/in-degree types: "CEFCON detects the regulon-like gene modules (RGMs) involving the identified driver regulators."
- [readme] AUCell activity measurement: "Moreover, CEFCON detects gene regulatory modules (i.e., RGMs) involving the identified driver regulators and measure their activities using AUCell"
- [methods] RGM_activity API step: "# Identify regulon-like gene modules
cefcon_results.RGM_activity()"
- [abstract] Module and activity finding: "then models cell fate dynamics through network control theory to identify driver regulators and the associated gene modules"
