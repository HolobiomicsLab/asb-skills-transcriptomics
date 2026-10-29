---
name: wes-wgs-read-based-genotyping
description: Use when a tumor-normal WES/WGS pipeline needs patient HLA class I typing
  from sequencing reads (not clinical HLA reports) so that pMHC binding/immunogenicity
  prediction can be restricted to the patient's own HLA alleles.
when_to_use_negative:
- Clinical-grade HLA typing results are already available (e.g., the TESLA dataset
  provides clinical-grade HLA typing), making read-based inference redundant
- The input is RNA-Seq rather than WES/WGS reads from a tumor-normal pair, or HLA
  class II typing is required (OptiType as embedded here covers class I only)
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_2830
- http://edamontology.org/topic_0622
tools:
- name: OptiType
  role: Performs HLA class I allele identification from the sequencing reads in module
    3 of TSNAD
- name: Trimmomatic
  role: Produces the cleaned FASTQ inputs that this typing module consumes
- name: TSNAD
  role: One-stop pipeline that embeds and orchestrates the OptiType-based HLA typing
    step and passes results to DeepHLApan
  repo: https://github.com/jiujiezz/tsnad
- name: DeepHLApan
  role: Downstream consumer of the HLA types for peptide-MHC binding and immunogenicity
    prediction
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.csbj.2021.08.016
    title: 'TSNAD v2.0: A one-stop software solution for tumor-specific neoantigen
      detection'
  source_content_hash: sha256:66dedb5b8b75d8a0c2454d166a81bf093250b442314aeabb37bbd2ceeece5bc6
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
  - TSNAD
  - TSNAD v2.0
  - Trimmomatic
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_2830
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# WES/WGS read-based genotyping

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Identify 4-digit HLA class I (HLA-A/-B/-C) alleles from trimmed tumor-normal FASTQ reads using OptiType v1.3.5, as embedded in the TSNAD v2.0.1 neoantigen prediction pipeline, producing per-sample genotype calls that feed the DeepHLApan-based neoantigen prediction module.

## When to use

Use when a tumor-normal WES/WGS pipeline needs patient HLA class I typing from sequencing reads (not clinical HLA reports) so that pMHC binding/immunogenicity prediction can be restricted to the patient's own HLA alleles.

## When NOT to use

- Clinical-grade HLA typing results are already available (e.g., the TESLA dataset provides clinical-grade HLA typing), making read-based inference redundant
- The input is RNA-Seq rather than WES/WGS reads from a tumor-normal pair, or HLA class II typing is required (OptiType as embedded here covers class I only)

## Inputs

- Cleaned (Trimmomatic-trimmed) FASTQ files for the tumor sample
- Cleaned (Trimmomatic-trimmed) FASTQ files for the matched normal sample
- OptiType 1.3.5 installation (in Tools/ of the TSNAD repository or the biopharm/tsnad Docker image)

## Outputs

- Per-sample 4-digit HLA class I genotype calls (two alleles each for HLA-A, HLA-B, HLA-C) for tumor and normal
- Consolidated HLA typing table (hlatype_result.tsv) consumable by the DeepHLApan-based neoantigen prediction module

## How to apply

Take the cleaned FASTQ files for the tumor and normal samples produced by the Trimmomatic 0.39 trimming step of TSNAD's module 1 and run OptiType 1.3.5 (kept in the Tools/ folder of the TSNAD repository) on each sample's reads to determine 4-digit HLA-A, HLA-B, and HLA-C alleles via read-count-based allele selection. Consolidate the per-sample class I genotype calls into a single results table (hlatype_result.tsv), one row per sample with its two alleles per locus, formatted so the DeepHLApan 1.1 module can consume the HLA types when scoring 8-11mer mutant peptides. Validation in TSNAD is by consistency: confirm OptiType returns a complete, single heterozygous pair of HLA-A/B/C alleles for both tumor and normal samples, with tumor-normal typing agreeing; the paper reports no separate numeric typing threshold for this module. Rationale: neoantigen candidates are only potential antigens relative to the patient's HLA background, so accurate read-based typing upstream of DeepHLApan is required before applying the binding score >0.5 and immunogenic score >0.5 filters.

## Related tools

- **OptiType** (Performs HLA class I allele identification from the sequencing reads in module 3 of TSNAD)
- **Trimmomatic** (Produces the cleaned FASTQ inputs that this typing module consumes)
- **TSNAD** (One-stop pipeline that embeds and orchestrates the OptiType-based HLA typing step and passes results to DeepHLApan) — https://github.com/jiujiezz/tsnad
- **DeepHLApan** (Downstream consumer of the HLA types for peptide-MHC binding and immunogenicity prediction)

## Examples

```
cd /home/tsnad; bash uncompress.sh; python TSNAD.py -I samples/ -R RNA-seq/ -V grch37 -O results/
```

## Evaluation signals

- OptiType returns a complete, single heterozygous pair of HLA-A, HLA-B, and HLA-C alleles (6 alleles total) for both the tumor and normal samples
- Tumor and normal HLA class I typings are consistent with each other (no discordant allele calls between the paired samples)
- hlatype_result.tsv contains one row per sample with two alleles per locus, in the format expected by the DeepHLApan module
- Alleles are reported at 4-digit resolution suitable for DeepHLApan pMHC prediction
- The pipeline completes end-to-end via `python TSNAD.py` and the final neoantigen results appear under results/deephlapan_results/, implying the typing step did not abort the run

## Limitations

- The paper reports no separate numeric typing threshold or accuracy metric for this module; validation is by tumor-normal consistency only
- Only HLA class I (HLA-A/-B/-C) alleles are typed; class II typing and TCR-pMHC interaction modeling are absent from TSNAD v2.0.1
- Typing quality depends on sufficient sequencing depth — the paper notes minimum depth should be 15x for WGS and 50x for WES (recommended 30x/100x)

## Evidence

- [other] Load the cleaned fastq files for the tumor-normal pair produced by the Trimmomatic trimming step: [evidence span withheld — anchor status: not_found]
