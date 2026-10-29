---
name: contaminant-rna-sequence-filtering
description: Use when processing Ribo-seq FastQ reads after Cutadapt trimming/length
  filtering and before genome alignment, to remove reads derived from rRNA, tRNA or
  other contaminant sequences prior to double-mapping.
when_to_use_negative:
- When fasta_outRNA is left empty in config.yaml — RiboDoc skips this step and passes
  trimmed reads straight to mapping
- When the input is raw, untrimmed FastQ — trimming and RPF length selection (25-35
  nt) must precede depletion
- When you need multimapper handling or genome alignment — that is the separate downstream
  HISAT2/Bowtie2 double-mapping step, not contaminant depletion
edam_operation: http://edamontology.org/operation_3187
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0632
tools:
- name: Bowtie 2
  role: Aligns trimmed reads against the contaminant-sequence index under default
    parameters; unaligned reads are retained
  repo: https://bowtie-bio.sourceforge.net/bowtie2/index.shtml
- name: Cutadapt
  role: Upstream trimming and read-length filtering that produces the reads fed into
    depletion
  repo: https://cutadapt.readthedocs.io/en/stable/
- name: FastQC
  role: Quality control of the depleted reads and read-number recording at this step
  repo: https://www.bioinformatics.babraham.ac.uk/projects/fastqc/
- name: RiboDoc
  role: Docker/Snakemake pipeline orchestrating the depletion step and its config.yaml
    (fasta_outRNA) skip logic
  repo: https://github.com/equipeGST/RiboDoc
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.csbj.2021.05.014
    title: 'RiboDoc: A Docker-based package for ribosome profiling analysis'
  source_content_hash: sha256:984278bf9f105db99d0044b174619ac53dcc9afc1801a55393eca3910e422241
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
  edam_operation: http://edamontology.org/operation_3187
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_0632
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# contaminant-rna-sequence-filtering

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Depletes unwanted RNA contaminants (rRNA, tRNA, viral RNA) from adapter- and quality-trimmed, RPF-length-selected reads by aligning them with Bowtie2 to a user-provided contaminant index and keeping only unaligned reads, as implemented in the RiboDoc ribosome profiling pipeline.

## When to use

Use when processing Ribo-seq FastQ reads after Cutadapt trimming/length filtering and before genome alignment, to remove reads derived from rRNA, tRNA or other contaminant sequences prior to double-mapping.

## When NOT to use

- When fasta_outRNA is left empty in config.yaml — RiboDoc skips this step and passes trimmed reads straight to mapping
- When the input is raw, untrimmed FastQ — trimming and RPF length selection (25-35 nt) must precede depletion
- When you need multimapper handling or genome alignment — that is the separate downstream HISAT2/Bowtie2 double-mapping step, not contaminant depletion

## Inputs

- Cutadapt-trimmed, RPF-length-filtered per-sample FastQ files (gzip-compressed, e.g. 25-35 nt fraction) in the project `fastq` folder
- User-provided contaminant sequences fasta (fasta_outRNA), e.g. organism rRNA sequences from Silva, Ensembl or NCBI
- RiboDoc config.yaml specifying the unwanted-RNA fasta path

## Outputs

- Per-sample contaminant-depleted FastQ files containing only reads with no alignment to unwanted sequences
- FastQC quality-control reports on the depleted reads
- Read-number counts at the depletion step feeding the RiboDoc report and table summary

## How to apply

Build a Bowtie2 index from the user-provided contaminant fasta (fasta_outRNA in config.yaml, e.g. organism rRNA sequences downloaded from the Silva, Ensembl or NCBI databases). Align the trimmed, RPF-length-selected reads (e.g. the Cutadapt 25-35 nt fraction) against this contaminant index with Bowtie2 under default alignment parameters. Retain only reads that fail to align to the unwanted sequences and write per-sample depleted FastQ files for the subsequent HISAT2/Bowtie2 genome and transcriptome alignment. Setting fasta_outRNA empty in config.yaml skips the step entirely. Run FastQC on the depleted reads and record read numbers at this stage so they feed the RiboDoc analysis report and table summary.

## Related tools

- **Bowtie 2** (Aligns trimmed reads against the contaminant-sequence index under default parameters; unaligned reads are retained) — https://bowtie-bio.sourceforge.net/bowtie2/index.shtml
- **Cutadapt** (Upstream trimming and read-length filtering that produces the reads fed into depletion) — https://cutadapt.readthedocs.io/en/stable/
- **FastQC** (Quality control of the depleted reads and read-number recording at this step) — https://www.bioinformatics.babraham.ac.uk/projects/fastqc/
- **RiboDoc** (Docker/Snakemake pipeline orchestrating the depletion step and its config.yaml (fasta_outRNA) skip logic) — https://github.com/equipeGST/RiboDoc

## Examples

```
bowtie2 -x rRNA_index -U trimmed_reads.fastq.gz --un depleted_reads.fastq.gz
```

## Evaluation signals

- Per-sample depleted FastQ files exist and contain only reads with no Bowtie2 alignment to the unwanted-RNA index
- Read counts at the depletion step are recorded and appear in the RiboDoc analysis report and table summary, showing the expected drop relative to the trimmed-read counts
- FastQC was run on the depleted reads (step 5 of the pipeline) and reports were generated
- Providing an empty fasta_outRNA in config.yaml verifiably skips the step, confirming the configuration toggle works
- Depleted reads successfully feed the subsequent conjoint HISAT2/Bowtie2 genome and transcriptome alignment

## Limitations

- Only reads matching the user-supplied contaminant fasta are removed — contaminants absent from the index (e.g. rRNA from Silva/Ensembl not included) will pass through
- Bowtie2 runs with default alignment parameters; stringency is not tuned per dataset in the documented workflow
- The step requires the user to correctly source contaminant sequences (Silva, Ensembl, NCBI); wrong or incomplete rRNA fasta leads to incomplete depletion
- Dataset must align enough material for downstream steps — the README warns the GEO GSE173856 example does not align enough material for riboWaltz

## Evidence
