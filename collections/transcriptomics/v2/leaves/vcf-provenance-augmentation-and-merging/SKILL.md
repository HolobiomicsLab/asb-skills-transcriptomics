---
name: vcf-provenance-augmentation-and-merging
description: Use when consolidating variant calls from multiple callers/modules in
  a pipeline (e.g. ScanNeo2's SplAdder, ScanExitron, transIndel, Mutect2, Arriba outputs)
  into per-variant-type VCFs that must carry caller-provenance INFO keys (GRP, SRC)
  for source-attributed downstream analysis.
when_to_use_negative:
- The input is a single caller's already-annotated final VCF with no multi-module
  merging or source attribution needed
- The workflow config has all module activate flags off or only a single caller is
  in use, so no cross-module provenance (GRP/SRC) needs to be maintained
- Downstream consumers ignore variant origin and only need a flat, caller-agnostic
  variant list
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3577
- http://edamontology.org/topic_0622
tools:
- name: ScanNeo2
  role: Snakemake workflow whose variation-calling stage performs the per-module VCF
    conversion, GRP/SRC augmentation, and merging into per-variant-type VCFs
  repo: https://github.com/ylab-hi/ScanNeo2/
- name: SplAdder
  role: Detects alternative splicing events; its splice-graph output is converted
    to per-group VCFs augmented with GRP/SRC INFO keys and merged into altsplicing.vcf.gz
- name: ScanExitron
  role: Calls exitron-splicing events; its <group>.exitron output is converted to
    VCF, augmented with GRP and SRC=exitron keys, sorted, and merged into exitrons.vcf.gz
  repo: https://github.com/ylab-hi/ScanExitron
- name: transIndel
  role: Calls long indels from DNA/RNA BAM builds; output is augmented with GRP/SRC=long_indel
    keys and merged into long.indels.vcf.gz
- name: GATK Mutect2
  role: Calls short indels and SNVs per split BAM; its filtered output is split into
    somatic.snvs.vcf.gz and somatic.short.indels.vcf.gz
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad659
    title: 'ScanNeo2: a comprehensive workflow for neoantigen detection and immunogenicity
      prediction from diverse genomic and transcriptomic alterations'
  source_content_hash: sha256:2c4c741a0bc6b6723c40566f97c014fa76473716d02e5813b05f7173cd683baf
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
  - http://edamontology.org/topic_3577
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# vcf-provenance-augmentation-and-merging

## Summary

A ScanNeo2 pattern in which variant calls from independent detection modules (alternative splicing, exitron-splicing, long/short indels, SNVs, gene fusions) are converted to VCF, augmented with GRP (group) and SRC (caller source) INFO keys, and merged into per-variant-type VCFs so downstream neoantigen prediction can trace every peptide back to its originating caller.

## When to use

Use when consolidating variant calls from multiple callers/modules in a pipeline (e.g. ScanNeo2's SplAdder, ScanExitron, transIndel, Mutect2, Arriba outputs) into per-variant-type VCFs that must carry caller-provenance INFO keys (GRP, SRC) for source-attributed downstream analysis.

## When NOT to use

- The input is a single caller's already-annotated final VCF with no multi-module merging or source attribution needed
- The workflow config has all module activate flags off or only a single caller is in use, so no cross-module provenance (GRP/SRC) needs to be maintained
- Downstream consumers ignore variant origin and only need a flat, caller-agnostic variant list

## Inputs

- Aligned DNA-seq BWA BAMs and RNA-seq BAMs from the alignment stage
- SplAdder splice-graph output (per group) for alternative splicing
- ScanExitron <group>.exitron output files
- transIndel detect_long_indel_ti_call output for long indels
- GATK Mutect2 per-read-group output (somatic SNVs and short indels)
- config/config.yml and sample sheet with activate flags and indel type/mode keys

## Outputs

- results/<sample>/variants/altsplicing.vcf.gz
- results/<sample>/variants/exitrons.vcf.gz
- results/<sample>/variants/long.indels.vcf.gz
- results/<sample>/variants/somatic.snvs.vcf.gz
- results/<sample>/variants/somatic.short.indels.vcf.gz
- Each output VCF carries GRP (group) and SRC (caller source) INFO keys on every record

## How to apply

Gate each calling module on its `activate` flag in config/config.yml, plus the indel-level keys (`type: short|long|all`, `mode: DNA|RNA|BOTH`), and use the aligned DNA-seq BWA BAMs and RNA-seq BAMs from the alignment stage as caller inputs. For each module, convert the tool-specific output to VCF — SplAdder splice-graph output to per-group VCFs, ScanExitron's <group>.exitron output to VCF, transIndel's detect_long_indel_ti_call output for long indels — and augment every record with the GRP (group) INFO key and an SRC key naming the caller source (e.g. SRC=exitron, SRC=long_indel). Sort the augmented records and merge per-group VCFs into one file per variant type under results/<sample>/variants/: altsplicing.vcf.gz, exitrons.vcf.gz, long.indels.vcf.gz, and from Mutect2's learned filters split output into somatic.snvs.vcf.gz and somatic.short.indels.vcf.gz. The rationale is that ScanNeo2 tracks candidates by source — 5671–34 981 candidate neoantigens per patient can originate from a single source (alternative splicing, exitron-splicing, gene fusion, indels, or SNVs) — so provenance keys must survive conversion and merging for later filtering (e.g. VAF ≥0.02, TPM ≥2) and immunogenicity prediction.

## Related tools

- **ScanNeo2** (Snakemake workflow whose variation-calling stage performs the per-module VCF conversion, GRP/SRC augmentation, and merging into per-variant-type VCFs) — https://github.com/ylab-hi/ScanNeo2/
- **SplAdder** (Detects alternative splicing events; its splice-graph output is converted to per-group VCFs augmented with GRP/SRC INFO keys and merged into altsplicing.vcf.gz)
- **ScanExitron** (Calls exitron-splicing events; its <group>.exitron output is converted to VCF, augmented with GRP and SRC=exitron keys, sorted, and merged into exitrons.vcf.gz) — https://github.com/ylab-hi/ScanExitron
- **transIndel** (Calls long indels from DNA/RNA BAM builds; output is augmented with GRP/SRC=long_indel keys and merged into long.indels.vcf.gz)
- **GATK Mutect2** (Calls short indels and SNVs per split BAM; its filtered output is split into somatic.snvs.vcf.gz and somatic.short.indels.vcf.gz)

## Evaluation signals

- Every activated module's merged per-variant-type VCF exists under results/<sample>/variants/ (altsplicing.vcf.gz, exitrons.vcf.gz, long.indels.vcf.gz, somatic.snvs.vcf.gz, somatic.short.indels.vcf.gz)
- Each VCF record carries both the GRP (group) and SRC (caller source) INFO keys, with SRC values correctly naming the originating caller (e.g. exitron, long_indel)
- Mutect2 output is correctly split into separate somatic.snvs.vcf.gz and somatic.short.indels.vcf.gz files after applying Mutect2's learned filters
- Merged VCFs are sorted and bgzipped (.vcf.gz), and per-group VCFs from SplAdder/ScanExitron were merged rather than left fragmented
- Module gating is respected: no VCF is produced for modules whose activate flag is false, and indel outputs respect the type (short|long|all) and mode (DNA|RNA|BOTH) keys

## Limitations

- Modules are individually gated by the config activate flag and indel type/mode keys, so a missing or misconfigured flag silently yields no merged VCF for that variant source
- Provenance depends on correct conversion of tool-native formats (SplAdder splice graph, ScanExitron exitron table, transIndel calls) to VCF; conversion errors lose GRP/SRC traceability
- Gene fusion calls from Arriba are handled as a separate module; per-source neoantigen counts in ScanNeo2 (5671–34 981 per patient) show candidates can be multi-source, so SRC keys must not be overwritten during merging

## Evidence

- [abstract] Unlike its predecessor, ScanNeo2 integrates multiple sources of somatic variants, including canonical- and exitron-splicing, gene fusion events, and various somatic variants.: "Unlike its predecessor, ScanNeo2 integrates multiple sources of somatic variants, including canonical- and exitron-splicing, gene fusion events, and various somatic variants."
- [results] 5671–34 981 candidate neoantigens originate from a single source, such as alternative splicing, exitron-splicing, gene fusion, indels, or SNVs: "5671–34 981 candidate neoantigens originate from a single source, such as alternative splicing, exitron-splicing, gene fusion, indels, or SNVs"
- [methods] Exitron events are called using [ScanExitron](https://github.com/ylab-hi/ScanExitron): "Exitron events are called using [ScanExitron](https://github.com/ylab-hi/ScanExitron)"
- [results] gatk ... is utilized for detection of short indels, as well as SNVs ... combined with transIndel (Yang et al. 2018) to detect long variants: [evidence span withheld — anchor status: not_found]
- [readme] A computational workflow for exitron splicing identification from RNA-Seq data.: [evidence span withheld — anchor status: external_doc]
