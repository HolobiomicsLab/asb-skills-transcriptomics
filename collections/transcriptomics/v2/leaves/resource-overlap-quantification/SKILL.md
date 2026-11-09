---
name: resource-overlap-quantification
description: Use when you need to compare the content of multiple ligand-receptor
  CCC resources (e.g. CellChatDB, CellPhoneDB, Ramilowski, ConnectomeDB, LRdb) to
  quantify how many interactions, transmitters, or receivers are unique to each resource
  versus shared.
when_to_use_negative:
- When benchmarking inference methods rather than resource contents — method-resource
  combinations and consensus are handled by the article's decoupled method benchmark,
  not by overlap statistics
- When measuring overlap of predicted interactions across cell types or datasets for
  a single resource — this skill quantifies resource content overlap, not prediction
  overlap
- When a method infers intracellular processes (e.g. NicheNet, Cytotalk, SoptSC),
  which the article states are not directly comparable
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: LIANA
  role: Provides liana::select_resource to load and harmonise the 16 CCC ligand-receptor
    resources and the framework in which the overlap statistics are computed
  repo: https://github.com/saezlab/liana
- name: OmnipathR
  role: Backend that provides the underlying ligand-receptor resource data retrieved
    by LIANA
  repo: saezlab/OmnipathR
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-022-30755-0
    title: Comparison of methods and resources for cell-cell communication inference
      from single-cell RNA-Seq data
  source_content_hash: sha256:7f8cbca1a037bea41217a4450ab2108c5dfcbecad00894f621fa2c75e672ba1a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/saezlab/liana
  tools:
  - LIANA
  - OmnipathR
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# resource-overlap-quantification

## Summary

Quantifies the uniqueness and pairwise overlap of cell-cell communication (CCC) ligand-receptor resources — unique interactions, receivers, and transmitters plus a Jaccard similarity matrix across the 16 resources harmonised in LIANA — to expose how much resource choice drives CCC inference.

## When to use

Use when you need to compare the content of multiple ligand-receptor CCC resources (e.g. CellChatDB, CellPhoneDB, Ramilowski, ConnectomeDB, LRdb) to quantify how many interactions, transmitters, or receivers are unique to each resource versus shared.

## When NOT to use

- When benchmarking inference methods rather than resource contents — method-resource combinations and consensus are handled by the article's decoupled method benchmark, not by overlap statistics
- When measuring overlap of predicted interactions across cell types or datasets for a single resource — this skill quantifies resource content overlap, not prediction overlap
- When a method infers intracellular processes (e.g. NicheNet, Cytotalk, SoptSC), which the article states are not directly comparable

## Inputs

- 16 cell-cell communication ligand-receptor resources accessible via liana::select_resource (CellChatDB, CellPhoneDB, Ramilowski, ConnectomeDB, LRdb, OmniPath-derived resources, etc.), backed by OmnipathR
- Resource tables containing transmitter (ligand) and receiver (receptor) genes, including heteromeric complex definitions

## Outputs

- Per-resource summary statistics: mean percentage of unique interactions, unique receivers, and unique transmitters relative to the other resources
- 16x16 pairwise Jaccard similarity matrix of interactions across resources
- Supplementary Table 1 (supplementary_table_1.csv)
- Resource-overlap visualization reproducing Fig 3

## How to apply

Load all 16 ligand-receptor resources through liana::select_resource in LIANA, which is backed by OmnipathR. Decomplexify heteromeric protein complexes so each complex-mediated interaction is represented at the level of its constituent subunits, making resources comparable at the protein-pair level. For each resource, compute the mean percentage of unique interactions, unique receivers, and unique transmitters relative to all other resources. Compute the pairwise Jaccard similarity matrix of interactions across all 16 resources. Assemble the per-resource statistics into Supplementary Table 1 (supplementary_table_1.csv) and reproduce the resource-overlap visualization of Fig 3; expected magnitudes are on average only 6.4% of receivers, 5.7% of transmitters, and 10.4% of interactions unique to any given resource.

## Related tools

- **LIANA** (Provides liana::select_resource to load and harmonise the 16 CCC ligand-receptor resources and the framework in which the overlap statistics are computed) — https://github.com/saezlab/liana
- **OmnipathR** (Backend that provides the underlying ligand-receptor resource data retrieved by LIANA) — saezlab/OmnipathR

## Examples

```
library(liana); resources <- liana::select_resource(c("CellChatDB", "CellPhoneDB", "Ramilowski", "ConnectomeDB", "LRdb"))
```

## Evaluation signals

- Per-resource mean uniqueness falls in the reported range: on average ~6.4% unique receivers, ~5.7% unique transmitters, and ~10.4% unique interactions per resource
- The Jaccard similarity matrix is 16x16, symmetric, with a diagonal of 1 and off-diagonal values reflecting the reported varying degree of overlap
- supplementary_table_1.csv contains, for each of the 16 resources, the mean percentage of unique interactions, unique receivers, and unique transmitters
- Heteromeric complexes are decomplexified so each complex-mediated interaction is counted at the level of its constituent subunits before any uniqueness or Jaccard computation
- Resource-overlap figure matches the structure of Fig 3 of the article (few unique interactions, varying pairwise overlap)

## Limitations

- No gold standard exists and the biological ground truth for CCC is largely unknown, so overlap statistics cannot identify a definitively best resource
- Variable overlap between resources, uneven functional distributions, and reported curation disagreements mean the statistics call for further large-scale curation efforts rather than a single authoritative resource
- Both the choice of method and resource strongly influence predicted intercellular interactions, so resource overlap alone does not determine downstream CCC predictions
- Findings reflect protein-level resource content after decomplexification; processes preceding transmitter-receiver interactions (translation, secretion, diffusion) and long-distance endocrine signalling are not captured

## Evidence

- [abstract] Among the resources, few unique interactions, a varying degree of overlap, and an uneven coverage of specific pathways and tissue-enriched proteins were found.: "Among the resources, we find few unique interactions, a varying degree of overlap, and an uneven coverage of specific pathways and tissue-enriched proteins."
- [intro] First, we explored the degree of overlap among resources and whether certain resources are biased toward specific biological terms, such as pathways and tissue-enriched proteins.: "First, we explored the degree of overlap among resources and whether certain resources are biased toward specific biological terms, such as pathways and tissue-enriched proteins."
- [methods] Decomplexify heteromeric protein complexes so each complex-mediated interaction is represented at the level of its constituent subunits.: [evidence span withheld — anchor status: not_found]
- [supplementary] Load all 16 cell-cell communication ligand-receptor resources through liana::select_resource in LIANA, backed by OmnipathR.: [evidence span withheld — anchor status: not_found]
- [readme] LIANA enables the use of any combination of ligand-receptor methods and resources, and their consensus.: "LIANA enables the use of any combination of ligand-receptor methods and resources, and their consensus."
