---
name: gene-name-alias-mapping
description: Use when refining an inferred GRN against curated databases (OmniPath,
  Signor, SignaLink, TRRUST) and gene/organism labels from a GenBank file may miss
  synonyms, causing incomplete interaction retrieval.
when_to_use_negative:
- The organism or its genes are not represented in curated databases (typical for
  many non-model microbes), where refinement steps cannot contribute verified interactions
- The task is Boolean Network conversion from an existing final GRN, which uses the
  Cell Collective database search rather than alias-based interaction retrieval
- Only promoter-based de novo TFBM refinement is needed and no database interaction
  search is planned
tools:
- name: Augusta
  role: Pipeline whose refineGRN step performs the alias extension and curated-database
    search before BN conversion
  repo: https://github.com/JanaMus/Augusta
- name: EcoNameTranslator
  role: Python package used to extend organism and gene names with scientific and
    common names to maximize database retrieval
- name: OmniPath
  role: Manually curated interaction database queried with alias-expanded names
- name: Signor
  role: Manually curated interaction database queried with alias-expanded names
- name: SignaLink
  role: Manually curated interaction database queried with alias-expanded names
- name: TRRUST
  role: Manually curated interaction database queried with alias-expanded names
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.csbj.2024.01.013
    title: 'Augusta: From RNA‐Seq to gene regulatory networks and Boolean models'
  source_content_hash: sha256:b4dc82a0a6f9471cfd990570a926c943659b40f5e879d9967cb009e900232796
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
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# gene-name-alias-mapping

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Expands organism and gene names into scientific and common-name aliases (e.g., via the EcoNameTranslator Python package) before querying curated regulatory databases, so that the maximum number of verified interactions can be retrieved for GRN refinement in Augusta.

## When to use

Use when refining an inferred GRN against curated databases (OmniPath, Signor, SignaLink, TRRUST) and gene/organism labels from a GenBank file may miss synonyms, causing incomplete interaction retrieval.

## When NOT to use

- The organism or its genes are not represented in curated databases (typical for many non-model microbes), where refinement steps cannot contribute verified interactions
- The task is Boolean Network conversion from an existing final GRN, which uses the Cell Collective database search rather than alias-based interaction retrieval
- Only promoter-based de novo TFBM refinement is needed and no database interaction search is planned

## Inputs

- Initial GRN adjacency matrix (from MI-based inference, e.g. a .csv GRN_input file)
- GenBank genome file (.gb) providing organism name and gene names
- RNA-Seq count table (.csv) used in the refinement workflow

## Outputs

- Alias-extended organism and gene name sets used for database queries
- Merged GRN with curated database edges (final_grn.csv adjacency matrix)

## How to apply

After inferring the initial GRN with mutual information, load the adjacency matrix and extract the organism name and gene names from the input GenBank file. Extend the organism and gene names with corresponding scientific and common names using the EcoNameTranslator Python package to maximize database retrieval, since curated databases index genes and organisms under heterogeneous aliases. Then search OmniPath, Signor, SignaLink, and TRRUST for regulatory interactions of the studied organism and add all found relationships to the GRN. On conflicts, prefer the database edge type over the inferred one; between conflicting databases adopt the edge type with more references, and drop the interaction when reference counts are equal. Export the merged network as final_grn.csv.

## Related tools

- **Augusta** (Pipeline whose refineGRN step performs the alias extension and curated-database search before BN conversion) — https://github.com/JanaMus/Augusta
- **EcoNameTranslator** (Python package used to extend organism and gene names with scientific and common names to maximize database retrieval)
- **OmniPath** (Manually curated interaction database queried with alias-expanded names)
- **Signor** (Manually curated interaction database queried with alias-expanded names)
- **SignaLink** (Manually curated interaction database queried with alias-expanded names)
- **TRRUST** (Manually curated interaction database queried with alias-expanded names)

## Examples

```
>>> Augusta.refineGRN(GRN_input = 'MyGRN_file.csv', genbank_file_input = 'MyGB_file.gb', count_table_input = 'MyCT_file.csv', promoter_length = My_number, motifs_max_time = My_seconds)
```

## Evaluation signals

- final_grn.csv exists and parses as an adjacency matrix in .csv format
- Merged adjacency matrix contains edges contributed by OmniPath, Signor, SignaLink, and/or TRRUST alongside MI-inferred edges
- Conflict rules are visibly applied: database edge types override inferred types, database-vs-database ties resolved by reference count, and equal-reference conflicts dropped
- Alias expansion increases the number of database hits relative to querying with raw GenBank gene names only

## Limitations

- Refinement steps cannot be performed without additional knowledge of a genome and are computationally very demanding
- The approach may simply yield no hits when the studied organism is absent from the databases, which is typically the case for non-model organisms including most microbes
- Knowledge gaps still exist that disable inferring perfect networks; inferred networks are approximations

## Evidence

- [other] workflow: alias extension: [evidence span withheld — anchor status: not_found]
- [readme] README refineGRN entry point: [evidence span withheld — anchor status: external_doc]
