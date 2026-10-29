---
name: variant-calling-from-scrna-alignment-bam
description: Use when you have a Cell Ranger genome BAM from scRNA-seq data and need
  de novo SNV genotypes per cell — e.g. to determine host vs donor origin in post-transplant
  bone marrow samples without a known donor genotype.
when_to_use_negative:
- When a reference donor/host genotype is already available and genotyping (rather
  than de novo discovery) suffices — this skill is for de novo calling without prior
  genotype knowledge
- When indels or structural variants are the target — the method explicitly ignores
  indels and calls only SNVs
- When expecting very low-level chimerism below ~4% of ~6,000 cells — K=1 selection
  only indicates the mixture fraction is below the detection level
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0199
tools:
- name: Freebayes 1.0.2
  role: SNV caller run on the Cell Ranger genome BAM with indels ignored
- name: Cell Ranger Single-Cell Software Suite
  role: Produces the genome BAM (sample demultiplexing, barcode processing, single-cell
    3' gene counting) that serves as the variant-calling input
  repo: 10XGenomics/cellranger
- name: STAR
  role: Aligns Read1, which contains the cDNA insert, to the hg19 reference genome
    prior to Cell Ranger counting
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/ncomms14049
    title: Massively parallel digital transcriptional profiling of single cells
  source_content_hash: sha256:1507edfb99a2fb6f84884614c5e411bc8c58283b02b006978b1319acb6dbb7de
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/10XGenomics/cellranger
  tools:
  - Cell Ranger Single-Cell Software Suite
  - Freebayes 1.0.2
  - STAR
  - Seurat
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0199
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# variant-calling-from-scrna-alignment-bam

## Summary

De novo SNV calling directly from the genome BAM produced by single-cell 3' RNA-seq alignment, used to infer host/donor chimerism and cell genotypes without prior genotype knowledge. It repurposes transcriptome alignments (STAR-aligned Read1 cDNA inserts via Cell Ranger) as a variant-calling substrate with Freebayes 1.0.2.

## When to use

Use when you have a Cell Ranger genome BAM from scRNA-seq data and need de novo SNV genotypes per cell — e.g. to determine host vs donor origin in post-transplant bone marrow samples without a known donor genotype.

## When NOT to use

- When a reference donor/host genotype is already available and genotyping (rather than de novo discovery) suffices — this skill is for de novo calling without prior genotype knowledge
- When indels or structural variants are the target — the method explicitly ignores indels and calls only SNVs
- When expecting very low-level chimerism below ~4% of ~6,000 cells — K=1 selection only indicates the mixture fraction is below the detection level

## Inputs

- Cell Ranger genome BAM from scRNA-seq (Read1 cDNA inserts aligned with STAR to hg19)
- Gene-cell-barcode matrices and per-cell barcode assignments from Cell Ranger

## Outputs

- Filtered SNV calls (indels ignored; >=2 cell barcodes, SNV Qual >=30, base Qual >=1) with per-cell-barcode allelic support
- Per-cell SNV fraction vectors feeding the K=1/K=2 Bayesian genotype mixture model and host/donor chimerism assignments

## How to apply

Run Freebayes 1.0.2 on the genome BAM produced by Cell Ranger, ignoring indels, so variants are SNVs called directly from the transcriptome alignments of Read1 cDNA inserts (aligned with STAR to hg19). Retain only SNVs with support from at least two cell barcodes, a minimal SNV Qual score >=30, and minimal SNV base Qual >=1; these thresholds are deliberately permissive relative to the Jurkat/293T high-confidence set (Qual>=100 with at least 10 UMI counts from at least two cells). Feed the filtered SNVs into the downstream Bayesian genotype-assignment step, which fits K=1 and K=2 mixtures and selects K=2 only when 90% of cells have posterior probability >75%. Note that selecting K=1 means the mixture fraction is below the detection level (4% of 6,000 cells in in silico mixing), so a K=1 call on a transplant sample is a sensitivity-limited all-one-genotype call, not proof of homogeneity.

## Related tools

- **Freebayes 1.0.2** (SNV caller run on the Cell Ranger genome BAM with indels ignored)
- **Cell Ranger Single-Cell Software Suite** (Produces the genome BAM (sample demultiplexing, barcode processing, single-cell 3' gene counting) that serves as the variant-calling input) — 10XGenomics/cellranger
- **STAR** (Aligns Read1, which contains the cDNA insert, to the hg19 reference genome prior to Cell Ranger counting)

## Examples

```
freebayes -p 1 --haplotype-length 0 --min-alternate-count 2 --min-alternate-q 30 --min-base-quality 1 --min-alternate-fraction 0 -v snvs.vcf cellranger_genome.bam
```

## Evaluation signals

- Every retained SNV has support from at least two cell barcodes with minimal SNV Qual >=30 and minimal SNV base Qual >=1, and no indels are present in the call set
- In the AML027 post-transplant validation, the K=2 model is selected and the minor genotype frequency is near 13.8%, with the major genotype ~97% similar to the host genotype, consistent with the clinical 12% donor chimerism assay
- In the AML035 validation, the K=1 (all-donor, 78% similarity) model is selected rather than K=2
- SNV calls reproduce known genotypes in mixing controls: high-quality SNVs (Qual>=100, >=10 UMI counts from >=2 cells) are observed only in Jurkat or 293T cells, and multiplets show SNV fractions between 0.2 and 0.8 from either species

## Limitations

- K=1 selection only indicates the mixture fraction is below the level of detection of the method (4% of 6,000 cells in in silico mixing experiments), limiting sensitivity for low-level chimerism
- The proof-of-principle chimerism study included BMMCs from only two transplant patients; larger patient cohorts were not yet performed
- Clinical chimerism testing with this method is described as aspirational ('in the near future'), not yet clinically validated
- Some earlier libraries used 5 nt UMI and 5 bp Read2, introducing protocol heterogeneity that may affect downstream SNV support

## Evidence

- [discussion] SNV calling on Cell Ranger BAM: "SNVs were called by running Freebayes 1.0.2 (ref. 36) on the genome BAM produced by Cell Ranger."
- [discussion] SNV inclusion filters: "Only SNVs with support from at least two cell barcodes, with a minimal SNV Qual score >=30, minimal SNV base Qual>=1 were included."
- [discussion] K=2 model selection criterion: "We applied a requirement that 90% of the cells have a posterior probability >75% to select the K=2 model over the K=1 model."
- [discussion] K=1 detection limit: "Selecting K=1 indicates that the mixture fraction is below the level of detection of the method, which in in silico mixing experiments was determined to be 4% of 6,000 cells."
- [abstract] Chimerism application finding: "we used sequence variation in the transcriptome data to determine host and donor chimerism at single-cell resolution from bone marrow mononuclear cells isolated from transplant patients."
- [readme] Cell Ranger pipeline role: "Cell Ranger is a set of analysis pipelines that perform sample demultiplexing, barcode processing, single cell 3' and 5' gene counting"
