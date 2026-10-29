---
name: bayesian-mixture-model-selection
description: Use when single-cell RNA-seq SNV calls yield a per-cell fraction of reads
  supporting each species/genotype and you must decide if the population is a mixture
  of two genotypes, assign cells, and quantify multiplets.
when_to_use_negative:
- The dataset is a single-genotype sample where K=1 is expected and the goal is not
  chimerism/mixture deconvolution (K=1 selection here only means mixture fraction
  is below the ~4% detection level, not a successful two-genotype assignment)
- Input is raw gene-barcode count data without SNV calls — this skill requires per-cell
  SNV fractions, not expression matrices
- Variants are indels rather than SNVs, since the pipeline explicitly ignores indels
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0102
tools:
- name: Freebayes 1.0.2
  role: Calls SNVs from the Cell Ranger genome BAM that feed the per-cell SNV fractions
    used in mixture model fitting
- name: Cell Ranger Single-Cell Software Suite
  role: Produces the genome BAM (STAR-aligned to hg19) and cell barcodes from which
    SNVs are scored per cell
  repo: https://github.com/10XGenomics/cellranger
- name: R (10XGenomics/single-cell-3prime-paper code)
  role: Secondary analysis code (mixture fitting, assignment tables) hosted in the
    paper's repository
  repo: https://github.com/10XGenomics/single-cell-3prime-paper
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/ncomms14049
    title: Massively parallel digital transcriptional profiling of single cells
  source_content_hash: sha256:7b28bbfa5889e81a31f3d768bb4364346b7d53f33b2fccd6dcd874fc152b6090
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/10XGenomics/cellranger
  tools:
  - Cell Ranger Single-Cell Software Suite
  - Freebayes 1.0.2
  - R (10XGenomics/single-cell-3prime-paper code)
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# bayesian-mixture-model-selection

## Summary

Fit K=1 vs K=2 Bayesian mixture models to per-cell SNV genotype fractions derived from scRNA-seq to decide whether a sample contains two distinct genotypes (e.g., Jurkat vs 293T or host vs donor), classify individual cells, and estimate multiplet rates.

## When to use

Use when single-cell RNA-seq SNV calls yield a per-cell fraction of reads supporting each species/genotype and you must decide if the population is a mixture of two genotypes, assign cells, and quantify multiplets.

## When NOT to use

- The dataset is a single-genotype sample where K=1 is expected and the goal is not chimerism/mixture deconvolution (K=1 selection here only means mixture fraction is below the ~4% detection level, not a successful two-genotype assignment)
- Input is raw gene-barcode count data without SNV calls — this skill requires per-cell SNV fractions, not expression matrices
- Variants are indels rather than SNVs, since the pipeline explicitly ignores indels

## Inputs

- Per-cell SNV fractions (fraction of reads supporting each species/genotype per cell barcoded cell) from Freebayes SNV calls on a Cell Ranger genome BAM (hg19)
- High-quality SNV set: Qual>=100, >=10 UMI counts from at least two cells, indels ignored, SNVs observed exclusively in Jurkat or exclusively in 293T cells

## Outputs

- Per-cell genotype assignments with posterior probabilities under the K=1 vs K=2 models
- Multiplet calls (SNV fraction from either species between 0.2 and 0.8) and inferred multiplet rate (2x observed)
- Consistency table of SNV-based vs marker-based assignments, e.g. snv_genotype_assignments.csv

## How to apply

Score each cell with SNVs supported by at least two cell barcodes, SNV Qual>=30 and minimal SNV base Qual>=1, computing the fraction of reads supporting each species per cell. Fit K=1 and K=2 mixture models to these per-cell SNV fractions; select the K=2 model only if 90% of the cells have posterior probability >75%. Classify cells whose SNV fraction from either species falls between 0.2 and 0.8 as multiplets and compute the inferred multiplet rate as 2x the observed multiplet fraction. Finally, compare SNV-based genotype assignments with marker-based (gene expression) assignments, as done for the Jurkat/293T 1:1 mix where 96% of 293T and 94% of Jurkat cells primarily harbored genotype-enriched SNVs with ~99% overall consistency. Note that selecting K=1 indicates the mixture fraction is below the method's detection level (4% of 6,000 cells in in silico mixing experiments).

## Related tools

- **Freebayes 1.0.2** (Calls SNVs from the Cell Ranger genome BAM that feed the per-cell SNV fractions used in mixture model fitting)
- **Cell Ranger Single-Cell Software Suite** (Produces the genome BAM (STAR-aligned to hg19) and cell barcodes from which SNVs are scored per cell) — https://github.com/10XGenomics/cellranger
- **R (10XGenomics/single-cell-3prime-paper code)** (Secondary analysis code (mixture fitting, assignment tables) hosted in the paper's repository) — https://github.com/10XGenomics/single-cell-3prime-paper

## Examples

```
Rscript fit_snv_mixtures.R --input per_cell_snv_fractions.csv --kmax 2 --posterior_cutoff 0.75 --min_frac_cells 0.9 --output snv_genotype_assignments.csv
```

## Evaluation signals

- K=2 model accepted only if >=90% of cells have posterior probability >75%; otherwise fall back to K=1
- SNV-based vs marker-based assignment consistency reaches ~99% in the Jurkat/293T 1:1 mix
- Within-genotype purity ~96% for 293T and ~94% for Jurkat cells
- Inferred multiplet rate ~3% (multiplets = SNV fraction from either species in 0.2–0.8; rate = 2x observed fraction)
- All scored SNVs meet inclusion thresholds: support from >=2 cell barcodes, SNV Qual>=30, minimal SNV base Qual>=1

## Limitations

- Selecting K=1 only indicates the mixture fraction is below the detection level, determined as 4% of 6,000 cells in in silico mixing experiments — low-level chimerism is not quantifiable
- Only high-quality SNVs (Qual>=100, >=10 UMI counts from at least two cells, indels ignored) exclusive to one genotype are usable, so sparse-coverage cells may be unassignable
- Multiplet rate is inferred as twice the observed fraction between 0.2 and 0.8, an approximation rather than a direct measurement

## Evidence

- [discussion] K=2 model selection threshold: "We applied a requirement that 90% of the cells have a posterior probability >75% to select the K=2 model over the K=1 model."
- [discussion] SNV inclusion filters for scoring: "Only SNVs with support from at least two cell barcodes, with a minimal SNV Qual score >=30, minimal SNV base Qual>=1 were included."
- [discussion] High-quality SNV calling criteria: "High-quality SNVs (SNV calling Qual>=100 with at least 10 UMI counts from at least two cells; indels are ignored)"
- [discussion] Multiplet definition and rate calculation: "Cells with a fraction of SNV from either species between 0.2 and 0.8 are considered multiplets. The inferred multiplet rate is 2* observed multiplet rate"
- [discussion] Detection-level limitation of K=1: "Selecting K=1 indicates that the mixture fraction is below the level of detection of the method, which in in silico mixing experiments was determined to be 4% of 6,000 cells."
