---
name: regulon-partitioning-from-inferred-grn
description: Use when an inferred GRN (e.g. from Augusta's mutual-information step)
  needs to be grouped into TF-specific regulons before promoter-based motif discovery,
  and small regulons must be pruned.
when_to_use_negative:
- No GenBank-annotated genome is available for the organism, since refinement steps
  cannot be performed without additional knowledge of a genome
- Only a curated-database-driven refinement is desired (e.g. when running Augusta.GRN_to_BN
  with add_dbs_info and no motif-based refinement is needed)
- The GRN is not gene-regulatory (e.g. a protein-protein interaction network), so
  TF-target regulons cannot be defined
edam_topics:
- http://edamontology.org/topic_0797
- http://edamontology.org/topic_0602
tools:
- name: Augusta
  role: Python package implementing the GRN refinement step (regulon grouping, promoter
    extraction, motif-based refinement) via Augusta.refineGRN
  repo: https://github.com/JanaMus/Augusta
- name: MEME Suite
  role: Performs the de novo TFBM discovery on each regulon's promoter sequences (run
    via Docker image)
- name: Python
  role: Language of the Augusta package; invoked as a Python module
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.csbj.2024.01.013
    title: 'Augusta: From RNA‐Seq to gene regulatory networks and Boolean models'
  source_content_hash: sha256:3433fe0c3ef2a00bb4c3525758ebbee0510258b3a39494019e93a7d0bce5629c
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
  - http://edamontology.org/topic_0797
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# regulon-partitioning-from-inferred-grn

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Partitions Augusta's inferred GRN into per-transcription-factor regulons and filters out TFs with fewer than five target genes, preparing regulons whose promoters can be scanned for de novo TFBMs via MEME Suite. This threshold-based grouping is the first step of the GRN refinement stage.

## When to use

Use when an inferred GRN (e.g. from Augusta's mutual-information step) needs to be grouped into TF-specific regulons before promoter-based motif discovery, and small regulons must be pruned.

## When NOT to use

- No GenBank-annotated genome is available for the organism, since refinement steps cannot be performed without additional knowledge of a genome
- Only a curated-database-driven refinement is desired (e.g. when running Augusta.GRN_to_BN with add_dbs_info and no motif-based refinement is needed)
- The GRN is not gene-regulatory (e.g. a protein-protein interaction network), so TF-target regulons cannot be defined

## Inputs

- Inferred GRN file (e.g. MyGRN_file.csv) with TF-target edges
- GenBank flat file of the organism's genome (e.g. MyGB_file.gb) for promoter sequence extraction
- Count table (e.g. MyCT_file.csv) used for the initial GRN inference
- promoter_length parameter (default 1000 bp upstream sequence)
- motifs_max_time parameter (seconds, per-regulon MEME run time limit)

## Outputs

- Set of retained TF regulons (each with >= 5 target genes)
- Upstream promoter sequences per regulon (1000 bp default)
- Discovered TFBMs exported per regulon (tfbm_motifs.txt in MEME format; Stockholm .sto motif format alongside the refined GRN)
- Refined GRN incorporating TFBM and database-derived interactions

## How to apply

Load the inferred GRN (e.g. MyGRN_file.csv from Augusta's RNASeq_to_GRN step) and group each transcription factor's target genes into a regulon. Retain only regulons containing five or more genes, since smaller regulons provide too few promoters for reliable de novo motif discovery. For each retained regulon, extract the upstream promoter sequences (default 1000 bp, adjustable via promoter_length in Augusta.refineGRN) from the GenBank flat file of the organism (e.g. the Escherichia coli BW25113 GenBank file or B. subtilis genome AL009126.3). Run de novo TFBM discovery on each regulon's promoter set with the MEME Suite via Docker (bounded by motifs_max_time). The refinement step in Augusta combines this motif evidence with verified interactions from curated databases (OmniPath, Signor, SignaLink, TRRUST). This step is computationally demanding and requires genome knowledge (the GenBank file), which may be unavailable for poorly annotated organisms.

## Related tools

- **Augusta** (Python package implementing the GRN refinement step (regulon grouping, promoter extraction, motif-based refinement) via Augusta.refineGRN) — https://github.com/JanaMus/Augusta
- **MEME Suite** (Performs the de novo TFBM discovery on each regulon's promoter sequences (run via Docker image))
- **Python** (Language of the Augusta package; invoked as a Python module)

## Examples

```
import Augusta; Augusta.refineGRN(GRN_input='MyGRN_file.csv', genbank_file_input='MyGB_file.gb', count_table_input='MyCT_file.csv', promoter_length=1000, motifs_max_time=600)
```

## Evaluation signals

- Every retained regulon contains at least five target genes; regulons below the threshold are excluded
- tfbm_motifs.txt exists and parses as a valid MEME-format motif file containing one motif per discovered regulon
- Each discovered motif corresponds to exactly one retained regulon's promoter set
- Promoter sequences extracted match the GenBank coordinates for the specified promoter_length (default 1000 bp) upstream of each target gene
- The refined GRN output preserves the TFBM-derived edges and database-verified interactions added during refinement

## Limitations

- GRN refinement is computationally very demanding, and no parallelization was yet implemented to reduce run time
- Refinement steps require additional knowledge of a genome (a GenBank flat file) and cannot be performed for organisms lacking genome annotation
- Inferred networks are approximations and the resulting BNs are first drafts, not final models; autoregulation inference remains a significant challenge
- Known operon structures are not yet incorporated into the refined networks

## Evidence

- [readme] refineGRN invocation: [evidence span withheld — anchor status: external_doc]
- [other] refinement inputs in workflow card: [evidence span withheld — anchor status: not_found]
