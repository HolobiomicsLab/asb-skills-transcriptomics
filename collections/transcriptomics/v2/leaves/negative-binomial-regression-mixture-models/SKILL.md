---
name: negative-binomial-regression-mixture-models
description: Use when you need to assign droplets from HTO-labeled pooled scRNA-seq/snRNA-seq
  experiments to their sample of origin (including multiplet detection) from an (HTOs
  x droplets) count matrix, optionally with per-droplet genes-detected counts as a
  covariate.
when_to_use_negative:
- The experiment uses genetic demultiplexing (e.g. demuxlet/freemuxlet sample-genotype
  calls) rather than HTO oligonucleotide barcoding — no HTO count matrix is available
  for mixture fitting
- Only full gene-expression profiles are available and you intend to pass them to
  the method — the article specifies passing the total number of genes detected per
  droplet, not the complete transcription profile
- Droplets are already demultiplexed and only a downstream quality metric like the
  overlap score is needed without refitting the mixture models
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: demuxmix
  role: R/Bioconductor package that preprocesses HTO counts, fits the naive and regression
    negative binomial mixture models via EM, selects between them by expected classification
    error, and performs posterior-probability classification with multiplet detection
  repo: https://doi.org/doi:10.18129/B9.bioc.demuxmix
- name: R
  role: Runtime environment in which the demuxmix package vignette workflows (e.g.
    demuxmix(hto, numGenes) fitting and classification) are executed
provenance:
  source_task_ids:
  - task_008
  - task_001
  - task_003
  - task_005
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btad481
    title: 'demuxmix: demultiplexing oligonucleotide-barcoded single-cell RNA sequencing
      data with regression mixture models'
  source_content_hash: sha256:a2037ffcb8b521705c0918438c8484591f2d75560c68fe944806adf336c2a0d9
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DemuxEM
  - GMM-Demux
  - HTODemux
  - MULTI-seq
  - R
  - demuxmix
  - hashedDrops
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# negative-binomial-regression-mixture-models

## Summary

Fits negative binomial regression mixture models to per-droplet hashtag oligonucleotide (HTO) counts to probabilistically demultiplex pooled single-cell RNA-seq data, exploiting the positive association between HTO counts and the number of detected genes. This is the core statistical method implemented in the demuxmix R/Bioconductor package.

## When to use

Use when you need to assign droplets from HTO-labeled pooled scRNA-seq/snRNA-seq experiments to their sample of origin (including multiplet detection) from an (HTOs x droplets) count matrix, optionally with per-droplet genes-detected counts as a covariate.

## When NOT to use

- The experiment uses genetic demultiplexing (e.g. demuxlet/freemuxlet sample-genotype calls) rather than HTO oligonucleotide barcoding — no HTO count matrix is available for mixture fitting
- Only full gene-expression profiles are available and you intend to pass them to the method — the article specifies passing the total number of genes detected per droplet, not the complete transcription profile
- Droplets are already demultiplexed and only a downstream quality metric like the overlap score is needed without refitting the mixture models

## Inputs

- HTO count matrix (HTOs x droplets) from oligonucleotide-labeled pooled single-cell sequencing
- Per-droplet number of detected genes vector (RNA library covariate for the regression model)
- Acceptance probability threshold pAcpt (default 0.9^n; e.g. 0.99 for single-HTO designs)

## Outputs

- Per-droplet demultiplexing assignments (sample of origin or multiplet) with posterior probabilities
- Droplets flagged as uncertain whose assignment error probability exceeds pAcpt
- Demultiplexing error rate estimates and QC metrics such as the overlap score (ovs) in (0, 1]

## How to apply

Follow demuxmix's three-step workflow: (1) preprocess the HTO count data per hashtag oligonucleotide; (2) fit negative binomial mixture models per HTO via the EM algorithm — both a naive model and a regression model using genes-detected-per-droplet as a covariate — then select the model minimizing expected classification error; (3) classify each droplet by posterior probability, discarding droplets whose assignment error probability exceeds the acceptance threshold pAcpt (default 0.9^n, where n is the number of HTOs; e.g. pAcpt=0.99 was used for the one-HTO CSF/PBMC analysis). The regression model is preferred over the naive model because the positive association between detected genes and HTO counts explains variance and improves droplet assignments. Model fits and posterior component probabilities can additionally be used to compute the overlap score (ovs, Equation 5) — the shared probability mass of the negative and positive components, weighted by mean detected genes for regression models and unweighted for naive models — as a per-HTO quality metric standardized to (0, 1].

## Related tools

- **demuxmix** (R/Bioconductor package that preprocesses HTO counts, fits the naive and regression negative binomial mixture models via EM, selects between them by expected classification error, and performs posterior-probability classification with multiplet detection) — https://doi.org/doi:10.18129/B9.bioc.demuxmix
- **R** (Runtime environment in which the demuxmix package vignette workflows (e.g. demuxmix(hto, numGenes) fitting and classification) are executed)

## Examples

```
if (!require("BiocManager")) install.packages("BiocManager"); BiocManager::install("demuxmix")
```

## Evaluation signals

- Every droplet receives a sample-of-origin or multiplet label together with a posterior probability, and droplets with error probability above pAcpt are classified as uncertain/removed (e.g. demuxmix removed 261 cells, 11.1%, at pAcpt=0.99 on the CSF/PBMC data)
- Regression-model fits must use the weighted mean detected-genes values, while naive-model quantities omit the weighting (checkable when computing the overlap score)
- All computed overlap scores (ovs) fall within (0, 1] after standardization
- Against genetic ground truth (freemuxlet on CSF/PBMC), regression-model metrics reach approximately specificity 0.985, sensitivity 0.980, versus naive-model 0.983/0.972 — the regression model should not underperform the naive model
- Model selection between naive and regression models minimizes expected classification error; the regression model should be retained when the genes-detected covariate carries signal

## Limitations

- Uncertain droplets are discarded rather than assigned, so a fraction of cells (6.3% for demuxmix versus up to 12.8% for other methods on the simulated human brain benchmark) is lost when HTO signal is attenuated
- Dataset quality varies: published GEO HTO datasets showed overlap scores spanning benchmark-level quality down to levels matching simulated low-quality data (small HTO-signal scaling factors), so method choice and performance depend on HTO data quality
- In designs with a single HTO (non-tagged cells pooled with tagged cells), the acceptance threshold and the benefit of the gene-count covariate determine how many true singlets are removed as uncertain

## Evidence

- [abstract] A new demultiplexing method based on negative binomial regression mixture models is introduced.: "A new demultiplexing method based on negative binomial regression mixture models is introduced."
- [abstract] demuxmix utilizes the positive association between detected genes in the RNA library and HTO counts to explain parts of the variance in the HTO data: "demuxmix utilizes the positive association between detected genes in the RNA library and HTO counts to explain parts of the variance in the HTO data"
- [intro] better results are often achieved if the total number of genes detected per droplet (not the complete transcription profile) is passed to the method along with the HTO counts: "better results are often achieved if the total number of genes detected per droplet (not the complete transcription profile) is passed to the method along with the HTO counts to leverage the positive"
- [readme] The package implements methods to fit regression mixture models for a probabilistic classification of cells, including multiplet detection. Demultiplexing error rates can be estimated, and methods: "The package implements methods to fit regression mixture models for a probabilistic classification of cells, including multiplet detection. Demultiplexing error rates can be estimated, and methods"
