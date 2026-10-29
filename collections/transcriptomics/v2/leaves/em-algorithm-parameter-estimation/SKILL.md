---
name: em-algorithm-parameter-estimation
description: Use when per-HTO droplet count distributions (e.g., an (HTOs x droplets)
  HTO count matrix) must be decomposed into unobserved signal/noise components, i.e.,
  mixture parameters and posteriors must be estimated before classifying droplets
  to samples.
when_to_use_negative:
- The data are plain gene expression counts without HTO/barcode oligonucleotide channels
  — there is no mixture over hashtag labels to decompose.
- Droplet-to-sample assignment is already known or the dataset is not pooled from
  labeled cells, so demultiplexing is unnecessary.
- Only the complete transcription profile (not total genes detected per droplet) is
  available and the naive mixture model suffices — but note the article indicates
  better results are often achieved with the genes-detected covariate.
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3517
- http://edamontology.org/topic_3308
tools:
- name: demuxmix
  role: R/Bioconductor package that fits the per-HTO negative binomial regression
    mixture models by EM and performs the probabilistic classification of droplets
  repo: https://doi.org/doi:10.18129/B9.bioc.demuxmix
- name: R
  role: Execution environment for installing and running the demuxmix package
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btad481
    title: 'demuxmix: demultiplexing oligonucleotide-barcoded single-cell RNA sequencing
      data with regression mixture models'
  source_content_hash: sha256:79af9189ba1d59c046a82a8e51a366cf9f6e40a1c2e649105f23ae8447bdb680
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - demuxmix
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3517
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# EM algorithm parameter estimation

## Summary

Fits mixture model parameters by expectation-maximization: in demuxmix, negative binomial mixture models are fitted per hashtag oligonucleotide (HTO) via the EM algorithm, either as a naive model or a regression model using the number of genes detected per droplet as covariate, enabling probabilistic droplet demultiplexing.

## When to use

Use when per-HTO droplet count distributions (e.g., an (HTOs x droplets) HTO count matrix) must be decomposed into unobserved signal/noise components, i.e., mixture parameters and posteriors must be estimated before classifying droplets to samples.

## When NOT to use

- The data are plain gene expression counts without HTO/barcode oligonucleotide channels — there is no mixture over hashtag labels to decompose.
- Droplet-to-sample assignment is already known or the dataset is not pooled from labeled cells, so demultiplexing is unnecessary.
- Only the complete transcription profile (not total genes detected per droplet) is available and the naive mixture model suffices — but note the article indicates better results are often achieved with the genes-detected covariate.

## Inputs

- (HTOs x droplets) HTO oligonucleotide count matrix
- Number of genes detected per droplet (for the regression model covariate)
- Number of HTOs (n), used to set the default acceptance threshold pAcpt = 0.9^n

## Outputs

- Fitted per-HTO negative binomial mixture model parameters (naive and regression variants)
- Per-droplet posterior probabilities over samples of origin and multiplet class
- Per-droplet demultiplexing assignments with posterior probabilities (e.g., demuxmix_results.csv)
- Estimated demultiplexing error rates

## How to apply

Load the (HTOs x droplets) HTO count matrix and the number of genes detected per droplet into R with the demuxmix package. Fit a negative binomial mixture model per HTO via the EM algorithm: fit both the naive model (HTO counts only) and the regression model that uses genes detected per droplet as a covariate, because the positive association between detected genes and HTO counts explains part of the variance in the HTO data and improves assignments. Select the model (naive vs. regression) that minimizes the expected classification error. Use the fitted models' posterior probabilities to classify each droplet to its sample of origin or to the multiplet class, discarding droplets below the acceptance threshold pAcpt (default 0.9^n for n HTOs) as uncertain.

## Related tools

- **demuxmix** (R/Bioconductor package that fits the per-HTO negative binomial regression mixture models by EM and performs the probabilistic classification of droplets) — https://doi.org/doi:10.18129/B9.bioc.demuxmix
- **R** (Execution environment for installing and running the demuxmix package)

## Examples

```
if (!require("BiocManager")) install.packages("BiocManager"); BiocManager::install("demuxmix"); library(demuxmix)
```

## Evaluation signals

- Both model variants (naive and regression with genes-detected covariate) are fitted per HTO and a model is selected by minimizing expected classification error, not by default choice.
- Every classified droplet has a posterior probability; droplets below pAcpt (default 0.9^n, n = number of HTOs) are discarded as uncertain rather than force-assigned.
- The demultiplexing output includes multiplet detection as an explicit class alongside sample assignments.
- Estimated demultiplexing error rates are reported and QC methods are applied before accepting assignments.
- Per-droplet assignments with posterior probabilities are written out (e.g., demuxmix_results.csv) matching the (HTOs x droplets) input droplets.

## Limitations

- Droplets with uncertain assignments (posterior below the pAcpt acceptance threshold) are discarded, reducing the number of classified cells.
- The regression model requires the number of genes detected per droplet; results differ depending on whether this covariate is passed alongside the HTO counts.
- Improved performance is assessed on real and simulated data but the method targets oligonucleotide-barcoded single-cell RNA-seq designs; feasibility of designs mixing non-labeled with labeled cells must be verified separately.

## Evidence

- [abstract] NB regression mixture fitting via EM: "A new demultiplexing method based on negative binomial regression mixture models is introduced. The method, called demuxmix, implements two significant improvements."
- [intro] Regression mixture models for probabilistic classification: "The package implements methods to fit regression mixture models for a probabilistic classification of cells, including multiplet detection."
- [intro] Genes-detected covariate improves assignments: "better results are often achieved if the total number of genes detected per droplet (not the complete transcription profile) is passed to the method along with the HTO counts"
- [abstract] Uncertain droplets discarded via error probabilities: "provides error probabilities for droplet assignments that can be used to discard uncertain droplets"
- [readme] Package scope from README: "Demultiplexing error rates can be estimated, and methods for quality control are provided."
