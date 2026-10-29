---
name: snv-based-chimerism-genotype-assignment
description: Use when post-transplant bone marrow (BMMC) or donor-mix scRNA-seq data
  are available as a Cell Ranger genome BAM, and you need per-cell host/donor genotype
  calls and mixture fractions from de novo SNVs.
when_to_use_negative:
- Input lacks per-cell allele UMI structure (e.g. bulk sequencing or pre-aggregated
  VCFs) — the method needs single-cell SNV fractions to fit genotype mixtures.
- Clinical chimerism reporting is required — the method is described as proof-of-principle
  ('in the near future'), not yet clinically validated.
- The variant of interest involves indels or structural variation — SNVs are called
  with indels ignored.
edam_topics:
- http://edamontology.org/topic_0625
tools:
- name: STAR
  role: Aligns Read1 cDNA inserts from scRNA-seq reads to the hg19 reference genome
    as the first step of the pipeline.
- name: Cell Ranger Single-Cell Software Suite
  role: Performs sample demultiplexing, barcode processing (posterior >=0.975 correction),
    and single-cell 3' gene counting to produce the genome BAM used for SNV calling.
  repo: 10XGenomics/cellranger
- name: Freebayes 1.0.2
  role: Calls SNVs de novo on the Cell Ranger genome BAM, ignoring indels, to generate
    the allele-level SNV candidates for genotyping.
- name: Seurat
  role: Takes the log-transformed gene-cell-barcode matrix to annotate cell types
    by correlation against reference populations for per-genotype composition analysis.
  repo: https://github.com/10XGenomics/single-cell-3prime-paper
provenance:
  source_task_ids:
  - task_005
  - task_006
  source_papers:
  - doi: 10.1038/ncomms14049
    title: Massively parallel digital transcriptional profiling of single cells
  source_content_hash: sha256:8d7bf84ff639a38f595f6cead87890740182d1b5bb7ed7ba3e262977bf1121d8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/10XGenomics/cellranger
  tools:
  - Cell Ranger Single-Cell Software Suite
  - Freebayes 1.0.2
  - STAR
  - Seurat
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0625
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# snv-based-chimerism-genotype-assignment

## Summary

De novo SNV-based Bayesian genotype assignment to determine host/donor chimerism at single-cell resolution from scRNA-seq allele UMI counts, without prior genotype knowledge, by fitting K=1 vs K=2 genotype mixtures with a Gibbs sampler.

## When to use

Use when post-transplant bone marrow (BMMC) or donor-mix scRNA-seq data are available as a Cell Ranger genome BAM, and you need per-cell host/donor genotype calls and mixture fractions from de novo SNVs.

## When NOT to use

- Input lacks per-cell allele UMI structure (e.g. bulk sequencing or pre-aggregated VCFs) — the method needs single-cell SNV fractions to fit genotype mixtures.
- Clinical chimerism reporting is required — the method is described as proof-of-principle ('in the near future'), not yet clinically validated.
- The variant of interest involves indels or structural variation — SNVs are called with indels ignored.

## Inputs

- scRNA-seq FASTQ files (e.g. SRA accession SRP073767)
- Cell Ranger genome BAM from single-cell 3' counting
- Allele-level UMI counts per cell for filtered SNVs
- In silico mixture allele-count profiles (e.g. Donor B/C PBMC pools at 12 minor-fraction ratios, 0–50%)
- Wet-lab mixed-sample datasets (50:50, 90:10, 99:1)

## Outputs

- Per-cell host/donor genotype assignments (with multiplet flags)
- Selected model per sample (K=1 vs K=2) and genotype mixture fractions (e.g. AML027 post: 13.8% minor / 86.2% major)
- Sensitivity and PPV per minor-genotype ratio table (mixing_sensitivity_ppv.csv)
- Per-cell genotype and cell-type composition table (aml_transplant_chimerism_results.csv)

## How to apply

