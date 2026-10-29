---
name: de-novo-tfbm-motif-discovery
description: Use when an initial GRN inferred from RNA-Seq time-series data needs
  refinement with binding motif evidence, and a GenBank genome file with promoter
  sequences is available.
when_to_use_negative:
- The organism lacks an annotated genome (GenBank file) for promoter extraction —
  the article notes refining steps cannot be performed without additional knowledge
  of a genome
- Only the initial MI-based GRN is needed and computational time is critical — the
  article reports refinement is computationally very demanding (714.18 s vs 83.07
  s without refinement on DREAM 4)
- Regulons of interest contain fewer than five genes, below the minimum size used
  for de novo TFBM discovery
edam_operation: http://edamontology.org/operation_0238
edam_topics:
- http://edamontology.org/topic_0204
tools:
- name: Augusta
  role: Python package implementing the GRN refinement step; exposes the refineGRN
    function that orchestrates regulon extraction, promoter parsing, and motif discovery
  repo: https://github.com/JanaMus/Augusta
- name: MEME Suite
  role: Performs de novo transcription factor binding motif discovery on regulon promoter
    sets, run via its Docker image with a time bound
- name: Python
  role: Language of the Augusta package; invoked as a Python module from the command
    line
  repo: https://github.com/JanaMus/Augusta
provenance:
  source_task_ids:
  - task_001
  - task_006
  - task_004
  source_papers:
  - doi: 10.1016/j.csbj.2024.01.013
    title: 'Augusta: From RNA‐Seq to gene regulatory networks and Boolean models'
  source_content_hash: sha256:a12617d78d60c587b7257de46e964247daf281f46e4a374b4f219a40f6d2f973
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
  - GENIE3
  - KBoost
  - MEME Suite
  - Python
  - TDAracne
  edam_operation: http://edamontology.org/operation_0238
  edam_topics:
  - http://edamontology.org/topic_0204
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# de-novo-tfbm-motif-discovery

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Refines an initial mutual-information-based GRN by discovering transcription factor binding motifs (TFBMs) de novo in promoter sequences of regulated genes using the MEME Suite, then exporting the discovered motifs for downstream motif search and edge re-typing.

## When to use

Use when an initial GRN inferred from RNA-Seq time-series data needs refinement with binding motif evidence, and a GenBank genome file with promoter sequences is available.

## When NOT to use

- The organism lacks an annotated genome (GenBank file) for promoter extraction — the article notes refining steps cannot be performed without additional knowledge of a genome
- Only the initial MI-based GRN is needed and computational time is critical — the article reports refinement is computationally very demanding (714.18 s vs 83.07 s without refinement on DREAM 4)
- Regulons of interest contain fewer than five genes, below the minimum size used for de novo TFBM discovery

## Inputs

- Initial GRN as adjacency matrix in CSV format (GRN_input)
- Annotated genome sequence in GenBank flat file format (genbank_file_input, e.g. E. coli BW25113 or C. beijerinckii NRRL B-598)
- RNA-Seq time-series count table in CSV format (count_table_input) for MSD-based edge typing
- Promoter length parameter (default 1000 bp upstream)
- MEME Suite motif discovery time limit (motifs_max_time, in seconds)

## Outputs

- Refined GRN as adjacency matrix in CSV format
- Exported TFBM motif file (tfbm_motifs.txt) in MEME/Stockholm (.sto) format, one motif per discovered regulon
- Updated edge directions and activation/repression signs from re-applied MSD time-lag evaluation

## How to apply

Group the initial GRN's target genes into regulons per transcription factor and retain only regulons containing five or more genes, since smaller regulons provide too little promoter signal for de novo motif discovery. Parse the annotated GenBank flat file to extract upstream promoter sequences (default 1000 bp, user-adjustable via the promoter_length parameter) for each retained regulon. Run de novo motif discovery on each regulon's promoter set with the MEME Suite executed via its Docker image, bounding the search with the motifs_max_time parameter. Export the discovered TFBMs (as Stockholm-format motif output alongside the inferred GRN and database interactions), then search the inferred motifs in promoters of other genes in the network to add new edges and re-apply the most significant difference (MSD) time-lag evaluation to assign edge direction and activation/repression sign. This step is optional refinement: run the pipeline with and without it to quantify its effect on sensitivity, specificity, and accuracy against a gold standard.

## Related tools

- **Augusta** (Python package implementing the GRN refinement step; exposes the refineGRN function that orchestrates regulon extraction, promoter parsing, and motif discovery) — https://github.com/JanaMus/Augusta
- **MEME Suite** (Performs de novo transcription factor binding motif discovery on regulon promoter sets, run via its Docker image with a time bound)
- **Python** (Language of the Augusta package; invoked as a Python module from the command line) — https://github.com/JanaMus/Augusta

## Examples

```
Augusta.refineGRN(GRN_input = 'MyGRN_file.csv', genbank_file_input = 'MyGB_file.gb', count_table_input = 'MyCT_file.csv', promoter_length = 1000, motifs_max_time = 600)
```

## Evaluation signals

- tfbm_motifs.txt exists and parses as a valid MEME-format motif file containing one motif per discovered regulon (regulons with ≥5 genes)
- Motifs are exported alongside the refined GRN and database interactions in the expected Stockholm (.sto) output
- Every retained regulon has a promoter sequence length equal to promoter_length (default 1000 bp) extracted from the GenBank file
- Newly introduced motif-derived edges are typed by MSD among adjacent time points, with activation/repression sign assigned
- Refined network counts match reported values when applied to the C. beijerinckii dataset: 2864 TFs / 1880 TGs (vs initial 4684 TFs / 4529 TGs)

## Limitations

- GRN refinement is computationally very demanding; on the DREAM 4 dataset the refined pipeline took 714.18 s vs 83.07 s for MI-only inference
- Refinement steps cannot be performed without additional knowledge of a genome (annotated GenBank file for promoter sequences)
- Inferred networks are approximations, not perfectly accurate representations of biological systems regulation
- Parallelization to reduce computational time is not yet implemented (noted as future work)

## Evidence

- [readme] README refineGRN invocation: [evidence span withheld — anchor status: external_doc]
