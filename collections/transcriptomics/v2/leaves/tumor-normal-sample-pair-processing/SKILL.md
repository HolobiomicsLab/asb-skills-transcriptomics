---
name: tumor-normal-sample-pair-processing
description: Use when processing paired tumor-normal WES/WGS FASTQ through TSNAD v2.0
  and HLA class I allele typing (4-digit HLA-A/B/C via OptiType 1.3.5) must be run
  per sample before neoantigen prediction.
when_to_use_negative:
- Only clinical-grade HLA typing results are already available (as in the TESLA Synapse
  syn21048999 dataset) — no in-pipeline OptiType run is needed
- Neoantigen prediction is required for HLA class II alleles — OptiType in TSNAD covers
  class I (HLA-A/B/C) only
- Input is not paired tumor-normal WES/WGS sequencing data from the same patient
edam_operation: http://edamontology.org/operation_3196
tools:
- name: OptiType
  role: Performs HLA class I allele identification (4-digit HLA-A/B/C) from cleaned
    tumor/normal FASTQ reads by read-count-based allele selection
- name: Trimmomatic
  role: Produces the cleaned FASTQ files per tumor and normal sample that are the
    direct input to the HLA typing module
- name: TSNAD v2.0
  role: One-stop pipeline wrapper that chains Trimmomatic, OptiType 1.3.5 and DeepHLApan
    1.1 for paired tumor-normal samples
  repo: https://github.com/jiujiezz/tsnad
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.csbj.2021.08.016
    title: 'TSNAD v2.0: A one-stop software solution for tumor-specific neoantigen
      detection'
  source_content_hash: sha256:2499ddf3d4639ede4dcb6037ef34ca7523e88cf7721b781233cff171620d49dd
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
  - OptiType
  - TSNAD v2.0
  - Trimmomatic
  edam_operation: http://edamontology.org/operation_3196
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# tumor-normal-sample-pair-processing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reconstructs the TSNAD v2.0.1 tumor-normal sample-pair handling, including the OptiType v1.3.5 HLA class I typing module that converts cleaned tumor/normal FASTQ files into 4-digit HLA-A/B/C genotypes (hlatype_result.tsv) feeding DeepHLApan-based neoantigen prediction.

## When to use

Use when processing paired tumor-normal WES/WGS FASTQ through TSNAD v2.0 and HLA class I allele typing (4-digit HLA-A/B/C via OptiType 1.3.5) must be run per sample before neoantigen prediction.

## When NOT to use

- Only clinical-grade HLA typing results are already available (as in the TESLA Synapse syn21048999 dataset) — no in-pipeline OptiType run is needed
- Neoantigen prediction is required for HLA class II alleles — OptiType in TSNAD covers class I (HLA-A/B/C) only
- Input is not paired tumor-normal WES/WGS sequencing data from the same patient

## Inputs

- Cleaned (Trimmomatic-trimmed) tumor FASTQ files
- Cleaned (Trimmomatic-trimmed) normal FASTQ files
- TSNAD Tools/ directory containing OptiType 1.3.5
- Reference genome selection (GRCh37 or GRCh38)

## Outputs

- hlatype_result.tsv — per-sample 4-digit HLA class I genotypes (two alleles each for HLA-A, HLA-B, HLA-C), formatted for the DeepHLApan module
- Final neoantigen results under results/deephlapan_results/ when run end-to-end

## How to apply

Load the trimmed FASTQ files for the tumor-normal pair produced by the Trimmomatic 0.39 step of TSNAD module 1, then run OptiType v1.3.5 on each sample's reads to identify 4-digit HLA class I alleles by read-count-based allele selection. Consolidate the per-sample calls for HLA-A, HLA-B and HLA-C into hlatype_result.tsv, one row per sample with two alleles per locus, formatted for the DeepHLApan prediction module (which keeps peptide-MHC pairs with binding score >0.5 and immunogenic score >0.5). Run within the TSNAD pipeline (python TSNAD.py -I samples/ -V grch37|grch38 -O results/), where OptiType 1.3.5 sits in the Tools/ folder. Validation is by completeness and consistency, not a numeric threshold: OptiType should return a complete single heterozygous pair of HLA-A/B/C alleles for both tumor and normal samples, with tumor-normal typing consistent. The upstream pipeline supplies depth requirements of at least 15x WGS / 50x WES (recommended 30x / 100x) that implicitly bound the reliability of per-sample genotyping.

## Related tools

- **OptiType** (Performs HLA class I allele identification (4-digit HLA-A/B/C) from cleaned tumor/normal FASTQ reads by read-count-based allele selection)
- **Trimmomatic** (Produces the cleaned FASTQ files per tumor and normal sample that are the direct input to the HLA typing module)
- **TSNAD v2.0** (One-stop pipeline wrapper that chains Trimmomatic, OptiType 1.3.5 and DeepHLApan 1.1 for paired tumor-normal samples) — https://github.com/jiujiezz/tsnad

## Examples

```
docker run -it -v [dir of WES/WGS]/:/home/tsnad/samples -v [output dir]:/home/tsnad/results biopharm/tsnad:latest /bin/bash; cd /home/tsnad; python TSNAD.py -I samples/ -V grch37 -O results/
```

## Evaluation signals

- OptiType returns a complete, single heterozygous pair of HLA-A/B/C alleles for BOTH the tumor and the normal sample (no missing locus, no allele dropout)
- Tumor and normal HLA typing calls are consistent with each other (same germline genotype expected across a patient's paired samples)
- hlatype_result.tsv contains one row per sample with two 4-digit alleles per locus, in the format consumed by the DeepHLApan module
- Input FASTQ files are the post-Trimmomatic cleaned reads, not raw reads, and satisfy minimum depth (15x WGS / 50x WES minimum; 30x / 100x recommended)
- Running TSNAD end-to-end places final neoantigen outputs in results/deephlapan_results/, confirming the HLA typing module fed the downstream prediction

## Limitations

- OptiType covers HLA class I (HLA-A, HLA-B, HLA-C) only; class II alleles are not typed by this module
- No separate numeric typing quality threshold is reported for the HLA module; validation relies on completeness and tumor-normal consistency
- TSNAD v1.0 supported only GRCh37; v2.0 adds GRCh38 — reference-genome choice must match the downstream variant-calling steps

## Evidence
