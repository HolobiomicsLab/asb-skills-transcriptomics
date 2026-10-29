---
name: protein-complex-decomplexification
description: Use when comparing or aggregating ligand-receptor CCC resources that
  encode heteromeric complexes, and interactions must be represented per constituent
  subunit to compute overlap, Jaccard similarity, or per-entity uniqueness statistics.
when_to_use_negative:
- When performing downstream CCC inference with a single resource where complex-level
  representation should be preserved for scoring (e.g., CellPhoneDB uses the minimum
  expression of heteromeric complex subunits)
- When the resources contain no heteromeric complexes and only single-protein ligand-receptor
  pairs
tools:
- name: LIANA
  role: Loads the 16 ligand-receptor resources via liana::select_resource and provides
    the data structures to be decomplexified
  repo: https://github.com/saezlab/liana
- name: OmnipathR
  role: Backend client that retrieves the underlying ligand-receptor resource data
  repo: https://github.com/saezlab/OmnipathR
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-022-30755-0
    title: Comparison of methods and resources for cell-cell communication inference
      from single-cell RNA-Seq data
  source_content_hash: sha256:19e1b0f47d4d10f5b43e477801746dc53fb0d0a14626e506fd15d871e9da6bda
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/saezlab/liana
  tools:
  - LIANA
  - OmnipathR
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# protein-complex-decomplexification

## Summary

Decomplexification of heteromeric ligand-receptor protein complexes into their constituent subunits so that complex-mediated cell-cell communication (CCC) interactions from multiple resources can be compared at the level of individual proteins. This enables cross-resource counting of unique interactions, receivers, and transmitters (e.g., the 6.4%/5.7%/10.4% uniqueness statistics across 16 CCC resources).

## When to use

Use when comparing or aggregating ligand-receptor CCC resources that encode heteromeric complexes, and interactions must be represented per constituent subunit to compute overlap, Jaccard similarity, or per-entity uniqueness statistics.

## When NOT to use

- When performing downstream CCC inference with a single resource where complex-level representation should be preserved for scoring (e.g., CellPhoneDB uses the minimum expression of heteromeric complex subunits)
- When the resources contain no heteromeric complexes and only single-protein ligand-receptor pairs

## Inputs

- Ligand-receptor CCC resources retrieved via liana::select_resource (backed by OmnipathR), including heteromeric complexes (e.g., CellPhoneDB, CellChatDB, Ramilowski, ConnectomeDB, LRdb)
- Resource identifier list specifying which of the 16 CCC resources to load

## Outputs

- Decomplexified interaction tables with complexes expanded to constituent subunits
- Per-resource summary statistics: mean percentage of unique interactions, unique receivers, and unique transmitters
- Pairwise Jaccard similarity matrix of interactions across the 16 resources
- Supplementary Table 1 (supplementary_table_1.csv) and the Fig 3 resource-overlap visualization

## How to apply

Load the 16 CCC ligand-receptor resources through liana::select_resource in LIANA, which is backed by OmnipathR for resource retrieval. For each resource, decomplexify heteromeric protein complexes so that each complex-mediated interaction is represented at the level of its constituent subunits, ensuring that a complex listed as a transmitter or receiver in one resource can be matched against single-protein entries or alternative subunit combinations in other resources. After decomplexification, compute per-resource summary statistics: the mean percentage of unique interactions, unique receivers, and unique transmitters relative to all other resources. Then compute the pairwise Jaccard similarity matrix of interactions across all resources to quantify overlap. Assemble the per-resource uniqueness statistics into a table (e.g., supplementary_table_1.csv) and use the Jaccard matrix to reproduce the resource-overlap visualization (Fig 3). The rationale is that without decomplexification, the same biological interaction represented as different complex compositions across resources would be falsely counted as unique.

## Related tools

- **LIANA** (Loads the 16 ligand-receptor resources via liana::select_resource and provides the data structures to be decomplexified) — https://github.com/saezlab/liana
- **OmnipathR** (Backend client that retrieves the underlying ligand-receptor resource data) — https://github.com/saezlab/OmnipathR

## Examples

```
library(liana); lr_resources <- select_resource(c("CellPhoneDB", "CellChatDB", "Ramilowski"))
```

## Evaluation signals

- Average uniqueness statistics should reproduce approximately 6.4% unique receivers, 5.7% unique transmitters, and 10.4% unique interactions per resource
- Every heteromeric complex in the loaded resources must appear as its constituent subunits in the decomplexified tables — no complex entities should remain unexpanded
- The pairwise Jaccard similarity matrix should be symmetric with a diagonal of 1 and dimensions 16 x 16
- supplementary_table_1.csv should contain per-resource rows with unique interaction, receiver, and transmitter percentages
- Cross-resource matching of the same interaction should succeed when subunits overlap, indicating decomplexification resolved complex-composition discrepancies

## Limitations

- A gold standard for CCC is unavailable, so overlap statistics cannot indicate which resource representation is biologically correct
- Resources show curation disagreements and uneven coverage of specific pathways and tissue-enriched proteins, so decomplexified comparisons reflect curation differences, not only biology
- Gene expression is a proxy of protein levels alone; decomplexification at the protein-interaction level does not resolve other layers of CCC inference bias

## Evidence

- [methods] Workflow: decomplexify complexes: [evidence span withheld — anchor status: not_found]
- [results] Uniqueness statistics: [evidence span withheld — anchor status: not_found]
- [abstract] Resource loading interface: "we provide LIANA, a LIgand-receptor ANalysis frAmework as an open-source interface to all the resources and methods"
- [readme] LIANA README purpose: "LIANA enables the use of any combination of ligand-receptor methods and resources, and their consensus."
