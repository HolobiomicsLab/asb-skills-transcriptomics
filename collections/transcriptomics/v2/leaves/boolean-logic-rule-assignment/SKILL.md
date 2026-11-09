---
name: boolean-logic-rule-assignment
description: Use when a refined GRN adjacency matrix is available and a draft Boolean
  Network model in SBML-qual format is needed for further manual editing or simulation.
when_to_use_negative:
- The user needs a final, validated Boolean model rather than a first draft — Augusta's
  BNs 'should be treated with caution as they rather present first drafts than final
  BNs'
- No refined GRN exists yet — the conversion stage requires the upstream GRN refinement
  output, not raw RNA-Seq count tables
- The organism's curated Boolean models must be fully accurate from the start without
  manual editing — the draft requires subsequent manual curation in SBML form
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0605
- http://edamontology.org/topic_0203
tools:
- name: Augusta
  role: Implements the GRN-to-BN conversion (Cell Collective database search plus
    Boolean rule assignment) via the GRN_to_BN function
  repo: https://github.com/JanaMus/Augusta
- name: CellNOpt
  role: Python package (converted to Python 3) used to generate the SBML-qual output
    file
- name: Cell Collective database
  role: Source of curated Boolean models from which logical functions are extracted
    for target genes present with all their transcription factors
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.csbj.2024.01.013
    title: 'Augusta: From RNA‐Seq to gene regulatory networks and Boolean models'
  source_content_hash: sha256:33204fc36e307926a203f434e2ba3bb00663dd79798ba6bc06e982e1105610d9
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0605
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# boolean-logic-rule-assignment

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Augusta's conversion of a refined Gene Regulatory Network (adjacency matrix) into a draft Boolean Network by transferring curated Boolean functions from the Cell Collective database where possible and generating generic logical rules (OR for same-sign edges, AND with NOT for mixed-sign edges), exported as an SBML-qual file.

## When to use

Use when a refined GRN adjacency matrix is available and a draft Boolean Network model in SBML-qual format is needed for further manual editing or simulation.

## When NOT to use

- The user needs a final, validated Boolean model rather than a first draft — Augusta's BNs 'should be treated with caution as they rather present first drafts than final BNs'
- No refined GRN exists yet — the conversion stage requires the upstream GRN refinement output, not raw RNA-Seq count tables
- The organism's curated Boolean models must be fully accurate from the start without manual editing — the draft requires subsequent manual curation in SBML form

## Inputs

- Refined Gene Regulatory Network provided as an adjacency matrix (e.g. CSV from Augusta.GRN_to_BN GRN_input)
- Cell Collective database of curated Boolean models
- Organism identifier used to filter CC models

## Outputs

- Draft Boolean Network in SBML-qual file format (editable, including disconnected nodes)
- Per-gene Boolean logic rules (transferred CC functions or generic OR / AND-NOT rules)

## How to apply

Load the final refined GRN as an adjacency matrix (upstream output of Augusta's GRN refinement). Search the Cell Collective (CC) database and extract only the curated Boolean models for the organism under study. For each target gene (TG), transfer its Boolean function from the extracted CC model only when the TG and all of its transcription factors are present in the CC model; otherwise construct a generic Boolean function, applying the logical OR operator when all incoming edges have the same sign (e.g. A = B or C) and the logical AND operator combined with NOT when both positive and negative edges influence the TG (e.g. G = (not H) and I). Generate the SBML-qual output with the CellNOpt Python package (converted to Python 3), preserving nodes not connected to the rest of the network. Validate that the file exists, parses as well-formed XML, and conforms to the SBML-qual qualitative-model format with every node, including disconnected ones, represented.

## Related tools

- **Augusta** (Implements the GRN-to-BN conversion (Cell Collective database search plus Boolean rule assignment) via the GRN_to_BN function) — https://github.com/JanaMus/Augusta
- **CellNOpt** (Python package (converted to Python 3) used to generate the SBML-qual output file)
- **Cell Collective database** (Source of curated Boolean models from which logical functions are extracted for target genes present with all their transcription factors)

## Examples

```
Augusta.GRN_to_BN(GRN_input = 'MyGRN_file.csv', promoter_length = My_number, genbank_file_input = 'MyGB_file.gb', add_dbs_info = 'My_string')
```

## Evaluation signals

- Exported SBML-qual file exists and parses as well-formed XML
- File conforms to the SBML-qual qualitative-model format
- Every network node, including disconnected ones, is represented in the output model
- Target genes present in the CC model together with all their transcription factors received the transferred CC Boolean function, not a generic one
- Generic rules follow sign logic: OR across same-sign incoming edges; AND with NOT when both positive and negative edges target a gene

## Limitations

- BNs produced by Augusta are first drafts rather than final networks and should be treated with caution
- Inferring autoregulations remains a significant challenge
- The transfer of CC functions only applies when the target gene and all of its transcription factors are present in the CC model; otherwise a generic fallback rule is used
- Nodes not connected to the rest of the network are preserved but have no biologically derived rules

## Evidence

- [methods] GRN to BN conversion via CC search and rule assignment: "the final GRN is converted into a Boolean Network (BN) by the Cell Collective (CC) database search and the addition of Boolean logic rules"
- [abstract] Draft BN by curated database search and logical rules: "a refined GRN is transformed into a draft BN by searching in the curated model database and setting logical rules to incoming edges of target genes"
- [abstract] SBML output supports manual editing: "can be further manually edited as the model is provided in the SBML file format"
- [intro] Augusta leverages CC for logical rules: "it leverages CC to adjust Augusta-inferred logical rules and therefore presents a new alternative to other tools used for BN inference"
- [readme] Augusta provides GRN_to_BN function: [evidence span withheld — anchor status: not_found]