Align Read1 cDNA inserts to hg19 with STAR and process with the Cell Ranger Single-Cell Software Suite (barcode correction only when posterior probability >=0.975) to obtain the genome BAM and gene-cell-barcode matrices. Call SNVs de novo by running Freebayes 1.0.2 on the genome BAM, ignoring indels, and retain only SNVs with support from at least two cell barcodes, SNV Qual >=30, and minimal SNV base Qual >=1. Load allele-level UMI counts per cell, then fit both K=1 and K=2 genotype-mixture models with the Gibbs sampler, selecting K=2 only when 90% of cells have posterior probability >75%; select K=1 (below detection level) otherwise. Flag cells with SNV fraction between 0.2 and 0.8 from either genotype as multiplets, and report minor-genotype fraction, sensitivity, and PPV across mixing ratios (in silico 0–50% plus wet-lab 50:50, 90:10, 99:1 mixes).

## Related tools

- **STAR** (Aligns Read1 cDNA inserts from scRNA-seq reads to the hg19 reference genome as the first step of the pipeline.)
- **Cell Ranger Single-Cell Software Suite** (Performs sample demultiplexing, barcode processing (posterior >=0.975 correction), and single-cell 3' gene counting to produce the genome BAM used for SNV calling.) — 10XGenomics/cellranger
- **Freebayes 1.0.2** (Calls SNVs de novo on the Cell Ranger genome BAM, ignoring indels, to generate the allele-level SNV candidates for genotyping.)
- **Seurat** (Takes the log-transformed gene-cell-barcode matrix to annotate cell types by correlation against reference populations for per-genotype composition analysis.) — https://github.com/10XGenomics/single-cell-3prime-paper

## Examples

```
freebayes -f hg19.fa cellranger_genome_bam.bam --no-indels
```

## Evaluation signals

- In silico mixing: minor-genotype detection sensitivity and PPV must exceed 95% for minor fractions down to 3%.
- The 99:1 wet-lab mix should yield K=1 selection, consistent with the reported detection limit of 4% of 6,000 cells.
- K=2 must be selected only when 90% of cells have posterior probability >75%; otherwise report the mixture as below detection level.
- AML027 post-transplant should reproduce a K=2 call with minor genotype near 13.8%, major genotype ~97% similar to host, matching the clinical 12% donor chimerism.
- All retained SNVs satisfy the filter set: at least two cell barcodes, SNV Qual >=30, minimal SNV base Qual >=1.

## Limitations

- Selecting K=1 only indicates the mixture fraction is below the detection level (4% of 6,000 cells); minor populations below ~3% mixing ratio cannot be detected.
- The proof-of-principle chimerism study included only two transplant patients; larger cohorts were not yet performed.
- Cells with SNV fraction between 0.2 and 0.8 from either genotype are ambiguous (multiplets) and flagged rather than confidently assigned.
- Clinical chimerism testing with this method is aspirational ('in the near future') and not yet clinically validated.

## Evidence

- [discussion] SNV filters: "Only SNVs with support from at least two cell barcodes, with a minimal SNV Qual score >=30, minimal SNV base Qual>=1 were included."
- [discussion] Model selection rule: "We applied a requirement that 90% of the cells have a posterior probability >75% to select the K=2 model over the K=1 model."
- [discussion] Detection limit: "Selecting K=1 indicates that the mixture fraction is below the level of detection of the method, which in in silico mixing experiments was determined to be 4% of 6,000 cells."
- [discussion] SNV calling: "SNVs were called by running Freebayes 1.0.2 (ref. 36) on the genome BAM produced by Cell Ranger."
- [discussion] Multiplet definition: "Cells with a fraction of SNV from either species between 0.2 and 0.8 are considered multiplets."
- [readme] Cell Ranger scope: "Cell Ranger is a set of analysis pipelines that perform sample demultiplexing, barcode processing, single cell 3' and 5' gene counting"
