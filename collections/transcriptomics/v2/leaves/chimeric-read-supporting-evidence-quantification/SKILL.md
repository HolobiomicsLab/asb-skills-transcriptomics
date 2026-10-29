---
name: chimeric-read-supporting-evidence-quantification
description: Use when benchmarking fusion-gene detectors on simulated paired-end FASTQ
  inputs and you need per-tool counts of spanning reads, split reads, and lowest coverage
  at which BCR-ABL1, CIC-DUX4, or DUX4-IGH is detected.
when_to_use_negative:
- The data are single-end RNA-seq, whole-genome, or whole-exome sequencing reads —
  Anchored-fusion only works on paired-end RNA-seq
- You need a de novo catalog of fusions not involving a pre-specified target gene
  — the anchored search only reports fusions containing the anchored gene
- Reads are already aligned and only downstream QC is needed — this skill requires
  paired-end FASTQ inputs through the BWA/BLAT alignment chain
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2640
tools:
- name: Anchored-fusion
  role: Primary anchored search pipeline that collects spanning and split reads supporting
    fusions involving the user-specified target gene
  repo: https://github.com/ShenLab-Genomics/Anchored-Fusion
- name: BWA-MEM
  role: Maps paired-end reads to the anchored-gene reference sequence within the anchoring
    step
  repo: https://github.com/ShenLab-Genomics/Anchored-Fusion
- name: BLAT
  role: Performs local alignment of candidate chimeric reads to predict the precise
    fusion location
- name: samtools
  role: Alignment file handling for the BWA-MEM mapping step
- name: bedtools
  role: Interval operations used in candidate filtering
- name: STAR-Fusion v1.12.0
  role: Competing bulk RNA-seq fusion detector whose spanning/split read support is
    tallied for comparison
  repo: https://github.com/STAR-Fusion
- name: FusionInspector v2.8.0
  role: Competing bulk RNA-seq fusion detector tallied in the benchmark
  repo: https://github.com/FusionInspector/FusionInspector
- name: FusionCatcher v1.30
  role: Competing bulk RNA-seq fusion detector; its detection coverage for DUX4-IGH
    sets the 1/4-fold comparison baseline
  repo: https://github.com/ndaniel/fusioncatcher
- name: Arriba v2.4.0
  role: Competing bulk RNA-seq fusion detector tallied in the benchmark
  repo: https://github.com/suhrig/arriba
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.crmeth.2024.100733
    title: Anchored-fusion enables targeted fusion search in bulk and single-cell
      RNA sequencing data
  source_content_hash: sha256:9ae722a4ee66632e230aa2a78e03814f9cf6148d62648060b0fd068940890097
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
  - Arriba
  - Arriba v2.4.0
  - BLAT
  - BWA
  - BWA-MEM
  - FusionCatcher
  - FusionCatcher v1.30
  - FusionInspector
  - FusionInspector v2.8.0
  - PyTorch
  - STAR-Fusion v1.12.0
  - bedtools
  - samtools
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2640
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5789473684210527
n_claims: 19
---

# chimeric-read-supporting-evidence-quantification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Quantifies spanning reads and split reads as supporting evidence for candidate gene fusions, tallying per-tool read support and minimum detection coverage in the Anchored-fusion simulated-data benchmark against STAR-Fusion, FusionInspector, FusionCatcher, and Arriba.

## When to use

Use when benchmarking fusion-gene detectors on simulated paired-end FASTQ inputs and you need per-tool counts of spanning reads, split reads, and lowest coverage at which BCR-ABL1, CIC-DUX4, or DUX4-IGH is detected.

## When NOT to use

- The data are single-end RNA-seq, whole-genome, or whole-exome sequencing reads — Anchored-fusion only works on paired-end RNA-seq
- You need a de novo catalog of fusions not involving a pre-specified target gene — the anchored search only reports fusions containing the anchored gene
- Reads are already aligned and only downstream QC is needed — this skill requires paired-end FASTQ inputs through the BWA/BLAT alignment chain

## Inputs

- Simulated paired-end FASTQ files of fusion transcripts (e.g., test_sample_1.fastq.gz)
- Target gene transcript FASTA (from NCBI Gene), genome FASTA (hg38 or hg19), and compatible gene annotation GTF
- Fusion detector outputs from STAR-Fusion v1.12.0, FusionInspector v2.8.0, FusionCatcher v1.30, and Arriba v2.4.0 on the same inputs

