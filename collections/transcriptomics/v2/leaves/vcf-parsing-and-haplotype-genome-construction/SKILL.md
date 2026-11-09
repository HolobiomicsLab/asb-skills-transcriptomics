---
name: vcf-parsing-and-haplotype-genome-construction
description: 'Use when performing allele-specific read assignment from RNA-seq FASTQ
  data: a per-sample VCF genotype directory and a reference FASTA are available and
  reads must be aligned to both haplotype-modified personal genomes.'
when_to_use_negative:
- The analysis goal is standard bulk RNA-seq differential expression or splicing quantification
  without allele-specific assignment — a single reference-genome STAR alignment suffices
  and personalization adds cost
- The genotype data are not phased heterozygous SNVs (e.g., haploid calls, structural
  variants only, or a common sample lacking a per-sample VCF subdirectory matching
  the sample name)
- SNPs of interest lie within or too far from alternative exons for short reads to
  carry them alongside the splice event — PAIRADISE cannot analyze such SNPs even
  with personalized genomes
edam_operation: http://edamontology.org/operation_3196
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0622
tools:
- name: PAIRADISE
  role: Provides the pairadise_personalize entry point that performs personalization,
    mapping, and assignment (Step 1) from VCF genotype directories and reference FASTA
  repo: github.com/Xinglab/PAIRADISE
- name: STAR
  role: Spliced aligner (>= 2.6.0a) used to build indices for each haplotype-modified
    personal genome and align the sample's gzipped fastq reads with at most 6 mismatches
    and canonical junctions
  repo: https://github.com/alexdobin/STAR
- name: pysam
  role: Python dependency (>= 0.15.2) used for reading/writing VCF and alignment files
    during personalization and haplotype assignment
  repo: github.com/Xinglab/PAIRADISE
- name: pybedtools
  role: Python dependency (>= 0.8.0) supporting genomic interval operations in the
    PAIRADISE personalization/assignment scripts
  repo: github.com/Xinglab/PAIRADISE
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.ajhg.2020.07.005
    title: Detecting Allele-Specific Alternative Splicing from Population-Scale RNA-Seq
      Data
  source_content_hash: sha256:8de51c73bd90d43534fcce2e886c2ac9debb70e91b12d10ab7c2b8af90adea17
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
  edam_operation: http://edamontology.org/operation_3196
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# vcf-parsing-and-haplotype-genome-construction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Constructs haplotype-modified personal genomes by substituting phased heterozygous alleles from per-chromosome VCF genotype files into the hg19.fa reference sequence, then builds STAR (>= 2.6.0a) indices for each haplotype genome using the Homo_sapiens.Ensembl.GRCh37.75.gtf annotation. This is the personalization foundation of the rPGA-style pipeline that PAIRADISE uses before read assignment to haplotypes.

## When to use

Use when performing allele-specific read assignment from RNA-seq FASTQ data: a per-sample VCF genotype directory and a reference FASTA are available and reads must be aligned to both haplotype-modified personal genomes.

## When NOT to use

- The analysis goal is standard bulk RNA-seq differential expression or splicing quantification without allele-specific assignment — a single reference-genome STAR alignment suffices and personalization adds cost
- The genotype data are not phased heterozygous SNVs (e.g., haploid calls, structural variants only, or a common sample lacking a per-sample VCF subdirectory matching the sample name)
- SNPs of interest lie within or too far from alternative exons for short reads to carry them alongside the splice event — PAIRADISE cannot analyze such SNPs even with personalized genomes

## Inputs

- Per-chromosome VCF genotype files, one subdirectory per sample/replicate inside the genotype directory
- Reference genome FASTA files (hg19.fa, Homo_sapiens.GRCh37.75.dna.primary_assembly.fa) in the genome directory
- Gene annotation GTF (Homo_sapiens.Ensembl.GRCh37.75.gtf)
- Gzipped FASTQ read files per sample (.gz extension) in the input directory
- Optional RNA editing positions file from RADAR (Human_AG_all_hg19_v2.txt) with the --rnaedit flag
- sample_name.txt file listing sample IDs and RNA-seq read lengths (3 columns)

