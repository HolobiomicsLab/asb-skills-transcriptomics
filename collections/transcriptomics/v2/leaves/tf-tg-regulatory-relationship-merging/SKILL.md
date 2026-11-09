---
name: tf-tg-regulatory-relationship-merging
description: Use when an initial GRN adjacency matrix inferred from expression data
  must be refined before Boolean Network conversion, and curated database interactions
  are available for the organism.
when_to_use_negative:
- The organism is absent from OmniPath, Signor, SignaLink, and TRRUST (typical for
  non-model microbes); the databases will contribute no edges and refinement cannot
  rely on verified interactions
- The goal is Boolean Network logic-rule setting rather than GRN refinement — that
  step uses the Cell Collective database search instead
- No additional knowledge of the genome (e.g., annotated GenBank file) is available,
  since refining steps cannot be performed without it
edam_operation: http://edamontology.org/operation_3566
edam_topics:
- http://edamontology.org/topic_0602
- http://edamontology.org/topic_3170
tools:
- name: Augusta
  role: Python package implementing GRN refinement; the refineGRN function merges
    curated-database interactions into the inferred network
  repo: https://github.com/JanaMus/Augusta
- name: EcoNameTranslator
  role: Extends organism and gene names with scientific and common names to maximize
    database retrieval
- name: OmniPath
  role: Manually curated database of gene/protein interactions searched for organism-specific
    regulatory edges
- name: Signor
  role: Manually curated database of gene/protein interactions searched for organism-specific
    regulatory edges
- name: SignaLink
  role: Manually curated database of gene/protein interactions searched for organism-specific
    regulatory edges
- name: TRRUST
  role: Manually curated database of transcriptional regulatory interactions searched
    for organism-specific edges
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.csbj.2024.01.013
    title: 'Augusta: From RNA‐Seq to gene regulatory networks and Boolean models'
  source_content_hash: sha256:731a65283f87e19446ca9ff8b7cbb375f5baa34c912004e2ae6575ec8afba181
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
  edam_operation: http://edamontology.org/operation_3566
  edam_topics:
  - http://edamontology.org/topic_0602
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 15
---

# TF-TG regulatory relationship merging

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Merges verified transcription factor–target gene regulatory interactions retrieved from curated databases (OmniPath, Signor, SignaLink, TRRUST) into an MI-inferred GRN, using name translation and explicit conflict-resolution rules to produce the final adjacency matrix.

## When to use

Use when an initial GRN adjacency matrix inferred from expression data must be refined before Boolean Network conversion, and curated database interactions are available for the organism.

## When NOT to use

- The organism is absent from OmniPath, Signor, SignaLink, and TRRUST (typical for non-model microbes); the databases will contribute no edges and refinement cannot rely on verified interactions
- The goal is Boolean Network logic-rule setting rather than GRN refinement — that step uses the Cell Collective database search instead
- No additional knowledge of the genome (e.g., annotated GenBank file) is available, since refining steps cannot be performed without it

## Inputs

- Initial GRN adjacency matrix (.csv) from mutual-information-based inference
- GenBank file of the organism's genome providing organism and gene names
- Count table (.csv) as required by Augusta.refineGRN
- Promoter length parameter and motifs_max_time timeout

## Outputs

- final_grn.csv — adjacency matrix of the refined GRN with merged database edges and conflict rules applied

## How to apply

Load the initial GRN adjacency matrix from the MI-based inference and extract organism and gene names from the input GenBank file. Extend organism and gene names with scientific and common names via the EcoNameTranslator Python package to maximize database retrieval. Query the curated databases OmniPath, Signor, SignaLink, and TRRUST for regulatory interactions of the organism under study, and add all available database relationships to the GRN. On edge-type mismatches, prefer the database edge type over the inferred one; between conflicting databases, adopt the edge type with the larger number of references, and drop the interaction if reference counts are equal. Export the merged network as the final GRN adjacency matrix (final_grn.csv).

## Related tools

- **Augusta** (Python package implementing GRN refinement; the refineGRN function merges curated-database interactions into the inferred network) — https://github.com/JanaMus/Augusta
- **EcoNameTranslator** (Extends organism and gene names with scientific and common names to maximize database retrieval)
- **OmniPath** (Manually curated database of gene/protein interactions searched for organism-specific regulatory edges)
- **Signor** (Manually curated database of gene/protein interactions searched for organism-specific regulatory edges)
- **SignaLink** (Manually curated database of gene/protein interactions searched for organism-specific regulatory edges)
- **TRRUST** (Manually curated database of transcriptional regulatory interactions searched for organism-specific edges)

## Examples

```
Augusta.refineGRN(GRN_input = 'MyGRN_file.csv', genbank_file_input = 'MyGB_file.gb', count_table_input = 'MyCT_file.csv', promoter_length = My_number, motifs_max_time = My_seconds)
```

## Evaluation signals

- final_grn.csv exists and parses as a valid adjacency matrix in .csv format
- All regulatory relationships available in the queried databases for the organism are present in the final GRN
- On inferred-vs-database edge-type mismatches, the final matrix carries the database edge type
- Between conflicting databases, the retained edge type matches the larger reference count, and equal-count conflicts result in the interaction being dropped
- Compare behavior with and without refinement: on DREAM 4, refinement shifted performance from sensitivity 0.38 / specificity 0.54 / accuracy 0.54 to sensitivity 0.06 / specificity 0.91 / accuracy 0.89

## Limitations

- Refinement of an initial GRN may be computationally very demanding
- Refining steps cannot be performed without additional knowledge of a genome
- If the organism is not available in the databases — typically the case for non-model organisms including most microbes — database merging contributes nothing
- Inferred networks should be considered approximations and not perfectly accurate representations of biological systems regulations

## Evidence

- [full_text] conflict rule from source card workflow: [evidence span withheld — anchor status: not_found]
- [readme] refineGRN invocation from README: [evidence span withheld — anchor status: external_doc]
