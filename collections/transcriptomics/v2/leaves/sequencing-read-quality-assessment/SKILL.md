---
name: sequencing-read-quality-assessment
description: Use when starting a sequencing analysis (DNA-seq or RNA-seq) from raw
  FASTQ data and the downstream pipeline needs quality-checked, adapter-free, length-
  and quality-filtered reads before alignment or genotyping.
when_to_use_negative:
- Input is already trimmed, adapter-free BAM/CRAM or aligned data — this skill operates
  on raw FASTQ only
- Processing long reads (PacBio/Nanopore); fastp is designed for short reads (e.g.
  Illumina NovaSeq, MGI), and the fastp README directs long-read users elsewhere
- The goal is downstream neoantigen prediction/immunogenicity ranking rather than
  the read pre-processing stage itself
edam_topics:
- http://edamontology.org/topic_3173
tools:
- name: FastQC
  role: Generates per-read-group quality reports for raw FASTQ files before trimming
  repo: https://github.com/ylab-hi/ScanNeo2/
- name: fastp
  role: Performs quality filtering, adapter removal, 3'-end sliding-window trimming,
    and length filtering; writes gzipped trimmed reads
  repo: https://github.com/OpenGene/fastp
- name: Snakemake
  role: Workflow engine orchestrating the pre-processing rules for every read group
  repo: https://github.com/ylab-hi/ScanNeo2/
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad659
    title: 'ScanNeo2: a comprehensive workflow for neoantigen detection and immunogenicity
      prediction from diverse genomic and transcriptomic alterations'
  source_content_hash: sha256:b2d673c654655ecf8245a724771bd62a127500bee0703047d5cca440ee33b30a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ylab-hi/ScanNeo2
  tools:
  - FastQC
  - ScanNeo2
  - Snakemake
  - fastp
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3173
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# sequencing-read-quality-assessment

## Summary

First stage of the ScanNeo2 Snakemake workflow: assess raw FASTQ quality with FastQC and trim reads with fastp (adapter removal, sliding-window quality trimming, length filtering) before HLA genotyping and variant calling.

## When to use

Use when starting a sequencing analysis (DNA-seq or RNA-seq) from raw FASTQ data and the downstream pipeline needs quality-checked, adapter-free, length- and quality-filtered reads before alignment or genotyping.

## When NOT to use

- Input is already trimmed, adapter-free BAM/CRAM or aligned data — this skill operates on raw FASTQ only
- Processing long reads (PacBio/Nanopore); fastp is designed for short reads (e.g. Illumina NovaSeq, MGI), and the fastp README directs long-read users elsewhere
- The goal is downstream neoantigen prediction/immunogenicity ranking rather than the read pre-processing stage itself

## Inputs

- Raw gzipped FASTQ read files (single-end or paired-end forward/reverse) per (sample, seqtype, group) read group
- Sample sheet config/samples.tsv mapping samples to raw read paths
- config.yml with preproc settings: activate=true, minlen=10, slidingwindow wsize=3, and global basequal=20 Phred threshold

## Outputs

- FastQC quality reports per read file under results/<sample>/{dnaseq,rnaseq}/qualitycontrol/
- Gzipped trimmed FASTQ reads under results/<sample>/{dnaseq,rnaseq}/reads/ as <group>_preproc.fq.gz (single-end) or <group>_preproc_r1.fq.gz / <group>_preproc_r2.fq.gz (paired-end)
- fastp before/after-filtering quality profiling (/HTML) with Q20/Q30, adapter contents, and quality curves

## How to apply

Read the sample sheet (config/samples.tsv) and config.yml to collect raw read paths per (sample, seqtype, group) and the preproc settings (activate=true, minlen=10, slidingwindow wsize=3, global basequal=20 Phred). Run FastQC on each raw read file per read group, producing separate forward/reverse reports for paired-end inputs under results/<sample>/{dnaseq,rnaseq}/qualitycontrol/. Then trim with fastp: cut from the 3'-end using a sliding window (window size 3), enforce a minimum read length (minlen 10), and apply the global base-quality cutoff, which also removes adapter sequences. Write gzipped trimmed reads to results/<sample>/{dnaseq,rnaseq}/reads/ as <group>_preproc.fq.gz (single-end) or <group>_preproc_r1.fq.gz / <group>_preproc_r2.fq.gz (paired-end). These trimmed reads are the input to HLA allele genotyping (yara/Optitype for class I, HLA-HD for class II).

## Related tools

- **FastQC** (Generates per-read-group quality reports for raw FASTQ files before trimming) — https://github.com/ylab-hi/ScanNeo2/
- **fastp** (Performs quality filtering, adapter removal, 3'-end sliding-window trimming, and length filtering; writes gzipped trimmed reads) — https://github.com/OpenGene/fastp
- **Snakemake** (Workflow engine orchestrating the pre-processing rules for every read group) — https://github.com/ylab-hi/ScanNeo2/

## Examples

```
fastp -i in.fq -o out.fq
```

## Evaluation signals

- Gzipped trimmed FASTQ files exist for every read group in the sample sheet, following the documented naming convention (<group>_preproc.fq.gz or <group>_preproc_r1/_r2.fq.gz) in results/<sample>/{dnaseq,rnaseq}/reads/
- FastQC reports exist for every raw read file per read group, with separate forward/reverse reports for paired-end inputs, under results/<sample>/{dnaseq,rnaseq}/qualitycontrol/
- fastp quality report shows Q20/Q30 statistics and adapter contents for both before- and after-filtering data, allowing comparison of quality curves pre/post trimming
- No output read is shorter than the minlen=10 minimum length; trimming respected the wsize=3 sliding window and basequal=20 Phred cutoff
- Paired-end inputs yield exactly two output files (R1/R2) per read group, single-end inputs one

## Limitations

- The truncating/quality parameters are configurable (minlen, wsize, basequal); the values cited here are ScanNeo2 defaults, and other pipelines may need different thresholds
- fastp targets short reads — long-read data (PacBio/Nanopore) requires a different preprocessing tool per the fastp README
- Trimming is lossy: reads below the length/quality cutoffs are discarded, so downstream read counts will be lower than raw input counts

## Evidence

- [results] the sequencing reads can be pre-processed, which includes quality filtering or removal of adapter sequences: "the sequencing reads can be pre-processed, which includes quality filtering or removal of adapter sequences. For that, fastp (Chen et al. 2018) is applied"
- [methods] minlen parameter: "Other parameters include the minimum length of the sequencing reads (`minlen`)"
- [methods] sliding window trimming: "that can be combined with a window trimming from the 3'-end with a defined window size (`wsize`)"
- [readme] fastp all-in-one preprocessing: [evidence span withheld — anchor status: external_doc]
- [readme] fastp quality profiling: [evidence span withheld — anchor status: external_doc]
- [readme] fastp sliding-window cutting: [evidence span withheld — anchor status: external_doc]
