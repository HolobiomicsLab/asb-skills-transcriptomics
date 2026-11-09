---
name: hla-read-source-mode-merging
description: Use when a neoantigen workflow must genotype HLA alleles from tumor samples
  using one or more read sources (DNA-seq, RNA-seq, or user-supplied allele files),
  with class-specific tool dispatch and merged multi-source allele output.
when_to_use_negative:
- Normal-sample rows should not be fed into HLA typing — the workflow excludes rows
  with type 'normal' by design.
- If HLA alleles are already known and provided as custom files only, running read-based
  DNA/RNA genotyping (yara/OptiType/HLA-HD) is unnecessary overhead.
- Do not attempt class II genotyping from DNA or RNA reads when the HLA-HD reference
  data (freq_data, split, dict) or hlahd.sh is missing — the workflow emits a '[config
  error]' at load time instead.
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3168
tools:
- name: ScanNeo2
  role: Snakemake workflow implementing the hlatyping block, class-conditional dispatch,
    mode merging, and TSV output
  repo: https://github.com/ylab-hi/ScanNeo2/
- name: yara
  role: Filters DNA-seq and RNA-seq reads for HLA reads prior to class I typing
- name: Optitype
  role: Aligns yara-filtered reads against a panel of HLA class I alleles to genotype
    class I
- name: HLA-HD
  role: Performs HLA class II genotyping from reads via hlahd.sh with freq_data, split,
    and dict reference data
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad659
    title: 'ScanNeo2: a comprehensive workflow for neoantigen detection and immunogenicity
      prediction from diverse genomic and transcriptomic alterations'
  source_content_hash: sha256:60082f2c711f5e353a7c3d4fd10006a4f5aa3657440d4252a32f1904e9c0e726
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ylab-hi/ScanNeo2
  tools:
  - HLA-HD
  - Optitype
  - ScanNeo2
  - yara
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# HLA read-source mode merging

## Summary

Configures and merges HLA genotyping results across multiple read sources (DNA, RNA, custom allele files) in ScanNeo2, dispatching class I typing to OptiType (yara-filtered reads) and class II typing to HLA-HD, and writing per-source allele rows to mhc-I.tsv / mhc-II.tsv.

## When to use

Use when a neoantigen workflow must genotype HLA alleles from tumor samples using one or more read sources (DNA-seq, RNA-seq, or user-supplied allele files), with class-specific tool dispatch and merged multi-source allele output.

## When NOT to use

- Normal-sample rows should not be fed into HLA typing — the workflow excludes rows with type 'normal' by design.
- If HLA alleles are already known and provided as custom files only, running read-based DNA/RNA genotyping (yara/OptiType/HLA-HD) is unnecessary overhead.
- Do not attempt class II genotyping from DNA or RNA reads when the HLA-HD reference data (freq_data, split, dict) or hlahd.sh is missing — the workflow emits a '[config error]' at load time instead.

## Inputs

- config/config.yml hlatyping block (class: I/II/BOTH; MHC-I_mode / MHC-II_mode comma-separated lists of DNA, RNA, custom)
- config/samples.tsv sample sheet with per-sample FASTQ/BAM paths, type column (tumor/normal), and custom_hla_I / custom_hla_II allele file columns
- DNA-seq and/or RNA-seq FASTQ or BAM reads for tumor samples
- User-supplied tab-delimited HLA allele files (custom mode)
- HLA-HD reference data directories: freq_data, HLA_gene.split.txt, dictionary

## Outputs

- results/<sample>/hla/mhc-I.tsv with rows formatted '<source>\t<allele>' (source = DNA/RNA/custom)
- results/<sample>/hla/mhc-II.tsv with rows formatted '<source>\t<allele>' (source = DNA/RNA/custom)

## How to apply

Parse the hlatyping block of config/config.yml to determine which MHC classes to type (class: I, II, or BOTH) and the per-class read-source modes (MHC-I_mode / MHC-II_mode as comma-separated lists of DNA, RNA, custom). Read config/samples.tsv to locate FASTQ/BAM reads per sample, using only tumor rows since rows with type 'normal' are excluded from HLA typing. For class I under DNA or RNA mode, filter reads for HLA reads with yara and type class I alleles with OptiType; for class II, run HLA-HD (hlahd.sh on PATH) with its reference data directories freq_data, split, and dict. For custom mode, load the user-supplied tab-delimited allele files given in the custom_hla_I / custom_hla_II sample-sheet columns. Merge allele sets across comma-separated modes (e.g. 'DNA, RNA' merges typing from both sources) while keeping read-based and custom sets when combined, and write per-source allele rows as results/<sample>/hla/mhc-I.tsv and mhc-II.tsv, each row formatted '<source>\t<allele>'.

## Related tools

- **ScanNeo2** (Snakemake workflow implementing the hlatyping block, class-conditional dispatch, mode merging, and TSV output) — https://github.com/ylab-hi/ScanNeo2/
- **yara** (Filters DNA-seq and RNA-seq reads for HLA reads prior to class I typing)
- **Optitype** (Aligns yara-filtered reads against a panel of HLA class I alleles to genotype class I)
- **HLA-HD** (Performs HLA class II genotyping from reads via hlahd.sh with freq_data, split, and dict reference data)

## Examples

```
snakemake --use-conda --configfile config/config.yml results/<sample>/hla/mhc-I.tsv results/<sample>/hla/mhc-II.tsv
```

## Evaluation signals

- At workflow-load time, typing class II via DNA or RNA without HLA-HD reference data or hlahd.sh on PATH must raise a '[config error]'.
- results/<sample>/hla/mhc-I.tsv and mhc-II.tsv must exist and contain source-labeled allele rows (DNA/RNA/custom).
- When MHC mode is a comma-separated list (e.g. 'DNA, RNA'), the output TSV must contain allele rows from all listed sources, merged rather than overwritten.
- No allele rows in the output may originate from samples whose type is 'normal' in config/samples.tsv.
- Class I rows must trace to yara/OptiType runs (DNA/RNA) or custom files; class II rows must trace to HLA-HD runs (DNA/RNA) or custom files — never class I tools for class II output.

## Limitations

- HLA-HD requires external reference data (freq_data, HLA_gene.split.txt, dictionary) and hlahd.sh on PATH; missing these fails class II read-based typing.
- Only tumor samples are eligible for HLA typing; normal rows are excluded.
- Read-source modes must be declared per MHC class; mixing modes requires understanding that comma-separated modes merge allele sets rather than replace them.

## Evidence

- [results] Class I genotyping dispatch: "DNA-seq and RNA-seq data are filtered for HLA reads using yara (Siragusa et al. 2013) and subsequently aligned against a panel of HLA class I alleles using Optitype"
- [results] Class II genotyping tool: "In a similar manner, HLA-HD (Kawaguchi et al. 2017) is used for HLA class II genotyping"
- [methods] HLA-HD reference data: "`freqdata`: ./hlahd_files/freq_data/"
- [abstract] Workflow source repo: "ScanNeo2 is freely available at https://github.com/ylab-hi/ScanNeo2/ and is accompanied by instruction and application data."
