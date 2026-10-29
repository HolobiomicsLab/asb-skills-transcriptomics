---
name: umi-based-variant-quality-filtering
description: Use when calling SNVs from UMI-based scRNA-seq data (e.g. Cell Ranger-aligned
  10x droplet data) to classify individual cells by genotype, quantify chimerism or
  multiplets, or need variant quality criteria tied to UMI/cell-barcode support rather
  than read depth alone.
when_to_use_negative:
- Input is already a variant VCF scored at read depth without UMI counts — UMI-level
  support filters cannot be applied retroactively.
- The goal is germline variant discovery from bulk exome/genome data; these thresholds
  are tuned for sparse single-cell 3' transcriptome coverage.
- Genotypes of the two cell populations are known and can be distinguished by expressed
  markers alone; SNV-based assignment adds value mainly for de novo chimerism without
  prior genotypes.
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3173
- http://edamontology.org/topic_3168
tools:
- name: Freebayes 1.0.2
  role: Haplotype-based SNV caller run on the Cell Ranger genome BAM to generate candidate
    SNVs for UMI-support filtering
- name: Cell Ranger Single-Cell Software Suite
  role: Performs sample demultiplexing, barcode processing, and single-cell 3' gene
    counting; produces the STAR-aligned genome BAM that SNVs are called from
  repo: https://github.com/10XGenomics/cellranger
- name: STAR
  role: Aligns Read1 (the cDNA insert) to the hg19 reference genome within the Cell
    Ranger pipeline
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/ncomms14049
    title: Massively parallel digital transcriptional profiling of single cells
  source_content_hash: sha256:6ed001b49b8eb311b6da45819c144d96b03a4f2d6d8d5687f95679cad508c59c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/10XGenomics/cellranger
  tools:
  - Cell Ranger Single-Cell Software Suite
  - Freebayes 1.0.2
  - STAR
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_3173
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# umi-based-variant-quality-filtering

## Summary

A filtering scheme for single nucleotide variants (SNVs) called from single-cell 3' UMI transcriptome data, in which variants called with Freebayes on the Cell Ranger/STAR genome BAM are retained only when backed by sufficient UMI counts and cell barcodes, then used for single-cell genotype/chimerism assignment. It enables de novo determination of cell origin from sequence variation without prior genotype knowledge.

## When to use

Use when calling SNVs from UMI-based scRNA-seq data (e.g. Cell Ranger-aligned 10x droplet data) to classify individual cells by genotype, quantify chimerism or multiplets, or need variant quality criteria tied to UMI/cell-barcode support rather than read depth alone.

## When NOT to use

- Input is already a variant VCF scored at read depth without UMI counts — UMI-level support filters cannot be applied retroactively.
- The goal is germline variant discovery from bulk exome/genome data; these thresholds are tuned for sparse single-cell 3' transcriptome coverage.
- Genotypes of the two cell populations are known and can be distinguished by expressed markers alone; SNV-based assignment adds value mainly for de novo chimerism without prior genotypes.

## Inputs

- Cell Ranger-produced genome BAM (STAR-aligned to hg19)
- FASTQ scRNA-seq reads from a 10x droplet library (e.g. SRA SRP073767 Jurkat/293T mixing data)
- Human reference genome (hg19) for variant calling
- Cell barcode / UMI gene-barcode matrix for cell identity

## Outputs

- Filtered set of genotype-exclusive SNVs (Qual>=100, >=10 UMI counts, >=2 cells, no indels)
- Per-cell SNV support fractions for each genotype
- SNV-based genotype assignment per cell, multiplet calls, and inferred multiplet rate (e.g. snv_genotype_assignments.csv)

## How to apply

Run Freebayes 1.0.2 on the STAR-aligned genome BAM produced by Cell Ranger (hg19 for human samples) and first retain only high-quality SNVs with Qual>=100 supported by at least 10 UMI counts from at least two cells, ignoring indels, and select SNVs observed exclusively in one genotype (e.g. Jurkat- or 293T-enriched). For per-cell scoring, include only SNVs with support from at least two cell barcodes, minimal SNV Qual score >=30, and minimal SNV base Qual>=1, then compute the fraction of reads supporting each species per cell. Fit K=1 and K=2 mixture models to the per-cell SNV fractions and accept the K=2 model only if 90% of cells have posterior probability >75%. Classify cells with an SNV fraction between 0.2 and 0.8 from either species as multiplets, and compute the inferred multiplet rate as twice the observed multiplet fraction.

## Related tools

- **Freebayes 1.0.2** (Haplotype-based SNV caller run on the Cell Ranger genome BAM to generate candidate SNVs for UMI-support filtering)
- **Cell Ranger Single-Cell Software Suite** (Performs sample demultiplexing, barcode processing, and single-cell 3' gene counting; produces the STAR-aligned genome BAM that SNVs are called from) — https://github.com/10XGenomics/cellranger
- **STAR** (Aligns Read1 (the cDNA insert) to the hg19 reference genome within the Cell Ranger pipeline)

## Examples

```
freebayes -f hg19.fa cellranger_genome.bam --pooled-continuous --min-alternate-count 10 | awk '$6>=100' > high_quality_snvs.vcf
```

## Evaluation signals

- Filtered SNV set should contain no indels and every retained variant must show Qual>=100 with at least 10 UMI counts from at least two cells and be exclusive to one genotype.
- Per-cell scoring should include only SNVs with support from at least two cell barcodes, minimal SNV Qual >=30, and minimal base Qual >=1.
- K=2 mixture model is acceptable only if 90% of cells have posterior probability >75%; otherwise K=1 (mixture below detection level) must be retained.
- SNV-based vs marker-based cell assignment consistency should reach ~99%, with within-genotype purity ~96% (293T) and ~94% (Jurkat).
- Inferred multiplet rate should be ~3% in the 1:1 Jurkat/293T mixing sample (multiplets = SNV fraction 0.2–0.8; rate = 2 × observed fraction).

## Limitations

- Selecting the K=1 model only indicates the mixture fraction is below the detection level, which in in silico mixing experiments was determined to be 4% of 6,000 cells, limiting sensitivity for low-level chimerism.
- Variant calling is restricted to SNVs (indels ignored), so genotype discrimination depends on transcriptome-expressed, barcode-supported SNVs only.
- Cell assignment by genotype can be low-confidence when the difference between the top two assignments is small, similar to the marker-based correlation caveat noted in the paper.

## Evidence

- [discussion] SNVs were called by running Freebayes 1.0.2 (ref. 36) on the genome BAM produced by Cell Ranger.: "SNVs were called by running Freebayes 1.0.2 (ref. 36) on the genome BAM produced by Cell Ranger."
- [discussion] High-quality SNVs definition: "High-quality SNVs (SNV calling Qual>=100 with at least 10 UMI counts from at least two cells; indels are ignored) that were only observed in Jurkat or 293T cells"
- [discussion] Per-cell SNV scoring filters: "Only SNVs with support from at least two cell barcodes, with a minimal SNV Qual score >=30, minimal SNV base Qual>=1 were included."
- [discussion] K=2 model selection rule: "We applied a requirement that 90% of the cells have a posterior probability >75% to select the K=2 model over the K=1 model."
- [discussion] Multiplet definition and rate calculation: "Cells with a fraction of SNV from either species between 0.2 and 0.8 are considered multiplets. The inferred multiplet rate is 2* observed multiplet rate"
- [readme] README: data availability: [evidence span withheld — anchor status: not_found]
