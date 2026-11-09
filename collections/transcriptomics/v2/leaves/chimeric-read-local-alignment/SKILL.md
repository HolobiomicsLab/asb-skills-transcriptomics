---
name: chimeric-read-local-alignment
description: Use when running Anchored-fusion on paired-end RNA-seq (FASTQ) data and
  you have candidate chimeric reads whose mates fail to map to the anchored target
  gene, requiring precise fusion-partner localization and breakpoint resolution.
when_to_use_negative:
- The sequencing data is single-end RNA-seq, whole-genome, or whole-exome data — Anchored-fusion
  requires paired-end RNA-seq
- You need to detect fusions that do not include the user-provided target/anchored
  gene — they will not be reported
- You only need genome-wide de novo fusion discovery without a gene of interest —
  use STAR-Fusion, FusionCatcher, or Arriba instead
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0080
tools:
- name: Anchored-fusion
  role: Pipeline that anchors the gene of interest, selects chimeric read pairs, and
    invokes BLAT for local alignment of candidate chimeric reads
  repo: https://github.com/ShenLab-Genomics/Anchored-Fusion
- name: BWA-MEM
  role: Aligner used to map reads to the anchored gene reference and the unmapped
    mates to the whole genome before BLAT refinement
- name: BLAT
  role: Performs local alignment on candidate chimeric reads to accurately predict
    the precise location of gene fusion
provenance:
  source_task_ids:
  - task_006
  - task_005
  - task_007
  source_papers:
  - doi: 10.1016/j.crmeth.2024.100733
    title: Anchored-fusion enables targeted fusion search in bulk and single-cell
      RNA sequencing data
  source_content_hash: sha256:e2cc3d83bc94b98146630ebb5150bb05f986211a4c78d2c896ae30bef820e54f
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
  - BLAT
  - BWA-MEM
  - FusionCatcher
  - FusionInspector
  - samtools
  - scFusion
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# chimeric-read-local-alignment

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Local-alignment refinement step of Anchored-fusion in which candidate chimeric read pairs (one mate mapped to the anchored gene, the unmapped mate aligned genome-wide by BWA-MEM) are realigned with BLAT to pinpoint the precise fusion breakpoint. It supports fusion calls using spanning and split reads as evidence.

## When to use

Use when running Anchored-fusion on paired-end RNA-seq (FASTQ) data and you have candidate chimeric reads whose mates fail to map to the anchored target gene, requiring precise fusion-partner localization and breakpoint resolution.

## When NOT to use

- The sequencing data is single-end RNA-seq, whole-genome, or whole-exome data — Anchored-fusion requires paired-end RNA-seq
- You need to detect fusions that do not include the user-provided target/anchored gene — they will not be reported
- You only need genome-wide de novo fusion discovery without a gene of interest — use STAR-Fusion, FusionCatcher, or Arriba instead

## Inputs

- Paired-end RNA-seq FASTQ files (--fastq1/--fastq2, e.g. Smart-seq2 scRNA-seq or CCLE bRNA-seq)
- Target gene transcript FASTA (--file_anchored_cds, e.g. BCR or NSD2 from NCBI Gene)
- Genome FASTA (--file_ref_seq, hg38 or hg19 from GENCODE/Ensembl)
- Gene annotation GTF (--file_ref_ann, version-compatible with the genome FASTA)
- BLAT >= v.35 installation (plus bwa >= 0.7.17, samtools >= 1.16, bedtools >= 2.30.0)

## Outputs

- Fusion gene calls including the anchored target gene, with fusion partner genes and precise fusion locations/breakpoints (e.g. chr4:1,902,353 and chr4:1,905,943 for NSD2-IGH)
- Per-fusion supporting evidence counts (spanning reads and split reads)
- Per-cell fusion-positive/negative calls when run across single cells

## How to apply

First map all paired-end reads to the target-gene transcript FASTA (e.g. BCR or NSD2 CDS from NCBI Gene) with BWA-MEM and collect reads aligned to the anchored gene. Select pairs where only one mate maps to the anchored gene and align the remaining unmapped mate against the whole hg38 genome (FASTA + compatible GTF from GENCODE/Ensembl). Then run BLAT local alignment on the candidate chimeric reads to accurately predict the precise location of the gene fusion, keeping paired-end reads spanning the fusion site (spanning reads) and single reads containing the fusion site (split reads) as supporting evidence. Candidates with uncertain fusion positions, or with significantly imbalanced spanning/split read counts, are filtered out before or after the HVLD deep-learning filter (probability threshold 0.1).

## Related tools

- **Anchored-fusion** (Pipeline that anchors the gene of interest, selects chimeric read pairs, and invokes BLAT for local alignment of candidate chimeric reads) — https://github.com/ShenLab-Genomics/Anchored-Fusion
- **BWA-MEM** (Aligner used to map reads to the anchored gene reference and the unmapped mates to the whole genome before BLAT refinement)
- **BLAT** (Performs local alignment on candidate chimeric reads to accurately predict the precise location of gene fusion)

## Examples

```
python Anchored_Fusion.py --file_anchored_cds=target_gene.fasta --fastq1=sample_1.fastq.gz --fastq2=sample_2.fastq.gz --out_folder=output_dir --file_ref_seq=reference.genome.fa --file_ref_ann=genome.annotation.gtf
```

## Evaluation signals

- Known-positive benchmark recovered: 35 BCR-ABL1-positive cells among 350 K562 Smart-seq2 cells (SRA SRP291312), 83.3% recovery of the 42 evidence-bearing cells, and 10/15 very-low-evidence cells
- MM cohort benchmark: 99 NSD2-IGH-positive single cells across the three t(4;14) patients (RRMM1, RRMM2, SMM0), 92.5% of all positive cells, with breakpoints at chr4:1,902,353 and chr4:1,905,943
- Detected fusions include the anchored target gene and have resolvable (non-uncertain) fusion positions with balanced spanning/split read support
- BLAT step runs on candidates defined as read pairs with only one mate mapped to the anchored gene — verify no all-mapped or all-unmapped pairs enter local alignment

## Limitations

- Fusion genes that do not include the target gene will not be detected
- Only works on paired-end RNA-seq data; cannot be used for single-end RNA-seq, whole-genome, or whole-exome sequencing data
- Requires users to supply the reference sequence of their target gene
- Candidate fusions with uncertain fusion positions, or significantly imbalanced spanning/split read numbers, are filtered out and may drop true events

## Evidence
