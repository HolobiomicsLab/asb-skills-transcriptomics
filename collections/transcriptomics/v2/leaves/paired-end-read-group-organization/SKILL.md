---
name: paired-end-read-group-organization
description: Use when a multi-sample sequencing workflow must ingest raw FASTQ inputs
  from a sample sheet into per-read-group QC and trimming steps, distinguishing paired-end
  (R1/R2) from single-end files for DNA-seq and RNA-seq.
when_to_use_negative:
- Input reads are already trimmed/filtered (no adapter or quality preprocessing needed)
  — re-running fastp trimming adds no value and risks over-trimming
- Data are long reads (Nanopore, PacBio) — fastp is designed for short reads (Illumina
  NovaSeq, MGI); the fastp README directs long-read users to fastplong
- The goal is downstream alignment or variant calling itself — this skill only covers
  the pre-processing stage before HLA genotyping, yara filtering, and variant calling
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0196
tools:
- name: ScanNeo2
  role: Snakemake workflow whose pre-processing stage organizes read groups, runs
    FastQC per read file, and applies fastp trimming before HLA genotyping
  repo: https://github.com/ylab-hi/ScanNeo2/
- name: fastp
  role: Performs quality filtering, adapter removal, 3'-end sliding-window trimming
    (wsize=3), and minimum-length filtering (minlen=10), writing gzipped trimmed reads
    per read group
  repo: https://github.com/OpenGene/fastp
- name: FastQC
  role: Generates per-read-group quality reports on raw reads, with separate forward/reverse
    reports for paired-end inputs
- name: Snakemake
  role: Workflow engine orchestrating per-(sample, seqtype, group) file organization
    and output naming conventions
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad659
    title: 'ScanNeo2: a comprehensive workflow for neoantigen detection and immunogenicity
      prediction from diverse genomic and transcriptomic alterations'
  source_content_hash: sha256:87cba6931b0195aa85da1a3ab54871c010a6104836c44f1bdb6b947ad1c7cddb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ylab-hi/ScanNeo2
  tools:
  - FastQC
  - ScanNeo2
  - Snakemake
  - fastp
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# paired-end-read-group-organization

## Summary

Organizes raw sequencing reads into per-(sample, seqtype, group) read groups and runs per-file quality control (FastQC) and trimming (fastp) as the first stage of the ScanNeo2 Snakemake workflow, keeping single-end and paired-end file conventions separate. This ensures every downstream HLA genotyping and variant-calling step consumes consistently named, quality-filtered gzipped FASTQ files.

## When to use

Use when a multi-sample sequencing workflow must ingest raw FASTQ inputs from a sample sheet into per-read-group QC and trimming steps, distinguishing paired-end (R1/R2) from single-end files for DNA-seq and RNA-seq.

## When NOT to use

- Input reads are already trimmed/filtered (no adapter or quality preprocessing needed) — re-running fastp trimming adds no value and risks over-trimming
- Data are long reads (Nanopore, PacBio) — fastp is designed for short reads (Illumina NovaSeq, MGI); the fastp README directs long-read users to fastplong
- The goal is downstream alignment or variant calling itself — this skill only covers the pre-processing stage before HLA genotyping, yara filtering, and variant calling

## Inputs

- Raw FASTQ read files (gzipped or plain), one or two files per read group for single-end / paired-end data
- Sample sheet config/samples.tsv defining (sample, seqtype, group) read groups and raw read paths
- config.yml with pre-processing settings (activate, minlen=10, wsize=3) and global basequal=20 Phred threshold

## Outputs

- Gzipped trimmed FASTQ files: results/<sample>/{dnaseq,rnaseq}/reads/<group>_preproc.fq.gz (single-end) or <group>_preproc_r1.fq.gz / <group>_preproc_r2.fq.gz (paired-end)
- FastQC quality reports per raw read file per read group, written to results/<sample>/{dnaseq,rnaseq}/qualitycontrol/ (separate forward/reverse reports for paired-end inputs)

