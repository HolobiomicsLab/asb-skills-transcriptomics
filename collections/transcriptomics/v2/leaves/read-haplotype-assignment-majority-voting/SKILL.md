---
name: read-haplotype-assignment-majority-voting
description: 'Use when performing allele-specific read assignment for ASAS analysis:
  you have phased per-chromosome VCFs, gzipped FASTQ reads, hg19.fa, and need to label
  each uniquely aligned read with haplotype 1, 2, or ''discarded''.'
when_to_use_negative:
- You only need gene/transcript quantification without allele-specific resolution
  — a single-genome STAR alignment is cheaper than dual haplotype-genome mapping
- The heterozygous SNP of interest lies within, or too far from, the alternative exon
  so that no read can cover both SNP and splice event — PAIRADISE cannot analyze these
  SNPs with short reads
- Your SNVs are putative RNA editing sites rather than germline heterozygous variants
  — PAIRADISE handles these separately via the RADAR-based --rnaedit flag, not via
  haplotype personalization
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_0625
tools:
- name: PAIRADISE
  role: Pipeline providing pairadise_personalize for genome personalization, mapping,
    and haplotype assignment (Step 1)
  repo: github.com/Xinglab/PAIRADISE
- name: STAR
  role: Spliced aligner used to align reads independently to each haplotype-modified
    personal genome (>= 2.6.0a)
  repo: https://github.com/alexdobin/STAR
- name: pysam
  role: Python dependency used to manipulate alignment records during filtering and
    assignment
- name: pybedtools
  role: Python dependency used for interval operations when retaining reads aligned
    to both haplotype genomes at the same location
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.ajhg.2020.07.005
    title: Detecting Allele-Specific Alternative Splicing from Population-Scale RNA-Seq
      Data
  source_content_hash: sha256:564ff98670d3d4f95c200d467b3f303bd14cd08b72629a170d1e7bf4737404aa
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
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_0625
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# read-haplotype-assignment-majority-voting

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

PAIRADISE-style assignment of RNA-seq reads to haplotype 1 or 2 after aligning to haplotype-modified personal genomes: reads are retained only if they map uniquely to both personal genome versions at the same location, then assigned by majority voting over covered heterozygous SNPs, with draws and ambiguous reads discarded.

## When to use

Use when performing allele-specific read assignment for ASAS analysis: you have phased per-chromosome VCFs, gzipped FASTQ reads, hg19.fa, and need to label each uniquely aligned read with haplotype 1, 2, or 'discarded'.

## When NOT to use

- You only need gene/transcript quantification without allele-specific resolution — a single-genome STAR alignment is cheaper than dual haplotype-genome mapping
- The heterozygous SNP of interest lies within, or too far from, the alternative exon so that no read can cover both SNP and splice event — PAIRADISE cannot analyze these SNPs with short reads
- Your SNVs are putative RNA editing sites rather than germline heterozygous variants — PAIRADISE handles these separately via the RADAR-based --rnaedit flag, not via haplotype personalization

## Inputs

- Phased per-chromosome VCF genotype files (one VCF per chromosome, in a per-sample genotype subdirectory)
- Gzipped FASTQ RNA-seq read files for the sample/replicate
- hg19.fa reference fasta file
- Homo_sapiens.Ensembl.GRCh37.75.gtf annotation
- STAR >= 2.6.0a genome indices built for each haplotype-modified personal genome

## Outputs

- Per-sample read-to-haplotype assignment table (read_haplotype_assignments.tsv) with labels 1, 2, or discarded for every uniquely aligned read
- Alignments (BAM) of uniquely mapped reads to each haplotype-modified personal genome

## How to apply

First personalize the reference by substituting the sample's phased heterozygous alleles from the per-chromosome VCF files (genotype directory) into hg19.fa, producing two haplotype-modified personal genomes. Build STAR (>= 2.6.0a) indices for both genomes with Homo_sapiens.Ensembl.GRCh37.75.gtf and align the sample's gzipped FASTQ reads independently to each (allowing at most 6 mismatches per read, canonical splice junctions only). Cross-compare the two alignments and keep only reads that map uniquely to both haplotype genomes at the same genomic location — this guards against mapping bias from spurious alignments. For each retained read, assign haplotype by majority voting over the heterozygous SNPs it covers: reads carrying haplotype 1 (or 2) alleles at the majority of SNP positions go to that haplotype; reads containing multiple SNPs whose votes end in a draw are discarded. Write the per-sample read-to-haplotype table, including reads filtered out as ambiguous, to read_haplotype_assignments.tsv.

## Related tools

- **PAIRADISE** (Pipeline providing pairadise_personalize for genome personalization, mapping, and haplotype assignment (Step 1)) — github.com/Xinglab/PAIRADISE
- **STAR** (Spliced aligner used to align reads independently to each haplotype-modified personal genome (>= 2.6.0a)) — https://github.com/alexdobin/STAR
- **pysam** (Python dependency used to manipulate alignment records during filtering and assignment)
- **pybedtools** (Python dependency used for interval operations when retaining reads aligned to both haplotype genomes at the same location)

## Examples

```
qsub -cwd -l h_vmem=90G qsub_files/pairadise_step1_NA12843.qsub
```

## Evaluation signals

- read_haplotype_assignments.tsv exists and contains a haplotype label (1, 2, or discarded) for every uniquely aligned read
- No retained read was assigned without unique alignment to both haplotype genomes at the same genomic location
- Reads whose multi-SNP votes end in a draw appear under 'discarded', not under haplotype 1 or 2
- STAR alignment settings used <= 6 mismatches per read and canonical splice junctions only
- Every uniquely aligned read maps concordantly across the two haplotype genome alignments (no multi-mapping or discordant reads retained)

## Limitations

- Step 1 is computationally expensive: typical runtime for one sample (e.g., in the Geuvadis CEU dataset) ranged from 2-5 hours with ~90G memory requested
- ASAS detection requires a heterozygous SNP outside the alternative exon but close enough to be detected on the same RNA-seq read
- SNPs located within or too distant from alternative exons cannot be analyzed using short-read RNA-seq data
- Similarity to WASP's allele-swapping/remapping strategy means the same confounding-by-mapping-bias concerns drive the unique-alignment requirement

## Evidence
