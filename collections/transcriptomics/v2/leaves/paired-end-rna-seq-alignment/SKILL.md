---
name: paired-end-rna-seq-alignment
description: Use when performing a targeted fusion search on paired-end RNA-seq FASTQ
  data by anchoring a gene of interest, and reads must be aligned to both the target
  gene transcript fasta and a whole-genome reference.
when_to_use_negative:
- Input is single-end RNA-seq data, whole-genome sequencing, or whole-exome sequencing
  — Anchored-fusion only supports paired-end RNA-seq
- The fusion of interest does not include the user-provided target (anchored) gene
  — such fusions will not be detected
- An untargeted, de novo fusion discovery across all possible fusion partners is required
  — the anchored search is limited to the user-specified gene
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_3170
tools:
- name: BWA-MEM
  role: 'Primary aligner: maps all paired-end reads to the anchored gene transcript
    and the unmapped mates to the whole genome'
- name: samtools
  role: SAM/BAM processing and alignment handling (version >= 1.16)
- name: BLAT
  role: Local alignment of candidate chimeric reads to pinpoint the precise fusion
    location (version >= v.35)
- name: bedtools
  role: Interval operations during candidate fusion filtering (version >= v2.30.0)
- name: Anchored-fusion
  role: Pipeline driver invoked as Anchored_Fusion.py, orchestrating the anchored
    alignment and downstream HVLD filtering
  repo: https://github.com/ShenLab-Genomics/Anchored-Fusion
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.crmeth.2024.100733
    title: Anchored-fusion enables targeted fusion search in bulk and single-cell
      RNA sequencing data
  source_content_hash: sha256:bf9d83dcfbd442079af6b36286201820b5208df69e5a6c56c0267070612e0f96
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
  - BWA
  - BWA-MEM
  - FusionCatcher
  - FusionInspector
  - PyTorch
  - bedtools
  - samtools
  - scikit-learn
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4
n_claims: 15
---

# paired-end-rna-seq-alignment

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Maps paired-end RNA-seq FASTQ reads to an anchored target gene and to the whole genome using BWA-MEM plus samtools, the alignment backbone of the Anchored-fusion targeted fusion search; it recovers multi-mapping and chimeric read pairs that de novo fusion detectors typically discard.

## When to use

Use when performing a targeted fusion search on paired-end RNA-seq FASTQ data by anchoring a gene of interest, and reads must be aligned to both the target gene transcript fasta and a whole-genome reference.

## When NOT to use

- Input is single-end RNA-seq data, whole-genome sequencing, or whole-exome sequencing — Anchored-fusion only supports paired-end RNA-seq
- The fusion of interest does not include the user-provided target (anchored) gene — such fusions will not be detected
- An untargeted, de novo fusion discovery across all possible fusion partners is required — the anchored search is limited to the user-specified gene

## Inputs

- Paired-end RNA-seq FASTQ files (fastq1 and fastq2, e.g. test_sample_1.fastq.gz)
- Target gene transcript reference fasta (downloaded via NCBI Gene, e.g. target_gene.fasta)
- Whole-genome reference fasta (from GENCODE or Ensembl, e.g. reference.genome.fa)
- Compatible gene annotation GTF (hg38 or hg19, e.g. genome.annotation.gtf)

## Outputs

- Reads aligned to the anchored gene X
- Alignments of unmapped mates against the whole genome (candidate chimeric read pairs)
- Spanning reads and split reads supporting predicted fusion genes, used downstream by BLAT and the HVLD filter

## How to apply

First map all paired-end reads to the reference sequence of the anchored gene X with BWA-MEM (bwa >= 0.7.17) and process/convert alignments with samtools (>= 1.16), collecting all reads aligned to gene X. Select paired-end reads in which only one read maps to the anchored gene and align the unmapped mate to the entire genome (genome fasta from GENCODE/Ensembl with a compatible gene annotation gtf, hg38 or hg19). Downstream, candidate chimeric reads undergo local alignment with BLAT (>= v.35) and interval operations with bedtools (>= v2.30.0), yielding spanning reads and split reads as supporting evidence at the predicted fusion site. The rationale is that reads with non-unique or homologous matches — which conventional fusion detectors exclude — are retained here, enabling detection of fusions like BCR-ABL1 and DUX4-IGH at low coverage.

## Related tools

- **BWA-MEM** (Primary aligner: maps all paired-end reads to the anchored gene transcript and the unmapped mates to the whole genome)
- **samtools** (SAM/BAM processing and alignment handling (version >= 1.16))
- **BLAT** (Local alignment of candidate chimeric reads to pinpoint the precise fusion location (version >= v.35))
- **bedtools** (Interval operations during candidate fusion filtering (version >= v2.30.0))
- **Anchored-fusion** (Pipeline driver invoked as Anchored_Fusion.py, orchestrating the anchored alignment and downstream HVLD filtering) — https://github.com/ShenLab-Genomics/Anchored-Fusion

## Examples

```
python Anchored_Fusion.py --file_anchored_cds=target_gene.fasta --fastq1=test_sample_1.fastq.gz --fastq2=test_sample_1.fastq.gz --out_folder=output_dir --file_ref_seq=reference.genome.fa --file_ref_ann=genome.annotation.gtf
```

## Evaluation signals

- All paired-end reads map to the anchored gene reference and only pairs with exactly one mate aligned to gene X are passed to whole-genome alignment of the unmapped mate
- Genome fasta and annotation gtf versions are compatible (hg38 fa with hg38 gtf, or hg19 with hg19)
- Tallied spanning reads and split reads per fusion candidate are consistent with the benchmark expectation, e.g. Anchored-fusion detects BCR-ABL1 and CIC-DUX4 at the lowest coverage among STAR-Fusion v1.12.0, FusionInspector v2.8.0, FusionCatcher v1.30, and Arriba v2.4.0
- DUX4-IGH detection coverage limit is approximately one-fourth of FusionCatcher's, matching Figure 3 of the paper
- Genes with homologous regions to the anchored gene are excluded from the potential fusion partner set before candidate filtering

## Limitations

- Only fusion genes that include the user-provided target gene can be detected; other fusions will not appear in the output
- Cannot be applied to single-end RNA-seq, whole-genome, or whole-exome sequencing data
- Users must supply the reference sequence of their target gene; small input datasets should use the default HVLD model parameters from GitHub rather than retrained ones

## Evidence
