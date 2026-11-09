---
name: fastq-naming-schema-validation
description: Use when raw GEX/ADT FASTQ files must be staged before a gExcite/cellranger
  run and you need to confirm the per-sample directory layout and SAMPLENAME_S[Number]_L00[Lane]_[ReadType]_001.fastq.gz
  naming before execution.
when_to_use_negative:
- The analysis starts after the cellranger count and CITE-Seq steps — then dummy files,
  not validated real FASTQs, are placed in the schema positions
- Input is already a count matrix or cellranger output rather than raw FASTQ files
- Running scAmpi with STARsolo where a different, non-cellranger FASTQ layout may
  be chosen — validate against that tool's expectations instead
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3170
tools:
- name: cellranger
  role: Consumer of the FASTQ naming schema — it parses sample name, lane and read
    type from the filenames and requires one sub-directory per sample
  repo: https://support.10xgenomics.com/single-cell-gene-expression/software/pipelines/latest/installation
- name: gExcite
  role: Snakemake pipeline whose 'Before running the pipeline' step requires ADT and
    GEX FASTQs in the specified folder structure and an adapted config.yaml
  repo: https://github.com/ETH-NEXUS/gExcite_pipeline
- name: Snakemake
  role: Used for a dry run (snakemake -n) to confirm the staged FASTQ inputs resolve
    correctly before executing the workflow
  repo: https://github.com/ETH-NEXUS/gExcite_pipeline
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad329
    title: 'gExcite: a start-to-end framework for single-cell gene expression, hashing,
      and antibody analysis'
  source_content_hash: sha256:72587505fbb1d8e02eed205c8fe374e128e5f1f4ba446601fc98426fed4002e4
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Snakemake
  - cellranger
  - gExcite
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# fastq-naming-schema-validation

## Summary

Validates that raw GEX and ADT FASTQ files are staged in cellranger's expected per-sample folder structure with the SAMPLENAME_S[Number]_L00[Lane]_[ReadType]_001.fastq.gz naming schema, so the gExcite Snakemake pipeline can consume them without aborting.

## When to use

Use when raw GEX/ADT FASTQ files must be staged before a gExcite/cellranger run and you need to confirm the per-sample directory layout and SAMPLENAME_S[Number]_L00[Lane]_[ReadType]_001.fastq.gz naming before execution.

## When NOT to use

- The analysis starts after the cellranger count and CITE-Seq steps — then dummy files, not validated real FASTQs, are placed in the schema positions
- Input is already a count matrix or cellranger output rather than raw FASTQ files
- Running scAmpi with STARsolo where a different, non-cellranger FASTQ layout may be chosen — validate against that tool's expectations instead

## Inputs

- GEX FASTQ files (I1, R1, R2 per sample) for the sequencing run
- ADT/CITE-seq FASTQ files (I1, R1, R2 per sample)
- gExcite config.yaml with FASTQ input paths relative to the working directory

## Outputs

- Validated per-sample FASTQ directory tree conforming to SAMPLENAME_S[Number]_L00[Lane]_[ReadType]_001.fastq.gz
- Dummy FASTQ files in place of real FASTQs when resuming after cellranger count/CITE-Seq
- A Snakemake dry run that resolves all inputs without errors

## How to apply

Lay out the FASTQs so each sample sits in its own sub-directory under input_fastqs_gex and input_fastqs_adt, e.g. /path/to/input_fastqs_gex/SAMPLENAME/SAMPLENAME_S4_L001_I1_001.fastq.gz, covering the I1 (sample index), R1 and R2 read types. Verify the naming schema SAMPLENAME_S[Number]_L00[Lane Number]_[Read Type]_001.fastq.gz for every file, since cellranger parses sample name, sample number, lane and read type from these tokens. Check that SAMPLENAME uses only the characters [a-zA-Z0-9_-]+ — a dot in the sample name makes cellranger stop immediately. Confirm the FASTQ input paths are specified relative to the working directory in config.yaml (this is mandatory in the current pipeline, not merely recommended). If starting downstream of the resource-intensive cellranger count and CITE-Seq steps, place dummy files in place of the FASTQs instead of real reads. Finally validate the staged inputs with a dry run of the Snakemake workflow (snakemake -n) before launching the full run.

## Related tools

- **cellranger** (Consumer of the FASTQ naming schema — it parses sample name, lane and read type from the filenames and requires one sub-directory per sample) — https://support.10xgenomics.com/single-cell-gene-expression/software/pipelines/latest/installation
- **gExcite** (Snakemake pipeline whose 'Before running the pipeline' step requires ADT and GEX FASTQs in the specified folder structure and an adapted config.yaml) — https://github.com/ETH-NEXUS/gExcite_pipeline
- **Snakemake** (Used for a dry run (snakemake -n) to confirm the staged FASTQ inputs resolve correctly before executing the workflow) — https://github.com/ETH-NEXUS/gExcite_pipeline

## Examples

```
snakemake -n --use-conda --configfile config/config.yaml
```

## Evaluation signals

- Every FASTQ filename matches the pattern SAMPLENAME_S[Number]_L00[Lane Number]_[Read Type]_001.fastq.gz with read types I1, R1 and R2 present per sample
- SAMPLENAME contains only characters from [a-zA-Z0-9_-]+ (no dots — cellranger stops otherwise)
- Each sample has its own sub-directory under input_fastqs_gex and input_fastqs_adt
- FASTQ input paths in config.yaml are specified relative to the working directory (mandatory)
- A dry run of the Snakemake workflow (snakemake -n) completes without errors

## Limitations

- SAMPLENAME with a dot causes cellranger to stop immediately, so sample names must be pre-checked against [a-zA-Z0-9_-]+
- Specifying FASTQ input relative to the working directory is mandatory in the current gExcite pipeline, not merely recommended; older absolute-path configs fail
- When resuming after cellranger count and CITE-Seq, dummy files must substitute the real FASTQs, so this validation stage confirms the dummy layout rather than raw data

## Evidence

- [methods] The pipeline expects the FASTQ files per sample to be in the following folder structure: "The pipeline expects the FASTQ files per sample to be in the following folder structure"
- [readme] The pipeline expects the FASTQ files per sample to be in the following folder structure, adhering to the naming schema:: "The pipeline expects the FASTQ files per sample to be in the following folder structure, adhering to the naming schema:"
- [readme] SAMPLENAME can only contain the following characters [a-zA-Z0-9_-]+ — a dot stops cellranger: "NOTE: SAMPLENAME can only contain the following characters [a-zA-Z0-9_-]+
  E.g., if the SAMPLENAME contains a dot, cellranger will stop right away."
- [readme] Starting processing after the resource-intensive cellranger count and CITE-Seq steps requires the presence of dummy files: "Starting processing after the resource-intensive cellranger count and CITE-Seq steps requires the presence of dummy files in place of the FASTQ files described in this chapter."
- [methods] In case of combined GEX and ADT NovaSeq sequencing data, scripts are provided to clean up the data before a run: "In case of combined GEX and ADT NovaSeq sequencing data, scripts are provided to clean up the data before a run"
