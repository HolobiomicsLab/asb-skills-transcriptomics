---
name: grn-edge-conflict-resolution
description: Use when refining a MI-inferred GRN with curated database interactions
  and the same regulator-target pair receives contradictory edge types (e.g., activation
  vs. inhibition) from inference or from multiple databases.
when_to_use_negative:
- The organism under study is a non-model organism absent from OmniPath, Signor, SignaLink,
  and TRRUST — database mining adds no verified edges, and refinement relies only
  on de novo TFBM prediction
- You only have a raw RNA-Seq count table and no initial GRN yet — infer the network
  first with Augusta.RNASeq_to_GRN before conflict resolution is meaningful
- Your goal is Boolean Network conversion with logical rules — that is handled downstream
  by the Cell Collective database search in Augusta.GRN_to_BN, not by edge conflict
  resolution
edam_operation: http://edamontology.org/operation_0279
edam_topics:
- http://edamontology.org/topic_0602
tools:
- name: Augusta
  role: Implements the GRN refinement and database-edge merge/conflict resolution
    via Augusta.refineGRN, exporting final_grn.csv
  repo: https://github.com/JanaMus/Augusta
- name: OmniPath
  role: Manually curated database searched for verified regulatory interactions to
    merge into the GRN
- name: Signor
  role: Manually curated database searched for verified regulatory interactions to
    merge into the GRN
- name: SignaLink
  role: Manually curated database searched for verified regulatory interactions to
    merge into the GRN
- name: TRRUST
  role: Manually curated database searched for verified regulatory interactions to
    merge into the GRN
- name: EcoNameTranslator
  role: Python package used to extend organism and gene names with scientific and
    common names to maximize database retrieval
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.csbj.2024.01.013
    title: 'Augusta: From RNA‐Seq to gene regulatory networks and Boolean models'
  source_content_hash: sha256:f985ec0d6d1e50c7feb15dcca3abc04f4a018e630d080172aadebb68ff23357a
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/WanluLiuLab/CRISPRTE
  repo_url: https://github.com/JanaMus/Augusta
  tools:
  - Augusta
  - EcoNameTranslator
  - OmniPath
  - Python
  - SignaLink
  - Signor
  - TRRUST
  edam_operation: http://edamontology.org/operation_0279
  edam_topics:
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6428571428571429
n_claims: 14
---

# grn-edge-conflict-resolution

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Resolves conflicting regulatory edge types that arise when merging manually curated database interactions (OmniPath, Signor, SignaLink, TRRUST) into an initial mutual-information-inferred GRN during Augusta's refinement step, producing a final GRN adjacency matrix.

## When to use

Use when refining a MI-inferred GRN with curated database interactions and the same regulator-target pair receives contradictory edge types (e.g., activation vs. inhibition) from inference or from multiple databases.

## When NOT to use

- The organism under study is a non-model organism absent from OmniPath, Signor, SignaLink, and TRRUST — database mining adds no verified edges, and refinement relies only on de novo TFBM prediction
- You only have a raw RNA-Seq count table and no initial GRN yet — infer the network first with Augusta.RNASeq_to_GRN before conflict resolution is meaningful
- Your goal is Boolean Network conversion with logical rules — that is handled downstream by the Cell Collective database search in Augusta.GRN_to_BN, not by edge conflict resolution

## Inputs

- Initial GRN as an adjacency matrix (e.g., 'MyGRN_file.csv')
- GenBank file of the organism's annotated genome (e.g., 'MyGB_file.gb')
- Gene count table (e.g., 'MyCT_file.csv') as used by Augusta.refineGRN
- Curated interaction records from OmniPath, Signor, SignaLink, and TRRUST, including edge type and reference counts

## Outputs

- final_grn.csv — adjacency matrix (.csv) of the refined GRN with merged database edges and conflict rules applied

## How to apply

Load the initial GRN adjacency matrix and extract organism and gene names from the input GenBank file, extending them with scientific and common names via EcoNameTranslator to maximize database retrieval. Query the manually curated databases OmniPath, Signor, SignaLink, and TRRUST for the organism's regulatory interactions and add all retrieved relationships to the GRN. Apply the conflict-resolution rules in order: on mismatch between an inferred and a database edge type, prefer the database edge type; between conflicting databases, adopt the edge type backed by the larger number of references; if reference counts are equal, drop the interaction entirely. Export the resolved network as final_grn.csv, an adjacency matrix in .csv format. The rationale is that curated, verified interactions should override de novo inference, with literature support (reference counts) arbitrating between equally curated sources.

## Related tools

- **Augusta** (Implements the GRN refinement and database-edge merge/conflict resolution via Augusta.refineGRN, exporting final_grn.csv) — https://github.com/JanaMus/Augusta
- **OmniPath** (Manually curated database searched for verified regulatory interactions to merge into the GRN)
- **Signor** (Manually curated database searched for verified regulatory interactions to merge into the GRN)
- **SignaLink** (Manually curated database searched for verified regulatory interactions to merge into the GRN)
- **TRRUST** (Manually curated database searched for verified regulatory interactions to merge into the GRN)
- **EcoNameTranslator** (Python package used to extend organism and gene names with scientific and common names to maximize database retrieval)

## Examples

```
Augusta.refineGRN(GRN_input = 'MyGRN_file.csv', genbank_file_input = 'MyGB_file.gb', count_table_input = 'MyCT_file.csv', promoter_length = My_number, motifs_max_time = My_seconds)
```

## Evaluation signals

- final_grn.csv exists and parses as an adjacency matrix in .csv format with database edges incorporated
- For every regulator-target pair where an inferred edge type contradicts a database edge type, the final matrix carries the database edge type
- For pairs where two curated databases conflict, the retained edge type corresponds to the larger number of references
- Pairs with conflicting edge types and equal reference counts are absent from final_grn.csv (interaction dropped)
- Retrieved relationships reflect the queried organism (names extended via EcoNameTranslator), not off-target matches from other species

## Limitations

- Curated databases may lack coverage of the organism; the approach is still applicable, but refinement then depends on de novo TFBM prediction alone
- GRN refinement, including database mining and conflict resolution, can be computationally very demanding
- Refinement steps cannot be performed without additional knowledge of the organism's genome (e.g., a GenBank file)
- Interactions with equal reference counts across conflicting databases are dropped, which may discard biologically real edges

## Evidence

- [readme] refineGRN invocation: [evidence span withheld — anchor status: external_doc]
