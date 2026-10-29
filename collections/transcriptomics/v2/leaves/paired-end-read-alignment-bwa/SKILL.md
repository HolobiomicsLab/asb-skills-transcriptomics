---
name: paired-end-read-alignment-bwa
description: 'Use when running Anchored-fusion or its benchmark against de novo fusion
  callers: paired-end FASTQ (R1/R2) must be mapped with BWA-MEM (>=0.7.17) to a target-gene
  fasta and genome fa (hg38/hg19) with matching GTF.'
when_to_use_negative:
- Input is single-end RNA-seq, whole-genome sequencing, or whole-exome sequencing
  data — Anchored-fusion's BWA-MEM workflow requires paired-end RNA-seq reads
- 'Goal is de novo discovery of all fusion events: only fusions that include the user-provided
  anchored gene are detected and reported'
- No compatible genome fasta/GTF pair is available — versions of the reference fa
  and annotation gtf must match (hg38 and hg19 both welcome)
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: BWA-MEM
  role: 'Alignment engine: maps all paired-end reads to the anchored gene reference
    and aligns unmapped mates of singly-mapped pairs to the whole genome'
  repo: https://github.com/ShenLab-Genomics/Anchored-Fusion
- name: BLAT
  role: Local alignment of candidate chimeric reads to precisely predict the gene
    fusion location after BWA-MEM mapping
  repo: https://github.com/ShenLab-Genomics/Anchored-Fusion
- name: samtools
  role: Read manipulation including generating subsampled depths (repeated read-pair
    halving) for the sensitivity benchmark
  repo: https://github.com/ShenLab-Genomics/Anchored-Fusion
- name: Anchored-fusion
  role: Pipeline wrapping the BWA-MEM alignment steps; invoked via Anchored_Fusion.py
    with --file_ref_seq and --file_ref_ann
  repo: https://github.com/ShenLab-Genomics/Anchored-Fusion
- name: Arriba
  role: Competing bRNA-seq fusion caller (v2.4.0, STAR-based) benchmarked against
    the BWA-MEM-based Anchored-fusion on downsampled data
  repo: https://github.com/suhrig/arriba
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.crmeth.2024.100733
    title: Anchored-fusion enables targeted fusion search in bulk and single-cell
      RNA sequencing data
  source_content_hash: sha256:33d68b4be3d2a62ed366d5ed9ee7eac84f7320e36451cbba891f82826ec51879
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
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# paired-end-read-alignment-bwa

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Align paired-end RNA-seq FASTQ reads with BWA-MEM as the core read-mapping step of Anchored-fusion: reads are first aligned to the anchored gene's transcript/CDS fasta, and the unmapped mates of singly-mapped pairs are then aligned against the whole genome to recover chimeric reads supporting fusions.

## When to use

Use when running Anchored-fusion or its benchmark against de novo fusion callers: paired-end FASTQ (R1/R2) must be mapped with BWA-MEM (>=0.7.17) to a target-gene fasta and genome fa (hg38/hg19) with matching GTF.

## When NOT to use

- Input is single-end RNA-seq, whole-genome sequencing, or whole-exome sequencing data — Anchored-fusion's BWA-MEM workflow requires paired-end RNA-seq reads
- Goal is de novo discovery of all fusion events: only fusions that include the user-provided anchored gene are detected and reported
- No compatible genome fasta/GTF pair is available — versions of the reference fa and annotation gtf must match (hg38 and hg19 both welcome)

## Inputs

- Paired-end FASTQ files (R1 and R2, gzipped fastq accepted) from bulk or single-cell RNA-seq
- Target gene transcript/CDS reference fasta (downloadable via NCBI GENE)
- Genome reference fasta (hg38 or hg19 from GENCODE/Ensembl)
- Gene annotation GTF file compatible with the genome fasta version

## Outputs

- BWA-MEM alignments of paired-end reads against the anchored gene reference (collected reads aligned to gene X)
- Whole-genome alignments of unmapped mates of singly-anchored read pairs
- Candidate chimeric read alignments passed to BLAT for precise fusion breakpoint prediction
- Anchored-fusion output folder listing detected fusion genes involving the target gene

