---
name: sbml-qual-model-export
description: Use when a finalized, refined GRN (e.g., an adjacency matrix of signed
  gene-gene interactions) needs to be turned into a draft Boolean Network in SBML
  file format for simulation or manual editing.
when_to_use_negative:
- The task is GRN inference or refinement from raw RNA-Seq count tables rather than
  BN construction from an already-refined GRN
- A curated, simulation-ready Boolean model is required — Augusta's output is explicitly
  a first draft to be manually edited
- The organism's genome (GenBank file) is unavailable, since add_dbs_info-based conversion
  depends on additional genomic knowledge
edam_operation: null
edam_topics:
- http://edamontology.org/topic_2258
tools:
- name: Augusta
  role: Provides the GRN_to_BN function that searches the Cell Collective database,
    assigns Boolean logic rules to incoming edges of target genes, and exports the
    draft BN
  repo: https://github.com/JanaMus/Augusta
- name: CellNOpt
  role: Python package (converted to Python 3) used to generate the SBML-qual output
    file for the draft Boolean Network
- name: Cell Collective database
  role: Source of curated Boolean models; only models for the studied organism are
    extracted, and CC functions are transferred to a target gene only when the gene
    and all its transcription factors are present
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.csbj.2024.01.013
    title: 'Augusta: From RNA‐Seq to gene regulatory networks and Boolean models'
  source_content_hash: sha256:98923c1bca9951464968d459d05c006368de4300eafc741e931403cf22f4598f
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
  - http://edamontology.org/topic_2258
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# sbml-qual-model-export

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Convert Augusta's refined Gene Regulatory Network (adjacency matrix) into a draft Boolean Network exported as an SBML-qual qualitative model, by combining curated Boolean functions retrieved from the Cell Collective database with generic logic rules built from the sign of incoming edges.

## When to use

Use when a finalized, refined GRN (e.g., an adjacency matrix of signed gene-gene interactions) needs to be turned into a draft Boolean Network in SBML file format for simulation or manual editing.

## When NOT to use

- The task is GRN inference or refinement from raw RNA-Seq count tables rather than BN construction from an already-refined GRN
- A curated, simulation-ready Boolean model is required — Augusta's output is explicitly a first draft to be manually edited
- The organism's genome (GenBank file) is unavailable, since add_dbs_info-based conversion depends on additional genomic knowledge

## Inputs

- Refined Gene Regulatory Network as an adjacency matrix (CSV, e.g., 'MyGRN_file.csv')
- GenBank file of the organism's genome (e.g., 'MyGB_file.gb')
- Promoter length parameter for upstream sequence extraction
- Cell Collective database of curated Boolean models

## Outputs

- Draft Boolean Network in SBML-qual (SBML qualitative model) file format with every network node represented, including disconnected ones
- Boolean rules assigned to each target gene (curated CC functions or generic OR / AND-NOT rules)

## How to apply

Load the final refined GRN as an adjacency matrix and search the Cell Collective (CC) database, extracting only curated Boolean models for the organism under study. For each target gene (TG), transfer its Boolean function from the extracted CC model only when the TG and all of its transcription factors are present in the CC model; otherwise build a generic Boolean function. Apply the logical OR operator when all incoming edges have the same sign (e.g., A = B or C), and combine AND with NOT when a target gene has both positive and negative regulators (e.g., G = (not H) and I). Generate the SBML-qual output with the CellNOpt Python package (converted to Python 3), preserving nodes not connected to the rest of the network. The result is a first-draft model meant for further manual editing, not a final network.

## Related tools

- **Augusta** (Provides the GRN_to_BN function that searches the Cell Collective database, assigns Boolean logic rules to incoming edges of target genes, and exports the draft BN) — https://github.com/JanaMus/Augusta
- **CellNOpt** (Python package (converted to Python 3) used to generate the SBML-qual output file for the draft Boolean Network)
- **Cell Collective database** (Source of curated Boolean models; only models for the studied organism are extracted, and CC functions are transferred to a target gene only when the gene and all its transcription factors are present)

## Examples

```
Augusta.GRN_to_BN(GRN_input = 'MyGRN_file.csv', promoter_length = My_number, genbank_file_input = 'MyGB_file.gb', add_dbs_info = 'My_string')
```

## Evaluation signals

- The exported file exists and parses as well-formed XML
- The file conforms to the SBML-qual qualitative-model format
- Every network node, including nodes not connected to the rest of the network, is represented in the SBML-qual model
- Target genes fully covered by the Cell Collective model carry the curated CC Boolean function; all others carry generic functions (OR for uniform-sign incoming edges; AND combined with NOT when both positive and negative edges influence the target)
- Each Boolean rule's regulators match the incoming edges of the target gene in the refined GRN adjacency matrix

## Limitations

- BNs provided by Augusta are first drafts, not final networks, and should be treated with caution
- Curated CC Boolean functions can only be transferred when the target gene and all of its transcription factors are present in the extracted Cell Collective model
- The refinement/BN construction steps cannot be performed without additional knowledge of the organism's genome (GenBank file)

## Evidence

- [readme] README invocation: [evidence span withheld — anchor status: external_doc]
