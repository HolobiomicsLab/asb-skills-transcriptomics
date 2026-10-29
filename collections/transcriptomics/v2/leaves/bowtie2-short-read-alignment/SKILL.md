---
name: bowtie2-short-read-alignment
description: Use when adapter-trimmed Ribo-seq FastQ reads (e.g. the 25-35 nt RPF
  fraction) may still contain contaminant RNA (rRNA, tRNA, viral RNA) and must be
  filtered against an unwanted-sequence reference before genome/transcriptome mapping.
when_to_use_negative:
- The unwanted-RNA fasta (fasta_outRNA) is empty in config.yaml — RiboDoc skips the
  depletion step entirely
- Splice-aware genome alignment is the goal on its own — RiboDoc pairs Bowtie2 with
  HISAT2 for the genome/transcriptome alignment step since Bowtie2 is not splice-aware
- Input is not short reads or not trimmed RPF fragments (e.g. long reads, or raw FastQ
  still containing adapters)
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_3308
tools:
- name: Bowtie 2
  role: Aligns trimmed reads against the unwanted-RNA index; reads failing to align
    are kept as depleted reads
  repo: https://bowtie-bio.sourceforge.net/bowtie2/index.shtml
- name: Cutadapt
  role: Upstream adapter/quality trimming and read-length filtering producing the
    input FastQ
  repo: https://cutadapt.readthedocs.io/en/stable/
- name: FastQC
  role: Quality control of the depleted reads after the Bowtie2 filtering step
  repo: https://www.bioinformatics.babraham.ac.uk/projects/fastqc/
- name: HISAT2
  role: Co-aligner with Bowtie2 for the subsequent genome and transcriptome alignment
    of depleted reads
  repo: http://daehwankimlab.github.io/hisat2/
- name: RiboDoc
  role: Docker/Snakemake pipeline that orchestrates the depletion step and its QC
  repo: https://github.com/equipeGST/RiboDoc
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.csbj.2021.05.014
    title: 'RiboDoc: A Docker-based package for ribosome profiling analysis'
  source_content_hash: sha256:c6010ecfbd869f5e19b510b9f87c86bb902c2281caf7e4630ec04f77dcc80f99
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
  - HISAT2
  - RiboDoc
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# bowtie2-short-read-alignment

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Align short sequencing reads with Bowtie2, as used in the RiboDoc Ribo-seq pipeline to deplete contaminant RNA (rRNA, tRNA, viral RNA) from adapter-trimmed, RPF-length-selected reads before the double-mapping (HISAT2 + Bowtie2) step.

## When to use

Use when adapter-trimmed Ribo-seq FastQ reads (e.g. the 25-35 nt RPF fraction) may still contain contaminant RNA (rRNA, tRNA, viral RNA) and must be filtered against an unwanted-sequence reference before genome/transcriptome mapping.

## When NOT to use

- The unwanted-RNA fasta (fasta_outRNA) is empty in config.yaml — RiboDoc skips the depletion step entirely
- Splice-aware genome alignment is the goal on its own — RiboDoc pairs Bowtie2 with HISAT2 for the genome/transcriptome alignment step since Bowtie2 is not splice-aware
- Input is not short reads or not trimmed RPF fragments (e.g. long reads, or raw FastQ still containing adapters)

## Inputs

- Trimmed, RPF-length-selected FastQ files (Cutadapt output, e.g. 25-35 nt fraction), gzip-compressed
- Unwanted-sequence reference fasta (fasta_outRNA), e.g. rRNA sequences from Silva, Ensembl, or NCBI
- Bowtie2 index built from the unwanted-sequence fasta

## Outputs

- Per-sample depleted FastQ files containing only reads with no alignment to the unwanted sequences
- FastQC quality-control report on the depleted reads
- Read counts at the depletion step, feeding the RiboDoc analysis report and table summary

## How to apply

