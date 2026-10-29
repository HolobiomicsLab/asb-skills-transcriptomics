---
name: hto-count-matrix-preprocessing
description: Use when you have a pooled single-cell scRNA-seq experiment where cells
  were labeled with hashtag oligonucleotides (HTOs) and you hold an (HTOs x droplets)
  HTO count matrix that must be preprocessed per HTO before mixture-model demultiplexing.
when_to_use_negative:
- The input is a (genes x droplets) RNA expression count matrix only — demultiplexing
  requires the HTO oligonucleotide count matrix, not the transcription profile
- Cells were not labeled with barcode oligonucleotides before pooling (e.g., genotype-
  or SNP-based demultiplexing), which is outside demuxmix's HTO-based scope
- Droplets have already been demultiplexed and you only need downstream sample-level
  analysis of the assignments
tools:
- name: demuxmix
  role: R/Bioconductor package performing the per-HTO preprocessing, NB regression
    mixture fitting, and probabilistic classification of droplets
  repo: https://doi.org/doi:10.18129/B9.bioc.demuxmix
- name: R
  role: Runtime environment in which the demuxmix package is installed and the preprocessing
    workflow is executed
- name: BiocManager
  role: Installation channel for the Bioconductor demuxmix package
  repo: https://bioconductor.org
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btad481
    title: 'demuxmix: demultiplexing oligonucleotide-barcoded single-cell RNA sequencing
      data with regression mixture models'
  source_content_hash: sha256:219f38ecdc0f48eef7cf91bcd0e5842541978356a67dd151e054ab122aec6a9a
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BiocManager
  - R
  - demuxmix
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# hto-count-matrix-preprocessing

## Summary

Per-HTO preprocessing of the (HTOs x droplets) hashtag oligonucleotide count matrix, the first step of the demuxmix three-step demultiplexing workflow (preprocessing, NB mixture fitting, classification). It prepares each HTO's count distribution for negative binomial mixture model fitting.

## When to use

Use when you have a pooled single-cell scRNA-seq experiment where cells were labeled with hashtag oligonucleotides (HTOs) and you hold an (HTOs x droplets) HTO count matrix that must be preprocessed per HTO before mixture-model demultiplexing.

## When NOT to use

- The input is a (genes x droplets) RNA expression count matrix only — demultiplexing requires the HTO oligonucleotide count matrix, not the transcription profile
- Cells were not labeled with barcode oligonucleotides before pooling (e.g., genotype- or SNP-based demultiplexing), which is outside demuxmix's HTO-based scope
- Droplets have already been demultiplexed and you only need downstream sample-level analysis of the assignments

## Inputs

- (HTOs x droplets) hashtag oligonucleotide (HTO) count matrix
- Number of genes detected per droplet, derived from the (genes x droplets) RNA read count matrix

## Outputs

- Per-HTO preprocessed HTO count data ready for negative binomial mixture model fitting
- Downstream per-droplet demultiplexing assignments with posterior probabilities (e.g., written to demuxmix_results.csv)

## How to apply

Load the (HTOs x droplets) HTO count matrix into R along with the number of genes detected per droplet (computed from the (genes x droplets) RNA read count matrix), since passing genes-detected-per-droplet leverages the positive association between detected genes and HTO counts. Run demuxmix's per-HTO preprocessing independently on the HTO count data for each hashtag oligonucleotide, producing per-HTO distributions from which negative binomial mixture models (naive and regression variants) can be fit via the EM algorithm. After preprocessing, the pipeline proceeds to mixture fitting, model selection minimizing expected classification error, and posterior-probability classification with acceptance threshold pAcpt (default 0.9^n, n = number of HTOs). Correctness is judged by whether preprocessed per-HTO data support well-separated positive/negative droplet components and, downstream, low expected classification error and confident assignments for most droplets.

## Related tools

- **demuxmix** (R/Bioconductor package performing the per-HTO preprocessing, NB regression mixture fitting, and probabilistic classification of droplets) — https://doi.org/doi:10.18129/B9.bioc.demuxmix
- **R** (Runtime environment in which the demuxmix package is installed and the preprocessing workflow is executed)
- **BiocManager** (Installation channel for the Bioconductor demuxmix package) — https://bioconductor.org

## Examples

```
BiocManager::install("demuxmix"); library(demuxmix); browseVignettes("demuxmix")
```

## Evaluation signals

- Input matrix has HTOs as rows and droplets as columns, and the genes-detected-per-droplet vector has length equal to the number of droplets
- Preprocessing is applied once per hashtag oligonucleotide (per-HTO), not on the pooled HTO matrix
- Preprocessed per-HTO distributions yield separable components when naive and regression negative binomial mixtures are fit via EM
- Model selection step reports minimized expected classification error between the naive and regression models
- Downstream classification applies the acceptance threshold pAcpt (default 0.9^n, n = number of HTOs) and discards droplets below it as uncertain

## Limitations

- Uncertain droplets are discarded rather than assigned, controlled by the acceptance threshold pAcpt (default 0.9^n), so low-quality droplets yield no sample assignment
- Improved assignments depend on passing the total number of genes detected per droplet (not the complete transcription profile) alongside HTO counts; without this covariate only the naive model is available
- Preprocessing quality depends on the positive association between detected genes and HTO counts; experiments where this association is weak may benefit less from the regression model

## Evidence

- [intro] demuxmix implements a method to demultiplex droplets based on HTO counts using negative binomial regression mixture models: "The *demuxmix* package implements a method to demultiplex droplets based on HTO counts using negative binomial regression mixture models."
- [intro] HTOs sequenced together with RNA produce an (HTOs x droplets) count matrix: "These oligonucleotides are termed hashtag oligonucleotides (HTOs) and are sequenced together with the RNA molecules of the cells resulting in an (HTOs x droplets) count matrix"
- [intro] Passing genes detected per droplet with HTO counts leverages positive association: "better results are often achieved if the total number of genes detected per droplet (not the complete transcription profile) is passed to the method along with the HTO counts to leverage the positive"
- [readme] demuxmix is a package for demultiplexing single-cell sequencing experiments of pooled cells labeled with barcode oligonucleotides: "demuxmix is a package for demultiplexing single-cell sequencing experiments
of pooled cells labeled with barcode oligonucleotides."
- [abstract] Error probabilities allow discarding uncertain droplets: "provides error probabilities for droplet assignments that can be used to discard uncertain droplets"
