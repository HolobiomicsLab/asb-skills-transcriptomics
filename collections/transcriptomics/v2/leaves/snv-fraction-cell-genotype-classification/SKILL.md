---
name: snv-fraction-cell-genotype-classification
description: Use when scRNA-seq data from a mixed-cell or chimerism sample requires
  per-cell genotype assignment from transcriptome SNVs, especially when no distinguishing
  marker genes exist or marker-based calls need orthogonal validation.
when_to_use_negative:
- Input is a 5'/3' gene-expression feature matrix without an accompanying aligned
  genome BAM, since SNVs must be called from aligned transcriptome reads
- The sample is a single pure genotype, where marker-based classification already
  suffices and no genotype-informative SNV set exists
- Analysis goals require detecting chimerism below the method's detection level (~4%
  of 6,000 cells), where the K=1 model will be selected
edam_operation: http://edamontology.org/operation_3259
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_3308
tools:
- name: Cell Ranger Single-Cell Software Suite
  role: Produces the genome BAM (STAR-aligned) and gene-cell-barcode counts that serve
    as input for SNV calling and marker-based classification
  repo: https://www.10xgenomics.com/support/software/cell-ranger
- name: Freebayes 1.0.2
  role: Calls SNVs on the Cell Ranger genome BAM; run with Qual>=100, >=10 UMI counts
    from at least two cells, indels ignored
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/ncomms14049
    title: Massively parallel digital transcriptional profiling of single cells
  source_content_hash: sha256:c67a0f214d2e2fe2dda18fcaa049b7700742f13bdc47cbe157af8c09bf9b82a4
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Cell Ranger Single-Cell Software Suite
  - Freebayes 1.0.2
  edam_operation: http://edamontology.org/operation_3259
  edam_topics:
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# SNV-fraction-based cell genotype classification in scRNA-seq mixtures

## Summary

Classifies individual cells in mixed scRNA-seq samples (e.g., Jurkat/293T, host/donor chimerism) by scoring genotype-enriched SNVs called directly from the transcriptome with Freebayes on the Cell Ranger genome BAM, independent of marker gene expression. Provides a de novo alternative to marker-based cell assignment plus a multiplet-rate estimator.

## When to use

Use when scRNA-seq data from a mixed-cell or chimerism sample requires per-cell genotype assignment from transcriptome SNVs, especially when no distinguishing marker genes exist or marker-based calls need orthogonal validation.

## When NOT to use

- Input is a 5'/3' gene-expression feature matrix without an accompanying aligned genome BAM, since SNVs must be called from aligned transcriptome reads
- The sample is a single pure genotype, where marker-based classification already suffices and no genotype-informative SNV set exists
- Analysis goals require detecting chimerism below the method's detection level (~4% of 6,000 cells), where the K=1 model will be selected

## Inputs

- Cell Ranger genome BAM (STAR-aligned, hg19) from the 10x droplet scRNA-seq run (SRA SRP073767)
- Marker-gene-based per-cell assignment table for cross-validation
- Human reference genome hg19 (or other appropriate reference) for variant calling

## Outputs

- snv_genotype_assignments.csv with per-cell SNV-based genotype assignment, consistency vs marker-based assignment, within-genotype purity, and multiplet rate
- Per-cell species SNV fractions used for mixture modeling

## How to apply

Start from the genome BAM produced by Cell Ranger (STAR-aligned to hg19 for Jurkat/293T) and call SNVs with Freebayes 1.0.2, keeping only high-quality SNVs (Qual>=100, at least 10 UMI counts from at least two cells, indels ignored) observed exclusively in one genotype. Score each cell using only SNVs supported by at least two cell barcodes with SNV Qual>=30 and minimal SNV base Qual>=1, computing the fraction of reads supporting each species per cell. Fit K=1 and K=2 mixture models to the per-cell SNV fractions, accepting the K=2 model only if 90% of cells have posterior probability >75% (K=1 means the mixture fraction is below the detection level, ~4% of 6,000 cells). Classify cells whose SNV fraction from either species lies between 0.2 and 0.8 as multiplets and estimate the multiplet rate as twice the observed multiplet fraction, then compare SNV assignments against marker-based assignments.

## Related tools

- **Cell Ranger Single-Cell Software Suite** (Produces the genome BAM (STAR-aligned) and gene-cell-barcode counts that serve as input for SNV calling and marker-based classification) — https://www.10xgenomics.com/support/software/cell-ranger
- **Freebayes 1.0.2** (Calls SNVs on the Cell Ranger genome BAM; run with Qual>=100, >=10 UMI counts from at least two cells, indels ignored)

## Examples

```
freebayes -f hg19.fa cellranger_genome_bam.bam --pooled-continuous --min-alternate-total 10 > snvs.vcf
```

## Evaluation signals

- SNV-based vs marker-based assignment consistency reaches ~99%
- Within-genotype purity reaches ~96% and ~94% for the two genotypes (e.g., 96% of 293T cells and 94% of Jurkat cells carry the enriched SNV set)
- Inferred multiplet rate is ~3% in the 1:1 mixing experiment (2x the observed 0.2-0.8 SNV-fraction fraction)
- K=2 mixture model is accepted only when 90% of cells have posterior probability >75%; otherwise K=1 is reported
- Retained SNVs satisfy Qual>=100 for calling and, for cell scoring, >=2 cell barcodes, SNV Qual>=30, and minimal base Qual>=1

## Limitations

- Selecting K=1 only indicates the mixture fraction is below the detection level, determined as ~4% of 6,000 cells in in silico mixing; low-level chimerism is not resolvable
- Multiplet calls between SNV fractions 0.2 and 0.8 are a heuristic; the inferred rate is twice the observed multiplet fraction
- Only SNVs observed exclusively in one genotype contribute to the assignment, so genotypes sharing most variants or low-coverage cells may be unclassifiable

## Evidence

- [discussion] High-quality SNVs are selected before scoring cells: "High-quality SNVs (SNV calling Qual>=100 with at least 10 UMI counts from at least two cells; indels are ignored) that were only observed in Jurkat or 293T cells"
- [discussion] Cell-scoring SNV inclusion thresholds: "Only SNVs with support from at least two cell barcodes, with a minimal SNV Qual score >=30, minimal SNV base Qual>=1 were included."
- [discussion] Mixture model selection criterion: "We applied a requirement that 90% of the cells have a posterior probability >75% to select the K=2 model over the K=1 model."
- [discussion] Multiplet definition and rate inference: "Cells with a fraction of SNV from either species between 0.2 and 0.8 are considered multiplets. The inferred multiplet rate is 2* observed multiplet rate"
- [discussion] Variant calling tool and input: "SNVs were called by running Freebayes 1.0.2 (ref. 36) on the genome BAM produced by Cell Ranger."
- [readme] Dataset availability: [evidence span withheld — anchor status: not_found]
