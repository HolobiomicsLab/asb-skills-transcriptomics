---
name: reference-genome-indexing
description: 'Use when preparing an RNA-seq alignment reference: a genome FASTA plus
  Ensembl GTF must be compiled into a STAR index, or the reference must be re-built
  after masking known variant positions with Bedtools to reduce reference-allele bias.'
when_to_use_negative:
- Input is already an aligned BAM produced with a consistent genome build and GTF
  — indexing is unnecessary and would not change results
- Genome and GTF come from mismatched builds/annotations (e.g., Gallus_gallus_6.0
  GTF with Gallus_gallus_5.0 FASTA) — rebuild with a coherent pair instead
- Short-read DNA variant calling against GRCh38/other non-chicken references — the
  index specifics here are Gallus gallus and RNA-seq oriented
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: STAR 2.5.2b
  role: Genome index builder and two-pass aligner; the index embeds splice junctions
    from the Ensembl GTF so RNA-seq reads align across introns with soft-clipping
    instead of trimming
- name: Bedtools
  role: Uses the merged global VCF to mask reference genome positions, enabling re-indexing/re-alignment
    that reduces reference-allele bias before ASE testing
- name: GATK 3.7
  role: Produces the per-sample g.vcf files that are merged into the global VCF used
    to create the masked reference genome
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41598-021-83459-8
    title: Investigation of allele specific expression in various tissues of broiler
      chickens using the detection tool VADT
  source_content_hash: sha256:0cc5d1d94204281a533f1acbab9092c9ca12aea73276a93116f85516602be207
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Bedtools
  - FastQC
  - GATK 3.7
  - Picard
  - STAR 2.5.2b
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# reference-genome-indexing

## Summary

Builds a STAR splice-aware genome index from a Gallus gallus reference genome build plus matching Ensembl GTF annotation (and, when needed, a Bedtools-masked variant-corrected genome) so that two-pass STAR RNA-seq alignment can run correctly.

## When to use

Use when preparing an RNA-seq alignment reference: a genome FASTA plus Ensembl GTF must be compiled into a STAR index, or the reference must be re-built after masking known variant positions with Bedtools to reduce reference-allele bias.

## When NOT to use

- Input is already an aligned BAM produced with a consistent genome build and GTF — indexing is unnecessary and would not change results
- Genome and GTF come from mismatched builds/annotations (e.g., Gallus_gallus_6.0 GTF with Gallus_gallus_5.0 FASTA) — rebuild with a coherent pair instead
- Short-read DNA variant calling against GRCh38/other non-chicken references — the index specifics here are Gallus gallus and RNA-seq oriented

## Inputs

- Gallus gallus reference genome FASTA (e.g., Gallus_gallus_5.0 or Gallus_gallus_6.0 build)
- Matching Ensembl GTF annotation file (e.g., release 86)
- Optional: global VCF file of variants to mask into the reference genome

## Outputs

- STAR genome index directory (genome + splice-junction database)
- Variant-masked reference genome FASTA (when Bedtools masking is applied)
- Aligned SAM/BAM files from two-pass STAR 2.5.2b with per-sample mapping statistics

## How to apply

Create the STAR index from the Gallus_gallus genome build together with its matching Ensembl GTF (this study used the Gallus_gallus_5.0 build with Ensembl GTF release 86 to make the initial index, and repeated GTF-dependent steps later with the Gallus_gallus_6.0 GTF), since the index embeds splice junctions needed for two-pass STAR 2.5.2b alignment. Keep GTF annotation and genome build version consistent across indexing and downstream counting/quantification steps. After variant calling, the global VCF can be used to mask the reference genome with Bedtools and the samples re-aligned to the masked genome; this reduces reference-allele bias, dropping the reference allele ratio from ~51.8-51.9% to ~50.2% in all three tissues. Verify the index build by aligning reads with STAR two-pass mode and checking mapping statistics (e.g., the reported average unique mapping rate of 87.24%).

## Related tools

- **STAR 2.5.2b** (Genome index builder and two-pass aligner; the index embeds splice junctions from the Ensembl GTF so RNA-seq reads align across introns with soft-clipping instead of trimming)
- **Bedtools** (Uses the merged global VCF to mask reference genome positions, enabling re-indexing/re-alignment that reduces reference-allele bias before ASE testing)
- **GATK 3.7** (Produces the per-sample g.vcf files that are merged into the global VCF used to create the masked reference genome)

## Examples

```
STAR --runMode genomeGenerate --genomeDir gg_index --genomeFastaFiles Gallus_gallus_5.0.fa --sjdbGTFfile Gallus_gallus.86.gtf --sjdbOverhang 100
```

## Evaluation signals

- STAR alignment statistics consistent with the paper's reported average unique mapping rate of 87.24%
- Index genome/GTF pair uses consistent build numbers (Gallus_gallus_5.0 with Ensembl release 86, or Gallus_gallus_6.0 with the 6.0 GTF)
- After Bedtools masking and re-alignment, reference allele ratio is near 50% (reported reduction from ~51.8-51.9% to ~50.17-50.19% across breast muscle, abdominal fat, and liver)
- Masked reference genome contains variant positions from all 68 samples' merged global VCF

## Limitations

- Mapping problems arose with muscle tissue samples, likely due to limitations of the current genome build and annotation of muscle-related genes
- Breast muscle samples had a significantly higher percentage of reads flagged by STAR as too short (avg. 15.39% vs ~4.5-5.3% for other tissues)

## Evidence

- [methods] STAR index from genome + Ensembl GTF: "The index file initially used for STAR was created using gtf file 86 from Ensembl"
- [methods] Two-pass STAR alignment to Gallus_gallus_5.0: "aligning sequencing reads using STAR (version 2.5.2b) with the two-pass setting29,30 to the Gallus_gallus_5.0 genome build"
- [methods] Bedtools masking of reference genome: "The global vcf file was then used to mask the reference genome using Bedtools"
- [methods] Masking reduced reference allele ratio toward 50%: "masked reference genome reduced the reference allele ration from 51.77 to 50.17% in breast muscle, 51.93% to 50.19% in abdominal fat and 51.91% to 50.17% in liver"
- [discussion] Muscle mapping issues attributed to genome build/annotation: "The biggest problem we encountered with our entire analysis pipeline was mapping issues with muscle tissue samples"
