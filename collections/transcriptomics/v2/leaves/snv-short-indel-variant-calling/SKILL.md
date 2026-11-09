---
name: snv-short-indel-variant-calling
description: Use when tumor WES/WGS and matched RNA-seq alignments must be converted
  into somatic SNV and short-indel VCFs as candidate neoantigen sources, with per-read-group
  parallel calling and Mutect2-learned filtering.
when_to_use_negative:
- The variant source of interest is a long indel — ScanNeo2 routes those to the transIndel
  module (build remapped BAM with redefined CIGARs, then detect_long_indel_ti_call),
  not Mutect2.
- The goal is alternative-splicing, exitron-splicing, or gene-fusion neoantigens —
  those require the SplAdder, ScanExitron, or Arriba modules respectively, not the
  SNV/short-indel module.
- Input is already a called VCF of somatic variants — this skill starts from raw aligned
  BAMs and would redo calling unnecessarily.
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: GATK Mutect2
  role: Somatic SNV and short-indel caller run per split BAM across read-groups, with
    learned filters applied before output splitting
- name: transIndel
  role: 'Boundary tool: handles long-indel detection that Mutect2''s short-variant
    scope does not cover'
- name: ScanNeo2
  role: Snakemake workflow that orchestrates and parameterizes the Mutect2-based SNV/short-indel
    module
  repo: https://github.com/ylab-hi/ScanNeo2/
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad659
    title: 'ScanNeo2: a comprehensive workflow for neoantigen detection and immunogenicity
      prediction from diverse genomic and transcriptomic alterations'
  source_content_hash: sha256:43f01ffe794616e27a3b21bed0ad4b2bb9071dfbc5b37464da7803cfa9ef7ef3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ylab-hi/ScanNeo2
  tools:
  - Arriba
  - GATK Mutect2
  - ScanExitron
  - ScanNeo2
  - SplAdder
  - transIndel
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# snv-short-indel-variant-calling

## Summary

The SNV/short-indel calling module of the ScanNeo2 Snakemake workflow, which uses GATK Mutect2 on BWA-aligned DNA- and RNA-seq BAMs to produce somatic SNV and short-indel VCFs that feed downstream neoantigen extraction. It complements ScanNeo2's other variant sources (alternative splicing, exitron-splicing, gene fusions, long indels).

## When to use

Use when tumor WES/WGS and matched RNA-seq alignments must be converted into somatic SNV and short-indel VCFs as candidate neoantigen sources, with per-read-group parallel calling and Mutect2-learned filtering.

## When NOT to use

- The variant source of interest is a long indel — ScanNeo2 routes those to the transIndel module (build remapped BAM with redefined CIGARs, then detect_long_indel_ti_call), not Mutect2.
- The goal is alternative-splicing, exitron-splicing, or gene-fusion neoantigens — those require the SplAdder, ScanExitron, or Arriba modules respectively, not the SNV/short-indel module.
- Input is already a called VCF of somatic variants — this skill starts from raw aligned BAMs and would redo calling unnecessarily.

## Inputs

- BWA-aligned DNA-seq BAM files (tumor/normal, split per read-group)
- BWA-aligned RNA-seq BAM files
- config/config.yml and sample sheet with `activate`, `type: short|long|all`, and `mode: DNA|RNA|BOTH` keys
- ENSEMBL reference genome at the configured `release` version

## Outputs

- somatic.snvs.vcf.gz (filtered somatic SNV calls with GRP and SRC INFO keys)
- somatic.short.indels.vcf.gz (filtered somatic short-indel calls with GRP and SRC INFO keys)
- Merged per-variant-type VCFs under results/<sample>/variants/

## How to apply

Gate the module on its `activate` flag in config/config.yml plus the indel-level keys (`type: short|long|all`, `mode: DNA|RNA|BOTH`), and consume the aligned DNA-seq BWA BAMs and RNA-seq BAMs produced by the alignment stage. Split each BAM per read-group and run GATK Mutect2 on the split BAMs in parallel, then apply Mutect2's learned filters to the raw calls. Split the filtered output into two variant-type-specific files, somatic.snvs.vcf.gz and somatic.short.indels.vcf.gz, and augment records with GRP (variant group) and SRC (caller source) INFO keys before merging under results/<sample>/variants/. Rationale: most existing neoantigen-prediction tools focus only on SNV- or indel-derived neoantigens, so this module supplies the canonical small-variant source while sibling modules (SplAdder, ScanExitron, transIndel, Arriba) cover splicing and fusion events.

## Related tools

- **GATK Mutect2** (Somatic SNV and short-indel caller run per split BAM across read-groups, with learned filters applied before output splitting)
- **transIndel** (Boundary tool: handles long-indel detection that Mutect2's short-variant scope does not cover)
- **ScanNeo2** (Snakemake workflow that orchestrates and parameterizes the Mutect2-based SNV/short-indel module) — https://github.com/ylab-hi/ScanNeo2/

## Examples

```
git clone --recurse-submodules https://github.com/ylab-hi/ScanNeo2
```

## Evaluation signals

- A merged per-variant-type VCF exists under results/<sample>/variants/ for every activated module.
- Output is split into exactly somatic.snvs.vcf.gz and somatic.short.indels.vcf.gz, matching the variant-type gating.
- Every VCF record carries the GRP (group) and SRC (caller source) INFO keys.
- The FILTER column is populated from Mutect2's learned filters rather than containing raw unfiltered calls.
- Calling honors the indel-level keys: short calls appear only when `type` is `short|all`, and the DNA/RNA/BOTH mode matches the BAMs supplied.

## Limitations

- Mutect2 covers only short variants; long indels require the separate transIndel build/detect steps (DNA build, RNA build, or both per indel.mode).
- For frameshift variants, only transcripts escaping nonsense-mediated decay are further considered downstream.
- ScanNeo2 requires on average ~22.5% more CPU time than nextNEOpi, a cost of its multi-source design.
- No specific Snakemake version was pinned in the publication text (the README badge indicates snakemake ≥ 9.0.0).

## Evidence

- [abstract] Unlike its predecessor, ScanNeo2 integrates multiple sources of somatic variants, including canonical- and exitron-splicing, gene fusion events, and various somatic variants.: "Unlike its predecessor, ScanNeo2 integrates multiple sources of somatic variants, including canonical- and exitron-splicing, gene fusion events, and various somatic variants."
- [results] gatk ... is utilized for detection of short indels, as well as SNVs ... combined with transIndel (Yang et al. 2018) to detect long variants: [evidence span withheld — anchor status: not_found]
- [results] we applied ScanNeo2 with more stringent parameters (immunogenicity score ≥0.5, TPM ≥2, ranking score ≥1000, and VAF ≥0.02): "we applied ScanNeo2 with more stringent parameters (immunogenicity score ≥0.5, TPM ≥2, ranking score ≥1000, and VAF ≥0.02)"
- [methods] git clone --recurse-submodules https://github.com/ylab-hi/ScanNeo2: "git clone --recurse-submodules https://github.com/ylab-hi/ScanNeo2"
