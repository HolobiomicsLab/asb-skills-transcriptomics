---
name: adjacency-matrix-network-export
description: Use when a mutual-information-inferred GRN needs to be refined with curated
  database interactions and persisted as an adjacency-matrix .csv (final_grn.csv)
  before downstream Boolean Network conversion.
when_to_use_negative:
- The organism under study is absent from OmniPath, Signor, SignaLink, and TRRUST
  and you cannot tolerate an unchanged or partially refined network (though Augusta
  still supports non-model organisms via TFBM-only refinement)
- You only need the initial MI-based GRN estimate and do not intend to convert it
  to a Boolean Network or incorporate verified interactions
- The input is raw, unnormalized RNA-Seq count data — normalization (CPM, RPKM, or
  TPM) must precede GRN inference and refinement
edam_operation: http://edamontology.org/operation_3925
edam_topics:
- http://edamontology.org/topic_0611
- http://edamontology.org/topic_0602
tools:
- name: Augusta
  role: Provides the refineGRN step that merges curated database interactions into
    the inferred GRN and exports final_grn.csv as an adjacency matrix
  repo: https://github.com/JanaMus/Augusta
- name: OmniPath
  role: Manually curated source of gene/protein regulatory interactions added to the
    GRN
- name: Signor
  role: Manually curated source of gene/protein regulatory interactions added to the
    GRN
- name: SignaLink
  role: Manually curated source of gene/protein regulatory interactions added to the
    GRN
- name: TRRUST
  role: Manually curated source of transcriptional regulatory interactions added to
    the GRN
- name: EcoNameTranslator
  role: Python package used to extend organism and gene names with scientific and
    common names to maximize database retrieval
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.csbj.2024.01.013
    title: 'Augusta: From RNA‐Seq to gene regulatory networks and Boolean models'
  source_content_hash: sha256:5e7a34195980f8b7a30499085635e5f98d2ccbf00fc75bc5145ac9a8c8e414bf
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
  edam_operation: http://edamontology.org/operation_3925
  edam_topics:
  - http://edamontology.org/topic_0611
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# adjacency-matrix-network-export

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Exports the database-refined Gene Regulatory Network from Augusta as an adjacency matrix in .csv format (final_grn.csv), merging verified interactions from OmniPath, Signor, SignaLink, and TRRUST into the MI-inferred network and applying edge-conflict rules before export.

## When to use

Use when a mutual-information-inferred GRN needs to be refined with curated database interactions and persisted as an adjacency-matrix .csv (final_grn.csv) before downstream Boolean Network conversion.

## When NOT to use

- The organism under study is absent from OmniPath, Signor, SignaLink, and TRRUST and you cannot tolerate an unchanged or partially refined network (though Augusta still supports non-model organisms via TFBM-only refinement)
- You only need the initial MI-based GRN estimate and do not intend to convert it to a Boolean Network or incorporate verified interactions
- The input is raw, unnormalized RNA-Seq count data — normalization (CPM, RPKM, or TPM) must precede GRN inference and refinement

## Inputs

- Initial GRN as an adjacency matrix from MI-based inference (e.g. MyGRN_file.csv)
- Annotated genome in GenBank file format (e.g. MyGB_file.gb)
- Normalized RNA-Seq count table (.csv, e.g. MyCT_file.csv)
- Promoter length parameter and motifs_max_time limit
- Curated interaction databases: OmniPath, Signor, SignaLink, TRRUST

## Outputs

- final_grn.csv — adjacency matrix (.csv) of the refined GRN with merged database edges and conflict rules applied

## How to apply

Load the initial GRN adjacency matrix produced by Augusta's MI-based inference and extract organism and gene names from the input GenBank file, extending them with scientific and common names via the EcoNameTranslator package to maximize database retrieval. Query the manually curated databases OmniPath, Signor, SignaLink, and TRRUST for regulatory interactions of the organism and add all retrieved relationships to the GRN. Resolve conflicts deterministically: on mismatch between a database edge and an inferred edge, prefer the database edge type; between conflicting databases, adopt the edge type with the larger number of references and drop the interaction if reference counts tie. Serialize the merged network as an adjacency matrix in a .csv file (final_grn.csv) so it can feed Augusta's Cell Collective database search and Boolean logic rule assignment step.

## Related tools

- **Augusta** (Provides the refineGRN step that merges curated database interactions into the inferred GRN and exports final_grn.csv as an adjacency matrix) — https://github.com/JanaMus/Augusta
- **OmniPath** (Manually curated source of gene/protein regulatory interactions added to the GRN)
- **Signor** (Manually curated source of gene/protein regulatory interactions added to the GRN)
- **SignaLink** (Manually curated source of gene/protein regulatory interactions added to the GRN)
- **TRRUST** (Manually curated source of transcriptional regulatory interactions added to the GRN)
- **EcoNameTranslator** (Python package used to extend organism and gene names with scientific and common names to maximize database retrieval)

## Examples

```
Augusta.refineGRN(GRN_input = 'MyGRN_file.csv', genbank_file_input = 'MyGB_file.gb', count_table_input = 'MyCT_file.csv', promoter_length = 300, motifs_max_time = 60)
```

## Evaluation signals

- final_grn.csv exists and parses as an adjacency matrix in .csv format
- All regulatory relationships retrieved from OmniPath, Signor, SignaLink, and TRRUST for the organism are present as edges in the exported matrix
- On edge-type mismatch, the database edge type overrides the inferred edge type in the exported matrix
- Between conflicting database annotations, the exported edge matches the type with the larger number of references; interactions with equal reference counts are dropped
- The exported adjacency matrix loads successfully as GRN_input for the downstream Augusta.GRN_to_BN step

## Limitations

- Refinement steps require additional knowledge of a genome and cannot be performed without it; refinement may also be computationally very demanding
- The approach depends on curated databases — organisms absent from them (typical for non-model microbes) gain no verified interactions from this step
- Inferred networks are approximations, not perfectly accurate representations of biological regulation, and knowledge gaps still prevent inferring perfect networks

## Evidence

- [methods] Database search and conflict-merge rationale: "Manually curated DBs containing interactions of genes or proteins, particularly OmniPath [41], Signor [27], SignaLink [10], and TRRUST [17]"
- [full_text] Export and validation target: [evidence span withheld — anchor status: not_found]
- [full_text] Conflict resolution rule: [evidence span withheld — anchor status: not_found]
- [readme] Concrete refineGRN invocation from the README: [evidence span withheld — anchor status: external_doc]
