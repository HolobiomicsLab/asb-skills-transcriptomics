---
name: chimeric-transcript-identification
description: Use when tumor RNA-Seq FASTQ reads are available and fusion-derived neoantigen
  candidates are needed alongside WES/WGS-based SNV/INDEL neoantigen prediction in
  TSNAD.
when_to_use_negative:
- Only WES/WGS data are available and no tumor RNA-Seq reads can be supplied — the
  fusion module is only performed when RNA-Seq data are provided
- The goal is gene expression quantification from RNA-Seq rather than fusion detection
  (TSNAD handles expression separately, e.g. filtering neoantigens from genes with
  TPM < 1)
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2640
tools:
- name: STAR
  role: Aligns RNA-Seq reads (v2.7) to the GRCh37/GRCh38 reference genome to support
    fusion detection
  repo: https://github.com/jiujiezz/tsnad
- name: Arriba
  role: Detects gene fusion transcripts (v1.1.0) from the STAR-aligned BAM; chosen
    over STAR-fusion as quicker and more accurate
  repo: https://github.com/jiujiezz/tsnad
- name: TSNAD v2.0
  role: Integrated pipeline that orchestrates the RNA-Seq fusion module and downstream
    fusion-derived neoantigen prediction
  repo: https://github.com/jiujiezz/tsnad
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.csbj.2021.08.016
    title: 'TSNAD v2.0: A one-stop software solution for tumor-specific neoantigen
      detection'
  source_content_hash: sha256:48c7eba71f37a58db218bddce9cf4dbc543202277c46867495e72e8d8b18a5ad
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
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2640
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# Chimeric transcript identification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Identification of gene fusion (chimeric) transcripts from tumor RNA-Seq data within TSNAD v2.0, using the STAR aligner (v2.7) for genome mapping followed by Arriba (v1.1.0) for fusion detection, performed only when RNA-Seq data are supplied.

## When to use

Use when tumor RNA-Seq FASTQ reads are available and fusion-derived neoantigen candidates are needed alongside WES/WGS-based SNV/INDEL neoantigen prediction in TSNAD.

## When NOT to use

- Only WES/WGS data are available and no tumor RNA-Seq reads can be supplied — the fusion module is only performed when RNA-Seq data are provided
- The goal is gene expression quantification from RNA-Seq rather than fusion detection (TSNAD handles expression separately, e.g. filtering neoantigens from genes with TPM < 1)

## Inputs

- Tumor RNA-Seq FASTQ reads
- Reference genome GRCh37 or GRCh38

## Outputs

- fusions.tsv with Arriba-reported fusion calls (gene-pair identifiers and supporting read information)
- Fusion-derived neoantigen candidates for downstream prediction

## How to apply

Load the tumor RNA-Seq FASTQ reads and align them to the reference genome (GRCh37 or GRCh38) with the STAR (v2.7) aligner to produce an alignment from which chimeric transcripts can be detected. Run Arriba (v1.1.0) on the STAR-aligned BAM to detect gene fusion transcripts; Arriba was chosen over STAR-fusion because it is both quicker and more accurate, so its output serves as the pipeline's canonical fusion result. Write detected fusion candidates to fusions.tsv, retaining gene-pair identifiers and supporting read information for downstream fusion-derived neoantigen prediction. This module is conditional: it is skipped entirely if RNA-Seq data are not provided to TSNAD (e.g. via the -R argument). Validate the run by confirming fusions.tsv exists and contains Arriba-reported fusion calls; fusions feed subsequent fusion-derived neoantigen prediction within TSNAD v2.0.

## Related tools

- **STAR** (Aligns RNA-Seq reads (v2.7) to the GRCh37/GRCh38 reference genome to support fusion detection) — https://github.com/jiujiezz/tsnad
- **Arriba** (Detects gene fusion transcripts (v1.1.0) from the STAR-aligned BAM; chosen over STAR-fusion as quicker and more accurate) — https://github.com/jiujiezz/tsnad
- **TSNAD v2.0** (Integrated pipeline that orchestrates the RNA-Seq fusion module and downstream fusion-derived neoantigen prediction) — https://github.com/jiujiezz/tsnad

## Examples

```
python TSNAD.py -I samples/ -R RNA-seq/ -V grch37 -O results/
```

## Evaluation signals

- fusions.tsv exists in the results directory and contains Arriba-reported fusion calls
- Each fusion entry retains gene-pair identifiers and supporting read information
- Fusion calls are consistent with the STAR (v2.7) alignment produced on the specified reference genome (GRCh37/GRCh38)
- The module was skipped when no RNA-Seq directory was supplied (RNA-seq is not necessary to provide per the TSNAD usage)
- Downstream fusion-derived neoantigen prediction consumes the detected fusion candidates

## Limitations

- Fusion detection is conditional on RNA-Seq data being provided; without it, no fusion neoantigens are predicted
- Arriba was preferred over STAR-fusion for speed and accuracy, but no independent fusion validation benchmark is reported in the article
- TSNAD v1.0 only supports GRCh37; GRCh38 support is new in v2.0, so older fusion results may not map across genome versions

## Evidence

- [methods] The detection of fusions is based on the RNA-Seq data. STAR (v2.7) aligner [18] is used for mapping reads to the reference genome, Arriba (v1.1.0) [19] is then used for fusion detection.: "The detection of fusions is based on the RNA-Seq data. STAR (v2.7) aligner [18] is used for mapping reads to the reference genome"
- [methods] Arriba is quicker and more accurate than STAR-fusion: "Compared with other fusion detectors such as STAR-fusion [20], Arriba is both quicker and more accurate."
- [readme] RNA-seq is not necessary to provide: "You need to enter the TSNAD running enviromont with your path of WES/WGS/RNA-seq as the following command (RNA-seq is not necessary to provide):"
- [intro] add the function of RNA-Seq data analysis including gene expression and gene fusion analyses: "We present here TSNAD v2.0 that implements new features and improvements including (i) update all the embedded tools into the latest version, (ii) add the function of RNA-Seq data analysis"