## How to apply

Prepare three references: a target gene transcript fasta from NCBI Gene, a genome fasta (GENCODE/Ensembl hg38 or hg19), and a version-matched GTF. Anchored-fusion first maps all paired-end reads to the reference sequence of the anchored gene X with BWA-MEM and collects reads aligned to gene X; for pairs where only one read maps to the anchored gene, the unmapped mate is aligned to the entire genome to find candidate fusion partners. Candidate chimeric reads are subsequently locally aligned with BLAT (>= v.35) to pinpoint the fusion breakpoint, with spanning and split reads used as supporting evidence. In the downsampling benchmark, the same BWA-MEM-based Anchored-fusion run (python Anchored_Fusion.py --file_anchored_cds=<target_gene.fasta> --fastq1=<R1> --fastq2=<R2> --out_folder=<dir> --file_ref_seq=<genome.fa> --file_ref_ann=<annotation.gtf>) is applied at each of 8 subsampled depths down to <1M read pairs on CCLE bRNA-seq from SRP186687. Because the pipeline deliberately recovers non-unique (multi-mapping) matches that conventional tools discard, BWA-MEM is chosen over tools with stricter unique-mapping defaults.

## Related tools

- **BWA-MEM** (Alignment engine: maps all paired-end reads to the anchored gene reference and aligns unmapped mates of singly-mapped pairs to the whole genome) — https://github.com/ShenLab-Genomics/Anchored-Fusion
- **BLAT** (Local alignment of candidate chimeric reads to precisely predict the gene fusion location after BWA-MEM mapping) — https://github.com/ShenLab-Genomics/Anchored-Fusion
- **samtools** (Read manipulation including generating subsampled depths (repeated read-pair halving) for the sensitivity benchmark) — https://github.com/ShenLab-Genomics/Anchored-Fusion
- **Anchored-fusion** (Pipeline wrapping the BWA-MEM alignment steps; invoked via Anchored_Fusion.py with --file_ref_seq and --file_ref_ann) — https://github.com/ShenLab-Genomics/Anchored-Fusion
- **Arriba** (Competing bRNA-seq fusion caller (v2.4.0, STAR-based) benchmarked against the BWA-MEM-based Anchored-fusion on downsampled data) — https://github.com/suhrig/arriba

## Examples

```
python Anchored_Fusion.py --file_anchored_cds=test/target_gene.fasta --fastq1=test/test_sample_1.fastq.gz --fastq2=test/test_sample_1.fastq.gz --out_folder=output_dir --file_ref_seq=reference.genome.fa --file_ref_ann=genome.annotation.gtf
```

## Evaluation signals

- Alignment sanity: paired-end reads map to the anchored gene transcript/CDS fasta, and only mates of pairs with exactly one read on the anchored gene are re-aligned genome-wide
- Detection matrix check: in the CCLE/SRP186687 benchmark, Anchored-fusion (BWA-MEM-based) still calls the expected fusion at all 7 halving steps below 1 million read pairs where STAR-Fusion v1.12.0, FusionInspector v2.8.0, FusionCatcher v1.30, and Arriba v2.4.0 all fail (Figure 4 pattern)
- Output-folder completeness: Anchored-fusion produces detected fusions that all include the anchored target gene; fusions lacking the target gene should be absent
- Reference consistency: genome fasta and GTF versions match (e.g., both hg38 or both hg19) so breakpoint coordinates are consistent
- HVLD filtering retains calls: candidate fusions with HVLD probability < 0.1 are filtered out, so surviving calls pass the deep-learning false-positive filter unless --not_filter_false_positive was set

## Limitations

- Fusion genes that do not include the user-provided target gene will not be detected
- Anchored-fusion can only be used on paired-end RNA-seq; single-end RNA-seq, whole-genome, and whole-exome sequencing data are unsupported
- Users must supply the reference sequence (fasta) of their target gene themselves, e.g. via NCBI GENE
- For very small input datasets, retraining the HVLD filter is unreliable; the default model parameters from the GitHub repo are recommended instead

## Evidence
