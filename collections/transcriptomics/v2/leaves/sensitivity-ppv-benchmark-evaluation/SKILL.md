---
name: sensitivity-ppv-benchmark-evaluation
description: Use when quantifying the detection limit of a genotype/chimerism assignment
  method — e.g. how low a minor donor fraction SNV-based assignment can detect in
  single-cell UMI count data.
when_to_use_negative:
- Input is a gene expression feature matrix rather than allele-level SNV UMI counts
  — this skill benchmarks genotype assignment, not expression deconvolution
- SNVs have not passed the genotype-assignment filters (>=2 cell barcodes, SNV Qual
  >=30, base Qual >=1); unfiltered calls will inflate error rates
- The goal is calling chimerism in a real clinical sample rather than establishing
  a detection limit via synthetic/in silico mixtures
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Cell Ranger Single-Cell Software Suite
  role: Performs sample demultiplexing, barcode processing and single-cell 3' gene
    counting that produces the barcode/UMI count matrices the benchmark consumes
  repo: https://github.com/10XGenomics/cellranger
- name: STAR
  role: Aligns Read1 cDNA inserts to the reference genome (hg19) to enable downstream
    SNV calling
- name: Freebayes 1.0.2
  role: Calls SNVs on the Cell Ranger genome BAM; these variants become the allele-level
    inputs for mixture benchmarking
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/ncomms14049
    title: Massively parallel digital transcriptional profiling of single cells
  source_content_hash: sha256:885621afd28146b3faf6c92eef2e27e8f4e478391aa0305e14e602875e45a75d
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
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# sensitivity-ppv-benchmark-evaluation

## Summary

Benchmark the SNV-based single-cell genotype-assignment method by constructing in silico donor mixtures, running Gibbs-sampled K=1/K=2 model comparisons, and tabulating minor-genotype detection sensitivity and positive predictive value across mixing ratios.

## When to use

Use when quantifying the detection limit of a genotype/chimerism assignment method — e.g. how low a minor donor fraction SNV-based assignment can detect in single-cell UMI count data.

## When NOT to use

- Input is a gene expression feature matrix rather than allele-level SNV UMI counts — this skill benchmarks genotype assignment, not expression deconvolution
- SNVs have not passed the genotype-assignment filters (>=2 cell barcodes, SNV Qual >=30, base Qual >=1); unfiltered calls will inflate error rates
- The goal is calling chimerism in a real clinical sample rather than establishing a detection limit via synthetic/in silico mixtures

## Inputs

- Allele-level UMI counts per cell barcodes for filtered donor SNVs (support >=2 cell barcodes, SNV Qual >=30, base Qual >=1)
- Single-cell allele-count profiles for each donor separately (e.g. Donor B and Donor C PBMCs)
- Observed wet-lab mixture datasets at known ratios (50:50, 90:10, 99:1)

## Outputs

- Per-ratio table of sensitivity, positive predictive value, and selected model (K=1/K=2), e.g. mixing_sensitivity_ppv.csv
- Per-mixture K=2 model-selection decision based on the 90% posterior-probability>75% rule
- Estimated minor-genotype detection limit (fraction of total cells)

## How to apply

Load allele-level UMI counts per cell for SNVs that passed genotype-assignment filters (support from at least two cell barcodes, SNV Qual >=30, SNV base Qual >=1). Pool single-cell allele-count profiles from two donors at multiple minor-fraction ratios (0–50%) plus observed wet-lab mixes (50:50, 90:10, 99:1). For each mixture, fit both K=1 and K=2 models with a Gibbs sampler and select K=2 over K=1 only when 90% of cells have posterior probability >75%; otherwise report the mixture as below the detection level. Compute sensitivity and positive predictive value for minor-genotype detection at each ratio, writing per-ratio results to a table such as mixing_sensitivity_ppv.csv. Selecting K=1 indicates the mixture fraction is below the method's level of detection (4% of 6,000 cells in in silico mixing for this dataset).

## Related tools

- **Cell Ranger Single-Cell Software Suite** (Performs sample demultiplexing, barcode processing and single-cell 3' gene counting that produces the barcode/UMI count matrices the benchmark consumes) — https://github.com/10XGenomics/cellranger
- **STAR** (Aligns Read1 cDNA inserts to the reference genome (hg19) to enable downstream SNV calling)
- **Freebayes 1.0.2** (Calls SNVs on the Cell Ranger genome BAM; these variants become the allele-level inputs for mixture benchmarking)

## Examples

```
python run_mixing_benchmark.py --alleles allele_umi_counts.csv --ratios 0-0.5 --models 1 2 --min_posterior 0.75 --out mixing_sensitivity_ppv.csv
```

## Evaluation signals

- Sensitivity and PPV must both exceed 95% for minor-genotype fractions down to 3%
- The 99:1 wet-lab mix should yield K=1 selection, consistent with a reported detection limit of 4% of 6,000 cells in in silico mixing
- K=2 model is selected only when 90% of cells have posterior probability >75%; ratios failing this rule are reported as below detection level
- mixing_sensitivity_ppv.csv contains one row per in silico ratio (0–50% minor fraction) plus the three wet-lab mixes with sensitivity, PPV, and K=1/K=2 columns

## Limitations

- Minor populations could not be detected below a 3% mixing ratio; K=1 selection only means the fraction is below the ~4% (of 6,000 cells) detection level, not that no minor population exists
- Selecting K=1 is an absence-of-evidence result, limiting sensitivity interpretation for low-level chimerism
- The method was demonstrated in a proof-of-principle setting and is not clinically validated; wet-lab mixing datasets are required to confirm in silico estimates

## Evidence

- [discussion] K=2 selection rule: "We applied a requirement that 90% of the cells have a posterior probability >75% to select the K=2 model over the K=1 model."
- [discussion] detection limit via in silico mixing: "Selecting K=1 indicates that the mixture fraction is below the level of detection of the method, which in in silico mixing experiments was determined to be 4% of 6,000 cells."
- [discussion] SNV genotype-assignment filters: "Only SNVs with support from at least two cell barcodes, with a minimal SNV Qual score >=30, minimal SNV base Qual>=1 were included."
- [readme] cellranger availability: "The officially supported release binaries are available at <https://www.10xgenomics.com/support/software/cell-ranger/downloads>."
