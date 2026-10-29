---
name: allelic-read-uniqueness-qc-filtering
description: 'Use when RNA-seq reads aligned to two haplotype-modified personal genomes
  must be allele-assigned: filter multi-mapping or between-genome discordant reads,
  then assign haplotypes by SNP majority voting before ASAS counting.'
when_to_use_negative:
- Reads are aligned to a single unmodified reference genome (e.g., STAR on hg19 only)
  — uniqueness across both haplotype-modified genomes cannot be assessed and majority
  voting over phased SNPs is not meaningful
- No phased per-sample VCF genotype data are available, so haplotype-modified personal
  genomes cannot be constructed
- The heterozygous SNPs of interest are inside alternative exons or too distant from
  the AS event to appear on short RNA-seq reads — reads cannot be allele-assigned
  for PAIRADISE analysis
edam_topics:
- http://edamontology.org/topic_3563
- http://edamontology.org/topic_0092
tools:
- name: PAIRADISE
  role: Pipeline whose Step 1 (personalization, mapping, and assignment) implements
    genome personalization, dual-genome STAR alignment, uniqueness cross-comparison,
    and majority-vote haplotype assignment
  repo: github.com/Xinglab/PAIRADISE
- name: STAR
  role: Spliced aligner (>= 2.6.0a) used to align each sample's reads independently
    to both haplotype-modified personal genome indices
  repo: https://github.com/alexdobin/STAR
- name: pysam
  role: Python dependency (>= 0.15.2) for reading/manipulating alignment records during
    read filtering and assignment
  repo: github.com/Xinglab/PAIRADISE
- name: pybedtools
  role: Python dependency (>= 0.8.0) for interval-based operations on reads and SNPs
    in the assignment step
  repo: github.com/Xinglab/PAIRADISE
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.ajhg.2020.07.005
    title: Detecting Allele-Specific Alternative Splicing from Population-Scale RNA-Seq
      Data
  source_content_hash: sha256:ed11dc431fb3e3f3f6cafa54b55e2e4329d23cd4fccff37332dabf1264cf86a1
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/Xinglab/PAIRADISE
  tools:
  - PAIRADISE
  - STAR
  - pybedtools
  - pysam
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3563
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# Allelic-read-uniqueness QC filtering

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A read-level quality-control and assignment filter from the PAIRADISE/rPGA workflow that retains only RNA-seq reads mapping uniquely to both haplotype-modified personal genomes at the same genomic location, then assigns each retained read to haplotype 1 or 2 by majority voting over covered heterozygous SNPs, discarding multi-SNP draws.

## When to use

Use when RNA-seq reads aligned to two haplotype-modified personal genomes must be allele-assigned: filter multi-mapping or between-genome discordant reads, then assign haplotypes by SNP majority voting before ASAS counting.

## When NOT to use

- Reads are aligned to a single unmodified reference genome (e.g., STAR on hg19 only) — uniqueness across both haplotype-modified genomes cannot be assessed and majority voting over phased SNPs is not meaningful
- No phased per-sample VCF genotype data are available, so haplotype-modified personal genomes cannot be constructed
- The heterozygous SNPs of interest are inside alternative exons or too distant from the AS event to appear on short RNA-seq reads — reads cannot be allele-assigned for PAIRADISE analysis

## Inputs

- gzipped fastq RNA-seq reads per sample/replicate (input directory)
- Per-chromosome VCF genotype files with phased heterozygous SNPs (genotype directory, one subdirectory per sample)
- hg19.fa reference genome fasta
- Homo_sapiens.Ensembl.GRCh37.75.gtf annotation
- Homo_sapiens.GRCh37.75.dna.primary_assembly.fa
- sample_name.txt mapping replicate IDs to samples with RNA-seq read lengths

## Outputs

- read_haplotype_assignments.tsv: per-sample read-to-haplotype assignment table with labels 1, 2, or discarded for every uniquely aligned read
- Per-sample uniquely aligned, allele-assigned read sets ready for Step 2 (joint annotation) and Step 3 (counting)

## How to apply

