---
name: gene-regulatory-network-neighborhood-extraction
description: Use when you have an inferred lineage-specific GRN plus driver regulators
  with influence scores and need gene modules around those regulators with per-cell
  activity scores.
when_to_use_negative:
- The driver regulators have not yet been identified (no driver_regulators.csv with
  influence scores) — run the network-control/driver-regulator stage first.
- The analysis targets multiple fate trajectories with bifurcations at once — CEFCON
  can only handle one cell lineage at a time.
- The gene modules you need must be canonical TF-only regulons — CEFCON's RGMs include
  non-TF genes, so they are regulon-like rather than strict regulons.
edam_operation: http://edamontology.org/operation_2436
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0602
tools:
- name: CEFCON
  role: Framework whose RGM_activity step detects regulon-like gene modules of the
    identified driver regulators and measures their activities
  repo: https://github.com/WPZgithub/CEFCON
- name: networkx
  role: Extract out-degree and in-degree neighborhoods of each driver regulator from
    the inferred GRN graph
- name: scanpy
  role: Environment for reading and handling the scRNA-seq expression matrix used
    for AUCell activity scoring
  repo: https://scanpy.readthedocs.io/en/stable/installation.html
- name: AUCell
  role: Computes per-cell activity scores for each regulon-like gene module
  repo: https://github.com/aertslab/AUCell
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-023-44103-3
    title: Deciphering driver regulators of cell fate decisions from single-cell transcriptomics
      data with CEFCON
  source_content_hash: sha256:53d72b9ec7b1b1b5834b448ebdc9b853bf08f820a7f1dabeadf29270386e54fc
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
triage_status: critique
yes_rate: 0.4375
n_claims: 16
---

# gene-regulatory-network-neighborhood-extraction

## Summary

Extracts regulon-like gene modules (RGMs) from a cell-lineage-specific GRN by taking the out-degree and in-degree neighborhoods of identified driver regulators, filters them by differential expression and a minimum module size of 10 genes, and scores per-cell module activity with AUCell.

## When to use

Use when you have an inferred lineage-specific GRN plus driver regulators with influence scores and need gene modules around those regulators with per-cell activity scores.

## When NOT to use

- The driver regulators have not yet been identified (no driver_regulators.csv with influence scores) — run the network-control/driver-regulator stage first.
- The analysis targets multiple fate trajectories with bifurcations at once — CEFCON can only handle one cell lineage at a time.
- The gene modules you need must be canonical TF-only regulons — CEFCON's RGMs include non-TF genes, so they are regulon-like rather than strict regulons.

## Inputs

- cell-lineage-specific GRN edge list (cell_lineage_GRN.csv)
- driver regulators with influence scores (driver_regulators.csv)
- gene differential expression file (log fold change per gene)
- scRNA-seq expression matrix (rows = cells, columns = genes)

## Outputs

- RGMs.csv — retained regulon-like gene modules (out-degree and in-degree types) passing DE and size filters
- AUCell_mtx.csv — per-cell activity scores for every RGM across all cells

## How to apply

Load the constructed cell-lineage-specific GRN (cell_lineage_GRN.csv) and driver regulators with their influence scores (driver_regulators.csv) using pandas. Using networkx on the inferred GRN graph, extract for each driver regulator its out-degree neighborhood (direct targets it regulates) and its in-degree neighborhood (targets co-regulated by shared incoming regulators); CEFCON detects all modules of driver regulators including non-TF genes, hence 'regulon-like'. Filter candidate modules against the gene differential expression file (log fold change per gene) and retain only modules passing the differential-expression criterion with at least 10 genes, saving the retained modules to RGMs.csv. Compute per-cell module activity with AUCell (via the pyscenic/scanpy environment) on the scRNA-seq expression matrix (rows = cells, columns = genes) and write AUCell_mtx.csv.

## Related tools

- **CEFCON** (Framework whose RGM_activity step detects regulon-like gene modules of the identified driver regulators and measures their activities) — https://github.com/WPZgithub/CEFCON
- **networkx** (Extract out-degree and in-degree neighborhoods of each driver regulator from the inferred GRN graph)
- **scanpy** (Environment for reading and handling the scRNA-seq expression matrix used for AUCell activity scoring) — https://scanpy.readthedocs.io/en/stable/installation.html
- **AUCell** (Computes per-cell activity scores for each regulon-like gene module) — https://github.com/aertslab/AUCell

## Examples

```
cefcon_results.RGM_activity()  # after cefcon_results.driver_regulators(); saves RGMs and per-cell AUCell activity
```

## Evaluation signals

- RGMs.csv contains only modules that pass the differential-expression filter and the >= 10 gene size threshold
- Each retained module is anchored on an identified driver regulator and is of out-degree or in-degree type
- AUCell_mtx.csv provides an activity score for every RGM across all cells (rows = cells, columns = modules)
- Modules include non-TF genes when the driver regulators are non-TFs, consistent with 'regulon-like' definition
- All module genes are members of the inferred cell-lineage-specific GRN

## Limitations

- RGMs are regulon-like rather than canonical regulons: they include all gene modules of the driver regulators, including non-TF genes
- Cannot process multiple fate trajectories with bifurcations simultaneously; one cell lineage at a time
- Module retention depends on the differential-expression criterion and the >= 10 gene size threshold, so small or non-DE modules are discarded

## Evidence

- [other] CEFCON detects all gene modules of the identified driver regulators (including non-TF genes), termed regulon-like gene modules (RGMs), which come in both out-degree and in-degree types: [evidence span withheld — anchor status: not_found]
- [other] per-cell RGM activities are subsequently measured with AUCell: [evidence span withheld — anchor status: not_found]
- [readme] CEFCON detects gene regulatory modules (i.e., RGMs) involving the identified driver regulators and measure their activities using AUCell: "CEFCON detects gene regulatory modules (i.e., RGMs) involving the identified driver regulators and measure 
their activities using [AUCell](https://github.com/aertslab/AUCell)"
- [methods] Identify regulon-like gene modules (RGMs) and their activity: "# Identify regulon-like gene modules
cefcon_results.RGM_activity()"
