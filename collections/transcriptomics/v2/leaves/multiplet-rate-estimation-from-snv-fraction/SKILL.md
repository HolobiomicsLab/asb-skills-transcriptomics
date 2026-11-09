---
name: multiplet-rate-estimation-from-snv-fraction
description: Use when scRNA-seq data come from mixed cell populations (e.g. Jurkat/293T
  or 293T/3T3 mixing experiments, transplant chimerism samples) and an observed multiplet/doublet
  rate is needed from genotype rather than expression markers.
when_to_use_negative:
- Single-genotype (pure) samples with no mixed species or genotypes, where per-cell
  SNV fractions carry no multiplet signal
- Low-level chimerism below the method's detection level (4% of 6,000 cells), where
  K=1 mixture models are selected and no multiplet rate can be inferred
- Datasets without a genotype BAM or sufficient UMI depth to call SNVs with >=10 UMI
  counts from at least two cells
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: Freebayes 1.0.2
  role: Calls SNVs on the Cell Ranger genome BAM; run with Qual>=100, >=10 UMI counts
    from at least two cells, indels ignored
- name: Cell Ranger Single-Cell Software Suite
  role: Produces the genome BAM and gene-barcode matrices (barcode/UMI processing
    and 3' gene counting) used as input for SNV calling
  repo: 10XGenomics/cellranger
provenance:
  source_task_ids:
  - task_001
  - task_003
  source_papers:
  - doi: 10.1038/ncomms14049
    title: Massively parallel digital transcriptional profiling of single cells
  source_content_hash: sha256:fe6f3a10270d39473ef9cafba0268a1d7ba96ad49ddeee3b3c5c2de804e9e372
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/10XGenomics/cellranger
  tools:
  - Cell Ranger Single-Cell Software Suite
  - Freebayes 1.0.2
  - STAR
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# Multiplet rate estimation from SNV fraction

## Summary

Estimates single-cell multiplet rates by scoring per-cell SNV fractions from scRNA-seq data: cells with a fraction of SNVs from either genotype/species between 0.2 and 0.8 are classified as multiplets, and the inferred multiplet rate is reported as twice the observed multiplet fraction. This provides a genotype-based, expression-independent doublet estimate in mixed samples.

## When to use

Use when scRNA-seq data come from mixed cell populations (e.g. Jurkat/293T or 293T/3T3 mixing experiments, transplant chimerism samples) and an observed multiplet/doublet rate is needed from genotype rather than expression markers.

## When NOT to use

- Single-genotype (pure) samples with no mixed species or genotypes, where per-cell SNV fractions carry no multiplet signal
- Low-level chimerism below the method's detection level (4% of 6,000 cells), where K=1 mixture models are selected and no multiplet rate can be inferred
- Datasets without a genotype BAM or sufficient UMI depth to call SNVs with >=10 UMI counts from at least two cells

## Inputs

- Genome BAM file of scRNA-seq alignments produced by Cell Ranger (e.g. STAR-aligned to hg19)
- FASTQ/deposited sequencing data (e.g. SRA accession SRP073767) from a mixed-cell sample
- Marker-based cell assignments (gene expression labels) for cross-validation

## Outputs

- Per-cell SNV fractions for each genotype/species
- Multiplet calls (cells with SNV fraction between 0.2 and 0.8 for either species)
- Inferred multiplet rate (2 x observed multiplet rate), e.g. tabulated in snv_genotype_assignments.csv

## How to apply

First call SNVs by running Freebayes 1.0.2 on the genome BAM produced by Cell Ranger (STAR-aligned, hg19), retaining high-quality SNVs with Qual>=100 supported by at least 10 UMI counts from at least two cells and ignoring indels, selecting SNVs observed exclusively in one genotype (e.g. only Jurkat or only 293T cells). Then score each cell using SNVs with support from at least two cell barcodes, minimal SNV Qual score >=30, and minimal SNV base Qual>=1, computing the fraction of reads supporting each genotype per cell. Optionally fit K=1 and K=2 mixture models to the per-cell SNV fractions, selecting the K=2 model only if 90% of cells have posterior probability >75%; otherwise the mixture fraction is below the method's detection level (4% of 6,000 cells). Classify cells with a SNV fraction from either species between 0.2 and 0.8 as multiplets, and report the inferred multiplet rate as twice the observed multiplet rate because a multiplet contains two cells. Validate by comparing SNV-based assignments to marker-based (gene expression) cell classification.

## Related tools

- **Freebayes 1.0.2** (Calls SNVs on the Cell Ranger genome BAM; run with Qual>=100, >=10 UMI counts from at least two cells, indels ignored)
- **Cell Ranger Single-Cell Software Suite** (Produces the genome BAM and gene-barcode matrices (barcode/UMI processing and 3' gene counting) used as input for SNV calling) — 10XGenomics/cellranger

## Examples

```
multiplets = ((snv_frac > 0.2) & (snv_frac < 0.8)).any(axis=1); inferred_rate = 2 * multiplets.mean()
```

## Evaluation signals

- In a 1:1 Jurkat/293T mixed sample, SNV-based versus marker-based assignment consistency reaches ~99%
- Within-genotype purity reaches ~96% (293T) and ~94% (Jurkat) for SNV-enriched assignments
- The inferred multiplet rate (2 x observed multiplet fraction) is ~3%, matching the reported value
- Cells classified as multiplets have SNV fraction strictly between 0.2 and 0.8 for at least one species; all others fall outside this interval
- K=2 mixture model is only accepted if 90% of cells have posterior probability >75%; otherwise K=1 is retained

## Limitations

- Cells with intermediate SNV fractions (0.2-0.8) are called multiplets, but the inferred rate assumes each observed multiplet represents exactly two cells (rate = 2 x observed)
- Low-level mixture fractions below 4% of 6,000 cells fall below the method's detection level (K=1 model selected)
- The inferred multiplet rate depends on prior Cell Ranger barcode/UMI processing quality; earlier libraries with 5 nt UMIs introduce protocol heterogeneity

## Evidence

- [methods] Cells with a fraction of SNV from either species between 0.2 and 0.8 are considered multiplets. The inferred multiplet rate is 2* observed multiplet rate: "Cells with a fraction of SNV from either species between 0.2 and 0.8 are considered multiplets. The inferred multiplet rate is 2* observed multiplet rate"
- [discussion] SNVs were called by running Freebayes 1.0.2 (ref. 36) on the genome BAM produced by Cell Ranger.: "SNVs were called by running Freebayes 1.0.2 (ref. 36) on the genome BAM produced by Cell Ranger."
- [methods] Only SNVs with support from at least two cell barcodes, with a minimal SNV Qual score >=30, minimal SNV base Qual>=1 were included.: "Only SNVs with support from at least two cell barcodes, with a minimal SNV Qual score >=30, minimal SNV base Qual>=1 were included."
- [methods] We applied a requirement that 90% of the cells have a posterior probability >75% to select the K=2 model over the K=1 model.: "We applied a requirement that 90% of the cells have a posterior probability >75% to select the K=2 model over the K=1 model."
- [readme] Cell Ranger is a set of analysis pipelines that perform sample demultiplexing, barcode processing, single cell 3' and 5' gene counting: "Cell Ranger is a set of analysis pipelines that perform sample demultiplexing, barcode processing, single cell 3' and 5' gene counting"
