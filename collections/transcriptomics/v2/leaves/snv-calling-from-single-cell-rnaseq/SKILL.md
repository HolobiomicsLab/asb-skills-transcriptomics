---
name: snv-calling-from-single-cell-rnaseq
description: Use when scRNA-seq data from a mixed-genotype or mixed-species sample
  (e.g. Jurkat/293T mixtures or transplant host/donor chimerism) must be deconvolved
  at single-cell resolution using sequence variation rather than gene expression markers.
when_to_use_negative:
- The goal is differential gene expression or cell-type clustering based on transcript
  abundance rather than genotype assignment — use marker-based approaches (e.g. Seurat)
  instead
- The mixture fraction is below the detection level of the method (~4% of 6,000 cells
  in in silico mixing experiments); K=1 selection would only indicate the minor genotype
  is undetectable
- Input data lack per-cell UMI counts or the sample is homogeneous with no expected
  sequence variation between cell populations
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2885
tools:
- name: Cell Ranger Single-Cell Software Suite
  role: Performs sample demultiplexing, barcode processing and single-cell 3' gene
    counting, and produces the genome BAM used as SNV-calling input
  repo: https://github.com/10XGenomics/cellranger
- name: Freebayes 1.0.2
  role: Calls SNVs on the Cell Ranger genome BAM, with indels ignored and high-quality
    filtering applied
- name: STAR
  role: Aligns Read1 (cDNA insert) to the reference genome (hg19) during Cell Ranger
    processing
- name: GemCode Single-Cell Instrument (10x Genomics)
  role: Generates single-cell GEMs from the Jurkat/293T cellular suspensions that
    provide the scRNA-seq data
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/ncomms14049
    title: Massively parallel digital transcriptional profiling of single cells
  source_content_hash: sha256:0d53eaf76a5337e998137a48a596b5a3766641cd633028b8a74630b7dbe5b613
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/10XGenomics/cellranger
  tools:
  - Cell Ranger Single-Cell Software Suite
  - Freebayes 1.0.2
  - GemCode Single-Cell Instrument (10x Genomics)
  - STAR
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2885
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# snv-calling-from-single-cell-rnaseq

## Summary

A method for calling and using single nucleotide variants directly from scRNA-seq transcriptome data (via Freebayes on the Cell Ranger genome BAM) to assign individual cells to their genotype of origin without prior knowledge of cell-specific markers or genotypes.

## When to use

Use when scRNA-seq data from a mixed-genotype or mixed-species sample (e.g. Jurkat/293T mixtures or transplant host/donor chimerism) must be deconvolved at single-cell resolution using sequence variation rather than gene expression markers.

## When NOT to use

- The goal is differential gene expression or cell-type clustering based on transcript abundance rather than genotype assignment — use marker-based approaches (e.g. Seurat) instead
- The mixture fraction is below the detection level of the method (~4% of 6,000 cells in in silico mixing experiments); K=1 selection would only indicate the minor genotype is undetectable
- Input data lack per-cell UMI counts or the sample is homogeneous with no expected sequence variation between cell populations

## Inputs

- Genome BAM produced by Cell Ranger (STAR-aligned, hg19) from Jurkat/293T mixing scRNA-seq data (SRA SRP073767)
- Cell barcodes and UMI counts per cell for filtering SNV support
- Marker-based (gene expression) cell classification for downstream consistency comparison

## Outputs

- List of high-quality genotype-exclusive SNVs (Qual>=100, >=10 UMI counts from >=2 cells, indels ignored)
- Per-cell SNV fraction of reads supporting each genotype/species
- Per-cell SNV-based genotype assignments with multiplet flags and inferred multiplet rate
- snv_genotype_assignments.csv tabulating consistency, within-genotype purity, and multiplet rate

## How to apply

