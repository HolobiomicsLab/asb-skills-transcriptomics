---
name: gene-regulatory-network-refinement
description: Use when an initial GRN inferred from RNA-Seq expression data needs refinement
  with TFBM predictions and curated database interactions, especially for non-model
  organisms lacking database coverage.
when_to_use_negative:
- No GenBank annotated genome is available for the organism, since refining steps
  cannot be performed without additional knowledge of a genome
- The user only needs a quick, low-cost GRN estimate — refinement is computationally
  very demanding (e.g., 714.18 s vs. 83.07 s unrefined on DREAM 4)
- The GRN input already contains verified TFBM annotations, making de novo motif discovery
  redundant
edam_operation: http://edamontology.org/operation_0239
edam_topics:
- http://edamontology.org/topic_0749
- http://edamontology.org/topic_0204
tools:
- name: Augusta
  role: Python package providing the refineGRN function that groups regulons, extracts
    promoters, and orchestrates TFBM-based GRN refinement
  repo: https://github.com/JanaMus/Augusta
- name: MEME Suite
  role: De novo motif discovery engine run on each regulon's promoter set via its
    Docker image
- name: Python
  role: Implementation language; Augusta requires Python 3.7 or 3.8 and Docker as
    dependencies
  repo: https://github.com/JanaMus/Augusta
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.csbj.2024.01.013
    title: 'Augusta: From RNA‐Seq to gene regulatory networks and Boolean models'
  source_content_hash: sha256:8cdb110451f9872b55f3be42d6fd061694d0064e22b7950c08d472907b261711
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
  edam_operation: http://edamontology.org/operation_0239
  edam_topics:
  - http://edamontology.org/topic_0749
  - http://edamontology.org/topic_0204
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 15
---

# gene-regulatory-network-refinement

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Refines an initial mutual-information-inferred GRN using de novo transcription factor binding motif (TFBM) discovery on regulon promoters and incorporation of verified database interactions, as implemented in Augusta's refineGRN module.

## When to use

Use when an initial GRN inferred from RNA-Seq expression data needs refinement with TFBM predictions and curated database interactions, especially for non-model organisms lacking database coverage.

## When NOT to use

- No GenBank annotated genome is available for the organism, since refining steps cannot be performed without additional knowledge of a genome
- The user only needs a quick, low-cost GRN estimate — refinement is computationally very demanding (e.g., 714.18 s vs. 83.07 s unrefined on DREAM 4)
- The GRN input already contains verified TFBM annotations, making de novo motif discovery redundant

## Inputs

- Initial GRN file (CSV) inferred from expression data
- GenBank flat file (.gb) with annotated genome sequence for promoter extraction
- RNA-Seq count table (CSV) for regulon grouping
- promoter_length parameter (default 1000 bp)
- motifs_max_time parameter (seconds, bounding MEME runs)

## Outputs

- Refined GRN with TFBM-based edges and verified database interactions
- Exported motif file tfbm_motifs.txt (Stockholm/MEME motif format, one motif per discovered regulon)

## How to apply

Load the initial GRN (mutual-information-inferred from RNA-Seq count data) and group target genes into regulons per transcription factor, retaining only regulons with five or more genes. Parse the GenBank flat file (e.g., the Escherichia coli BW25113 genome in GenBank format) and extract upstream promoter sequences for each retained regulon, using a default length of 1000 bp that the user can adjust via the promoter_length parameter. Run de novo motif discovery on each regulon's promoter set using the MEME Suite via its Docker image, bounded by the motifs_max_time parameter. Export the discovered TFBMs (tfbm_motifs.txt, Stockholm/motif format) alongside the refined GRN with verified database interactions incorporated; optionally chain into BN inference via Cell Collective database search and Boolean logic rules.

## Related tools

- **Augusta** (Python package providing the refineGRN function that groups regulons, extracts promoters, and orchestrates TFBM-based GRN refinement) — https://github.com/JanaMus/Augusta
- **MEME Suite** (De novo motif discovery engine run on each regulon's promoter set via its Docker image)
- **Python** (Implementation language; Augusta requires Python 3.7 or 3.8 and Docker as dependencies) — https://github.com/JanaMus/Augusta

## Examples

```
Augusta.refineGRN(GRN_input = 'MyGRN_file.csv', genbank_file_input = 'MyGB_file.gb', count_table_input = 'MyCT_file.csv', promoter_length = 1000, motifs_max_time = 600)
```

## Evaluation signals

- tfbm_motifs.txt exists and parses as a valid MEME-format motif file containing one motif per discovered regulon
- Every retained regulon used for motif discovery contains at least five genes
- Extracted promoter sequences match the requested promoter_length (default 1000 bp upstream of each target gene in the GenBank file)
- On DREAM 4, refinement should increase specificity/accuracy relative to MI-only inference (specificity 0.91 vs 0.54, accuracy 0.89 vs 0.54)
- The refined GRN reflects both TFBM evidence and verified interactions obtained from curated databases

## Limitations

- GRN refinement is computationally very demanding and parallelization is not yet implemented
- Refining steps cannot be performed without additional knowledge of a genome (GenBank annotation)
- Motif discovery runtime is capped by motifs_max_time, so short budgets may miss motifs
- Inferred networks are approximations, not perfectly accurate representations of biological regulation

## Evidence

- [readme] README refineGRN invocation shows inputs and parameters: [evidence span withheld — anchor status: external_doc]
- [readme] Python and Docker dependencies: [evidence span withheld — anchor status: external_doc]
