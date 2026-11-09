---
name: neoantigen-pipeline-hla-input-preparation
description: Use when preparing patient-level HLA class I typing from tumor-normal
  FASTQ inputs as a required input for MHC binding/immunogenicity neoantigen prediction
  (e.g., DeepHLApan in TSNAD).
when_to_use_negative:
- Clinical-grade HLA typing is already available (e.g., the TESLA dataset on Synapse
  syn21048999 provides clinical-grade HLA typing directly)
- The prediction target is HLA class II alleles — OptiType as configured in TSNAD
  covers HLA class I (A/B/C) only
- Input is raw untrimmed reads — the module expects fastq files already processed
  by the Trimmomatic trimming step
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_2830
- http://edamontology.org/topic_0199
tools:
- name: OptiType
  role: Performs HLA class I allele identification (4-digit HLA-A/B/C calling) from
    the trimmed reads; ships as OptiType 1.3.5 in the TSNAD Tools/ folder
- name: TSNAD v2.0
  role: One-stop pipeline orchestrating the HLA typing module and feeding hlatype_result.tsv
    into the DeepHLApan neoantigen prediction step
  repo: https://github.com/jiujiezz/tsnad
- name: Trimmomatic
  role: Upstream trimming/cropping of raw WES/WGS reads that produce the FASTQ input
    consumed by OptiType
- name: DeepHLApan
  role: Downstream consumer of the HLA genotype for peptide-MHC binding and immunogenicity
    prediction; replaced NetMHCpan in TSNAD v2.0
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.csbj.2021.08.016
    title: 'TSNAD v2.0: A one-stop software solution for tumor-specific neoantigen
      detection'
  source_content_hash: sha256:cfb609e4ef1f3b82d9bd621687aa496d674a830d05242423d91eaed4236cc0cc
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
  - DeepHLApan
  - OptiType
  - TSNAD v2.0
  - Trimmomatic
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_2830
  - http://edamontology.org/topic_0199
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# neoantigen-pipeline-hla-input-preparation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reconstruct the HLA class I allele typing step of the TSNAD v2.0.1 neoantigen pipeline, which uses OptiType v1.3.5 to derive 4-digit HLA-A/B/C genotypes from trimmed WES/WGS reads and outputs them in a format consumable by the DeepHLApan-based neoantigen prediction module.

## When to use

Use when preparing patient-level HLA class I typing from tumor-normal FASTQ inputs as a required input for MHC binding/immunogenicity neoantigen prediction (e.g., DeepHLApan in TSNAD).

## When NOT to use

- Clinical-grade HLA typing is already available (e.g., the TESLA dataset on Synapse syn21048999 provides clinical-grade HLA typing directly)
- The prediction target is HLA class II alleles — OptiType as configured in TSNAD covers HLA class I (A/B/C) only
- Input is raw untrimmed reads — the module expects fastq files already processed by the Trimmomatic trimming step

## Inputs

- Trimmed/cleaned WES or WGS FASTQ files for tumor and normal samples (from Trimmomatic 0.39 step)
- Reference genome build selection (GRCh37 or GRCh38) for the TSNAD run

## Outputs

- hlatype_result.tsv: one row per sample with two 4-digit alleles per HLA class I locus (HLA-A, HLA-B, HLA-C), formatted for the DeepHLApan module
- Final neoantigen candidates in results/deephlapan_results/ when run as part of the full TSNAD pipeline

## How to apply

Take the cleaned fastq files for the tumor-normal pair produced by the Trimmomatic (v0.39) trimming step and run OptiType v1.3.5 on each sample's reads to identify 4-digit HLA class I (HLA-A, HLA-B, HLA-C) alleles by read-count-based allele selection. Consolidate the per-sample genotype calls into hlatype_result.tsv, one row per sample with two alleles per locus, formatted for consumption by the DeepHLApan (v1.1) neoantigen prediction module embedded in TSNAD (run via 'python TSNAD.py' with GRCh37 or GRCh38 reference selection). Validate that OptiType returns a complete, single heterozygous pair of HLA-A/B/C alleles for both tumor and normal samples; the paper reports no separate numeric typing threshold for this module. Note the article replaces NetMHCpan with DeepHLApan downstream, so the HLA genotype must be compatible with DeepHLApan's supported HLA-I allele set.

## Related tools

- **OptiType** (Performs HLA class I allele identification (4-digit HLA-A/B/C calling) from the trimmed reads; ships as OptiType 1.3.5 in the TSNAD Tools/ folder)
- **TSNAD v2.0** (One-stop pipeline orchestrating the HLA typing module and feeding hlatype_result.tsv into the DeepHLApan neoantigen prediction step) — https://github.com/jiujiezz/tsnad
- **Trimmomatic** (Upstream trimming/cropping of raw WES/WGS reads that produce the FASTQ input consumed by OptiType)
- **DeepHLApan** (Downstream consumer of the HLA genotype for peptide-MHC binding and immunogenicity prediction; replaced NetMHCpan in TSNAD v2.0)

## Examples

```
cd /home/tsnad; python TSNAD.py -I samples/ -V grch38 -O results/
```

## Evaluation signals

- OptiType returns a complete, single heterozygous pair of HLA-A/B/C alleles for both the tumor and the normal sample (consistent tumor-normal typing)
- hlatype_result.tsv contains one row per sample with two alleles per locus in 4-digit resolution
- The HLA genotype files are present and formatted such that the DeepHLApan module of TSNAD runs without input errors
- Downstream peptide-MHC calls use the typed alleles, with binding score >0.5 and immunogenic score >0.5 defining potential neoantigens as a sanity check on the prediction step consuming the HLA input

## Limitations

- The paper reports no separate numeric typing threshold for the HLA typing module; validation relies on completeness/consistency of the called allele pairs
- OptiType in TSNAD covers HLA class I (A/B/C) only; class II typing is out of scope
- Downstream prediction is restricted to alleles supported by DeepHLApan (which replaced NetMHCpan); alleles outside its supported set cannot be predicted
- TSNAD v2.0 requires sufficient sequencing depth (minimum 15x for WGS and 50x for WES; recommended 30x WGS / 100x WES), which affects read-count-based allele selection reliability

## Evidence
