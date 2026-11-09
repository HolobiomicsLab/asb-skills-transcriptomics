---
name: expected-classification-error-model-selection
description: Use when demultiplexing HTO-labeled scRNA-seq droplets with demuxmix
  and both a naive NB mixture and a genes-detected regression mixture have been fit
  per HTO, so the model with the lower expected classification error must be chosen
  for classification.
when_to_use_negative:
- The input is not an (HTOs x droplets) hashtag-oligonucleotide count matrix — demuxmix's
  NB mixture models are specific to HTO count data.
- The number of genes detected per droplet cannot be supplied and comparison of the
  regression model against the naive model is not feasible.
- Droplets are being demultiplexed by sample genotype or SNP data rather than oligonucleotide
  barcodes.
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: demuxmix
  role: Fits the naive and regression negative binomial mixture models per HTO, computes
    expected classification error for model selection, and classifies droplets by
    posterior probability
  repo: https://doi.org/doi:10.18129/B9.bioc.demuxmix
- name: R
  role: Runtime environment for the demuxmix R/Bioconductor package and its EM fitting
    and classification routines
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btad481
    title: 'demuxmix: demultiplexing oligonucleotide-barcoded single-cell RNA sequencing
      data with regression mixture models'
  source_content_hash: sha256:96fa7594e6a55d0a6014e13570d403ea20a9e37f7b83d2ed4cb7b2e64ee624f4
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - demuxmix
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# Expected classification error model selection

## Summary

Selects between demuxmix's two negative binomial mixture models per HTO — the naive model and the regression model using genes-detected-per-droplet as covariate — by minimizing the expected classification error, before using the winning model's posteriors to demultiplex droplets.

## When to use

Use when demultiplexing HTO-labeled scRNA-seq droplets with demuxmix and both a naive NB mixture and a genes-detected regression mixture have been fit per HTO, so the model with the lower expected classification error must be chosen for classification.

## When NOT to use

- The input is not an (HTOs x droplets) hashtag-oligonucleotide count matrix — demuxmix's NB mixture models are specific to HTO count data.
- The number of genes detected per droplet cannot be supplied and comparison of the regression model against the naive model is not feasible.
- Droplets are being demultiplexed by sample genotype or SNP data rather than oligonucleotide barcodes.

## Inputs

- Per-HTO fitted negative binomial mixture models from demuxmix (naive and regression variants, fit by EM)
- HTO count matrix (HTOs x droplets) after per-HTO preprocessing
- Number of genes detected per droplet (regression covariate)

## Outputs

- Selected mixture model per HTO (naive or regression) minimizing expected classification error
- Per-droplet demultiplexing assignments with posterior probabilities, uncertain droplets discarded below pAcpt (default 0.9^n)

## How to apply

After preprocessing the (HTOs x droplets) HTO count matrix per hashtag oligonucleotide, fit two negative binomial mixture models per HTO via the EM algorithm: the naive model on HTO counts alone, and the regression model that uses the number of genes detected per droplet as covariate to explain variance in HTO counts (demuxmix leverages the positive association between detected genes and HTO counts). For each HTO, compute the expected classification error of both fitted models and select the model with the smaller expected error — this is the explicit model-selection decision point of the demuxmix workflow (preprocessing -> mixture fitting -> classification). Use the selected models' posterior probabilities to classify each droplet to its sample of origin or to a multiplet, applying the acceptance threshold pAcpt (default 0.9^n, n = number of HTOs) and discarding droplets whose assignment error probability exceeds it. Rationale: the regression model improves droplet assignments when the genes-detected association holds, but model selection by expected classification error avoids assuming it a priori for every HTO.

## Related tools

- **demuxmix** (Fits the naive and regression negative binomial mixture models per HTO, computes expected classification error for model selection, and classifies droplets by posterior probability) — https://doi.org/doi:10.18129/B9.bioc.demuxmix
- **R** (Runtime environment for the demuxmix R/Bioconductor package and its EM fitting and classification routines)

## Examples

```
if (!require("BiocManager")) install.packages("BiocManager"); BiocManager::install("demuxmix"); library(demuxmix)
```

## Evaluation signals

- Both the naive and the regression mixture model were fit per HTO (two EM fits per hashtag oligonucleotide), not just one.
- The model selected per HTO is the one with the smaller expected classification error.
- Classification posterior probabilities are reported alongside assignments, and droplets below the acceptance threshold pAcpt (0.9^n) are discarded rather than assigned.
- The regression model uses the number of genes detected per droplet as its covariate, matching the genes-detected column loaded with the HTO matrix.

## Limitations

- Droplets whose assignment error probabilities exceed pAcpt are discarded, so noisy or low-HTO-count droplets may receive no sample assignment.
- The regression model's benefit depends on the positive association between detected genes and HTO counts; the naive model must be fit as the fallback comparison in case the association is absent for a given HTO.
- demuxmix relies on cells being labeled with hashtag oligonucleotides before pooling; experimental designs without HTO labeling fall outside its scope.

## Evidence

- [intro] regression model fits HTO counts with covariate: "better results are often achieved if the total number of genes detected per droplet (not the complete transcription profile) is passed to the method along with the HTO counts"
- [abstract] probabilistic classification with error probabilities: "provides error probabilities for droplet assignments that can be used to discard uncertain droplets"
- [full_text] three-step workflow defines the selection step: [evidence span withheld — anchor status: not_found]
- [readme] package scope from README: "The package implements methods to fit regression mixture models for a probabilistic classification of cells, including multiplet detection."
