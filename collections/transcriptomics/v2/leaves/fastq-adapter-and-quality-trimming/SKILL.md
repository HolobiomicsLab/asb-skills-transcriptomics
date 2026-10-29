---
name: fastq-adapter-and-quality-trimming
description: Use when raw Illumina short-read FASTQ (single-end or paired-end, DNA-seq
  or RNA-seq) must be quality-filtered, adapter-trimmed, and QC-reported before alignment
  or genotyping in a pipeline.
when_to_use_negative:
- Input is long-read data (Nanopore/PacBio) — the fastp README states it is designed
  for processing short reads and points to fastplong for long reads
- Reads are already adapter-trimmed and quality-filtered by an upstream pre-processing
  stage
- Input is alignment data (BAM/SAM) rather than raw FASTQ — this skill operates at
  the FASTQ level only
edam_operation: http://edamontology.org/operation_3180
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0102
tools:
- name: fastp
  role: Performs quality filtering, adapter removal, 3'-end sliding-window trimming,
    and minimum-length filtering of raw FASTQ reads; writes gzipped trimmed reads
  repo: https://github.com/OpenGene/fastp
- name: FastQC
  role: Generates per-read-group quality-control reports on raw reads (separate forward/reverse
    reports for paired-end input) before trimming
- name: Snakemake
  role: Workflow engine implementing the pre-processing rules driven by config/samples.tsv
    and config.yml
  repo: https://github.com/ylab-hi/ScanNeo2/
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad659
    title: 'ScanNeo2: a comprehensive workflow for neoantigen detection and immunogenicity
      prediction from diverse genomic and transcriptomic alterations'
  source_content_hash: sha256:d5b3bc01f462ecbc5435e25b4bfe2bf480182a18e79ea45ecc605cff9fa781a7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/OpenGene/fastp
  tools:
  - FastQC
  - ScanNeo2
  - Snakemake
  - fastp
  edam_operation: http://edamontology.org/operation_3180
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5625
n_claims: 16
---

# fastq-adapter-and-quality-trimming

## Summary

Pre-processing stage of the ScanNeo2 Snakemake workflow that applies fastp to raw FASTQ reads for quality filtering and adapter removal, preceded by FastQC quality reports, before downstream HLA genotyping and variant calling.

## When to use

Use when raw Illumina short-read FASTQ (single-end or paired-end, DNA-seq or RNA-seq) must be quality-filtered, adapter-trimmed, and QC-reported before alignment or genotyping in a pipeline.

## When NOT to use

- Input is long-read data (Nanopore/PacBio) — the fastp README states it is designed for processing short reads and points to fastplong for long reads
- Reads are already adapter-trimmed and quality-filtered by an upstream pre-processing stage
- Input is alignment data (BAM/SAM) rather than raw FASTQ — this skill operates at the FASTQ level only

## Inputs

- Raw single-end FASTQ read files per read group
- Raw paired-end FASTQ read files (forward/reverse R1/R2)
- config/samples.tsv sample sheet mapping reads to (sample, seqtype, group)
- config.yml with preproc settings (activate, minlen=10, slidingwindow wsize=3, basequal=20)

## Outputs

- Gzipped trimmed FASTQ files: results/<sample>/{dnaseq,rnaseq}/reads/<group>_preproc.fq.gz (single-end) or <group>_preproc_r1.fq.gz / <group>_preproc_r2.fq.gz (paired-end)
- FastQC quality-control reports per read group in results/<sample>/{dnaseq,rnaseq}/qualitycontrol/
- fastp HTML/JSON quality-control and filtering reports

## How to apply

Read the sample sheet (config/samples.tsv) and config.yml to collect raw read paths per (sample, seqtype, group) and the pre-processing settings: activate=true, minlen=10, slidingwindow wsize=3, with a global basequal=20 Phred threshold. First run FastQC on each raw read file per read group (separate forward/reverse reports for paired-end input), writing reports to results/<sample>/{dnaseq,rnaseq}/qualitycontrol/. Then trim reads with fastp using 3'-end sliding-window trimming (window size 3), minimum-length filtering (minlen 10), and the global base-quality cutoff, writing gzipped trimmed reads to results/<sample>/{dnaseq,rnaseq}/reads/ as <group>_preproc.fq.gz (single-end) or <group>_preproc_r1.fq.gz / <group>_preproc_r2.fq.gz (paired-end). The rationale is that quality filtering and adapter removal upstream improves the reliability of subsequent yara/Optitype HLA genotyping and variant calling; the step is orchestrated as a Snakemake rule driven by the per-sample configuration.

## Related tools

- **fastp** (Performs quality filtering, adapter removal, 3'-end sliding-window trimming, and minimum-length filtering of raw FASTQ reads; writes gzipped trimmed reads) — https://github.com/OpenGene/fastp
- **FastQC** (Generates per-read-group quality-control reports on raw reads (separate forward/reverse reports for paired-end input) before trimming)
- **Snakemake** (Workflow engine implementing the pre-processing rules driven by config/samples.tsv and config.yml) — https://github.com/ylab-hi/ScanNeo2/

## Examples

```
fastp -i in.fq -o out.fq
```

## Evaluation signals

- Trimmed gzipped FASTQ files exist for every read group in the sample sheet, following the documented naming (<group>_preproc.fq.gz or <group>_preproc_r1/_r2.fq.gz) and directory conventions (results/<sample>/{dnaseq,rnaseq}/reads/)
- FastQC reports exist in results/<sample>/{dnaseq,rnaseq}/qualitycontrol/ for every read group, with separate forward/reverse reports for paired-end inputs
- fastp JSON/HTML reports confirm the expected parameters were applied: minlen=10, sliding window size 3, and base-quality cutoff Phred 20
- Trimmed output is gzip-compressed and read counts in trimmed FASTQ do not exceed raw input counts (quality/length filtering removed reads)
- Snakemake validation passes: the preproc rule marked activate=true produces all declared outputs before downstream HLA genotyping rules run

## Limitations

- The pre-processing parameters (minlen, wsize, basequal) are configured per the ScanNeo2 defaults in config.yml; very short or heavily degraded libraries may lose most reads at minlen=10 combined with the Phred-20 sliding-window trim
- fastp is designed for short reads; the README directs long-read users to fastplong instead
- Truncated repository README for ScanNeo2 leaves some per-parameter CLI details unverifiable from the provided context

## Evidence

- [results] the sequencing reads can be pre-processed, which includes quality filtering or removal of adapter sequences: "the sequencing reads can be pre-processed, which includes quality filtering or removal of adapter sequences. For that, fastp (Chen et al. 2018) is applied"
- [methods] Other parameters include the minimum length of the sequencing reads (`minlen`): [evidence span withheld — anchor status: not_found]
- [readme] fastp README tool description: [evidence span withheld — anchor status: external_doc]
- [readme] cut low quality bases per read via sliding window: [evidence span withheld — anchor status: external_doc]