## Outputs

- Two haplotype-modified personal genome FASTA sequences per sample (haplotype 1 and haplotype 2)
- STAR genome indices for both haplotype-modified genomes
- Per-sample STAR alignment (BAM) files against each haplotype genome
- Intermediate personalized-genome outputs in the directory specified by -o

## How to apply

For each sample, parse the per-chromosome VCF files in the genotype directory and substitute the phased heterozygous alleles into the hg19.fa reference sequence to create two haplotype-modified personal genomes. Build a STAR (>= 2.6.0a) genome index for each haplotype genome using the Homo_sapiens.Ensembl.GRCh37.75.gtf annotation, then align the sample's gzipped FASTQ reads independently to each genome with STAR configured for at most 6 mismatches per read and canonical splice junctions only. PAIRADISE's pairadise_personalize entry point wraps this step, accepting a VCF genotype directory (-v), reference FASTA (-r), gzipped-read flag (--gz), an optional RADAR RNA-editing file (-e with --rnaedit), and an output directory (-o). The rationale is that allele-specific read assignment later requires uniquely aligned reads at the same location in both haplotype genomes, so personalization must faithfully embed all phased heterozygous alleles; an optional RNA-editing check against RADAR prevents editing sites from being mistaken for haplotype alleles.

## Related tools

- **PAIRADISE** (Provides the pairadise_personalize entry point that performs personalization, mapping, and assignment (Step 1) from VCF genotype directories and reference FASTA) — github.com/Xinglab/PAIRADISE
- **STAR** (Spliced aligner (>= 2.6.0a) used to build indices for each haplotype-modified personal genome and align the sample's gzipped fastq reads with at most 6 mismatches and canonical junctions) — https://github.com/alexdobin/STAR
- **pysam** (Python dependency (>= 0.15.2) used for reading/writing VCF and alignment files during personalization and haplotype assignment) — github.com/Xinglab/PAIRADISE
- **pybedtools** (Python dependency (>= 0.8.0) supporting genomic interval operations in the PAIRADISE personalization/assignment scripts) — github.com/Xinglab/PAIRADISE

## Examples

```
pairadise_personalize --gz -v genotype/NA12843 -o output/ -r genome/hg19.fa NA12843
```

## Evaluation signals

- pairadise_personalize -h prints the expected usage block (with --gz, -e, --rnaedit, -v, -o, -r arguments), confirming installation and argument wiring
- Each sample has a subdirectory inside the genotype directory whose name matches the sample name, and one VCF per chromosome is present
- Both haplotype-modified genomes differ from hg19.fa exactly at the phased heterozygous SNP positions and nowhere else
- STAR indices build successfully for both haplotype genomes with the Homo_sapiens.Ensembl.GRCh37.75.gtf annotation (STAR >= 2.6.0a, ~32 GB RAM for mammalian genomes)
- Step 1 qsub job completes for each sample (typical run-time 2-5 hours per sample for the Geuvadis CEU dataset with h_vmem=90G), producing usable alignments for downstream cross-genome comparison

## Limitations

- Step 1 is slow: typical run-time for one sample on the CEU dataset is 2-5 hours and the qsub job requests h_vmem=90G
- Mammalian STAR index construction requires at least 16 GB of RAM, ideally 32 GB
- RNA editing must be accounted for with a RADAR file (-e with --rnaedit) or editing sites may be misinterpreted as allelic differences
- Only heterozygous SNPs outside the alternative exon but close enough to be co-detected on the same short read ultimately enable allele-specific assignment; SNPs within or too distant from alternative exons cannot be analyzed

## Evidence

- [other] Construct two haplotype-modified personal genomes for the sample by substituting the phased heterozygous alleles from the per-chromosome VCF files (genotype directory) into the hg19.fa reference sequence.: [evidence span withheld — anchor status: not_found]
