---
name: tfbm-motif-file-export
description: Use when refining an inferred GRN and you need de novo TFBM discovery
  over regulon promoters plus a validated exported motif file as an auxiliary refinement
  output.
when_to_use_negative:
- No GenBank flat file (or equivalent annotated genome) is available, since refinement
  steps cannot be performed without additional knowledge of a genome
- You only need the initial MI-based GRN and want to skip the computationally very
  demanding refinement stage
- Regulons contain fewer than five genes, which are dropped before motif discovery
edam_operation: http://edamontology.org/operation_0238
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0749
tools:
- name: Augusta
  role: Python package whose refineGRN step runs de novo TFBM discovery on regulon
    promoters and exports the motif file
  repo: https://github.com/JanaMus/Augusta
- name: MEME Suite
  role: De novo motif discovery engine executed via Docker on each retained regulon's
    promoter set
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.csbj.2024.01.013
    title: 'Augusta: From RNA‐Seq to gene regulatory networks and Boolean models'
  source_content_hash: sha256:7ecfe04bfa14d104aa409bba22e7466a97cbc86937b8952274ba7dbb687d686e
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
  edam_operation: http://edamontology.org/operation_0238
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0749
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# tfbm-motif-file-export

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Exports the transcription factor binding motifs (TFBMs) discovered during Augusta's GRN refinement stage, running de novo motif discovery on regulon promoter sequences with the MEME Suite and writing the motifs to the motif file output (tfbm_motifs.txt / Stockholm .sto format) alongside the refined GRN.

## When to use

Use when refining an inferred GRN and you need de novo TFBM discovery over regulon promoters plus a validated exported motif file as an auxiliary refinement output.

## When NOT to use

- No GenBank flat file (or equivalent annotated genome) is available, since refinement steps cannot be performed without additional knowledge of a genome
- You only need the initial MI-based GRN and want to skip the computationally very demanding refinement stage
- Regulons contain fewer than five genes, which are dropped before motif discovery

## Inputs

- Initial GRN as a CSV edge list (GRN_input, e.g. MyGRN_file.csv)
- Annotated genome in GenBank flat file format (genbank_file_input, e.g. MyGB_file.gb)
- RNA-Seq count table CSV (count_table_input, e.g. MyCT_file.csv)
- Upstream promoter sequences (default 1000 bp) extracted from the GenBank file per regulon

## Outputs

- Exported TFBM motif file (tfbm_motifs.txt) in MEME format, one motif per discovered regulon
- Motifs in Stockholm (.sto) file format alongside the inferred GRN and database interactions
- Refined GRN with motif-based and curated-database-supported edges

## How to apply

Load the initial GRN (e.g. inferred via mutual information from RNA-Seq count data) and group target genes into regulons per transcription factor, retaining only regulons with five or more genes. Parse the GenBank flat file to extract upstream promoter sequences (default length 1000 bp, user-adjustable via promoter_length) for the genes of each retained regulon. Run de novo motif discovery on each regulon's promoter set using the MEME Suite via its Docker image, bounding runtime with motifs_max_time. Write the discovered TFBMs to the exported motif file (tfbm_motifs.txt) as the auxiliary output of the refinement step, alongside the refined GRN and database interaction information. Validate that the motif file exists and parses as a valid MEME-format motif file containing one motif per discovered regulon.

## Related tools

- **Augusta** (Python package whose refineGRN step runs de novo TFBM discovery on regulon promoters and exports the motif file) — https://github.com/JanaMus/Augusta
- **MEME Suite** (De novo motif discovery engine executed via Docker on each retained regulon's promoter set)

## Examples

```
import Augusta; Augusta.refineGRN(GRN_input='MyGRN_file.csv', genbank_file_input='MyGB_file.gb', count_table_input='MyCT_file.csv', promoter_length=1000, motifs_max_time=My_seconds)
```

## Evaluation signals

- tfbm_motifs.txt exists as an auxiliary output of the refinement step and parses as a valid MEME-format motif file
- The motif file contains one motif per discovered regulon (one-to-one regulon-to-motif correspondence)
- Only regulons with five or more genes were passed to motif discovery
- Promoter sequences were extracted with the configured upstream length (default 1000 bp) from the GenBank file
- Discovered motifs appear in Stockholm (.sto) format alongside the refined GRN and database interactions

## Limitations

- GRN refinement, including TFBM discovery, is computationally very demanding and is bounded by the motifs_max_time parameter
- Refinement steps cannot be performed without additional knowledge of a genome (GenBank annotation required)
- Motif discovery is limited to regulons with at least five genes and depends on the MEME Suite running via Docker

## Evidence

- [full_text] Finding: motifs exported in Stockholm format: [evidence span withheld — anchor status: not_found]
- [readme] README refineGRN invocation shows the refinement entry point and its parameters: [evidence span withheld — anchor status: external_doc]