First align the scRNA-seq reads (Read1 containing the cDNA insert) with STAR via Cell Ranger against the appropriate reference (hg19 for human cells) to produce a genome BAM. Run Freebayes 1.0.2 on that genome BAM, ignoring indels, and retain high-quality SNVs with Qual>=100 supported by at least 10 UMI counts from at least two cells, selecting SNVs observed exclusively in one genotype (e.g. Jurkat-only or 293T-only). Score each cell using only SNVs with support from at least two cell barcodes, SNV Qual>=30, and minimal SNV base Qual>=1, computing the fraction of reads supporting each species per cell. Fit K=1 and K=2 mixture models to the per-cell SNV fractions and select K=2 only if 90% of cells have posterior probability >75%; classify cells with SNV fraction from either species between 0.2 and 0.8 as multiplets and report the inferred multiplet rate as twice the observed multiplet fraction. Finally compare SNV-based genotype assignments with marker-based (gene expression) assignments to validate consistency.

## Related tools

- **Cell Ranger Single-Cell Software Suite** (Performs sample demultiplexing, barcode processing and single-cell 3' gene counting, and produces the genome BAM used as SNV-calling input) — https://github.com/10XGenomics/cellranger
- **Freebayes 1.0.2** (Calls SNVs on the Cell Ranger genome BAM, with indels ignored and high-quality filtering applied)
- **STAR** (Aligns Read1 (cDNA insert) to the reference genome (hg19) during Cell Ranger processing)
- **GemCode Single-Cell Instrument (10x Genomics)** (Generates single-cell GEMs from the Jurkat/293T cellular suspensions that provide the scRNA-seq data)

## Examples

```
freebayes -f hg19.fa --pooled-continuous genome_bam.bam | awk 'NR==1 || ($5>=100)' > snvs.vcf
```

## Evaluation signals

- SNV-based vs marker-based cell assignment consistency reaches ~99%
- Within-genotype purity reaches ~96% (293T) and ~94% (Jurkat) in the 1:1 mixed sample
- Inferred multiplet rate is ~3% (cells with SNV fraction from either species between 0.2 and 0.8, rate = 2 x observed multiplet fraction)
- K=2 mixture model is accepted only if 90% of cells have posterior probability >75%
- Output snv_genotype_assignments.csv contains consistency, within-genotype purity, and multiplet rate columns

## Limitations

- Selecting K=1 indicates the mixture fraction is below the detection level, determined as ~4% of 6,000 cells in in silico mixing experiments, limiting sensitivity for low-level chimerism
- Only SNVs with support from at least two cell barcodes, minimal SNV Qual >=30, and minimal SNV base Qual >=1 are included; indels are ignored, restricting callable variation
- Multiplet classification (SNV fraction 0.2–0.8) is an inferred estimate; the inferred multiplet rate is defined as twice the observed multiplet rate rather than a direct measurement

## Evidence

- [discussion] SNV calling thresholds from Freebayes on Cell Ranger BAM: "SNVs were called by running Freebayes 1.0.2 (ref. 36) on the genome BAM produced by Cell Ranger."
- [discussion] High-quality SNV filter criteria: "High-quality SNVs (SNV calling Qual>=100 with at least 10 UMI counts from at least two cells; indels are ignored) that were only observed in Jurkat or 293T cells"
- [discussion] Genotype assignment SNV inclusion filters: "Only SNVs with support from at least two cell barcodes, with a minimal SNV Qual score >=30, minimal SNV base Qual>=1 were included."
- [discussion] Mixture model selection rule: "We applied a requirement that 90% of the cells have a posterior probability >75% to select the K=2 model over the K=1 model."
- [discussion] Multiplet definition and inferred rate: "Cells with a fraction of SNV from either species between 0.2 and 0.8 are considered multiplets. The inferred multiplet rate is 2* observed multiplet rate"
- [discussion] Data availability: "Single-cell RNA-seq data have been deposited in the Short Read Archive under accession number SRP073767."
- [readme] README data access pointer: [evidence span withheld — anchor status: not_found]
