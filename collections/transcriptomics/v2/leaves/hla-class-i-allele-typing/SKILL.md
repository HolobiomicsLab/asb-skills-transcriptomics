---
name: hla-class-i-allele-typing
description: Use when tumor-normal WES/WGS or RNA-Seq fastq files require patient-specific
  HLA class I genotyping as a prerequisite for neoantigen/pMHC binding prediction.
when_to_use_negative:
- Clinical-grade HLA typing is already available (as with the TESLA Synapse syn21048999
  dataset), so in-pipeline OptiType typing is redundant
- The goal is HLA class II (e.g. HLA-DR/DQ/DP) typing — OptiType in TSNAD covers only
  HLA class I alleles
- The task is peptide-MHC binding/immunogenicity scoring rather than genotype determination
  — that is handled by the DeepHLApan module, not this typing step
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_2830
- http://edamontology.org/topic_3120
tools:
- name: OptiType
  role: Performs HLA class I allele identification (v1.3.5) from the cleaned fastq
    reads via read-count-based 4-digit allele selection
- name: TSNAD v2.0
  role: One-stop pipeline that embeds and orchestrates the OptiType HLA typing module
    between Trimmomatic trimming and DeepHLApan neoantigen prediction
  repo: https://github.com/jiujiezz/tsnad
- name: Trimmomatic
  role: Upstream trimming tool (v0.39) that produces the cleaned fastq files consumed
    by the OptiType typing step
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.csbj.2021.08.016
    title: 'TSNAD v2.0: A one-stop software solution for tumor-specific neoantigen
      detection'
  source_content_hash: sha256:5894e387f344927f67adafe31f31dddc29a4261a5afcb30dbae0b40964154e8d
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
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_2830
  - http://edamontology.org/topic_3120
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# hla-class-i-allele-typing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Identify 4-digit HLA class I (HLA-A, HLA-B, HLA-C) alleles from tumor-normal sequencing reads using OptiType v1.3.5, as embedded in the TSNAD v2.0.1 neoantigen prediction pipeline, so that downstream DeepHLApan peptide-MHC predictions use patient-specific HLA genotypes.

## When to use

Use when tumor-normal WES/WGS or RNA-Seq fastq files require patient-specific HLA class I genotyping as a prerequisite for neoantigen/pMHC binding prediction.

## When NOT to use

- Clinical-grade HLA typing is already available (as with the TESLA Synapse syn21048999 dataset), so in-pipeline OptiType typing is redundant
- The goal is HLA class II (e.g. HLA-DR/DQ/DP) typing — OptiType in TSNAD covers only HLA class I alleles
- The task is peptide-MHC binding/immunogenicity scoring rather than genotype determination — that is handled by the DeepHLApan module, not this typing step

## Inputs

- Cleaned (Trimmomatic-trimmed) fastq files for tumor sample
- Cleaned (Trimmomatic-trimmed) fastq files for matched normal sample
- OptiType 1.3.5 bundled in Tools/ of TSNAD (jiujiezz/tsnad) or biopharm/tsnad Docker image

## Outputs

- hlatype_result.tsv with one row per sample and two 4-digit HLA-A/B/C alleles per locus
- Per-sample HLA class I genotype calls (4-digit resolution) formatted for the DeepHLApan module

## How to apply

Take the cleaned fastq files for the tumor-normal pair produced by the Trimmomatic 0.39 trimming step of TSNAD module 1 and run OptiType v1.3.5 (shipped in the Tools/ folder of the TSNAD GitHub install or bundled in the biopharm/tsnad Docker image) on each sample's reads. OptiType selects 4-digit HLA class I alleles by read-count-based allele selection, returning two alleles per locus for HLA-A, HLA-B, and HLA-C. Consolidate the per-sample genotype calls into hlatype_result.tsv, one row per sample with two alleles per locus, formatted for consumption by the DeepHLApan-based neoantigen prediction module. Validate that OptiType returns a complete, single heterozygous pair of HLA-A/B/C alleles for both tumor and normal samples, confirming consistent tumor-normal typing; the paper reports no separate numeric typing threshold for this module.

## Related tools

- **OptiType** (Performs HLA class I allele identification (v1.3.5) from the cleaned fastq reads via read-count-based 4-digit allele selection)
- **TSNAD v2.0** (One-stop pipeline that embeds and orchestrates the OptiType HLA typing module between Trimmomatic trimming and DeepHLApan neoantigen prediction) — https://github.com/jiujiezz/tsnad
- **Trimmomatic** (Upstream trimming tool (v0.39) that produces the cleaned fastq files consumed by the OptiType typing step)

## Examples

```
docker run -it -v [dir of WES/WGS]/:/home/tsnad/samples -v [output dir]:/home/tsnad/results biopharm/tsnad:latest /bin/bash; cd /home/tsnad; bash uncompress.sh; python TSNAD.py -I samples/ -V grch37 -O results/
```

## Evaluation signals

- OptiType returns a complete, single heterozygous pair of HLA-A/B/C alleles for both the tumor and normal samples
- Tumor and normal HLA class I typings are consistent (matching genotype across the tumor-normal pair)
- hlatype_result.tsv has one row per sample with two alleles per locus in the format expected by the DeepHLApan module
- Calls are 4-digit resolution (e.g. HLA-A*02:01-style alleles) derived by OptiType's read-count-based allele selection
- Paper reports no separate numeric typing threshold for this module, so absence of additional filtering criteria in the output is expected

## Limitations

- The paper does not report a separate numeric typing threshold or accuracy benchmark specific to the HLA typing module; validation is structural (complete heterozygous pairs, tumor-normal consistency)
- OptiType covers HLA class I only; no HLA class II typing is included in the TSNAD v2.0.1 pipeline
- Accuracy depends on the depth of the input WES/WGS data (minimum 15x WGS / 50x WES, recommended 30x WGS / 100x WES per the results section)

## Evidence
