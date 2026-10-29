---
name: promoter-sequence-extraction-from-genbank
description: Use when refining an inferred GRN by de novo transcription factor binding
  motif prediction and a GenBank-format genome annotation is available to pull upstream
  promoter regions for regulon genes.
when_to_use_negative:
- No annotated GenBank file exists for the organism of interest — the article notes
  that 'refining steps cannot be performed without additional knowledge of a genome'
- The input GRN regulons contain fewer than five genes, which Augusta excludes from
  refinement
- The goal is to use curated transcription factor databases (OmniPath, Signor, SignaLink,
  TRRUST) rather than de novo promoter-based motif prediction
edam_topics:
- http://edamontology.org/topic_0204
- http://edamontology.org/topic_0102
tools:
- name: Augusta
  role: Implements the GenBank parsing and promoter extraction step within its GRN
    refinement function refineGRN
  repo: https://github.com/JanaMus/Augusta
- name: MEME Suite
  role: Consumes the extracted promoter sequences for de novo TFBM discovery via its
    Docker image
- name: Python
  role: Language in which the Augusta package (including GenBank parsing) is implemented
  repo: https://github.com/JanaMus/Augusta
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.csbj.2024.01.013
    title: 'Augusta: From RNA‐Seq to gene regulatory networks and Boolean models'
  source_content_hash: sha256:ee6e94f9a7b5fb15fcdef72542ba54052f8f4eb3d9ed7dff8740e5519e1b7867
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
  - MEME Suite
  - Python
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0204
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# promoter-sequence-extraction-from-genbank

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

In Augusta's GRN refinement, an annotated GenBank flat file is parsed to extract upstream promoter sequences (default 1000 bp, user-adjustable) for genes in retained regulons, supplying the raw sequence input for de novo TFBM discovery with MEME.

## When to use

Use when refining an inferred GRN by de novo transcription factor binding motif prediction and a GenBank-format genome annotation is available to pull upstream promoter regions for regulon genes.

## When NOT to use

- No annotated GenBank file exists for the organism of interest — the article notes that 'refining steps cannot be performed without additional knowledge of a genome'
- The input GRN regulons contain fewer than five genes, which Augusta excludes from refinement
- The goal is to use curated transcription factor databases (OmniPath, Signor, SignaLink, TRRUST) rather than de novo promoter-based motif prediction

## Inputs

- Annotated genome in GenBank flat file format (.gb)
- Inferred GRN in CSV form grouping genes into regulons per transcription factor (only regulons with >= 5 genes)
- Count table (CSV) as required by the refineGRN call
- promoter_length parameter (default 1000 bp, user-adjustable)

## Outputs

- Upstream promoter sequences (default 1000 bp) for the genes of each retained regulon, passed to de novo motif discovery
- Downstream auxiliary output: tfbm_motifs.txt, a MEME-format motif file with one motif per discovered regulon

## How to apply

Group target genes into regulons per transcription factor from the initial GRN and retain only regulons containing five or more genes. Parse the organism's GenBank flat file (e.g., the complete annotated Escherichia coli BW25113 genome or B. subtilis AL009126.3) and extract the upstream promoter sequence for every gene in each retained regulon, using the default promoter length of 1000 bp or a user-supplied promoter_length value. In Augusta this is invoked through the refineGRN function, where genbank_file_input and promoter_length are explicit parameters. The rationale is that motif discovery needs candidate cis-regulatory sequence, which the annotation provides even for organisms absent from curated databases. Downstream, the extracted promoter sets feed MEME Suite (via Docker) for de novo TFBM discovery, and the discovered motifs are exported to tfbm_motifs.txt.

## Related tools

- **Augusta** (Implements the GenBank parsing and promoter extraction step within its GRN refinement function refineGRN) — https://github.com/JanaMus/Augusta
- **MEME Suite** (Consumes the extracted promoter sequences for de novo TFBM discovery via its Docker image)
- **Python** (Language in which the Augusta package (including GenBank parsing) is implemented) — https://github.com/JanaMus/Augusta

## Examples

```
Augusta.refineGRN(GRN_input = 'MyGRN_file.csv', genbank_file_input = 'MyGB_file.gb', count_table_input = 'MyCT_file.csv', promoter_length = 1000, motifs_max_time = 3600)
```

## Evaluation signals

- Every extracted promoter sequence is upstream of a gene belonging to a regulon with >= 5 genes in the input GRN
- Each extracted sequence length equals the promoter_length setting (default 1000 bp) or the maximum available upstream distance from the GenBank annotation
- The downstream tfbm_motifs.txt exists and parses as a valid MEME-format motif file containing one motif per discovered regulon
- Sequences are consistent with the GenBank coordinates of the input organism (e.g., E. coli BW25113 genome)

## Limitations

- Refinement steps, including promoter extraction and TFBM discovery, cannot be performed without additional knowledge of a genome (a GenBank annotation)
- GRN refinement, including the motif-discovery stage fed by these promoters, is computationally very demanding
- Inferring autoregulations remains a significant challenge, so promoter extraction may miss regulatory structure for self-regulating TFs

## Evidence

- [readme] Concrete refineGRN invocation showing genbank_file_input and promoter_length parameters: [evidence span withheld — anchor status: external_doc]
