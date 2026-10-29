---
name: gibbs-sampler-two-genome-mixture-modeling
description: Use when single-cell allele-level UMI counts at filtered SNVs are available
  and the goal is to detect a minor genotype, quantify mixing fraction, or decide
  K=1 vs K=2 in in silico or wet-lab chimerism mixtures.
when_to_use_negative:
- Input cells have not been genotyped via SNV calling — the model requires allele-level
  UMI counts at filtered donor SNVs, not raw expression matrices
- The mixture fraction is expected far below ~3–4% of profiled cells — K=1 selection
  would only mean the fraction is below the method's detection level, not that no
  minor population exists
- The analysis goal is gene-expression clustering (e.g. Seurat PBMC clustering) rather
  than genotype-based chimerism assignment
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_0622
tools:
- name: Cell Ranger Single-Cell Software Suite
  role: Produces the sample demultiplexing, barcode processing and single-cell 3'
    gene counting output from which allele-level UMI counts are derived
  repo: https://github.com/10XGenomics/cellranger
- name: STAR
  role: Aligns Read1 (cDNA insert) to the reference genome upstream of SNV calling
- name: Freebayes 1.0.2
  role: Calls SNVs on the Cell Ranger genome BAM; these SNVs feed the genotype-assignment
    filter and the mixture model
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/ncomms14049
    title: Massively parallel digital transcriptional profiling of single cells
  source_content_hash: sha256:9dd2802ccc1dbdbacf86138c8066bbd852817c041fa74ac7e42b664db61b28f9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/10XGenomics/cellranger
  tools:
  - Cell Ranger Single-Cell Software Suite
  - Freebayes 1.0.2
  - STAR
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# gibbs-sampler-two-genome-mixture-modeling

## Summary

A Gibbs-sampler-based two-genome (K=1 vs K=2) mixture model that fits per-cell allele-level UMI counts at donor SNVs to detect and quantify minor-genotype populations in chimeric single-cell RNA-seq mixtures, e.g. Donor B/C PBMC mixing experiments.

## When to use

Use when single-cell allele-level UMI counts at filtered SNVs are available and the goal is to detect a minor genotype, quantify mixing fraction, or decide K=1 vs K=2 in in silico or wet-lab chimerism mixtures.

## When NOT to use

- Input cells have not been genotyped via SNV calling — the model requires allele-level UMI counts at filtered donor SNVs, not raw expression matrices
- The mixture fraction is expected far below ~3–4% of profiled cells — K=1 selection would only mean the fraction is below the method's detection level, not that no minor population exists
- The analysis goal is gene-expression clustering (e.g. Seurat PBMC clustering) rather than genotype-based chimerism assignment

## Inputs

- Allele-level UMI counts per cell at SNVs passing genotype-assignment filters (>=2 cell barcodes, SNV Qual >=30, SNV base Qual >=1)
- Single-cell allele-count profiles from pure donor samples (e.g. Donor B and Donor C PBMCs) for constructing in silico mixtures
- Wet-lab mixture datasets (e.g. 50:50, 90:10, 99:1 donor mixes) for validation

## Outputs

- Per-mixture selected model (K=1 or K=2)
- Posterior probability of K=2 assignment per cell
- Per-ratio minor-genotype sensitivity and positive predictive value table (e.g. mixing_sensitivity_ppv.csv)

## How to apply

Start from allele-level UMI counts per cell for donor SNVs that pass the genotype-assignment filters (support from at least two cell barcodes, SNV Qual >=30, SNV base Qual >=1). Construct in silico mixtures by pooling single-cell allele-count profiles across a ratio series (here 12 minor-fraction ratios from 0–50%) and compare against observed wet-lab 50:50, 90:10, and 99:1 mixes. For each mixture, run the Gibbs sampler fitting K=1 (single genotype) and K=2 (two-genome mixture) models to the allele UMI counts. Select K=2 over K=1 only when 90% of the cells have a posterior probability >75%; otherwise report the mixture as below the detection level. Compute sensitivity and positive predictive value for the minor genotype at each ratio, and tabulate sensitivity, PPV, and the selected model per ratio (e.g. to mixing_sensitivity_ppv.csv). The rationale is that the K=2 posterior criterion enforces cell-level confidence across the population before declaring a second genotype, preventing spurious minor-genotype calls from stochastic allelic dropout.

## Related tools

- **Cell Ranger Single-Cell Software Suite** (Produces the sample demultiplexing, barcode processing and single-cell 3' gene counting output from which allele-level UMI counts are derived) — https://github.com/10XGenomics/cellranger
- **STAR** (Aligns Read1 (cDNA insert) to the reference genome upstream of SNV calling)
- **Freebayes 1.0.2** (Calls SNVs on the Cell Ranger genome BAM; these SNVs feed the genotype-assignment filter and the mixture model)

## Evaluation signals

- Sensitivity and PPV for the minor genotype exceed 95% for in silico minor-fraction ratios down to 3%
- Minor populations are not detected below a 3% mixing ratio (K=1 selected)
- The observed 99:1 wet mix yields K=1 selection, consistent with the reported in silico detection limit of 4% of 6,000 cells
- K=2 model is selected only when 90% of cells have posterior probability >75%
- Output table per ratio contains sensitivity, PPV, and selected model (K=1/K=2), written to mixing_sensitivity_ppv.csv

## Limitations

- Selecting K=1 only indicates the mixture fraction is below the level of detection (determined as 4% of 6,000 cells in in silico mixing experiments); it does not prove absence of a minor population
- The proof-of-principle chimerism study contained BMMCs from only two transplant patients; larger cohorts were not yet performed
- Clinical chimerism testing with this method was described as aspirational and not yet clinically validated

## Evidence

- [discussion] Only SNVs with support from at least two cell barcodes, with a minimal SNV Qual score >=30, minimal SNV base Qual>=1 were included.: "Only SNVs with support from at least two cell barcodes, with a minimal SNV Qual score >=30, minimal SNV base Qual>=1 were included."
- [discussion] 90% posterior criterion: "We applied a requirement that 90% of the cells have a posterior probability >75% to select the K=2 model over the K=1 model."
- [discussion] detection limit: "Selecting K=1 indicates that the mixture fraction is below the level of detection of the method, which in in silico mixing experiments was determined to be 4% of 6,000 cells."
- [readme] readme repo pointer: [evidence span withheld — anchor status: external_doc]
