---
name: tg-oriented-boolean-function-transfer
description: Use when a refined GRN adjacency matrix must be turned into a draft Boolean
  Network model, and curated logical rules (Cell Collective) should be reused where
  possible with generic fallbacks elsewhere.
when_to_use_negative:
- The GRN is only an initial mutual-information-based estimate that has not yet been
  refined (TFBM prediction / curated database interactions), since BN conversion expects
  the final refined GRN
- A fully curated, simulation-ready Boolean model already exists in Cell Collective
  for the organism, making draft BN generation unnecessary
edam_operation: http://edamontology.org/operation_3562
edam_topics:
- http://edamontology.org/topic_0605
- http://edamontology.org/topic_0204
tools:
- name: Augusta
  role: Implements the GRN-to-BN conversion (CC database search, Boolean rule assignment,
    SBML-qual export)
  repo: https://github.com/JanaMus/Augusta
- name: CellNOpt
  role: Python package used to generate the SBML-qual output file (converted to Python
    3)
- name: Cell Collective database
  role: Source of curated Boolean functions transferred to target genes present with
    all their transcription factors
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.csbj.2024.01.013
    title: 'Augusta: From RNA‐Seq to gene regulatory networks and Boolean models'
  source_content_hash: sha256:ce8d419e6f1601dc938dda9837618178304986f0d1db1153d32ecbaacd9d5ba1
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
  - Cell Collective database
  - CellNOpt
  - Python
  edam_operation: http://edamontology.org/operation_3562
  edam_topics:
  - http://edamontology.org/topic_0605
  - http://edamontology.org/topic_0204
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# TG-oriented Boolean function transfer

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Converts a refined Gene Regulatory Network (GRN) into a draft Boolean Network by transferring curated Boolean functions from the Cell Collective database to each target gene (TG) when the TG and all its transcription factors are present, and otherwise constructing generic Boolean functions from incoming edge signs, exported as SBML-qual via CellNOpt.

## When to use

Use when a refined GRN adjacency matrix must be turned into a draft Boolean Network model, and curated logical rules (Cell Collective) should be reused where possible with generic fallbacks elsewhere.

## When NOT to use

- The GRN is only an initial mutual-information-based estimate that has not yet been refined (TFBM prediction / curated database interactions), since BN conversion expects the final refined GRN
- A fully curated, simulation-ready Boolean model already exists in Cell Collective for the organism, making draft BN generation unnecessary

## Inputs

- Final refined GRN provided as an adjacency matrix
- Cell Collective curated Boolean models for the studied organism

## Outputs

- Draft Boolean Network in SBML-qual file format (editable model including disconnected nodes)

## How to apply

Load the final refined GRN as an adjacency matrix and search the Cell Collective (CC) database, extracting only curated Boolean models for the organism under study. For each target gene (TG), transfer its Boolean function from the extracted CC model only when the TG and all of its transcription factors are present in the CC model; otherwise fall back to a generic function. Build generic functions by applying the logical OR operator when all incoming edges share the same sign (e.g. A = B or C), and the AND operator combined with NOT when both positive and negative edges influence the TG (e.g. G = (not H) and I). Generate the SBML-qual output file using the CellNOpt Python package (converted to Python 3), preserving nodes not connected to the rest of the network.

## Related tools

- **Augusta** (Implements the GRN-to-BN conversion (CC database search, Boolean rule assignment, SBML-qual export)) — https://github.com/JanaMus/Augusta
- **CellNOpt** (Python package used to generate the SBML-qual output file (converted to Python 3))
- **Cell Collective database** (Source of curated Boolean functions transferred to target genes present with all their transcription factors)

## Examples

```
Augusta.GRN_to_BN(GRN_input = 'MyGRN_file.csv', promoter_length = My_number, genbank_file_input = 'MyGB_file.gb', add_dbs_info = 'My_string')
```

## Evaluation signals

- Exported file exists and parses as well-formed XML
- File conforms to the SBML-qual qualitative-model format
- Every network node, including disconnected ones, is represented in the exported SBML-qual model
- Target genes present in Cell Collective together with all their transcription factors carry the curated CC Boolean function rather than a generic one
- Same-sign incoming edges yield OR rules, mixed-sign edges yield AND-with-NOT rules (e.g. G = (not H) and I)

## Limitations

- BNs produced by Augusta are first drafts, not final networks, and should be treated with caution
- When a target gene or any of its transcription factors is absent from Cell Collective, only a generic Boolean function can be assigned
- Inferred networks are approximations, not perfectly accurate representations of biological regulation

## Evidence
