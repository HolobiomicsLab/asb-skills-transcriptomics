---
name: boolean-network-inference-from-grn
description: Use when a refined GRN adjacency matrix is available and a draft Boolean
  model suitable for further manual editing (SBML-qual) is needed, especially for
  organisms with or without models in the Cell Collective database.
when_to_use_negative:
- No GRN has been inferred/refined yet — the conversion expects the refined GRN adjacency
  matrix, not raw RNA-Seq count tables (use RNASeq_to_BN instead).
- A final, fully validated Boolean model is required — Augusta BNs are first drafts,
  not final networks.
- The goal is dynamic network simulation rather than drafting a logical model (e.g.
  SQUAD-style GUI simulation).
edam_operation: http://edamontology.org/operation_1781
edam_topics:
- http://edamontology.org/topic_0602
- http://edamontology.org/topic_2259
tools:
- name: Augusta
  role: Open-source Python package implementing the GRN-to-BN conversion (Augusta.GRN_to_BN),
    including CC database search and generic Boolean rule construction
  repo: https://github.com/JanaMus/Augusta
- name: CellNOpt
  role: Python package (converted to Python 3) used to generate the SBML-qual output
    file
- name: Cell Collective database
  role: Source of curated Boolean models; only models for the studied organism are
    extracted and Boolean functions transferred from them
- name: Python
  role: Implementation language of the Augusta package (versions 3.7 or 3.8 required)
  repo: https://github.com/JanaMus/Augusta
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.csbj.2024.01.013
    title: 'Augusta: From RNA‐Seq to gene regulatory networks and Boolean models'
  source_content_hash: sha256:0b6a816c62f3341f39b9e66e3787385bd48d957f0657b2784fb19d2069b49a20
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
  edam_operation: http://edamontology.org/operation_1781
  edam_topics:
  - http://edamontology.org/topic_0602
  - http://edamontology.org/topic_2259
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# boolean-network-inference-from-grn

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Converts a refined Gene Regulatory Network (adjacency matrix) into a draft Boolean Network by transferring curated Boolean functions from the Cell Collective database where target gene and all transcription factors match, and by building generic logical rules (OR for same-sign edges, AND with NOT for mixed-sign edges) for remaining nodes, exported as an SBML-qual file via CellNOpt.

## When to use

Use when a refined GRN adjacency matrix is available and a draft Boolean model suitable for further manual editing (SBML-qual) is needed, especially for organisms with or without models in the Cell Collective database.

## When NOT to use

- No GRN has been inferred/refined yet — the conversion expects the refined GRN adjacency matrix, not raw RNA-Seq count tables (use RNASeq_to_BN instead).
- A final, fully validated Boolean model is required — Augusta BNs are first drafts, not final networks.
- The goal is dynamic network simulation rather than drafting a logical model (e.g. SQUAD-style GUI simulation).

## Inputs

- Final refined GRN provided as an adjacency matrix (CSV, e.g. 'MyGRN_file.csv')
- Cell Collective curated Boolean model database
- Promoter length parameter (promoter_length)
- GenBank genome file (genbank_file_input, e.g. 'MyGB_file.gb')
- Flag controlling incorporation of curated database information (add_dbs_info)

## Outputs

- Draft Boolean Network in SBML-qual file format (further manually editable)
- Boolean logical rules assigned to incoming edges of each target gene

## How to apply

Load the final refined GRN as an adjacency matrix (the upstream output of Augusta's GRN refinement). Search the Cell Collective (CC) database and extract only the curated Boolean models for the organism under study; for each target gene, transfer its Boolean function from the CC model only when the target gene and all of its transcription factors are present in that model, otherwise fall back to a generic function. Construct generic Boolean functions for remaining nodes: apply logical OR when all incoming edges share the same sign (e.g. A = B or C), and logical AND combined with NOT when both positive and negative edges influence the target gene (e.g. G = (not H) and I). Generate the SBML-qual output using the CellNOpt Python package (converted to Python 3), preserving nodes not connected to the rest of the network.

## Related tools

- **Augusta** (Open-source Python package implementing the GRN-to-BN conversion (Augusta.GRN_to_BN), including CC database search and generic Boolean rule construction) — https://github.com/JanaMus/Augusta
- **CellNOpt** (Python package (converted to Python 3) used to generate the SBML-qual output file)
- **Cell Collective database** (Source of curated Boolean models; only models for the studied organism are extracted and Boolean functions transferred from them)
- **Python** (Implementation language of the Augusta package (versions 3.7 or 3.8 required)) — https://github.com/JanaMus/Augusta

## Examples

```
>>> Augusta.GRN_to_BN(GRN_input = 'MyGRN_file.csv', promoter_length = My_number, genbank_file_input = 'MyGB_file.gb', add_dbs_info = 'My_string')
```

## Evaluation signals

- Exported SBML-qual file exists at the expected output path.
- The output file parses as well-formed XML.
- The file conforms to the SBML-qual qualitative-model format.
- Every network node, including nodes not connected to the rest of the network (disconnected ones), is represented in the output model.
- Target genes fully present in the Cell Collective model (TG and all its transcription factors) carry the transferred CC Boolean function; others carry generic OR / AND-NOT rules consistent with edge signs.

## Limitations

- BNs provided by Augusta should be treated with caution as they rather present first drafts than final BNs.
- Inferred networks should be considered as approximations, not perfectly accurate representations of biological systems regulations.
- Only curated models for the studied organism can be transferred from the Cell Collective; for non-model organisms most functions come from generic rules.
- Augusta requires Python 3.7 or 3.8 and Docker; the README notes installation in a virtual environment is highly recommended.

## Evidence

- [abstract] a refined GRN is transformed into a draft BN by searching in the curated model database and setting logical rules to incoming edges of target genes: "a refined GRN is transformed into a draft BN by searching in the curated model database and setting logical rules to incoming edges of target genes"
- [abstract] can be further manually edited as the model is provided in the SBML file format: "can be further manually edited as the model is provided in the SBML file format"
- [methods] the final GRN is converted into a Boolean Network (BN) by the Cell Collective (CC) database search and the addition of Boolean logic rules: "the final GRN is converted into a Boolean Network (BN) by the Cell Collective (CC) database search and the addition of Boolean logic rules"
- [readme] >>> Augusta.GRN_to_BN(GRN_input = 'MyGRN_file.csv', promoter_length = My_number, genbank_file_input = 'MyGB_file.gb', add_dbs_info = 'My_string'): [evidence span withheld — anchor status: external_doc]
