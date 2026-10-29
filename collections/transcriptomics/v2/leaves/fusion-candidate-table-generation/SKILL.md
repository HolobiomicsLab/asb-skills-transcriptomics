---
name: fusion-candidate-table-generation
description: Use when tumor RNA-Seq fastq reads are available as part of a neoantigen
  prediction workflow and gene fusion candidates must be identified to support downstream
  fusion-derived neoantigen calling.
when_to_use_negative:
- No RNA-Seq data are provided — per the README, RNA-seq is not necessary to provide,
  and the fusion module is skipped; WES/WGS-only runs proceed with SNV/INDEL-based
  neoantigen prediction
- The goal is quantifying gene expression rather than detecting fusions — TSNAD v2.0
  handles expression via a separate RNA-Seq analysis module (StringTie/TPM-based)
- Fusions are already available from another caller and only peptide prediction is
  needed — feed fusion-derived peptides directly to the DeepHLApan step instead
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0199
tools:
- name: STAR
  role: Aligns RNA-Seq reads to the GRCh37/GRCh38 reference genome to produce the
    sorted BAM consumed by the fusion detector
  repo: https://github.com/jiujiezz/tsnad
- name: Arriba
  role: Detects gene fusion transcripts from the STAR-aligned BAM and writes fusion
    candidates to fusions.tsv; chosen over STAR-fusion for speed and accuracy
  repo: https://github.com/jiujiezz/tsnad
- name: TSNAD v2.0
  role: Wraps and orchestrates the STAR + Arriba fusion module inside the one-stop
    neoantigen prediction pipeline
  repo: https://github.com/jiujiezz/tsnad
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.csbj.2021.08.016
    title: 'TSNAD v2.0: A one-stop software solution for tumor-specific neoantigen
      detection'
  source_content_hash: sha256:8765bfa108f94d9ba43642a4de9b0473a795b55ec74fd3652528fe220fd7269b
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/jiujiezz/tsnad
  repo_url: https://github.com/jiujiezz/tsnad
  tools:
  - Arriba
  - STAR
  - TSNAD v2.0
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0199
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# fusion-candidate-table-generation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Detects gene fusion transcripts from tumor RNA-Seq data by aligning reads with STAR (v2.7) and calling fusions with Arriba (v1.1.0), producing a fusions.tsv table that feeds fusion-derived neoantigen prediction in TSNAD v2.0.

## When to use

Use when tumor RNA-Seq fastq reads are available as part of a neoantigen prediction workflow and gene fusion candidates must be identified to support downstream fusion-derived neoantigen calling.

## When NOT to use

- No RNA-Seq data are provided — per the README, RNA-seq is not necessary to provide, and the fusion module is skipped; WES/WGS-only runs proceed with SNV/INDEL-based neoantigen prediction
- The goal is quantifying gene expression rather than detecting fusions — TSNAD v2.0 handles expression via a separate RNA-Seq analysis module (StringTie/TPM-based)
- Fusions are already available from another caller and only peptide prediction is needed — feed fusion-derived peptides directly to the DeepHLApan step instead

## Inputs

- Tumor RNA-Seq fastq reads (single-end or paired-end) from the sample directory
- Reference genome index (GRCh37 or GRCh38) for STAR alignment
- STAR-produced sorted BAM alignment (input to Arriba)

## Outputs

- fusions.tsv — Arriba-reported fusion calls with gene-pair identifiers and supporting read information
- Sorted RNA-Seq BAM alignment from STAR (intermediate)

## How to apply

Run this module only when RNA-Seq data are provided (RNA-seq is optional in TSNAD's docker run command). First align the RNA-Seq fastq reads to the reference genome (GRCh37 or GRCh38) using the STAR v2.7 aligner to produce a sorted BAM; then run Arriba v1.1.0 on the STAR-aligned BAM to detect gene fusion transcripts. Arriba was selected over STAR-fusion because, per the methods, it is both quicker and more accurate, so its output is the pipeline's canonical fusion result. Write the detected fusion candidates to fusions.tsv, retaining gene-pair identifiers and supporting read information so that fusion-derived peptides can later be scored by DeepHLApan alongside SNV/INDEL-derived neoantigens.

## Related tools

- **STAR** (Aligns RNA-Seq reads to the GRCh37/GRCh38 reference genome to produce the sorted BAM consumed by the fusion detector) — https://github.com/jiujiezz/tsnad
- **Arriba** (Detects gene fusion transcripts from the STAR-aligned BAM and writes fusion candidates to fusions.tsv; chosen over STAR-fusion for speed and accuracy) — https://github.com/jiujiezz/tsnad
- **TSNAD v2.0** (Wraps and orchestrates the STAR + Arriba fusion module inside the one-stop neoantigen prediction pipeline) — https://github.com/jiujiezz/tsnad

## Examples

```
docker run -it -v [dir of WES/WGS]/:/home/tsnad/samples -v [dir of RNA-seq]:/home/tsnad/RNA-seq -v [output dir]:/home/tsnad/results biopharm/tsnad:latest /bin/bash
```

## Evaluation signals

- fusions.tsv exists and is non-empty (when RNA-Seq data were supplied) and contains Arriba-reported fusion calls
- Each fusion entry retains gene-pair identifiers plus supporting read information usable by downstream neoantigen prediction
- The STAR step completed with a sorted BAM built against the genome version matching the -V flag (grch37 or grch38)
- Fusion calls are traceable to Arriba (v1.1.0) output rather than STAR-fusion, since Arriba is the pipeline's canonical fusion result

## Limitations

- Fusion analysis is conditional: it is only performed when RNA-Seq data are provided; the docker invocation accepts the RNA-seq directory as optional
- The module depends on RNA quality and read depth of the tumor RNA-Seq sample; no minimum RNA-Seq depth threshold is specified in the article (the 15x/50x depth guidance applies to WGS/WES)
- Fusion-derived neoantigens feed into the same DeepHLApan thresholds (binding score >0.5, immunogenic score >0.5); low-support fusion calls may not yield candidate peptides

## Evidence

- [methods] Gene fusion detection from RNA-Seq data: "The detection of fusions is based on the RNA-Seq data. STAR (v2.7) aligner [18] is used for mapping reads to the reference genome, Arriba (v1.1.0) [19] is then used for fusion detection."
- [methods] Arriba chosen over STAR-fusion: "Compared with other fusion detectors such as STAR-fusion [20], Arriba is both quicker and more accurate."
- [readme] RNA-Seq is optional input: "You need to enter the TSNAD running enviromont with your path of WES/WGS/RNA-seq as the following command (RNA-seq is not necessary to provide):"