## How to apply

Read the sample sheet (config/samples.tsv) and config.yml to collect raw read paths per (sample, seqtype, group), together with pre-processing settings: activate=true, minlen=10, sliding window size wsize=3, and the global basequal=20 Phred threshold. For each raw read file per read group, run FastQC, generating separate forward/reverse reports for paired-end inputs and writing them to results/<sample>/{dnaseq,rnaseq}/qualitycontrol/. Then trim the reads with fastp using 3'-end sliding-window trimming (window size 3), minimum-length filtering (minlen 10), and the global base-quality cutoff, writing gzipped trimmed reads to results/<sample>/{dnaseq,rnaseq}/reads/ as <group>_preproc.fq.gz (single-end) or <group>_preproc_r1.fq.gz / <group>_preproc_r2.fq.gz (paired-end). The rationale is that fastp applies quality filtering or adapter-sequence removal before reads are passed on to HLA allele genotyping and downstream variant calling, so consistent per-group naming is a precondition for the rest of the Snakemake DAG.

## Related tools

- **ScanNeo2** (Snakemake workflow whose pre-processing stage organizes read groups, runs FastQC per read file, and applies fastp trimming before HLA genotyping) — https://github.com/ylab-hi/ScanNeo2/
- **fastp** (Performs quality filtering, adapter removal, 3'-end sliding-window trimming (wsize=3), and minimum-length filtering (minlen=10), writing gzipped trimmed reads per read group) — https://github.com/OpenGene/fastp
- **FastQC** (Generates per-read-group quality reports on raw reads, with separate forward/reverse reports for paired-end inputs)
- **Snakemake** (Workflow engine orchestrating per-(sample, seqtype, group) file organization and output naming conventions)

## Examples

```
fastp -i in.fq -o out.fq
```

## Evaluation signals

- Trimmed gzipped FASTQ files exist for every read group in the sample sheet, following the documented naming convention (<group>_preproc.fq.gz or _r1/_r2 for paired-end)
- FastQC reports exist under results/<sample>/{dnaseq,rnaseq}/qualitycontrol/ for each raw read file, with paired-end inputs yielding separate forward and reverse reports
- Trimmed reads respect the minimum-length filter (minlen=10) and 3'-end sliding window (wsize=3) with the global base-quality cutoff (basequal=20 Phred)
- Output reads remain gzipped FASTQ in results/<sample>/{dnaseq,rnaseq}/reads/, matching directory conventions per seqtype
- Downstream rules (HLA genotyping, yara filtering) resolve their inputs from the preproc output names without missing-file errors

## Limitations

- fastp is designed for short reads (Illumina NovaSeq, MGI); long-read data (Nanopore, PacBio) require fastplong instead
- No specific Snakemake version is pinned in the article text (the README badge indicates snakemake >=9.0.0), so behavior may vary across versions
- Parameter defaults such as minlen=10 and wsize=3 are workflow config choices and may need adjustment for other data

## Evidence

- [results] the sequencing reads can be pre-processed, which includes quality filtering or removal of adapter sequences: "the sequencing reads can be pre-processed, which includes quality filtering or removal of adapter sequences"
- [methods] Other parameters include the minimum length of the sequencing reads (`minlen`): "Other parameters include the minimum length of the sequencing reads (`minlen`)"
- [methods] that can be combined with a window trimming from the 3'-end with a defined window size (`wsize`): "that can be combined with a window trimming from the 3'-end with a defined window size (`wsize`)"
- [methods] [fastp](https://github.com/OpenGene/fastp) writes the trimmed reads: "[fastp](https://github.com/OpenGene/fastp) writes the trimmed reads"
- [readme] A tool designed to provide ultrafast all-in-one preprocessing and quality control for FastQ data.: [evidence span withheld — anchor status: external_doc]
