---
name: marker-vs-genotype-assignment-concordance-analysis
description: Use when you have single-cell RNA-seq data from a mixture of two genotypically
  distinguishable cell populations and need to assign each cell to its genotype from
  transcriptome SNVs, then validate those assignments against marker-based cell classification.
when_to_use_negative:
- Input is a single homogeneous cell population with no second genotype to distinguish
  — K=1 will be selected and per-cell genotype assignment is meaningless.
- Mixture fraction is below the method's detection level (4% of 6,000 cells in in
  silico mixing experiments); selecting K=1 only indicates the minor population cannot
  be detected.
- Data are bulk RNA-seq or lack cell barcodes/UMIs — per-cell SNV fraction scoring
  and multiplet detection cannot be performed.
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0625
tools:
- name: Cell Ranger Single-Cell Software Suite
  role: Performs sample demultiplexing, barcode processing and single-cell 3' gene
    counting, producing the genome BAM on which SNV calling is run
  repo: https://github.com/10XGenomics/cellranger
- name: Freebayes 1.0.2
  role: Variant caller run on the Cell Ranger genome BAM to detect SNVs (Qual>=100,
    >=10 UMI counts, >=2 cells, indels ignored) used for genotype assignment
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/ncomms14049
    title: Massively parallel digital transcriptional profiling of single cells
  source_content_hash: sha256:b0405b6badf81904fb59fca4b08d754118e4888464661fdb66e166888d5d4cc9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/10XGenomics/cellranger
  tools:
  - Cell Ranger Single-Cell Software Suite
  - Freebayes 1.0.2
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0625
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# Marker-vs-genotype assignment concordance analysis

## Summary

Classifies individual cells in a mixed Jurkat/293T 10x Genomics droplet scRNA-seq sample using SNVs called directly from the transcriptome (Freebayes on the Cell Ranger genome BAM), then cross-tabulates these genotype-based assignments against marker-gene-expression-based classifications to quantify consistency, within-genotype purity, and multiplet rate.

## When to use

Use when you have single-cell RNA-seq data from a mixture of two genotypically distinguishable cell populations and need to assign each cell to its genotype from transcriptome SNVs, then validate those assignments against marker-based cell classification.

## When NOT to use

- Input is a single homogeneous cell population with no second genotype to distinguish — K=1 will be selected and per-cell genotype assignment is meaningless.
- Mixture fraction is below the method's detection level (4% of 6,000 cells in in silico mixing experiments); selecting K=1 only indicates the minor population cannot be detected.
- Data are bulk RNA-seq or lack cell barcodes/UMIs — per-cell SNV fraction scoring and multiplet detection cannot be performed.

## Inputs

- Cell Ranger genome BAM (STAR-aligned, hg19) from the Jurkat/293T mixing experiment (SRA SRP073767)
- Gene-cell-barcode count matrix for marker-based (gene expression) cell classification
- Cell barcode identities for Jurkat and 293T populations

## Outputs

- snv_genotype_assignments.csv with per-cell SNV-based genotype, marker-based class, consistency, within-genotype purity, and inferred multiplet rate

## How to apply

Download the Jurkat/293T mixing data (SRA SRP073767) and the STAR-aligned hg19 genome BAM produced by Cell Ranger. Run Freebayes 1.0.2 on the genome BAM and retain high-quality SNVs with Qual>=100 supported by at least 10 UMI counts from at least two cells, ignoring indels, keeping only SNVs observed exclusively in Jurkat or exclusively in 293T cells. Score each cell with SNVs having support from at least two cell barcodes, SNV Qual>=30, and minimal SNV base Qual>=1, computing the per-cell fraction of reads supporting each species. Fit K=1 and K=2 mixture models to the per-cell SNV fractions, selecting K=2 only if 90% of cells have posterior probability >75%; classify cells with a SNV fraction between 0.2 and 0.8 from either species as multiplets (inferred multiplet rate = 2 x observed multiplet fraction). Finally, cross-tabulate SNV-based genotype assignments against marker-based (gene expression) assignments to compute consistency, within-genotype purity, and multiplet rate.

## Related tools

- **Cell Ranger Single-Cell Software Suite** (Performs sample demultiplexing, barcode processing and single-cell 3' gene counting, producing the genome BAM on which SNV calling is run) — https://github.com/10XGenomics/cellranger
- **Freebayes 1.0.2** (Variant caller run on the Cell Ranger genome BAM to detect SNVs (Qual>=100, >=10 UMI counts, >=2 cells, indels ignored) used for genotype assignment)

## Examples

```
freebayes -f hg19.fa --pooled-continuous position_basis Qual>=100 cellranger_genome.bam > jurkat_293t_snvs.vcf
```

## Evaluation signals

- SNV-based vs marker-based assignment consistency reaches ~99% in the 1:1 mixed sample
- Within-genotype purity: ~96% of 293T cells and ~94% of Jurkat cells primarily harbor their species-enriched SNVs
- Inferred multiplet rate is ~3% (2 x observed fraction of cells with SNV fraction between 0.2 and 0.8)
- K=2 mixture model accepted only when 90% of cells have posterior probability >75% over the K=1 model
- All scored SNVs pass the filter: >=2 cell barcodes, SNV Qual>=30, minimal SNV base Qual>=1

## Limitations

- Selecting K=1 only indicates the mixture fraction is below the level of detection, which was 4% of 6,000 cells in in silico mixing experiments, limiting sensitivity for low-level chimerism.
- Cells with a SNV fraction from either species between 0.2 and 0.8 are called multiplets, so genotype assignment for cells near this boundary is ambiguous.
- Indels are ignored during SNV calling, so genotype discrimination relies solely on SNVs observed exclusively in one species.

## Evidence

- [methods] High-quality SNVs: Qual>=100 with at least 10 UMI counts from at least two cells: "High-quality SNVs (SNV calling Qual>=100 with at least 10 UMI counts from at least two cells; indels are ignored) that were only observed in Jurkat or 293T cells"
- [methods] SNV scoring filters for genotype assignment: "Only SNVs with support from at least two cell barcodes, with a minimal SNV Qual score >=30, minimal SNV base Qual>=1 were included."
- [discussion] K=2 model selection criterion: "We applied a requirement that 90% of the cells have a posterior probability >75% to select the K=2 model over the K=1 model."
- [discussion] Multiplet definition and rate inference: "Cells with a fraction of SNV from either species between 0.2 and 0.8 are considered multiplets. The inferred multiplet rate is 2* observed multiplet rate"
- [discussion] SNV calling tool and input: "SNVs were called by running Freebayes 1.0.2 (ref. 36) on the genome BAM produced by Cell Ranger."
