---
name: ccc-resource-comparison-across-databases
description: Use when the goal is to quantify how much a set of ligand-receptor CCC
  resources (e.g., CellChatDB, CellPhoneDB, Ramilowski, ConnectomeDB, LRdb) overlap
  or differ in unique interactions, receivers, and transmitters before choosing one
  for downstream inference.
when_to_use_negative:
- The aim is to benchmark how method-resource combinations affect CCC predictions
  on single-cell expression data — that requires the separate method-vs-resource combination
  analysis, not the resource-intrinsic overlap statistics
- Only one resource is needed for a downstream CCC inference run and no cross-resource
  comparison is requested
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0605
- http://edamontology.org/topic_3170
tools:
- name: LIANA
  role: Provides a unified interface to load and compare all 16 CCC ligand-receptor
    resources via liana::select_resource
  repo: https://github.com/saezlab/liana
- name: OmnipathR
  role: Back-end resource retrieval layer through which the ligand-receptor resources
    are accessed
  repo: https://github.com/saezlab/OmnipathR
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-022-30755-0
    title: Comparison of methods and resources for cell-cell communication inference
      from single-cell RNA-Seq data
  source_content_hash: sha256:1cad24233f6faadd2060e7cf615bb4ec259ece39634cb64dbfe1fe363b8e253f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/saezlab/liana
  tools:
  - LIANA
  - OmnipathR
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0605
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# ccc-resource-comparison-across-databases

## Summary

Systematically quantifies the overlap and uniqueness of cell-cell communication (CCC) ligand-receptor resources — in this study, the 16 resources accessible via LIANA backed by OmnipathR — by computing unique receivers, transmitters, and interactions per resource and pairwise Jaccard similarity of interactions.

## When to use

Use when the goal is to quantify how much a set of ligand-receptor CCC resources (e.g., CellChatDB, CellPhoneDB, Ramilowski, ConnectomeDB, LRdb) overlap or differ in unique interactions, receivers, and transmitters before choosing one for downstream inference.

## When NOT to use

- The aim is to benchmark how method-resource combinations affect CCC predictions on single-cell expression data — that requires the separate method-vs-resource combination analysis, not the resource-intrinsic overlap statistics
- Only one resource is needed for a downstream CCC inference run and no cross-resource comparison is requested

## Inputs

- The 16 CCC ligand-receptor resources retrievable via liana::select_resource (including CellChatDB, CellPhoneDB, Ramilowski, ConnectomeDB, LRdb), accessed through OmnipathR
- Resource-specific interaction records with transmitter (ligand) and receiver (receptor) gene identifiers, including heteromeric complex subunits

## Outputs

- supplementary_table_1.csv — per-resource summary statistics (mean % unique interactions, unique receivers, unique transmitters)
- Pairwise Jaccard similarity matrix of interactions across the 16 resources
- Resource-overlap visualization reproducing Fig 3

## How to apply

Load all 16 CCC ligand-receptor resources through liana::select_resource in LIANA, which is backed by OmnipathR. Decomplexify heteromeric protein complexes so that each complex-mediated interaction is represented at the level of its constituent subunits, avoiding artefactual uniqueness from differing complex notation. Compute per-resource summary statistics: the mean percentage of unique interactions, unique receivers, and unique transmitters relative to every other resource. Compute the pairwise Jaccard similarity matrix of interactions across all 16 resources to quantify pairwise overlap. Assemble the statistics into a table (Supplementary Table 1, supplementary_table_1.csv) and visualize the resource-overlap structure as in Fig 3. The paper's finding that only on average 6.4% of receivers, 5.7% of transmitters, and 10.4% of interactions are unique to any given resource is the reference benchmark for this analysis.

## Related tools

- **LIANA** (Provides a unified interface to load and compare all 16 CCC ligand-receptor resources via liana::select_resource) — https://github.com/saezlab/liana
- **OmnipathR** (Back-end resource retrieval layer through which the ligand-receptor resources are accessed) — https://github.com/saezlab/OmnipathR

## Examples

```
resources <- c('CellChatDB','CellPhoneDB','Ramilowski2015','ConnectomeDB2020','LRdb','Baccin2019','Compendium2019','EMBRACE','Guide2Pharma','HuRRA','ICELLNET','iTALK','Kirouac2010','OmniPath','SCA','scML sent'); res <- lapply(resources, function(r) liana::select_resource(r)[[1]])
```

## Evaluation signals

- The per-resource mean uniqueness values should fall near the reported benchmarks: ~6.4% unique receivers, ~5.7% unique transmitters, ~10.4% unique interactions on average across the 16 resources
- The Jaccard similarity matrix must be symmetric with diagonal values of 1 and cover all 16 x 16 resource pairs
- All 16 resources must load successfully through liana::select_resource; missing or empty resources invalidate the statistics
- Decomplexification must be verified: complex-mediated interactions are represented by constituent subunits, so no heteromeric complex notation remains in the comparison sets
- supplementary_table_1.csv must contain, for each of the 16 resources, the mean percentage of unique interactions, unique receivers, and unique transmitters

## Limitations

- Resource overlap is measured on resource content alone; it does not indicate which resource or method yields biologically correct predictions, since a gold standard for CCC is largely unknown
- Results are restricted to the 16 human-focused resources available in LIANA at the time of analysis; species and coverage are limited to what those resources curate
- The variable overlap, uneven functional distributions, and curation disagreements between resources themselves call for further large-scale curation efforts

## Evidence

- [results] Across the 16 resources there is limited uniqueness: on average only 6.4% of receivers, 5.7% of transmitters, and 10.4% of interactions are unique to any given resource.: [evidence span withheld — anchor status: not_found]
- [other] Load all 16 cell-cell communication ligand-receptor resources through liana::select_resource in LIANA, backed by OmnipathR.: [evidence span withheld — anchor status: not_found]
- [abstract] Among the resources, we find few unique interactions, a varying degree of overlap, and an uneven coverage of specific pathways and tissue-enriched proteins.: "Among the resources, we find few unique interactions, a varying degree of overlap, and an uneven coverage of specific pathways and tissue-enriched proteins."
- [readme] LIANA enables the use of any combination of ligand-receptor methods and resources, and their consensus.: "LIANA enables the use of any combination of ligand-receptor methods and resources, and their consensus."
- [discussion] the variable overlap between the resources, their uneven functional distributions, and the reported curation disagreements are a call for further large-scale curation efforts: "the variable overlap between the resources, their uneven functional distributions, and the reported curation disagreements are a call for further large-scale curation efforts"
