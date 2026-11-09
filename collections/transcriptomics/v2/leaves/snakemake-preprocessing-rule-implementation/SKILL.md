---
name: snakemake-preprocessing-rule-implementation
description: Use when a Snakemake workflow must turn raw FASTQ reads per (sample,
  seqtype, group) into trimmed, gzipped pre-processed reads plus QC reports prior
  to HLA genotyping or variant calling.
when_to_use_negative:
- Input reads are already trimmed/pre-processed (e.g. the pipeline consumes <group>_preproc.fq.gz
  rather than raw FASTQ) — re-trimming would double-filter the data
- Data are long reads (Nanopore/PacBio); fastp as applied here targets short reads
  (Illumina NovaSeq, MGI), for which the fastp README recommends fastplong instead
- The task is downstream variant or neoantigen calling (e.g. SplAdder/ScanExitron/Arriba
  steps), which consumes the trimmed reads rather than producing them
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_0160
tools:
- name: Snakemake
  role: Workflow engine implementing the pre-processing rules per read group with
    config-driven parameters
- name: fastp
  role: Performs quality filtering, adapter removal, 3'-end sliding-window trimming
    and minlen filtering, writing the gzipped pre-processed FASTQs
  repo: https://github.com/OpenGene/fastp
- name: FastQC
  role: Generates per-read-group quality reports on the raw FASTQ inputs before trimming
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad659
    title: 'ScanNeo2: a comprehensive workflow for neoantigen detection and immunogenicity
      prediction from diverse genomic and transcriptomic alterations'
  source_content_hash: sha256:3c67c5852e6ea15436032681c5a4eb49e1da71fe4310303a9a021b6c81df53c5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/OpenGene/fastp
  tools:
  - FastQC
  - ScanNeo2
  - Snakemake
  - fastp
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_0160
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# snakemake-preprocessing-rule-implementation

## Summary

Implement the read pre-processing stage of a Snakemake-based pipeline (as in ScanNeo2): FastQC quality reports on raw reads followed by fastp trimming with sliding-window 3'-end trimming, minimum-length filtering, and a global base-quality cutoff, before downstream HLA genotyping and variant calling.

## When to use

Use when a Snakemake workflow must turn raw FASTQ reads per (sample, seqtype, group) into trimmed, gzipped pre-processed reads plus QC reports prior to HLA genotyping or variant calling.

## When NOT to use

- Input reads are already trimmed/pre-processed (e.g. the pipeline consumes <group>_preproc.fq.gz rather than raw FASTQ) — re-trimming would double-filter the data
- Data are long reads (Nanopore/PacBio); fastp as applied here targets short reads (Illumina NovaSeq, MGI), for which the fastp README recommends fastplong instead
- The task is downstream variant or neoantigen calling (e.g. SplAdder/ScanExitron/Arriba steps), which consumes the trimmed reads rather than producing them

## Inputs

- Raw FASTQ files referenced per (sample, seqtype, group) in config/samples.tsv
- config.yml with pre-processing settings (activate=true, minlen=10, wsize=3, basequal=20)
- Paired-end FASTQ pairs (forward/reverse) or single-end FASTQ per read group

## Outputs

- Gzipped trimmed FASTQ per read group: results/<sample>/{dnaseq,rnaseq}/reads/<group>_preproc.fq.gz (single-end) or <group>_preproc_r1.fq.gz and <group>_preproc_r2.fq.gz (paired-end)
- FastQC HTML reports per raw read file under results/<sample>/{dnaseq,rnaseq}/qualitycontrol/

## How to apply

Read the sample sheet (config/samples.tsv) and config.yml to collect raw read paths per (sample, seqtype, group) plus pre-processing settings: activate=true, minlen=10, sliding-window wsize=3, with the global basequal=20 Phred threshold. First run FastQC on each raw read file per read group, generating separate forward/reverse reports for paired-end inputs under results/<sample>/{dnaseq,rnaseq}/qualitycontrol/. Then trim reads with fastp using 3'-end sliding-window trimming (window size 3, analogous to fastp's sliding-window quality cutting), minimum-length filtering (minlen=10), and the global base-quality cutoff, writing gzipped trimmed reads to results/<sample>/{dnaseq,rnaseq}/reads/ as <group>_preproc.fq.gz (single-end) or <group>_preproc_r1.fq.gz/_r2.fq.gz (paired-end). The rationale is that quality filtering and adapter removal upstream stabilizes the reads fed to HLA read filtering (yara/Optitype/HLA-HD) and variant calling downstream, and Snakemake rule granularity ensures each read group is validated independently.

## Related tools

- **Snakemake** (Workflow engine implementing the pre-processing rules per read group with config-driven parameters)
- **fastp** (Performs quality filtering, adapter removal, 3'-end sliding-window trimming and minlen filtering, writing the gzipped pre-processed FASTQs) — https://github.com/OpenGene/fastp
- **FastQC** (Generates per-read-group quality reports on the raw FASTQ inputs before trimming)

## Examples

```
snakemake --cores 8 --use-conda results/<sample>/reads/<group>_preproc.fq.gz
```

## Evaluation signals

- Trimmed gzipped FASTQ files exist for every read group in the sample sheet, following the naming convention <group>_preproc.fq.gz (SE) or <group>_preproc_r1.fq.gz / <group>_preproc_r2.fq.gz (PE)
- FastQC reports exist for each raw read file, with separate forward/reverse reports for paired-end inputs, under results/<sample>/{dnaseq,rnaseq}/qualitycontrol/
- fastp output paths match results/<sample>/{dnaseq,rnaseq}/reads/ for the correct seqtype directory (dnaseq vs rnaseq)
- Configured parameters were honored: minlen=10 minimum length, wsize=3 sliding-window size, basequal=20 Phred cutoff
- Downstream rules resolve: trimmed reads feed yara/Optitype/HLA-HD HLA genotyping without missing-input errors in the Snakemake DAG

## Limitations

- No specific Snakemake version was pinned in the article text; the ScanNeo2 README badge indicates snakemake ≥9.0.0 for current releases
- fastp as applied here is intended for short reads (Illumina NovaSeq, MGI); long-read platforms are out of scope
- The reported text does not state the article's own license, and some settings (basequal=20) come from the workflow configuration rather than the paper prose

## Evidence

- [results] the sequencing reads can be pre-processed, which includes quality filtering or removal of adapter sequences. For that, fastp (Chen et al. 2018) is applied: "the sequencing reads can be pre-processed, which includes quality filtering or removal of adapter sequences. For that, fastp (Chen et al. 2018) is applied"
- [methods] Other parameters include the minimum length of the sequencing reads (`minlen`): "Other parameters include the minimum length of the sequencing reads (`minlen`)"
- [methods] that can be combined with a window trimming from the 3'-end with a defined window size (`wsize`): "that can be combined with a window trimming from the 3'-end with a defined window size (`wsize`)"
- [readme] cut low quality bases for per read in its 5' and 3' by evaluating the mean quality from a sliding window (like Trimmomatic but faster).: [evidence span withheld — anchor status: external_doc]
- [discussion] It has been implemented in snakemake to ensure ease of installation, usage as well as high portability.: "It has been implemented in snakemake to ensure ease of installation, usage as well as high portability."
