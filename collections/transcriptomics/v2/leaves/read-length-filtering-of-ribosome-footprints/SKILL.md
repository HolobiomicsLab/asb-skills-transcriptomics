---
name: read-length-filtering-of-ribosome-footprints
description: Use when preprocessing Ribo-seq FastQ files and only footprints within
  a defined nucleotide-length window (e.g. 25–35 nt, or a user-fixed range such as
  20–22 nt) should be retained for analysis.
when_to_use_negative:
- Input reads are already trimmed and length-selected (the already_trimmed flag in
  config.yaml is set), so rerunning the Cutadapt filtering step is unnecessary.
- The analysis targets RNA-seq or another long-read assay where fragments are not
  expected to fall in the ~25–35 nt RPF range, so the 25–35 nt window would incorrectly
  discard valid reads.
edam_operation: http://edamontology.org/operation_3216
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_3170
tools:
- name: Cutadapt
  role: Performs adapter and quality trimming plus the read-length filtering step
    that keeps only reads between readsLength_min and readsLength_max nucleotides
  repo: https://cutadapt.readthedocs.io/en/stable/
- name: FastQC
  role: Runs on the trimmed, length-filtered reads to quality-control the outcome
    of the filtering step
  repo: https://www.bioinformatics.babraham.ac.uk/projects/fastqc/
- name: RiboDoc
  role: Docker/Snakemake pipeline that wraps the Cutadapt length-filtering step with
    configurable readsLength_min/readsLength_max parameters from config.yaml
  repo: https://github.com/equipeGST/RiboDoc
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.csbj.2021.05.014
    title: 'RiboDoc: A Docker-based package for ribosome profiling analysis'
  source_content_hash: sha256:9945d276e63b7af0b6428d2f551bf67dd1548f449bd240dd92f19165440ba812
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/equipeGST/RiboDoc
  tools:
  - Cutadapt
  - FastQC
  - RiboDoc
  edam_operation: http://edamontology.org/operation_3216
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# Read-length filtering of ribosome footprints

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Selects ribosome-protected fragments of the expected RPF size range (default 25–35 nt in RiboDoc) during the Cutadapt preprocessing step, dropping reads that are too long or too short before downstream alignment and quantification.

## When to use

Use when preprocessing Ribo-seq FastQ files and only footprints within a defined nucleotide-length window (e.g. 25–35 nt, or a user-fixed range such as 20–22 nt) should be retained for analysis.

## When NOT to use

- Input reads are already trimmed and length-selected (the already_trimmed flag in config.yaml is set), so rerunning the Cutadapt filtering step is unnecessary.
- The analysis targets RNA-seq or another long-read assay where fragments are not expected to fall in the ~25–35 nt RPF range, so the 25–35 nt window would incorrectly discard valid reads.

## Inputs

- gzip-compressed FastQ files from the project's fastq/ subfolder (named [CONDITION]_[NAME].[REPLICATE].fastq.gz)
- config.yaml parameters: readsLength_min ('25'), readsLength_max ('35'), adapt_sequence, already_trimmed

## Outputs

- trimmed_reads.fastq.gz containing trimmed, length-filtered reads in valid gzip FASTQ format
- per-sample adapter information text files in the adapter_lists/ folder
- FastQC reports in the fastqc_after_trimming folder

## How to apply

Read readsLength_min and readsLength_max from the project's config.yaml (defaults 25 and 35, the typical RiboSeq size range around 30 nt) and pass them to the Cutadapt read-length filtering step together with adapter trimming (3' adapter, error rate 12.5%) and filtering of reads with more than one N. If the user fixes a narrower range (e.g. 20–22 bases), apply it instead of the default. Cutadapt writes the trimmed, length-filtered reads to trimmed_reads.fastq.gz, after which FastQC reports are generated in the fastqc_after_trimming folder. The rationale is that genuine ribosome footprints cluster around ~30 nt (28 nt in the paper's example), so length filtering removes non-RPF fragments before contaminant removal (Bowtie2) and alignment (HISAT2/Bowtie2).

## Related tools

- **Cutadapt** (Performs adapter and quality trimming plus the read-length filtering step that keeps only reads between readsLength_min and readsLength_max nucleotides) — https://cutadapt.readthedocs.io/en/stable/
- **FastQC** (Runs on the trimmed, length-filtered reads to quality-control the outcome of the filtering step) — https://www.bioinformatics.babraham.ac.uk/projects/fastqc/
- **RiboDoc** (Docker/Snakemake pipeline that wraps the Cutadapt length-filtering step with configurable readsLength_min/readsLength_max parameters from config.yaml) — https://github.com/equipeGST/RiboDoc

## Examples

```
cutadapt -a $adapt_sequence -e 0.125 --max-n 1 -m 25 -M 35 -o trimmed_reads.fastq.gz input.fastq.gz
```

## Evaluation signals

- trimmed_reads.fastq.gz exists and is a valid gzip FASTQ file
- All retained reads fall within the configured readsLength_min–readsLength_max range (default 25–35 nt)
- A read-length distribution plot of the filtered reads shows the expected RPF peak (e.g. majority of reads 28 nt long as in the paper's example)
- Each sample has an adapter_lists entry recording the adapter used
- FastQC reports for the trimmed reads are present in the fastqc_after_trimming folder

## Limitations

- The default 25–35 nt window assumes canonical ~30 nt RPFs; samples with footprints outside this range (or a user-desired 20–22 nt range) require editing the config.yaml parameters.
- Filtering parameters are stated at the dataset level; reads that are too long or too short are removed without per-sample adaptation of the window.
- RiboDoc's published materials do not state a version number for the pipeline or Docker package, so exact parameter defaults may differ across retrieved versions.

## Evidence
