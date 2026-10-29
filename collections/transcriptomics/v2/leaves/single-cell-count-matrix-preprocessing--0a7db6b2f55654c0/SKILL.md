---
name: single-cell-count-matrix-preprocessing
description: Use when pooled scRNA-seq/snRNA-seq droplets have been labeled with hashtag
  oligonucleotides and you need to load HTO and RNA count matrices, derive per-droplet
  detected-gene totals, and check HTO-vs-gene correlations before demultiplexing.
when_to_use_negative:
- Input is already demultiplexed cell assignments or a demuxmix classification output
  rather than raw HTO/RNA count matrices
- The experiment uses no hashtag oligonucleotide labeling (e.g. genotype-based or
  lipid-tag demultiplexing), so no HTO count matrix exists to correlate with detected
  genes
- Only full transcription profiles (not count summaries) are available and per-droplet
  detected-gene totals cannot be computed for the regression covariate
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: demuxmix
  role: R/Bioconductor package providing the regression mixture model framework whose
    covariate (per-droplet detected genes) is computed here and used for HTO-based
    demultiplexing
  repo: https://doi.org/doi:10.18129/B9.bioc.demuxmix
- name: R
  role: Environment in which the HTO and RNA count matrices are loaded, per-droplet
    detected-gene totals and HTO-gene correlations are computed, and results are exported
    to CSV
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad481
    title: 'demuxmix: demultiplexing oligonucleotide-barcoded single-cell RNA sequencing
      data with regression mixture models'
  source_content_hash: sha256:170ed77734f64960104a8a76161cf844ead12de3a76ace548a07ea5c23731075
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - demuxmix
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# single-cell-count-matrix-preprocessing

## Summary

Prepare (HTOs x droplets) HTO count matrices and (genes x droplets) RNA count matrices from hashtag-oligonucleotide-labeled, pooled single-cell experiments, and compute per-droplet covariates such as the total number of detected genes so they can be passed to demuxmix regression mixture models.

## When to use

Use when pooled scRNA-seq/snRNA-seq droplets have been labeled with hashtag oligonucleotides and you need to load HTO and RNA count matrices, derive per-droplet detected-gene totals, and check HTO-vs-gene correlations before demultiplexing.

## When NOT to use

- Input is already demultiplexed cell assignments or a demuxmix classification output rather than raw HTO/RNA count matrices
- The experiment uses no hashtag oligonucleotide labeling (e.g. genotype-based or lipid-tag demultiplexing), so no HTO count matrix exists to correlate with detected genes
- Only full transcription profiles (not count summaries) are available and per-droplet detected-gene totals cannot be computed for the regression covariate

## Inputs

- (HTOs x droplets) HTO count matrix for each labeled sample
- (genes x droplets) RNA read count matrix for each sample
- Human brain snRNA-seq dataset, cell line mixture dataset (GEO GSM3501446, GSM3501447), and CSF dataset count matrices

## Outputs

- Per-droplet total number of detected genes (covariate vector for demuxmix)
- Per-sample correlation table of tagging-HTO and background-HTO counts vs detected genes
- hto_gene_correlations.csv with columns dataset, sample, HTO type (tagging/background), correlation coefficient, and reference-range match

## How to apply

Load the (HTOs x droplets) HTO count matrix and the (genes x droplets) RNA read count matrix for each sample in R, using the demuxmix package installed via BiocManager. For each droplet, compute the total number of genes detected per droplet — this is the covariate demuxmix uses alongside HTO counts to leverage the positive association between detected genes and HTO counts. Per sample, compute the Pearson correlation between tagging HTO counts and detected genes (expected ~0.67-0.86 in the human brain snRNA-seq dataset) and between background HTO counts and detected genes (expected ~0.07-0.13 in the brain dataset; 0.48 for tagged PBMCs and 0.18 for non-tagged CSF cells in the CSF dataset; weaker in the cell line mixture dataset GEO GSM3501446/GSM3501447). Assemble these correlations into a table (dataset, sample, HTO type, coefficient, match to reference range) written to hto_gene_correlations.csv. A substantially positive tagging correlation justifies passing detected-gene counts to demuxmix to explain variance in HTO data and improve droplet assignments.

## Related tools

- **demuxmix** (R/Bioconductor package providing the regression mixture model framework whose covariate (per-droplet detected genes) is computed here and used for HTO-based demultiplexing) — https://doi.org/doi:10.18129/B9.bioc.demuxmix
- **R** (Environment in which the HTO and RNA count matrices are loaded, per-droplet detected-gene totals and HTO-gene correlations are computed, and results are exported to CSV)

## Examples

```
BiocManager::install("demuxmix"); library(demuxmix); nGenes <- colSums(rnaCounts > 0); cor(htoCounts[hto, ], nGenes)
```

## Evaluation signals

- Brain snRNA-seq tagging-HTO vs detected-gene correlations fall within the reported 0.67-0.86 range in every sample
- Brain background-HTO correlations fall within the reported 0.07-0.13 range
- CSF dataset correlations match the reported values: tagged PBMCs r = 0.48, non-tagged CSF cells r = 0.18
- Tagging correlations are substantially larger than background correlations within each sample, consistent with the positive association demuxmix exploits
- Output CSV hto_gene_correlations.csv contains dataset, sample, HTO type, correlation coefficient, and reference-range match for every sample in all three datasets

## Limitations

- The strength of the HTO-vs-gene association is dataset dependent: the cell line mixture dataset shows a weaker correlation than the brain dataset, so the covariate adds less information there
- Background HTO counts show only weak correlations (0.07-0.13 in brain), so correlations computed on background HTOs should not be used to justify the regression covariate
- Non-labeled cells pooled with labeled cells (as in the CSF design) show lower tagging correlations (0.18 for CSF cells), limiting how much variance detected genes can explain

## Evidence

- [abstract] demuxmix utilizes the positive association between detected genes in the RNA library and HTO counts to explain parts of the variance in the HTO data resulting in improved droplet assignments: "demuxmix utilizes the positive association between detected genes in the RNA library and HTO counts to explain parts of the variance in the HTO data"
- [intro] The demuxmix package implements a method to demultiplex droplets based on HTO counts using negative binomial regression mixture models.: "The *demuxmix* package implements a method to demultiplex droplets based on HTO counts using negative binomial regression mixture models."
- [intro] better results are often achieved if the total number of genes detected per droplet (not the complete transcription profile) is passed to the method along with the HTO counts: "better results are often achieved if the total number of genes detected per droplet (not the complete transcription profile) is passed to the method along with the HTO counts"
- [intro] These oligonucleotides are termed hashtag oligonucleotides (HTOs) and are sequenced together with the RNA molecules of the cells resulting in an (HTOs x droplets) count matrix: "These oligonucleotides are termed hashtag oligonucleotides (HTOs) and are sequenced together with the RNA molecules of the cells resulting in an (HTOs x droplets) count matrix"
- [readme] BiocManager::install("demuxmix"): "The package is available at [Bioconductor](https://bioconductor.org)
and can be installed via BiocManager::install"
