---
name: ligand-receptor-resource-loading
description: Use when an analysis needs to enumerate, compare, or overlap multiple
  CCC ligand-receptor resources (e.g., CellChatDB, CellPhoneDB, Ramilowski, ConnectomeDB,
  LRdb) before downstream inference or benchmarking.
when_to_use_negative:
- When only a single, already-chosen resource is needed for one CCC inference run
  — decomplexification and cross-resource uniqueness statistics are unnecessary
- When comparing CCC inference METHODS rather than resources — method comparison requires
  applying scoring schemes to expression data, not loading resource databases
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: LIANA
  role: Provides select_resource to load all 16 ligand-receptor resources in a common
    format
  repo: https://github.com/saezlab/liana
- name: OmnipathR
  role: Backend client retrieving the underlying ligand-receptor interaction data
    from the OmniPath web service
  repo: https://github.com/saezlab/OmnipathR
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-022-30755-0
    title: Comparison of methods and resources for cell-cell communication inference
      from single-cell RNA-Seq data
  source_content_hash: sha256:b24250838e69aa108ad450b32715d3b81e4686b6560e40c673d9a30fbcfe5216
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/saezlab/liana
  tools:
  - LIANA
  - OmnipathR
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# ligand-receptor-resource-loading

## Summary

Loads and harmonizes the 16 curated cell-cell communication ligand-receptor resources via liana::select_resource in LIANA (backed by OmnipathR) so their unique interactions, receivers, and transmitters can be quantified and cross-compared.

## When to use

Use when an analysis needs to enumerate, compare, or overlap multiple CCC ligand-receptor resources (e.g., CellChatDB, CellPhoneDB, Ramilowski, ConnectomeDB, LRdb) before downstream inference or benchmarking.

## When NOT to use

- When only a single, already-chosen resource is needed for one CCC inference run — decomplexification and cross-resource uniqueness statistics are unnecessary
- When comparing CCC inference METHODS rather than resources — method comparison requires applying scoring schemes to expression data, not loading resource databases

## Inputs

- 16 cell-cell communication ligand-receptor resources (e.g., CellChatDB, CellPhoneDB, Ramilowski, ConnectomeDB, LRdb) accessible via liana::select_resource
- OmniPath web service data accessed through OmnipathR as the backend provider

## Outputs

- Per-resource summary statistics: mean percentage of unique interactions, unique receivers, and unique transmitters (supplementary_table_1.csv)
- 16 x 16 pairwise Jaccard similarity matrix of interactions across resources
- Resource-overlap visualization reproducing Fig 3

## How to apply

Retrieve each of the 16 ligand-receptor resources through liana::select_resource, which is backed by OmnipathR as the data provider. Decomplexify heteromeric protein complexes so that each complex-mediated interaction is represented at the level of its constituent subunits, ensuring interactions are comparable across resources. For each resource, compute the percentage of unique interactions, unique receivers, and unique transmitters relative to the other 15 resources, and assemble a pairwise Jaccard similarity matrix of interactions across all 16 resources. Assemble these statistics into a supplementary table (supplementary_table_1.csv) and use them to reproduce the resource-overlap visualization (Fig 3). Correct application is judged by recovering the paper's reported statistics: on average only 6.4% of receivers, 5.7% of transmitters, and 10.4% of interactions are unique to any given resource, confirming limited uniqueness across resources.

## Related tools

- **LIANA** (Provides select_resource to load all 16 ligand-receptor resources in a common format) — https://github.com/saezlab/liana
- **OmnipathR** (Backend client retrieving the underlying ligand-receptor interaction data from the OmniPath web service) — https://github.com/saezlab/OmnipathR

## Examples

```
resources <- liana::select_resource(get = "all"); jaccard_mat <- proxyC::simil(sapply(resources, function(r) unique(paste0(r$source_genesymbol, "->", r$target_genesymbol))) |> as.matrix(), method = "jaccard")
```

## Evaluation signals

- Per-resource unique-fraction statistics reproduce the reported means: ~6.4% unique receivers, ~5.7% unique transmitters, ~10.4% unique interactions
- Every resource contains exactly the expected interaction schema (source/complex subunits, target) after decomplexification, with heteromeric complexes expanded to subunits
- The Jaccard similarity matrix is 16 x 16, symmetric, with unit diagonal and off-diagonal values in [0,1] reflecting varying degrees of overlap
- supplementary_table_1.csv contains one row per resource with unique interaction/receiver/transmitter percentages that sum against the 16-resource universe

## Limitations

- Resources show uneven coverage of specific pathways and tissue-enriched proteins, so 'unique' entries may reflect curation gaps rather than true biology
- No gold standard for CCC exists and the biological ground truth is largely unknown, so overlap statistics cannot identify a definitively best resource
- Curation disagreements between resources are reported, meaning identical interactions may be annotated differently (e.g., complex vs subunit level) without careful decomplexification

## Evidence

- [methods] Load resources via LIANA: [evidence span withheld — anchor status: not_found]
- [results] Limited uniqueness finding: [evidence span withheld — anchor status: not_found]
- [abstract] Systematic comparison scope: "we systematically compare 16 cell-cell communication inference resources and 7 methods, plus the consensus between the methods’ predictions"
- [abstract] Resource overlap conclusion: "Among the resources, we find few unique interactions, a varying degree of overlap, and an uneven coverage of specific pathways and tissue-enriched proteins."
- [readme] LIANA as interface: "LIANA enables the use of any combination of ligand-receptor methods and resources, and their consensus."
