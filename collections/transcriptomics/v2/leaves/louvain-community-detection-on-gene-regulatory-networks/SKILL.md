---
name: louvain-community-detection-on-gene-regulatory-networks
description: Use when you have a filtered differential GRN edge list (e.g., BoostDiff
  output post filter_network) and need to partition it into gene modules for pathway
  enrichment (gseapy Enrichr KEGG) or module-level interpretation.
when_to_use_negative:
- Input is the raw boostdiff_network_<keyword>.txt before postprocessing — Louvain
  should be run on the filtered differential network (filter_network with p=3), not
  the unfiltered model output.
- The task is single-condition GRN inference rather than comparing two conditions
  — BoostDiff/Louvain here targets differential networks from disease vs. control
  expression matrices (rows = genes, first column named 'Gene').
edam_operation: http://edamontology.org/operation_0279
edam_topics:
- http://edamontology.org/topic_0611
- http://edamontology.org/topic_0203
tools:
- name: python-louvain
  role: Louvain community detection library (pip name python-louvain, imported as
    community) used to partition the filtered differential network into modules
  repo: https://github.com/taynaud/python-louvain
- name: networkx
  role: Graph construction and drawing of the Louvain partition over the filtered
    BoostDiff edge list
- name: gseapy
  role: Runs KEGG pathway enrichment (Enrichr) on the Louvain module/network genes
    and exports the enrichment table to CSV
- name: BoostDiff
  role: Upstream producer of the differential network whose filtered edge list is
    the input to community detection
  repo: https://github.com/scibiome/boostdiff_inference
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioadv/vbae034
    title: Inference of differential gene regulatory networks using boosted differential
      trees
  source_content_hash: sha256:7473a5c0208d733b0b898fdaea433c049783e8d621d65fb5c37ee2d45bdc5fbe
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/taynaud/python-louvain
  tools:
  - BoostDiff
  - GEOquery
  - Python
  - conda
  - gseapy
  - networkx
  - pip
  - python-louvain
  edam_operation: http://edamontology.org/operation_0279
  edam_topics:
  - http://edamontology.org/topic_0611
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# Louvain community detection on gene regulatory networks

## Summary

Detects modular community structure in a filtered differential gene regulatory network (e.g., the top ~1000 TF-filtered BoostDiff edges per condition) using the Louvain algorithm from python-louvain, exporting module assignments that feed downstream KEGG pathway enrichment.

## When to use

Use when you have a filtered differential GRN edge list (e.g., BoostDiff output post filter_network) and need to partition it into gene modules for pathway enrichment (gseapy Enrichr KEGG) or module-level interpretation.

## When NOT to use

- Input is the raw boostdiff_network_<keyword>.txt before postprocessing — Louvain should be run on the filtered differential network (filter_network with p=3), not the unfiltered model output.
- The task is single-condition GRN inference rather than comparing two conditions — BoostDiff/Louvain here targets differential networks from disease vs. control expression matrices (rows = genes, first column named 'Gene').

## Inputs

- Filtered BoostDiff differential network edge list (boostdiff_network_<keyword>.txt after filter_network with p=3, top ~1000 TF-filtered edges, TSV)
- Gene/module gene list for enrichment (nodes of the filtered differential network)

## Outputs

- Module assignment CSV mapping each network gene to its Louvain community
- Downstream KEGG pathway enrichment table CSV (via gseapy Enrichr) per module/network

## How to apply

First obtain the filtered differential network: apply the BoostDiff postprocessing filter_network function with p=3 (3rd percentile of the largest mean differences in prediction error between conditions) to the raw boostdiff_network_<keyword>.txt, retaining the top ~1000 TF-filtered edges per condition. Load this edge list as an undirected graph in networkx and run community_louvain.best_partition(G); the pip package python-louvain is imported as 'community'. Export the resulting node-to-module assignment to a CSV, then run KEGG pathway enrichment on the module/network genes with gseapy Enrichr. Correct application is judged by whether module genes yield disease-relevant terms — for the GSE156063 COVID-19 network, enrichment should surface 'Coronavirus disease', 'TNF signaling', 'Th17 differentiation', 'NF-kappa B', 'NOD-like receptor', and 'Toll-like receptor' pathways.

## Related tools

- **python-louvain** (Louvain community detection library (pip name python-louvain, imported as community) used to partition the filtered differential network into modules) — https://github.com/taynaud/python-louvain
- **networkx** (Graph construction and drawing of the Louvain partition over the filtered BoostDiff edge list)
- **gseapy** (Runs KEGG pathway enrichment (Enrichr) on the Louvain module/network genes and exports the enrichment table to CSV)
- **BoostDiff** (Upstream producer of the differential network whose filtered edge list is the input to community detection) — https://github.com/scibiome/boostdiff_inference

## Examples

```
import community as community_louvain; partition = community_louvain.best_partition(G)
```

## Evaluation signals

- Module assignment CSV exists and every node of the filtered differential network appears exactly once with a valid community ID.
- Downstream gseapy Enrichr KEGG table exists as a CSV and contains the reported disease-relevant terms: 'Coronavirus disease', 'TNF signaling', 'Th17 differentiation', 'NF-kappa B', 'NOD-like receptor', and 'Toll-like receptor' signaling pathways.
- Edge-list TSV, module CSV, and enrichment CSV are all present as the workflow's required validation artifacts.
- Partition is computed on the filtered network (p=3, top ~1000 TF-filtered edges), not on the raw BoostDiff output.

## Limitations

- Package naming mismatch: pip installs 'python-louvain' but it must be imported as 'community' in Python.
- Module quality depends on the upstream filter threshold (p=3, top ~1000 TF-filtered edges per condition); running detection on the raw or differently filtered network changes the modules and thus the enrichment results.
- BoostDiff-side caveat: the ideal input is at least 30 samples per condition, and n_estimators=50 is recommended to avoid overfitting — poor upstream inference degrades downstream modules.

## Evidence

- [readme] The package name on pip is :code:`python-louvain` 
but it is imported as :code:`community` in python.: "The package name on pip is :code:`python-louvain` 
but it is imported as :code:`community` in python."
- [readme] # compute the best partition
    partition = community_louvain.best_partition(G): "# compute the best partition
    partition = community_louvain.best_partition(G)"
- [other] Detect community structure on the filtered differential network with the Louvain algorithm (python-louvain, taynaud/python-louvain) and export module assignments to a CSV.: [evidence span withheld — anchor status: not_found]
- [methods] df_dis = filter_network(file_net_dis, file_diff_dis, p=3, n_top_edges=150): "df_dis = filter_network(file_net_dis, file_diff_dis, p=3, n_top_edges=150)"
