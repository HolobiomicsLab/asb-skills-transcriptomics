---
name: genome-personalization-from-phased-vcf
description: Use when RNA-seq reads must be assigned to a specific haplotype (e.g.
  for allele-specific alternative splicing counting) and per-sample phased VCFs plus
  a reference FASTA are available.
when_to_use_negative:
- The variant of interest lies within the alternative exon or is too distant from
  it — PAIRADISE requires a heterozygous SNP outside but close to the AS event so
  both alleles can be detected on the same short RNA-seq read
- Genotypes are unphased or only low-confidence calls exist, since haplotype genome
  construction and SNP voting both depend on phased heterozygous alleles
- The goal is whole-genome assembly or calling novel variants — this skill only substitutes
  known phased alleles into a reference and re-assigns reads
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_0622
tools:
- name: PAIRADISE
  role: Pipeline whose Step 1 (personalization, mapping, and assignment) implements
    genome personalization and haplotype read assignment
  repo: https://github.com/Xinglab/PAIRADISE
- name: STAR
  role: Spliced aligner (>= 2.6.0a) used to map reads to each haplotype-modified personal
    genome with <= 6 mismatches and canonical junctions
  repo: https://github.com/alexdobin/STAR
- name: pysam
  role: Python dependency used to parse alignments and coordinates for cross-genome
    comparison and SNP voting
- name: pybedtools
  role: Python dependency used for interval operations when intersecting reads with
    heterozygous SNP positions
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.ajhg.2020.07.005
    title: Detecting Allele-Specific Alternative Splicing from Population-Scale RNA-Seq
      Data
  source_content_hash: sha256:1bee163d1ca1c091825b920511578ae1d1eadbdd079b576e314aad0fb6fe25ae
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/Xinglab/PAIRADISE
  repo_url: https://github.com/Xinglab/PAIRADISE
  tools:
  - PAIRADISE
  - STAR
  - pybedtools
  - pysam
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# genome-personalization-from-phased-vcf

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Constructs two haplotype-modified personal genomes by substituting phased heterozygous alleles from per-chromosome VCFs into hg19.fa, aligns RNA-seq reads to both with STAR, and assigns each uniquely-aligned read to haplotype 1 or 2 via majority voting over covered heterozygous SNPs — the rPGA-style personalization used to feed PAIRADISE's allele-specific counting.

## When to use

Use when RNA-seq reads must be assigned to a specific haplotype (e.g. for allele-specific alternative splicing counting) and per-sample phased VCFs plus a reference FASTA are available.

## When NOT to use

- The variant of interest lies within the alternative exon or is too distant from it — PAIRADISE requires a heterozygous SNP outside but close to the AS event so both alleles can be detected on the same short RNA-seq read
- Genotypes are unphased or only low-confidence calls exist, since haplotype genome construction and SNP voting both depend on phased heterozygous alleles
- The goal is whole-genome assembly or calling novel variants — this skill only substitutes known phased alleles into a reference and re-assigns reads

## Inputs

- Per-sample phased VCF files, one per chromosome (genotype directory with a subdirectory per sample/replicate)
- Reference genome FASTA (hg19.fa or Homo_sapiens.GRCh37.75.dna.primary_assembly.fa)
- Gene annotation GTF (Homo_sapiens.Ensembl.GRCh37.75.gtf)
- Gzipped RNA-seq FASTQ files per sample/replicate (input directory)
- Optional RNA editing positions file from RADAR (Human_AG_all_hg19_v2.txt) with the --rnaedit flag

## Outputs

- Per-sample read-to-haplotype assignment table (read_haplotype_assignments.tsv) with a haplotype label (1, 2, or discarded) for every uniquely aligned read
- STAR BAM alignments against both haplotype-modified personal genomes

## How to apply

Build two haplotype-modified personal genomes for the sample by substituting the phased heterozygous alleles from the per-chromosome VCFs in the genotype directory into the hg19.fa reference. Build STAR (>= 2.6.0a) indices for both haplotype genomes using the Homo_sapiens.Ensembl.GRCh37.75.gtf annotation, then align the sample's gzipped FASTQ reads independently to each genome with STAR configured for at most 6 mismatches per read and canonical splice junctions only. Cross-compare the two alignments and retain only reads that map uniquely to both haplotype genomes at the same genomic location — this guards against spurious alignment or mapping bias introduced by the personalization. Assign each retained read to haplotype 1 or 2 by majority voting over the heterozygous SNPs it covers, discarding reads whose SNP votes draw, and write the read-to-haplotype table (including ambiguous reads) to read_haplotype_assignments.tsv. The rationale is that allele-specific read assignment enables PAIRADISE to test one SNP at a time for ASAS with both alleles sharing the same cellular environment.

## Related tools

- **PAIRADISE** (Pipeline whose Step 1 (personalization, mapping, and assignment) implements genome personalization and haplotype read assignment) — https://github.com/Xinglab/PAIRADISE
- **STAR** (Spliced aligner (>= 2.6.0a) used to map reads to each haplotype-modified personal genome with <= 6 mismatches and canonical junctions) — https://github.com/alexdobin/STAR
- **pysam** (Python dependency used to parse alignments and coordinates for cross-genome comparison and SNP voting)
- **pybedtools** (Python dependency used for interval operations when intersecting reads with heterozygous SNP positions)

## Examples

```
pairadise_personalize --gz -v genotype/NA12843 -o output/NA12843 -r genome/hg19.fa input/NA12843_R1.fastq.gz input/NA12843_R2.fastq.gz
```

## Evaluation signals

- read_haplotype_assignments.tsv exists and contains a haplotype label (1, 2, or discarded) for every uniquely aligned read
- No read was assigned to a haplotype without unique alignment to both haplotype-modified genomes at the same genomic location
- No retained read was assigned when its covered SNPs produced a majority-voting draw (such reads must be discarded)
- STAR indices were built from both personalized genomes with the Ensembl GRCh37.75 GTF, and alignments used at most 6 mismatches per read with canonical splice junctions only
- Per-sample Step 1 completes within the expected runtime (2-5 hours per sample for the Geuvadis CEU dataset) without dropping samples before Step 2 joint annotation

## Limitations

- SNPs located within or too distant from alternative exons cannot be analyzed using short-read RNA-seq data, because the allele-specific SNP must be detectable on the same read
- Causal SNPs affecting splicing may lie within the alternative exon or nearby intronic regions and are not tested — only tagging SNPs outside the exon are identified
- Step 1 is computationally expensive (90G h_vmem, 2-5 hours per sample on the CEU dataset), which limits throughput at population scale

## Evidence
