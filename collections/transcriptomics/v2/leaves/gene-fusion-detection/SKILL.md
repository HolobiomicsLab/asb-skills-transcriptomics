---
name: gene-fusion-detection
description: Use when tumor RNA-Seq FASTQ reads are supplied to TSNAD v2.0 and fusion-derived
  neoantigen candidates must be generated alongside WES/WGS-based SNV/INDEL neoantigens.
when_to_use_negative:
- No RNA-Seq data are available for the sample — the README notes RNA-seq is optional
  ('RNA-seq is not necessary to provide'), and fusion detection is RNA-Seq-based
- You need to reproduce calls with STAR-fusion instead — the article's canonical fusion
  result is Arriba's output
- Only genomic DNA (WES/WGS) is present and no expressed-transcript evidence is needed
  for fusion neoantigens
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0622
tools:
- name: STAR 2.7
  role: Aligns tumor RNA-Seq reads to the reference genome to produce the BAM that
    Arriba consumes for fusion detection
- name: Arriba 1.1.0
  role: Performs gene fusion detection on the STAR-aligned BAM; chosen over STAR-fusion
    as quicker and more accurate
- name: TSNAD v2.0
  role: Integrates and orchestrates the fusion module within the one-stop neoantigen
    prediction pipeline
  repo: https://github.com/jiujiezz/tsnad
provenance:
  source_task_ids:
  - task_006
  - task_003
  source_papers:
  - doi: 10.1016/j.csbj.2021.08.016
    title: 'TSNAD v2.0: A one-stop software solution for tumor-specific neoantigen
      detection'
  source_content_hash: sha256:ba86955f6cf43d2187d0e9ec13d1639dd956bd19d019e9d671d60ae89d6007c9
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
  - Arriba 1.1.0
  - BWA
  - DeepHLApan
  - Docker
  - GATK
  - HISAT2
  - OptiType
  - SAMtools
  - STAR
  - STAR 2.7
  - StringTie
  - TSNAD v2.0
  - Trimmomatic
  - VEP
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# gene-fusion-detection

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Detects tumor gene fusions from tumor RNA-Seq data inside the TSNAD v2.0 pipeline by aligning reads with STAR v2.7 and calling fusions with Arriba v1.1.0, producing fusion candidates (fusions.tsv) for downstream fusion-derived neoantigen prediction.

## When to use

Use when tumor RNA-Seq FASTQ reads are supplied to TSNAD v2.0 and fusion-derived neoantigen candidates must be generated alongside WES/WGS-based SNV/INDEL neoantigens.

## When NOT to use

- No RNA-Seq data are available for the sample — the README notes RNA-seq is optional ('RNA-seq is not necessary to provide'), and fusion detection is RNA-Seq-based
- You need to reproduce calls with STAR-fusion instead — the article's canonical fusion result is Arriba's output
- Only genomic DNA (WES/WGS) is present and no expressed-transcript evidence is needed for fusion neoantigens

## Inputs

- Tumor RNA-Seq FASTQ reads (mounted at /home/tsnad/RNA-seq)
- Reference genome index GRCh37 or GRCh38 (selected via TSNAD.py -V)

## Outputs

- fusions.tsv containing Arriba-reported fusion calls with gene-pair identifiers and supporting read information
- Fusion-derived mutant peptide sequences (8–11 aa) passed to DeepHLApan for pMHC binding/immunogenicity prediction

## How to apply

Run this module only when RNA-Seq data are provided (the TSNAD docker run accepts an optional RNA-seq directory mounted at /home/tsnad/RNA-seq). First align the tumor RNA-Seq FASTQ reads to the reference genome (GRCh37 or GRCh38, matching the -V argument) with STAR v2.7 to produce a sorted BAM. Then run Arriba v1.1.0 on the STAR-aligned BAM to detect fusion transcripts, writing candidates to fusions.tsv with gene-pair identifiers and supporting read information. Arriba was chosen over STAR-fusion because the authors state it is 'both quicker and more accurate', so its output is treated as the pipeline's canonical fusion result. The detected fusion-derived peptides then flow into the same DeepHLApan v1.1 prediction and filtering steps (binding score >0.5, immunogenic score >0.5, TPM >= 1) used for SNV/INDEL-derived neoantigens.

## Related tools

- **STAR 2.7** (Aligns tumor RNA-Seq reads to the reference genome to produce the BAM that Arriba consumes for fusion detection)
- **Arriba 1.1.0** (Performs gene fusion detection on the STAR-aligned BAM; chosen over STAR-fusion as quicker and more accurate)
- **TSNAD v2.0** (Integrates and orchestrates the fusion module within the one-stop neoantigen prediction pipeline) — https://github.com/jiujiezz/tsnad

## Examples

```
docker run -it -v [dir of WES/WGS]/:/home/tsnad/samples -v [dir of RNA-seq]:/home/tsnad/RNA-seq -v [output dir]:/home/tsnad/results biopharm/tsnad:latest /bin/bash; cd /home/tsnad; python TSNAD.py -I samples/ -R RNA-seq/ -V grch38 -O results/
```

## Evaluation signals

- fusions.tsv exists in the results directory and contains Arriba-reported fusion calls (gene-pair identifiers plus supporting read information)
- Fusion calls are produced only when the RNA-Seq directory is mounted; running TSNAD.py without -R RNA-seq/ yields no fusion output
- The alignment BAM was generated with STAR v2.7 against the same reference version (GRCh37/GRCh38) specified via -V
- Fusion-derived peptides (lengths 8–11) enter DeepHLApan and are filtered with binding score >0.5, immunogenic score >0.5, and TPM >= 1
- End-to-end reproduction on the five TESLA samples yields 80 high-confidence neoantigens per sample (400 total), matching the paper's reported output

## Limitations

- Fusion detection is only performed when RNA-Seq data are provided; WES/WGS-only runs produce no fusion neoantigens
- Fusion specificity depends on tumor RNA-Seq depth/quality; the paper's sequencing-depth guidance (minimum 15x WGS / 50x WES, recommended 30x/100x) applies to the tumor-normal sequencing
- Fusion-derived neoantigens are ultimately limited by DeepHLApan's scoring, which does not model TCR-pMHC interactions — the paper notes a TCR-pMHC binding model is left for future work

## Evidence