## Outputs

- Per-tool counts of spanning reads and split reads supporting each simulated fusion event
- Minimum read coverage at which each tool detects BCR-ABL1, CIC-DUX4, and DUX4-IGH
- Compiled table simulated_fusion_benchmark_results.csv

## How to apply

Anchor a gene of interest and, per the Anchored-fusion workflow, map paired-end reads to the target-gene transcript reference with BWA-MEM (>=0.7.17) and samtools (>=1.16), align the unmapped mate of half-mapped pairs genome-wide, then use BLAT (>=v.35) to locally align candidate chimeric reads to predict the precise fusion location. Tally paired-end reads spanning the fusion site (spanning reads) and single reads containing the fusion site (split reads) as the support for each candidate. Run the competing bulk RNA-seq detectors STAR-Fusion v1.12.0, FusionInspector v2.8.0, FusionCatcher v1.30, and Arriba v2.4.0 on the same simulated paired-end FASTQ inputs, and count each tool's spanning/split reads for the simulated events. Determine the minimum read coverage at which each tool detects each fusion; validation for the paper's Figure 3 benchmark requires Anchored-fusion to detect BCR-ABL1 and CIC-DUX4 at the lowest coverage and DUX4-IGH at one-fourth of FusionCatcher's coverage limit, compiled into simulated_fusion_benchmark_results.csv.

## Related tools

- **Anchored-fusion** (Primary anchored search pipeline that collects spanning and split reads supporting fusions involving the user-specified target gene) — https://github.com/ShenLab-Genomics/Anchored-Fusion
- **BWA-MEM** (Maps paired-end reads to the anchored-gene reference sequence within the anchoring step) — https://github.com/ShenLab-Genomics/Anchored-Fusion
- **BLAT** (Performs local alignment of candidate chimeric reads to predict the precise fusion location)
- **samtools** (Alignment file handling for the BWA-MEM mapping step)
- **bedtools** (Interval operations used in candidate filtering)
- **STAR-Fusion v1.12.0** (Competing bulk RNA-seq fusion detector whose spanning/split read support is tallied for comparison) — https://github.com/STAR-Fusion
- **FusionInspector v2.8.0** (Competing bulk RNA-seq fusion detector tallied in the benchmark) — https://github.com/FusionInspector/FusionInspector
- **FusionCatcher v1.30** (Competing bulk RNA-seq fusion detector; its detection coverage for DUX4-IGH sets the 1/4-fold comparison baseline) — https://github.com/ndaniel/fusioncatcher
- **Arriba v2.4.0** (Competing bulk RNA-seq fusion detector tallied in the benchmark) — https://github.com/suhrig/arriba

## Examples

```
python Anchored_Fusion.py --file_anchored_cds=target_gene.fasta --fastq1=test_sample_1.fastq.gz --fastq2=test_sample_1.fastq.gz --out_folder=output_dir --file_ref_seq=reference.genome.fa --file_ref_ann=genome.annotation.gtf
```

## Evaluation signals

- Anchored-fusion detects BCR-ABL1 and CIC-DUX4 at the lowest coverage among all benchmarked tools (highest sensitivity, Figure 3A and 3B)
- Anchored-fusion's DUX4-IGH detection limit equals exactly one-fourth of FusionCatcher's coverage limit
- Every reported fusion event is supported by both spanning reads and split reads tallied per tool
- The compiled simulated_fusion_benchmark_results.csv matches the per-tool counts and coverage limits reported in the paper
- Fusion predictions from Anchored-fusion contain the anchored target gene; other fusions are absent from the output

## Limitations

- Fusion genes not including the user-provided target gene cannot be detected
- Anchored-fusion only works on paired-end RNA-seq data and cannot be applied to single-end RNA-seq, whole-genome, or whole-exome sequencing data
- Users must supply the reference sequence of their target gene
- If the input dataset is too small for HVLD retraining, the default model parameters from GitHub should be used instead of a retrained filter

## Evidence

- [results] Anchored-fusion best detection capability: [evidence span withheld — anchor status: not_found]
