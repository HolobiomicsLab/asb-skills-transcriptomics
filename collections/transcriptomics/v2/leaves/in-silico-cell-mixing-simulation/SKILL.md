---
name: in-silico-cell-mixing-simulation
description: Use when quantifying how low a minor donor genotype an SNV-based chimerism
  method can detect, by simulating cell mixtures from single-cell allele UMI counts
  and comparing model selection (K=1 vs K=2) and detection metrics per mixing ratio.
when_to_use_negative:
- Input is a gene expression matrix without SNV allele counts per cell — genotype
  assignment cannot be run, so simulated mixing is meaningless
- The goal is de novo chimerism determination on a single patient sample rather than
  benchmarking detection performance against known mixing fractions
- Sample has fewer than the 6,000-cell scale used to calibrate the detection limit,
  where the 3-4% minor-fraction threshold is not calibrated
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0092
tools:
- name: Cell Ranger Single-Cell Software Suite
  role: Produces the genome BAM and single-cell 3' gene counting that underlie the
    allele-level UMI counts used for mixture simulation
  repo: https://github.com/10XGenomics/cellranger
- name: STAR
  role: Aligns Read1 (cDNA insert) to the reference genome prior to SNV calling for
    donor genotypes
- name: Freebayes 1.0.2
  role: Calls the donor SNVs from the Cell Ranger genome BAM; only SNVs with at least
    two cell barcodes, SNV Qual >=30 and base Qual >=1 enter the mixing simulation
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/ncomms14049
    title: Massively parallel digital transcriptional profiling of single cells
  source_content_hash: sha256:1bb88e12a8d715d5f20271a26b2f61ad04254470e74419dd943399f70a5683d0
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
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# in-silico-cell-mixing-simulation

## Summary

A benchmarking skill that pools single-cell allele-level UMI count profiles from two donors (e.g., Donors B and C PBMCs) into in silico mixtures to measure the sensitivity and positive predictive value of SNV-based genotype assignment across minor-population fractions, and to establish the detection limit of the K=1/K=2 Gibbs-sampler mixture model.

## When to use

Use when quantifying how low a minor donor genotype an SNV-based chimerism method can detect, by simulating cell mixtures from single-cell allele UMI counts and comparing model selection (K=1 vs K=2) and detection metrics per mixing ratio.

## When NOT to use

- Input is a gene expression matrix without SNV allele counts per cell — genotype assignment cannot be run, so simulated mixing is meaningless
- The goal is de novo chimerism determination on a single patient sample rather than benchmarking detection performance against known mixing fractions
- Sample has fewer than the 6,000-cell scale used to calibrate the detection limit, where the 3-4% minor-fraction threshold is not calibrated

## Inputs

- Allele-level UMI counts per cell for donor SNVs passing genotype-assignment filters (>=2 cell barcodes, SNV Qual >=30, SNV base Qual >=1)
- Single-cell allele-count profiles from purified Donor B and Donor C PBMCs
- Observed wet-lab mixing datasets at 50:50, 90:10, and 99:1 donor ratios

## Outputs

- mixing_sensitivity_ppv.csv: per-ratio table of minor-genotype sensitivity, positive predictive value, and selected model (K=1 or K=2)
- Estimated detection limit of the minor genotype (reportable minimum minor fraction)

## How to apply

Load allele-level UMI counts per cell for the donor SNVs that passed genotype-assignment filters (support from at least two cell barcodes, SNV Qual >=30, SNV base Qual >=1). Construct in silico mixtures by pooling Donor B and Donor C single-cell allele-count profiles at 12 minor-fraction ratios spanning 0-50%, alongside the observed wet-lab 50:50, 90:10, and 99:1 mix datasets. For each mixture, run the Gibbs sampler fitting both K=1 (single genotype) and K=2 (two-genome mixture) models, and select K=2 over K=1 only when 90% of the cells have a posterior probability >75%; otherwise report the mixture as below the detection level. Compute sensitivity and positive predictive value for detecting the minor genotype at each ratio and write the per-ratio table (sensitivity, PPV, selected model) to mixing_sensitivity_ppv.csv. The rationale is that in silico pooling at known fractions provides ground-truth minor-population labels that wet-lab mixtures cannot, letting you calibrate the K=1/K=2 decision rule against a known detection limit (~3-4% minor fraction of 6,000 cells).

## Related tools

- **Cell Ranger Single-Cell Software Suite** (Produces the genome BAM and single-cell 3' gene counting that underlie the allele-level UMI counts used for mixture simulation) — https://github.com/10XGenomics/cellranger
- **STAR** (Aligns Read1 (cDNA insert) to the reference genome prior to SNV calling for donor genotypes)
- **Freebayes 1.0.2** (Calls the donor SNVs from the Cell Ranger genome BAM; only SNVs with at least two cell barcodes, SNV Qual >=30 and base Qual >=1 enter the mixing simulation)

## Evaluation signals

- Minor-genotype sensitivity and PPV exceed 95% for in silico minor fractions down to 3%
- Minor populations are not detected below a 3% mixing ratio (reported as below detection level)
- The 99:1 wet mix yields K=1 model selection, consistent with the reported detection limit of 4% of 6,000 cells in in silico mixing
- K=2 is selected only when 90% of cells have a posterior probability >75%; mixtures failing this criterion are labeled below detection
- mixing_sensitivity_ppv.csv contains one row per ratio across 12 in silico ratios spanning 0-50% plus the three wet-mix datasets, with sensitivity, PPV, and K=1/K=2 columns

## Limitations

- Selecting K=1 only indicates the mixture fraction is below the level of detection (~3% minor / 4% of 6,000 cells), so low-level chimerism below this threshold is unresolvable
- Minor populations cannot be detected below a 3% mixing ratio, so performance claims are bounded by this floor
- Wet-lab mixtures are limited to 50:50, 90:10, and 99:1 ratios; finer low-fraction validation relies on in silico pooling assumptions (Poisson cell loading and ~50% capture efficiency) rather than physical mixtures

## Evidence

- [discussion] K=2 selection requires 90% of cells posterior >75%: "We applied a requirement that 90% of the cells have a posterior probability >75% to select the K=2 model over the K=1 model."
- [discussion] SNV genotype-assignment filters: "Only SNVs with support from at least two cell barcodes, with a minimal SNV Qual score >=30, minimal SNV base Qual>=1 were included."
- [discussion] Detection limit rationale: "Selecting K=1 indicates that the mixture fraction is below the level of detection of the method, which in in silico mixing experiments was determined to be 4% of 6,000 cells."
- [readme] Data availability for reproducing mixing analyses: [evidence span withheld — anchor status: not_found]
