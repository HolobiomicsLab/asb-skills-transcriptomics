---
name: long-indel-remapped-alignment-calling
description: 'Use when reconstructing ScanNeo2''s variant-calling stage for long indels:
  matched DNA-seq and/or RNA-seq BAMs exist, config sets indel `type: long` (or `all`)
  with `mode: DNA|RNA|BOTH`, and long variants beyond Mutect2''s short-indel reach
  are needed.'
when_to_use_negative:
- The target variants are short indels or SNVs — ScanNeo2 routes those to GATK Mutect2
  run per split BAM across read-groups, not to the transIndel module.
- No matched DNA-seq or RNA-seq BAM alignment is available for the transIndel build
  step (indel.mode of DNA, RNA, or BOTH all require at least one aligned BAM).
- The neoantigen source of interest is alternative splicing (SplAdder), exitron-splicing
  (ScanExitron), or gene fusion (Arriba) rather than indel events.
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: transIndel
  role: Builds the remapped BAM with redefined CIGARs (DNA build, RNA build, or both
    per indel.mode) and calls long indels via detect_long_indel_ti_call
- name: ScanNeo2
  role: Snakemake workflow wrapping the long-indel module; merges per-group VCFs into
    long.indels.vcf.gz alongside the other caller modules
  repo: https://github.com/ylab-hi/ScanNeo2/
- name: Snakemake
  role: Workflow engine that gates the module on the config activate flag and indel
    type/mode keys and runs the caller steps
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad659
    title: 'ScanNeo2: a comprehensive workflow for neoantigen detection and immunogenicity
      prediction from diverse genomic and transcriptomic alterations'
  source_content_hash: sha256:10ee91d355a0bba90b2602a85f2435aa4fbada9320d444830db6459c71af168c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ylab-hi/ScanNeo2
  tools:
  - Arriba
  - GATK Mutect2
  - ScanExitron
  - ScanNeo2
  - Snakemake
  - SplAdder
  - transIndel
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# long-indel-remapped-alignment-calling

## Summary

ScanNeo2's long-indel detection module, which calls long variants with transIndel by building a remapped BAM with redefined CIGARs from DNA-seq and/or RNA-seq alignments, complementing the Mutect2 short-indel/SNV module.

## When to use

Use when reconstructing ScanNeo2's variant-calling stage for long indels: matched DNA-seq and/or RNA-seq BAMs exist, config sets indel `type: long` (or `all`) with `mode: DNA|RNA|BOTH`, and long variants beyond Mutect2's short-indel reach are needed.

## When NOT to use

- The target variants are short indels or SNVs — ScanNeo2 routes those to GATK Mutect2 run per split BAM across read-groups, not to the transIndel module.
- No matched DNA-seq or RNA-seq BAM alignment is available for the transIndel build step (indel.mode of DNA, RNA, or BOTH all require at least one aligned BAM).
- The neoantigen source of interest is alternative splicing (SplAdder), exitron-splicing (ScanExitron), or gene fusion (Arriba) rather than indel events.

## Inputs

- BWA-aligned DNA-seq BAM(s) from the ScanNeo2 alignment stage
- aligned RNA-seq BAM(s) from the ScanNeo2 alignment stage
- config/config.yml with indel-level keys: `type: short|long|all` and `mode: DNA|RNA|BOTH`

## Outputs

- Remapped BAM with redefined CIGARs produced by the transIndel build step
- Per-group long-indel VCFs annotated with GRP (group) and SRC=long_indel INFO keys
- Merged results/<sample>/variants/long.indels.vcf.gz

## How to apply

Read config/config.yml and the sample sheet, and gate the module on the indel-level keys `type: short|long|all` and `mode: DNA|RNA|BOTH`; use the BWA-aligned DNA-seq BAMs and RNA-seq BAMs from the alignment stage as caller inputs. First build a remapped BAM with redefined CIGARs via the transIndel build step — run the DNA build, the RNA build, or both depending on `indel.mode`. Then call indels with `detect_long_indel_ti_call`, augment the per-group VCFs with the GRP (group) and SRC=long_indel INFO keys, and merge into `long.indels.vcf.gz` under results/<sample>/variants/. The rationale is that transIndel (Yang et al. 2018) captures long variants that the short-indel/SNV caller (GATK Mutect2 on split BAMs) is not designed to detect, giving ScanNeo2 coverage of indel-derived neoantigens across sizes.

## Related tools

- **transIndel** (Builds the remapped BAM with redefined CIGARs (DNA build, RNA build, or both per indel.mode) and calls long indels via detect_long_indel_ti_call)
- **ScanNeo2** (Snakemake workflow wrapping the long-indel module; merges per-group VCFs into long.indels.vcf.gz alongside the other caller modules) — https://github.com/ylab-hi/ScanNeo2/
- **Snakemake** (Workflow engine that gates the module on the config activate flag and indel type/mode keys and runs the caller steps)

## Examples

```
snakemake --use-conda --cores 8 results/<sample>/variants/long.indels.vcf.gz  # with config/config.yml: indel: {type: long, mode: BOTH}
```

## Evaluation signals

- results/<sample>/variants/long.indels.vcf.gz exists for every activated group and is a valid bgzipped VCF
- Every record carries the GRP (group) and SRC=long_indel INFO keys added during augmentation
- The indel.mode setting is respected: DNA-only, RNA-only, or BOTH produce the corresponding transIndel build inputs
- Merged output contains contributions from all per-group calls, not just a single group
- The remapped BAM input to detect_long_indel_ti_call has redefined CIGARs produced by the transIndel build step

## Limitations

- Long-indel calling is deliberately separate from the Mutect2 short-indel/SNV module; using Mutect2 alone will miss long variants.
- Downstream, frameshift-derived candidates are restricted to transcripts escaping nonsense-mediated decay, so not every long indel yields candidate peptides.
- The paper text does not pin a specific Snakemake version for reproducibility of the module.

## Evidence

- [results] combined with transIndel (Yang et al. 2018) to detect long variants: [evidence span withheld — anchor status: not_found]
- [abstract] Unlike its predecessor, ScanNeo2 integrates multiple sources of somatic variants, including canonical- and exitron-splicing, gene fusion events, and various somatic variants.: "Unlike its predecessor, ScanNeo2 integrates multiple sources of somatic variants, including canonical- and exitron-splicing, gene fusion events, and various somatic variants."
- [intro] most focus mainly on finding neoantigens that come from single-nucleotide variants (SNVs), or indel events.: "most focus mainly on finding neoantigens that come from single-nucleotide variants (SNVs), or indel events."
- [results] In the case of a frameshift, only the transcripts are further considered that escape nonsense-mediated decay: "In the case of a frameshift, only the transcripts are further considered that escape nonsense-mediated decay"
