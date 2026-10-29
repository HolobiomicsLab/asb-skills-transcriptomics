---
name: riboseq-pipeline-configuration
description: 'Use when starting a Ribo-seq analysis project from raw FastQ files with
  RiboDoc: building the project folder, naming *.fastq.gz samples, choosing genome/rRNA
  fasta references, and setting config.yaml options (RPF lengths 25-35 nt, p-val,
  logFC, riboWaltz vs TRiP).'
when_to_use_negative:
- The dataset is not ribosome-profiling data or the goal is only P-site offset calculation
  in isolation (use riboWaltz directly rather than configuring the full RiboDoc pipeline)
- Input reads are already trimmed and depleted (alignment-level BAMs) — the configuration
  of upstream FastQ-to-depletion steps would be redundant
- A non-Docker/non-Snakemake RiboSeq pipeline is already in place and only individual
  steps (e.g. rRNA depletion) are needed
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: RiboDoc
  role: The Docker/Snakemake pipeline being configured; orchestrates all steps from
    FastQC QC to DESeq2 and riboWaltz/TRiP
  repo: https://github.com/equipeGST/RiboDoc
- name: Bowtie 2
  role: 'Configured twice: for removal of contaminants RNA (rRNA, tRNA, viral RNA)
    and for genome alignment alongside Hisat2'
- name: Cutadapt
  role: Configured for adapter/quality trimming and read-length filtering (default
    RPF selection 25-35 nt)
- name: riboWaltz
  role: Optional offset-prediction/periodicity backend selected in configuration when
    the dataset aligns enough material and compute allows
  repo: https://github.com/LabTranslationalArchitectomics/riboWaltz
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.csbj.2021.05.014
    title: 'RiboDoc: A Docker-based package for ribosome profiling analysis'
  source_content_hash: sha256:81613417d3533e50deeaa88bf54e51be5008e3b9576f80bb175b213cbf802d73
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/equipeGST/RiboDoc
  repo_url: https://github.com/equipeGST/RiboDoc
  tools:
  - Bowtie 2
  - Cutadapt
  - FastQC
  - RiboDoc
  - riboWaltz
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# riboseq-pipeline-configuration

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Configures the RiboDoc Docker/Singularity Ribo-seq pipeline (built on Snakemake) by setting up the project folder, naming gzipped FastQ samples, supplying genome and unwanted-RNA fasta references, and filling config.yaml so the FastQ-to-DESeq2 workflow runs reproducibly.

## When to use

Use when starting a Ribo-seq analysis project from raw FastQ files with RiboDoc: building the project folder, naming *.fastq.gz samples, choosing genome/rRNA fasta references, and setting config.yaml options (RPF lengths 25-35 nt, p-val, logFC, riboWaltz vs TRiP).

## When NOT to use

- The dataset is not ribosome-profiling data or the goal is only P-site offset calculation in isolation (use riboWaltz directly rather than configuring the full RiboDoc pipeline)
- Input reads are already trimmed and depleted (alignment-level BAMs) — the configuration of upstream FastQ-to-depletion steps would be redundant
- A non-Docker/non-Snakemake RiboSeq pipeline is already in place and only individual steps (e.g. rRNA depletion) are needed

## Inputs

- Gzip-compressed Ribo-seq FastQ files named [CONDITION]_[NAME].[REPLICATE].fastq.gz
- Reference genome fasta (e.g. Ensembl Saccharomyces_cerevisiae.R64-1-1.dna.toplevel.fa.gz)
- Unwanted-RNA fasta (fasta_outRNA, e.g. rRNA sequences from Silva), optional
- config.yaml with pipeline options (RPF lengths, p-val, logFC, offset tool choice)
- Docker or Singularity installation

## Outputs

- Fully configured RiboDoc project directory ready to run the Snakemake workflow
- Pipeline outputs per sample: FastQC QC reports, Cutadapt-trimmed reads, Bowtie2-depleted FastQs, Hisat2/Bowtie2 BAM alignments, htseq-count tables, DESeq2 differential expression results, riboWaltz/TRiP periodicity plots

## How to apply

Create a project folder named after the project (it becomes the Docker volume) containing a lowercase-named `fastq` subfolder holding gzip-compressed FastQ files named `[CONDITION]_[NAME].[REPLICATE].fastq.gz` (e.g. `Wild_Type.56.fastq.gz`), avoiding dashes, parentheses, spaces and all special characters except underscores. Supply reference fasta files: the genome assembly (e.g. `Saccharomyces_cerevisiae.R64-1-1.dna.toplevel.fa.gz` from Ensembl) and an unwanted-RNA fasta (`fasta_outRNA`, e.g. organism rRNAs from Silva or NCBI); leaving `fasta_outRNA` empty in config.yaml skips the Bowtie2 rRNA-depletion step. Set analysis parameters in config.yaml including RPF length selection (default 25-35 nt) and, for differential expression, the `p-val` and `logFC` thresholds used with DESeq2. Choose the offset tool per available compute: `riboWaltz` for well-aligning datasets or `TRiP` when material aligns poorly or computer power is limited. Pull the container (`docker pull equipegst/ribodoc`, or use Singularity on clusters without super-user rights) and launch; RiboDoc can be rerun performing only the steps that need recalculation after sample removal.

## Related tools

- **RiboDoc** (The Docker/Snakemake pipeline being configured; orchestrates all steps from FastQC QC to DESeq2 and riboWaltz/TRiP) — https://github.com/equipeGST/RiboDoc
- **Bowtie 2** (Configured twice: for removal of contaminants RNA (rRNA, tRNA, viral RNA) and for genome alignment alongside Hisat2)
- **Cutadapt** (Configured for adapter/quality trimming and read-length filtering (default RPF selection 25-35 nt))
- **riboWaltz** (Optional offset-prediction/periodicity backend selected in configuration when the dataset aligns enough material and compute allows) — https://github.com/LabTranslationalArchitectomics/riboWaltz

## Examples

```
docker pull equipegst/ribodoc
```

## Evaluation signals

- Project folder contains lowercase-named subfolders and a `fastq` subfolder of gzip-compressed files matching the [CONDITION]_[NAME].[REPLICATE].fastq.gz pattern with no dashes, parentheses, spaces or special characters
- config.yaml points to valid genome and unwanted-RNA fasta files; if `fasta_outRNA` is empty, the Bowtie2 depletion step is intentionally skipped rather than failing
- Pipeline runs the expected ordered steps: FastQC on raw reads, Cutadapt trimming, FastQC on trimmed reads, Bowtie2 depletion, FastQC on depleted reads, Hisat2/Bowtie2 alignment, samtools sort/index, htseq-count, DESeq2, riboWaltz or TRiP
- Configured RPF length filter defaults to 25-35 nt reads, and per-step read numbers are recorded for the analysis report and table summary
- Configuration reproduces on the example GEO dataset GSE173856 (using TRiP rather than riboWaltz for that dataset)

## Limitations

- No version number is stated for RiboDoc or the Docker package, complicating exact reproducibility pins
- riboWaltz cannot be used for datasets that align too little material (e.g. the example dataset GSE173856); TRiP must be selected instead, and the two tools depend on available computer power
- On Windows, file extensions can be hidden, which can silently break the required [CONDITION]_[NAME].[REPLICATE].fastq.gz naming
- Subfolder names must not contain uppercase letters or the analysis will not proceed correctly

## Evidence
