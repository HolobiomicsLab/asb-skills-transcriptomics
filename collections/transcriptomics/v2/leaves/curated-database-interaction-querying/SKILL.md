---
name: curated-database-interaction-querying
description: Use when a data-driven GRN (e.g., adjacency matrix from mutual information
  inference) needs curation with verified interactions, and an annotated GenBank file
  supplies organism/gene names for database lookup.
when_to_use_negative:
- The organism is a non-model microbe absent from OmniPath, Signor, SignaLink, and
  TRRUST — no verified interactions can be retrieved
- The goal is de novo motif-based refinement only (TFBM discovery with MEME), not
  database interaction incorporation
- The input is raw count data rather than an already inferred GRN adjacency matrix
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0602
- http://edamontology.org/topic_0204
tools:
- name: Augusta
  role: Python package implementing the GRN refinement and database merge step (refineGRN)
  repo: https://github.com/JanaMus/Augusta
- name: Python
  role: Implementation language; EcoNameTranslator used to extend organism/gene names
    before database queries
- name: OmniPath
  role: Curated database searched for verified regulatory interactions
- name: Signor
  role: Curated database searched for verified regulatory interactions
- name: SignaLink
  role: Curated database searched for verified regulatory interactions
- name: TRRUST
  role: Curated database searched for verified regulatory interactions
provenance:
  source_task_ids:
  - task_007
  - task_003
  - task_001
  source_papers:
  - doi: 10.1016/j.csbj.2024.01.013
    title: 'Augusta: From RNA‐Seq to gene regulatory networks and Boolean models'
  source_content_hash: sha256:84baee30d6223a389f94fde04787bebdea2c8aba00301764def1e91b83480297
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
  - GENIE3
  - KBoost
  - MEME Suite
  - OmniPath
  - Python
  - SignaLink
  - Signor
  - TDAracne
  - TRRUST
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0602
  - http://edamontology.org/topic_0204
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 18
---

# curated-database-interaction-querying

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Refines an MI-inferred GRN by querying manually curated interaction databases (OmniPath, Signor, SignaLink, TRRUST) for verified regulatory interactions of the studied organism and merging them into the network with explicit conflict-resolution rules.

## When to use

Use when a data-driven GRN (e.g., adjacency matrix from mutual information inference) needs curation with verified interactions, and an annotated GenBank file supplies organism/gene names for database lookup.

## When NOT to use

- The organism is a non-model microbe absent from OmniPath, Signor, SignaLink, and TRRUST — no verified interactions can be retrieved
- The goal is de novo motif-based refinement only (TFBM discovery with MEME), not database interaction incorporation
- The input is raw count data rather than an already inferred GRN adjacency matrix

## Inputs

- Initial GRN adjacency matrix from MI-based inference (.csv)
- Annotated genome GenBank file (e.g., E. coli BW25113, GenBank ID AL009126.3)
- Curated interaction databases: OmniPath, Signor, SignaLink, TRRUST

## Outputs

- final_grn.csv — adjacency matrix of the refined GRN with database interactions merged and conflict rules applied

## How to apply

Load the initial GRN adjacency matrix and extract organism and gene names from the input GenBank file, then extend both scientific and common names using the EcoNameTranslator Python package to maximize database retrieval. Search OmniPath, Signor, SignaLink, and TRRUST for regulatory interactions of the organism and add all found relationships to the GRN. On edge-type mismatches, prefer the database edge type over the inferred one; between conflicting databases, adopt the edge type with the larger number of references, and drop the interaction entirely if reference counts are equal. Export the merged network as final_grn.csv, an adjacency matrix. Note this step complements TFBM-based refinement and can be skipped when the organism is absent from all databases (as for many non-model microbes).

## Related tools

- **Augusta** (Python package implementing the GRN refinement and database merge step (refineGRN)) — https://github.com/JanaMus/Augusta
- **Python** (Implementation language; EcoNameTranslator used to extend organism/gene names before database queries)
- **OmniPath** (Curated database searched for verified regulatory interactions)
- **Signor** (Curated database searched for verified regulatory interactions)
- **SignaLink** (Curated database searched for verified regulatory interactions)
- **TRRUST** (Curated database searched for verified regulatory interactions)

## Examples

```
Augusta.refineGRN(GRN_input='initial_grn.csv', genbank_file_input='AL009126.3.gb', count_table_input='counts.csv', promoter_length=1000, motifs_max_time=600)
```

## Evaluation signals

- final_grn.csv exists and parses as an adjacency matrix in .csv format
- All regulatory relationships found in the four databases for the organism are present as edges in final_grn.csv
- Edge-type mismatches are resolved by preferring the database edge type over the inferred type
- Conflicting database edges are resolved by the larger reference count, and edges with equal reference counts are dropped
- On the B. subtilis benchmark, accuracy of the refined GRN exceeds the unrefined Augusta run (refinement improved accuracy)

## Limitations

- Refinement steps cannot be performed without additional knowledge of a genome (annotated GenBank input required)
- GRN refinement may be computationally very demanding
- The approach is applicable even if information about the organism under study is not available in the databases, which is typically the case for non-model organisms including most microbes

## Evidence

- [readme] database search output in refineGRN call: [evidence span withheld — anchor status: external_doc]
