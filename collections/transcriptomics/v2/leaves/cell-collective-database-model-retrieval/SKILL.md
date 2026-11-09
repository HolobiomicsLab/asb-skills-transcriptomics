---
name: cell-collective-database-model-retrieval
description: Use when a refined GRN (adjacency matrix) must be converted into a draft
  Boolean Network and curated Boolean functions are available in the Cell Collective
  for the organism under study.
when_to_use_negative:
- The organism has no curated Boolean models in the Cell Collective database (e.g.
  many non-model microbes), so only generic Boolean functions will be produced and
  no curated rules can be transferred.
- The goal is a final, publication-quality Boolean Network — the article explicitly
  warns that BNs provided by Augusta are first drafts, not final networks.
- The upstream GRN has not been refined (no TFBM prediction or curated database interactions
  incorporated), since this conversion stage expects the refined GRN as input.
edam_operation: http://edamontology.org/operation_3660
edam_topics:
- http://edamontology.org/topic_0602
- http://edamontology.org/topic_0204
- http://edamontology.org/topic_2259
tools:
- name: Augusta
  role: 'Implements the GRN-to-BN conversion: CC database search, Boolean rule transfer/generic
    rule construction, and SBML-qual export via the GRN_to_BN function'
  repo: https://github.com/JanaMus/Augusta
- name: Cell Collective
  role: Source database of curated Boolean models from which organism-specific Boolean
    functions are extracted
- name: CellNOpt
  role: Python package (converted to Python 3) used to generate the SBML-qual output
    file
- name: Python
  role: Language in which Augusta and the CellNOpt conversion are implemented
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.csbj.2024.01.013
    title: 'Augusta: From RNA‐Seq to gene regulatory networks and Boolean models'
  source_content_hash: sha256:68f07ae928f319501e0e908a70622ab3845b2edf38dea5ccd26dca209544d62a
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
  - Cell Collective
  - Cell Collective database
  - CellNOpt
  - Python
  edam_operation: http://edamontology.org/operation_3660
  edam_topics:
  - http://edamontology.org/topic_0602
  - http://edamontology.org/topic_0204
  - http://edamontology.org/topic_2259
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# cell-collective-database-model-retrieval

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Retrieves curated Boolean models for a studied organism from the Cell Collective (CC) database and transfers their Boolean logic rules onto a refined Gene Regulatory Network to construct a draft Boolean Network exported as an SBML-qual file.

## When to use

Use when a refined GRN (adjacency matrix) must be converted into a draft Boolean Network and curated Boolean functions are available in the Cell Collective for the organism under study.

## When NOT to use

- The organism has no curated Boolean models in the Cell Collective database (e.g. many non-model microbes), so only generic Boolean functions will be produced and no curated rules can be transferred.
- The goal is a final, publication-quality Boolean Network — the article explicitly warns that BNs provided by Augusta are first drafts, not final networks.
- The upstream GRN has not been refined (no TFBM prediction or curated database interactions incorporated), since this conversion stage expects the refined GRN as input.

## Inputs

- Final refined GRN provided as an adjacency matrix (CSV, e.g. GRN_input file)
- Cell Collective database of curated Boolean models
- Optional: GenBank file and promoter length parameters passed through the Augusta GRN_to_BN interface

## Outputs

- Draft Boolean Network in SBML-qual file format (parseable XML, qualitative-model compliant) suitable for further manual editing

## How to apply

Load the final refined GRN as an adjacency matrix (the upstream output of Augusta's GRN refinement), then search the Cell Collective database and extract only the curated Boolean models corresponding to the organism under study. For each target gene (TG) in the GRN, transfer its Boolean function from the extracted CC model only when the TG and all of its transcription factors are present in the CC model; otherwise construct a generic function. Build generic Boolean functions for remaining nodes using the logical OR operator when incoming edges are all of the same sign (e.g. A = B or C) and logical AND combined with NOT when both positive and negative edges influence the TG (e.g. G = (not H) and I). Generate the SBML-qual output file using the CellNOpt Python package (converted to Python 3), preserving nodes not connected to the rest of the network. The output is a first draft BN meant to be further manually edited, so validate that it parses as well-formed XML and conforms to the SBML-qual qualitative-model format with every node (including disconnected ones) represented.

## Related tools

- **Augusta** (Implements the GRN-to-BN conversion: CC database search, Boolean rule transfer/generic rule construction, and SBML-qual export via the GRN_to_BN function) — https://github.com/JanaMus/Augusta
- **Cell Collective** (Source database of curated Boolean models from which organism-specific Boolean functions are extracted)
- **CellNOpt** (Python package (converted to Python 3) used to generate the SBML-qual output file)
- **Python** (Language in which Augusta and the CellNOpt conversion are implemented)

## Examples

```
import Augusta; Augusta.GRN_to_BN(GRN_input = 'MyGRN_file.csv', promoter_length = My_number, genbank_file_input = 'MyGB_file.gb', add_dbs_info = 'My_string')
```

## Evaluation signals

- Exported SBML-qual file exists and parses as well-formed XML.
- Exported file conforms to the SBML-qual qualitative-model format.
- Every network node, including nodes not connected to the rest of the network, is represented in the SBML-qual output.
- Boolean rules for target genes were transferred from the CC model only when the TG and all of its transcription factors were present in the CC model; fallback generic rules use OR for same-sign edges and (not X) and Y for mixed-sign edges.
- Only curated models for the studied organism were extracted from the Cell Collective database.

## Limitations

- BNs provided by Augusta should be treated with caution as they present first drafts rather than final BNs.
- Inferring autoregulations remains a significant challenge.
- The approach works even when information about the organism is not in the databases, but then no curated CC rules can be transferred and only generic functions are generated.
- Inferred networks are approximations, not perfectly accurate representations of biological system regulation.

## Evidence

- [readme] Augusta.GRN_to_BN(GRN_input = 'MyGRN_file.csv', promoter_length = My_number, genbank_file_input = 'MyGB_file.gb', add_dbs_info = 'My_string'): [evidence span withheld — anchor status: external_doc]
