---
name: ribosome-profiling-read-depletion
description: Use when a Ribo-seq pipeline needs contaminant RNA removed from Cutadapt-trimmed,
  25–35 nt RPF-length-selected FastQ reads before the double-mapping step with Bowtie2/HISAT2.
when_to_use_negative:
- The fasta_outRNA entry in config.yaml is empty — RiboDoc skips the depletion step
  and reads pass through unchanged
- The reads have not yet been adapter-trimmed and RPF-length-filtered by Cutadapt
  (e.g. raw reads straight from the sequencer)
- You need multimapping/uniquely-mapped read filtering — that happens at the later
  Bowtie2/HISAT2 double-mapping step, not during contaminant depletion
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0659
tools:
- name: Bowtie 2
  role: Aligns trimmed, RPF-length-selected reads against the unwanted-RNA index;
    unaligned reads are retained as depleted output
  repo: https://bowtie-bio.sourceforge.net/bowtie2/index.shtml
- name: FastQC
  role: Quality control of the depleted reads after contaminant removal
- name: Cutadapt
  role: Produces the adapter-trimmed, read-length-filtered input (25–35 nt RPF selection)
    that feeds the depletion step
  repo: https://cutadapt.readthedocs.io/en/stable/
- name: RiboDoc
  role: Pipeline wrapping the depletion step; configured via config.yaml (fasta_outRNA)
    inside a Docker container
  repo: https://github.com/equipeGST/RiboDoc
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.csbj.2021.05.014
    title: 'RiboDoc: A Docker-based package for ribosome profiling analysis'
  source_content_hash: sha256:8666ce4bf4831f00d299b3136dbfb2c70152983fe2e09da6505c3bd2935776b1
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/equipeGST/RiboDoc
  tools:
  - Bowtie 2
  - Cutadapt
  - FastQC
  - RiboDoc
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0659
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# ribosome-profiling-read-depletion

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Depletes unwanted RNA contaminants (rRNA, tRNA, viral RNA) from adapter-trimmed, RPF-length-selected reads by aligning them with Bowtie2 against a user-provided contaminant index and retaining only non-aligning reads, so downstream genome/transcriptome mapping reflects ribosome-protected fragments rather than abundant contaminant RNA.

## When to use

Use when a Ribo-seq pipeline needs contaminant RNA removed from Cutadapt-trimmed, 25–35 nt RPF-length-selected FastQ reads before the double-mapping step with Bowtie2/HISAT2.

## When NOT to use

- The fasta_outRNA entry in config.yaml is empty — RiboDoc skips the depletion step and reads pass through unchanged
- The reads have not yet been adapter-trimmed and RPF-length-filtered by Cutadapt (e.g. raw reads straight from the sequencer)
- You need multimapping/uniquely-mapped read filtering — that happens at the later Bowtie2/HISAT2 double-mapping step, not during contaminant depletion

## Inputs

- Cutadapt-trimmed, RPF-length-selected FastQ files (e.g. 25–35 nt fraction) in the project's fastq folder
- User-provided unwanted-RNA sequences fasta (fasta_outRNA, e.g. organism rRNA from Silva, Ensembl, or NCBI)

## Outputs

- Per-sample depleted FastQ files containing only reads with no alignment to the unwanted sequences
- FastQC quality-control report of depleted reads
- Read counts at the depletion step for the analysis report and table summary

## How to apply

Build a Bowtie2 index from a user-supplied unwanted-RNA fasta (fasta_outRNA), e.g. organism rRNA sequences downloaded from Silva, Ensembl, or NCBI. Align the trimmed, RPF-length-selected reads (e.g. the 25–35 nt fraction selected by Cutadapt with default RiboDoc settings) against this index with Bowtie2 under default alignment parameters. Keep only reads with no alignment to the unwanted sequences and write per-sample depleted FastQ files for the subsequent Bowtie2/HISAT2 genome and transcriptome alignment. Run FastQC on the depleted reads and record read numbers at this step so they feed the analysis report and table summary. Leaving fasta_outRNA empty in config.yaml skips the depletion step entirely.

## Related tools

- **Bowtie 2** (Aligns trimmed, RPF-length-selected reads against the unwanted-RNA index; unaligned reads are retained as depleted output) — https://bowtie-bio.sourceforge.net/bowtie2/index.shtml
- **FastQC** (Quality control of the depleted reads after contaminant removal)
- **Cutadapt** (Produces the adapter-trimmed, read-length-filtered input (25–35 nt RPF selection) that feeds the depletion step) — https://cutadapt.readthedocs.io/en/stable/
- **RiboDoc** (Pipeline wrapping the depletion step; configured via config.yaml (fasta_outRNA) inside a Docker container) — https://github.com/equipeGST/RiboDoc

## Examples

```
bowtie2 -x fasta_outRNA_index -U trimmed_reads.fastq.gz --un depleted_reads.fastq.gz
```

## Evaluation signals

- Depleted FastQ files are produced per sample, and their read counts drop relative to the trimmed-read counts by roughly the contaminant fraction
- FastQC is successfully run on the depleted reads (third QC checkpoint after raw and trimmed reads)
- Read numbers at the depletion step appear in the final analysis report and table summary
- Only reads with no Bowtie2 alignment to the unwanted sequences are present in the output — no residual rRNA-matching reads
- Setting fasta_outRNA empty in config.yaml cleanly skips the step without breaking downstream alignment steps

## Limitations

- Depletion is only as complete as the contaminant fasta: incomplete rRNA/tRNA reference sequences (e.g. partial Silva or Ensembl downloads) leave residual contaminants
- No version number is stated for RiboDoc or the Docker package, so reproducing the exact depletion behavior across releases may be difficult
- Bowtie2 runs under default alignment parameters; non-default contaminant-alignment stringency is not configurable through the described workflow

## Evidence