Build a Bowtie2 index from the user-provided unwanted-RNA fasta (fasta_outRNA), e.g. organism rRNA sequences downloaded from Silva, Ensembl, or NCBI. Align the Cutadapt-trimmed, RPF-length-selected reads (default 25-35 nt) against this index with Bowtie2 under default alignment parameters, then retain only reads that failed to align, writing per-sample depleted FastQ files. Leave fasta_outRNA empty in config.yaml to skip the depletion step entirely. Run FastQC on the depleted reads and record read numbers at this step so they feed the RiboDoc analysis report and summary table. Depleted reads then proceed to joint genome and transcriptome alignment with HISAT2 and Bowtie2.

## Related tools

- **Bowtie 2** (Aligns trimmed reads against the unwanted-RNA index; reads failing to align are kept as depleted reads) — https://bowtie-bio.sourceforge.net/bowtie2/index.shtml
- **Cutadapt** (Upstream adapter/quality trimming and read-length filtering producing the input FastQ) — https://cutadapt.readthedocs.io/en/stable/
- **FastQC** (Quality control of the depleted reads after the Bowtie2 filtering step) — https://www.bioinformatics.babraham.ac.uk/projects/fastqc/
- **HISAT2** (Co-aligner with Bowtie2 for the subsequent genome and transcriptome alignment of depleted reads) — http://daehwankimlab.github.io/hisat2/
- **RiboDoc** (Docker/Snakemake pipeline that orchestrates the depletion step and its QC) — https://github.com/equipeGST/RiboDoc

## Examples

```
docker pull equipegst/ribodoc
```

## Evaluation signals

- Depleted FastQ files exist for each sample and contain only reads with no alignment to the unwanted-RNA index
- FastQC was run on the depleted reads as the pipeline's post-depletion QC
- Read numbers recorded at the depletion step appear in the analysis report and table summary (raw reads → filtered reads → multimapping → unique map chain continues)
- Setting fasta_outRNA empty in config.yaml causes the step to be skipped, confirming the config wiring
- Downstream HISAT2/Bowtie2 alignment consumes the depleted FastQ files, not the trimmed pre-depletion files

## Limitations

- Bowtie2 is run with default alignment parameters; no custom tuning of scoring or seeding is documented in the pipeline
- Depletion completeness depends on the user-supplied unwanted-sequence fasta (e.g. Silva rRNA); incomplete references leave residual contaminant reads
- Bowtie2 is not splice-aware, so RiboDoc uses it jointly with HISAT2 for the genome/transcriptome alignment step
- No version number is stated for RiboDoc or its Docker package, so exact Bowtie2 version inside the container must be checked

## Evidence

- [methods] Removal of contaminants RNA (rRNA, tRNA, viral RNA, ...) with [`Bowtie2`]: "Removal of contaminants RNA (rRNA, tRNA, viral RNA, ...) with [`Bowtie2`]"
- [methods] You can generate the fasta files associated to the orgnanism rRNAs from the [Silva](https://www.arb-silva.de) database for example.: "You can generate the fasta files associated to the orgnanism rRNAs from the [Silva](https://www.arb-silva.de) database for example."
- [readme] Genome and transcriptome alignment of reads conjointly with [`Hisat2`](http://daehwankimlab.github.io/hisat2/) and [`Bowtie2`](https://bowtie-bio.sourceforge.net/bowtie2/index.shtml): "Genome and transcriptome alignment of reads conjointly with [`Hisat2`](http://daehwankimlab.github.io/hisat2/) and [`Bowtie2`](https://bowtie-bio.sourceforge.net/bowtie2/index.shtml)"
- [readme] Quality Control of depleted reads with [`FastQC`](https://www.bioinformatics.babraham.ac.uk/projects/fastqc/): "Quality Control of depleted reads with [`FastQC`](https://www.bioinformatics.babraham.ac.uk/projects/fastqc/)"
- [methods] Default values select reads from 25 to 35 nucleotides long as RPF are usually around 30 nucleotides long: "Default values select reads from 25 to 35 nucleotides long as RPF are usually around 30 nucleotides long"