First construct two haplotype-modified personal genomes by substituting phased heterozygous alleles from per-chromosome VCFs (genotype directory) into hg19.fa, build STAR (>= 2.6.0a) indices with the Homo_sapiens.Ensembl.GRCh37.75.gtf annotation, and align each sample's gzipped fastq reads independently to each haplotype genome with at most 6 mismatches per read and canonical splice junctions only. Cross-compare the two alignments and require that any read assigned to a specific haplotype be aligned uniquely to both versions of the haplotype-modified personal genomes at the same location; discard multi-mapping or discordant reads. For each retained read, assign it to haplotype 1 or 2 by majority voting over the heterozygous SNPs it covers, and discard reads containing multiple SNPs whose votes result in a draw. The rationale is to suppress alignment bias before allele-specific counting: an allele-informative read must be positionally consistent across both personalized references, otherwise allelic ratios are confounded by mapping artifacts. Run this as Step 1 (personalization, mapping, and assignment) via the generated qsub files, e.g. pairadise_step1_NA12843.qsub with h_vmem=90G; expect 2-5 hours per sample for a population like Geuvadis CEU.

## Related tools

- **PAIRADISE** (Pipeline whose Step 1 (personalization, mapping, and assignment) implements genome personalization, dual-genome STAR alignment, uniqueness cross-comparison, and majority-vote haplotype assignment) — github.com/Xinglab/PAIRADISE
- **STAR** (Spliced aligner (>= 2.6.0a) used to align each sample's reads independently to both haplotype-modified personal genome indices) — https://github.com/alexdobin/STAR
- **pysam** (Python dependency (>= 0.15.2) for reading/manipulating alignment records during read filtering and assignment) — github.com/Xinglab/PAIRADISE
- **pybedtools** (Python dependency (>= 0.8.0) for interval-based operations on reads and SNPs in the assignment step) — github.com/Xinglab/PAIRADISE

## Examples

```
qsub -cwd -l h_vmem=90G qsub_files/pairadise_step1_NA12843.qsub
```

## Evaluation signals

- read_haplotype_assignments.tsv exists and contains a haplotype label (1, 2, or discarded) for every uniquely aligned read in the sample
- No retained read was assigned to a haplotype without unique alignment to both haplotype-modified genomes at the same genomic location
- Every read spanning multiple SNPs whose hap-1/hap-2 allele votes tie is recorded as discarded, not arbitrarily assigned
- STAR was run at version >= 2.6.0a with at most 6 mismatches per read and canonical splice junctions only, and both haplotype genome indices were built with the same GTF
- Counts of discarded multi-mapping/discordant reads are consistent between the two haplotype alignments (reads surviving must be the intersection of unique placements)

## Limitations

- Requires a heterozygous SNP outside the alternative exon but close enough to the AS event to be detected on the same RNA-seq read; SNPs located within or too distant from alternative exons cannot be analyzed with short-read RNA-seq data
- Step 1 (personalization, mapping, and assignment) is slow: typical run-time was 2-5 hours per sample for the Geuvadis CEU dataset with h_vmem=90G
- Mammalian STAR genome indices require at least 16GB of RAM, ideally 32GB, doubling the index-building burden since two haplotype genomes must be indexed
- Reads discarded as ambiguous (multi-mapping, discordant, or voting draws) reduce allelic count depth downstream

## Evidence

- [discussion] Uniqueness filter across both personalized genomes: "we require that any read assigned to a specific haplotype has to be aligned to both versions of the haplotype-modified personal genomes uniquely at the same location"
- [discussion] Majority voting and draw discard rule: "If a read contains multiple SNPs, the read is assigned to a specific haplotype based on majority voting or discarded if there is a draw."
- [discussion] rPGA-style haplotype-modified genome alignment: "RNA-seq reads are aligned to haplotype-modified personal genomes following the procedures in our rPGA pipeline."
- [readme] Step 1 execution via qsub: "Submit each of the qsub files corresponding to Step 1 of the PAIRADISE pipeline (personalization, mapping, and assignment):"
- [readme] STAR version requirement for the alignment step: "STAR >= (2.6.0a) (can be downloaded <a href="https://github.com/alexdobin/STAR/releases/tag/2.6.0a" target="_blank">here</a>)"
