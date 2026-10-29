---
name: somatic-snv-indel-calling
description: Use when you have a tumor-normal pair of cleaned, aligned, recalibrated
  BAM files and need to detect somatic SNVs as well as INDELs for downstream neoantigen
  or mutation analysis.
when_to_use_negative:
- Input is raw FASTQ without a matched normal sample — tumor-only calling is outside
  the tumor-normal design described here
- Sequencing depth is below the stated minimums (15x WGS / 50x WES), where calls would
  not be reliable
- You only need germline variant calling — Mutect2 in this pipeline is used for somatic
  SNVs/INDELs from tumor-normal pairs
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: GATK
  role: Provides the Mutect2 module used to call somatic SNVs/INDELs, and is also
    used to remove repeat sequences and recalibrate base quality scores producing
    the final BAM
  repo: https://github.com/broadinstitute/gatk
- name: Trimmomatic
  role: Trims and crops raw WES/WGS reads to remove artifacts harmful to downstream
    variant calling
- name: BWA
  role: Maps cleaned short reads to the reference genome (GRCh37 or GRCh38)
- name: SAMtools
  role: Converts BWA alignment output from SAM to BAM to save storage space
- name: VEP
  role: Annotates the called SNVs/INDELs at the amino-acid level downstream of variant
    calling
  repo: https://github.com/Ensembl/ensembl-vep
- name: TSNAD v2.0
  role: Integrated pipeline that embeds the Trimmomatic-BWA-SAMtools-GATK/Mutect2
    workflow for somatic variant calling as part of neoantigen prediction
  repo: https://github.com/jiujiezz/tsnad
provenance:
  source_task_ids:
  - task_004
  - task_003
  source_papers:
  - doi: 10.1016/j.csbj.2021.08.016
    title: 'TSNAD v2.0: A one-stop software solution for tumor-specific neoantigen
      detection'
  source_content_hash: sha256:af2c74959bb01de56c98f5721ec1d12f87b257273f191bb77f15c52d26380c02
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/jiujiezz/tsnad
  repo_url: https://github.com/broadinstitute/gatk
  tools:
  - Arriba
  - BWA
  - DeepHLApan
  - Docker
  - GATK
  - HISAT2
  - Mutect2
  - OptiType
  - SAMtools
  - STAR
  - StringTie
  - TSNAD v2.0
  - Trimmomatic
  - VEP
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.45
n_claims: 20
---

# somatic-snv-indel-calling

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Calls somatic SNVs and INDELs from tumor-normal WES/WGS pairs using the Mutect2 module of GATK v4.2.0.0, as implemented in the TSNAD v2.0 pipeline, writing results to a VCF (somatic_snvs_indels.vcf).

## When to use

Use when you have a tumor-normal pair of cleaned, aligned, recalibrated BAM files and need to detect somatic SNVs as well as INDELs for downstream neoantigen or mutation analysis.

## When NOT to use

- Input is raw FASTQ without a matched normal sample — tumor-only calling is outside the tumor-normal design described here
- Sequencing depth is below the stated minimums (15x WGS / 50x WES), where calls would not be reliable
- You only need germline variant calling — Mutect2 in this pipeline is used for somatic SNVs/INDELs from tumor-normal pairs

## Inputs

- Tumor WES/WGS BAM file (after Trimmomatic trimming, BWA mapping, SAMtools SAM-to-BAM conversion, GATK repeat removal and base quality score recalibration)
- Matched normal WES/WGS BAM file (same preprocessing)
- Reference genome FASTA (GRCh37 or GRCh38)

## Outputs

- somatic_snvs_indels.vcf (valid VCF containing somatic SNV/INDEL records called by Mutect2)

## How to apply

First preprocess the raw WES/WGS reads with Trimmomatic v0.39 to trim and crop adapter artifacts, map with BWA v0.7.17 to GRCh37 or GRCh38, convert SAM to BAM with SAMtools v1.13, then use GATK v4.2.0.0 to remove repeat sequences and recalibrate base quality scores to create the final BAM. Run GATK's Mutect2 on the tumor-normal BAM pair so that both SNVs and INDELs are detected, not just SNVs; TSNAD v2.0 uses Mutect2 specifically to extend calling beyond SNVs to INDELs. Ensure adequate sequencing depth: minimum 15x for WGS and 50x for WES (recommended 30x WGS and 100x WES). Output the calls to somatic_snvs_indels.vcf, which is validated for existence, valid VCF format, and presence of SNV/INDEL records called by Mutect2 from the final BAM. The resulting VCF is subsequently annotated at the amino-acid level with VEP v104 to extract mutant peptides of length 8-11.

## Related tools

- **GATK** (Provides the Mutect2 module used to call somatic SNVs/INDELs, and is also used to remove repeat sequences and recalibrate base quality scores producing the final BAM) — https://github.com/broadinstitute/gatk
- **Trimmomatic** (Trims and crops raw WES/WGS reads to remove artifacts harmful to downstream variant calling)
- **BWA** (Maps cleaned short reads to the reference genome (GRCh37 or GRCh38))
- **SAMtools** (Converts BWA alignment output from SAM to BAM to save storage space)
- **VEP** (Annotates the called SNVs/INDELs at the amino-acid level downstream of variant calling) — https://github.com/Ensembl/ensembl-vep
- **TSNAD v2.0** (Integrated pipeline that embeds the Trimmomatic-BWA-SAMtools-GATK/Mutect2 workflow for somatic variant calling as part of neoantigen prediction) — https://github.com/jiujiezz/tsnad

## Examples

```
gatk Mutect2 -I tumor.final.bam -I normal.final.bam -normal NORMAL_SAMPLE -R GRCh37.fa -O somatic_snvs_indels.vcf
```

## Evaluation signals

- somatic_snvs_indels.vcf exists in the output directory
- somatic_snvs_indels.vcf is a valid VCF file
- The VCF contains SNV/INDEL records attributable to Mutect2 calls from the final recalibrated BAM
- Input BAMs passed GATK base quality score recalibration and repeat-removal before calling
- Sequencing depth meets the minimum of 15x for WGS or 50x for WES (recommended 30x/100x)

## Limitations

- Both tumor and matched normal BAMs are required; the module is designed for tumor-normal pairs
- Variant calling quality depends on sequencing depth — below 15x (WGS) or 50x (WES) calls are not supported; recommended depths are 30x WGS and 100x WES
- Reference genome version choice (GRCh37 vs GRCh38) must be passed explicitly in TSNAD (-V grch37/grch38)

## Evidence

- [other] Call somatic SNVs/INDELs from the tumor-normal BAM pair using the Mutect2 module of GATK and write the results to somatic_snvs_indels.vcf: [evidence span withheld — anchor status: not_found]
