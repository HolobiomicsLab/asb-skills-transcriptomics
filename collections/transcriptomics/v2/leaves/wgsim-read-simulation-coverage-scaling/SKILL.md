---
name: wgsim-read-simulation-coverage-scaling
description: Use when constructing synthetic benchmark RNA-seq datasets with known
  fusion transcripts at graded expression/coverage levels, or when evaluating fusion-detection
  sensitivity across low-to-high sequencing depth.
when_to_use_negative:
- When evaluating de novo fusion discovery rather than targeted (anchored) detection
  — the simulated fusions are user-specified partner pairs
- When the input is already real tumor RNA-seq with validated fusions (e.g., CCLE/SRA:SRP186687
  benchmarks) — simulation is unnecessary and may not capture real background complexity
- For single-end RNA-seq or whole-genome/exome data, since the downstream detection
  requires paired-end RNA-seq
edam_operation: http://edamontology.org/operation_3580
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3512
tools:
- name: wgsim
  role: Simulates paired-end reads from each fusion transcript reference at seven
    doubling coverage levels (2x to 128x)
- name: Anchored-fusion
  role: Downstream targeted fusion detector benchmarked on the simulated test datasets
    via --fastq1/--fastq2 inputs
  repo: https://github.com/ShenLab-Genomics/Anchored-Fusion
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.crmeth.2024.100733
    title: Anchored-fusion enables targeted fusion search in bulk and single-cell
      RNA sequencing data
  source_content_hash: sha256:5104e6f3c8c24a07058148d1c822eb49de6467faae705d888e0856759278c535
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/ShenLab-Genomics/Anchored-Fusion
  repo_url: https://github.com/ShenLab-Genomics/Anchored-Fusion
  tools:
  - Anchored-fusion
  - wgsim
  edam_operation: http://edamontology.org/operation_3580
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3512
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# wgsim-read-simulation-coverage-scaling

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Simulate paired-end reads from constructed fusion-transcript references using wgsim across a doubling coverage ladder (2x to 128x), then spike the simulated fusion reads into real GM12878 RNA-seq background to build benchmark test datasets for targeted fusion detection (Anchored-fusion).

## When to use

Use when constructing synthetic benchmark RNA-seq datasets with known fusion transcripts at graded expression/coverage levels, or when evaluating fusion-detection sensitivity across low-to-high sequencing depth.

## When NOT to use

- When evaluating de novo fusion discovery rather than targeted (anchored) detection — the simulated fusions are user-specified partner pairs
- When the input is already real tumor RNA-seq with validated fusions (e.g., CCLE/SRA:SRP186687 benchmarks) — simulation is unnecessary and may not capture real background complexity
- For single-end RNA-seq or whole-genome/exome data, since the downstream detection requires paired-end RNA-seq

## Inputs

- Fusion partner gene transcript FASTA sequences (twelve genes, from NCBI Gene)
- Six constructed fusion transcript references (partner pairs joined at breakpoint junctions)
- Paired-end RNA-seq background reads from the GM12878 cell line

## Outputs

- Six gene pairs x seven coverage levels = 42 paired gzip-compressed FASTQ test datasets (fastq1/fastq2, R1/R2)
- Simulated fusion reads spanning each breakpoint junction embedded in GM12878 background

## How to apply

First build each fusion transcript reference by downloading the transcript FASTA of both partner genes from NCBI Gene and joining them at the breakpoint junction (e.g., BCR-ABL1, CIC-DUX4, DUX4-IGH, TMPRSS2-ERG, EML4-ALK, TPM3-NTRK1). Run wgsim on each fusion reference to generate paired-end reads at seven coverage levels doubling from 2x up to 128x. Combine each set of simulated fusion reads with paired-end RNA-seq reads from the GM12878 cell line as realistic background. Write the results as paired gzip-compressed FASTQ files (R1/R2) in the format expected by downstream tools such as Anchored_Fusion.py (--fastq1/--fastq2). Rationale: the coverage ladder tests whether a targeted detector can recover fusions from shallow sequencing, which is the stated motivation for Anchored-fusion's design.

## Related tools

- **wgsim** (Simulates paired-end reads from each fusion transcript reference at seven doubling coverage levels (2x to 128x))
- **Anchored-fusion** (Downstream targeted fusion detector benchmarked on the simulated test datasets via --fastq1/--fastq2 inputs) — https://github.com/ShenLab-Genomics/Anchored-Fusion

## Examples

```
wgsim -1 100 -2 100 -N <num_reads> fusion_transcript.fasta simulated_r1.fastq simulated_r2.fastq; gzip simulated_r1.fastq simulated_r2.fastq
```

## Evaluation signals

- All 6 fusion gene pairs (BCR-ABL1, CIC-DUX4, DUX4-IGH, TMPRSS2-ERG, EML4-ALK, TPM3-NTRK1) produced paired fastq.gz test datasets
- Each pair exists at all 7 coverage levels (2x, 4x, 8x, 16x, 32x, 64x, 128x)
- Test datasets contain reads spanning the simulated fusion junction embedded within GM12878 background reads
- Output FASTQ files are gzip-compressed and paired (R1/R2) in the input format expected by Anchored_Fusion.py
- Coverage levels double consistently from a base coverage of 128 down to 2

## Limitations

- Simulated chimeric reads are artificial cDNA fusions and may not represent natural chromosomal fusion artifacts; Anchored-fusion's HVLD model is explicitly designed to distinguish these
- Simulation fidelity depends on compatible reference versions — genome FASTA (from GENCODE or Ensembl) and GTF annotation (hg38 or hg19) must match
- Only fusion pairs assembled into a transcript reference can be simulated; the ladder tests a fixed set of six known gene pairs

## Evidence
