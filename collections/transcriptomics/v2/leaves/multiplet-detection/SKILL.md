---
name: multiplet-detection
description: Use when demultiplexing HTO-labeled pooled scRNA-seq droplets with demuxmix
  and droplets must be classified as singlets of a specific sample or as multiplets
  via posterior probabilities, with uncertain assignments discarded.
when_to_use_negative:
- Input data are not oligonucleotide/HTO-barcoded droplet counts (e.g. cells were
  not labeled with hashtag oligonucleotides before pooling), since the method demultiplexes
  based on HTO count matrices.
- The complete per-gene transcription profile is available instead of genes-detected-per-droplet
  — pass only the total number of genes detected per droplet as the covariate, not
  the full RNA profile.
- A non-probabilistic, hard-assignment demultiplexer without error probabilities is
  needed and uncertainty cannot be discarded by a posterior-probability threshold.
edam_operation: http://edamontology.org/operation_0310
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3697
tools:
- name: demuxmix
  role: R/Bioconductor package that fits negative binomial (regression) mixture models
    per HTO via the EM algorithm and performs probabilistic classification of droplets,
    including multiplet detection, error-rate estimation, and QC to discard uncertain
    droplets
  repo: https://doi.org/doi:10.18129/B9.bioc.demuxmix
- name: R
  role: Execution environment; demuxmix is installed via BiocManager::install and
    loaded with library(demuxmix)
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btad481
    title: 'demuxmix: demultiplexing oligonucleotide-barcoded single-cell RNA sequencing
      data with regression mixture models'
  source_content_hash: sha256:811fc55c7a49f573ccdbb22a3ad241c212c64eed5a39e727dd8833e523435989
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - demuxmix
  edam_operation: http://edamontology.org/operation_0310
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# multiplet-detection

## Summary

Identify multiplets (droplets containing cells from more than one sample) during HTO-based demultiplexing of pooled single-cell RNA-seq experiments by fitting negative binomial regression mixture models per hashtag oligonucleotide and assigning droplets by posterior probability, including a multiplet class.

## When to use

Use when demultiplexing HTO-labeled pooled scRNA-seq droplets with demuxmix and droplets must be classified as singlets of a specific sample or as multiplets via posterior probabilities, with uncertain assignments discarded.

## When NOT to use

- Input data are not oligonucleotide/HTO-barcoded droplet counts (e.g. cells were not labeled with hashtag oligonucleotides before pooling), since the method demultiplexes based on HTO count matrices.
- The complete per-gene transcription profile is available instead of genes-detected-per-droplet — pass only the total number of genes detected per droplet as the covariate, not the full RNA profile.
- A non-probabilistic, hard-assignment demultiplexer without error probabilities is needed and uncertainty cannot be discarded by a posterior-probability threshold.

## Inputs

- (HTOs x droplets) hashtag oligonucleotide (HTO) count matrix from the sequencing library
- Vector of the number of genes detected per droplet (from the paired (genes x droplets) RNA count matrix)
- Number of HTOs (n), used for the pAcpt acceptance threshold (default 0.9^n)

## Outputs

- Per-droplet demultiplexing assignments to sample of origin or multiplet class
- Posterior probabilities and error probabilities per droplet assignment (used to discard uncertain droplets)
- Estimated demultiplexing error rates; per-droplet assignments with posterior probabilities written to demuxmix_results.csv

## How to apply

Load the (HTOs x droplets) HTO count matrix and the number of genes detected per droplet into R, then run demuxmix's per-HTO preprocessing on the HTO counts. Fit negative binomial mixture models per HTO via the EM algorithm, fitting both the naive model and the regression model that uses genes-detected-per-droplet as a covariate; the regression model is preferred because of the positive association between detected genes and HTO counts, which explains variance in HTO data and improves assignments. Select the model minimizing expected classification error between the naive and regression variants. Classify each droplet to its sample of origin (or to the multiplet class) by posterior probability, applying the acceptance threshold pAcpt (default 0.9^n, n = number of HTOs), and discard droplets whose assignment error probabilities exceed the threshold.

## Related tools

- **demuxmix** (R/Bioconductor package that fits negative binomial (regression) mixture models per HTO via the EM algorithm and performs probabilistic classification of droplets, including multiplet detection, error-rate estimation, and QC to discard uncertain droplets) — https://doi.org/doi:10.18129/B9.bioc.demuxmix
- **R** (Execution environment; demuxmix is installed via BiocManager::install and loaded with library(demuxmix))

## Examples

```
library(demuxmix); dmm <- demuxmix(htoCounts, nGeneVec = nGenesDetected); cls <- classify(object = dmm, pAcpt = 0.9^numHtos)
```

## Evaluation signals

- Every droplet receives a posterior probability with its assignment to a sample of origin or to the multiplet class; assignments lacking probabilities indicate the mixture fit failed.
- Uncertain droplets are discarded when posterior acceptance under pAcpt (default 0.9^n for n HTOs) is not met — verify the multiplet/singlet call report matches this threshold.
- Model selection step minimizes expected classification error between the naive and regression mixture models; confirm the regression model (genes-detected covariate) was fitted and compared, not only the naive model.
- Estimated demultiplexing error rates are reported, and QC outputs list dropped droplets; error rates near zero for all droplets suggest the posterior probabilities were not actually used.
- Output file demuxmix_results.csv contains one row per droplet with assignment and posterior probability, matching the (HTOs x droplets) input droplet count.

## Limitations

- Droplets with uncertain assignments (posterior probability below pAcpt, default 0.9^n) are discarded rather than forced into a class, so low-confidence droplets are lost from the output.
- Improvement of the regression model relies on the positive association between number of genes detected per droplet and HTO counts; better results are achieved by passing only the total number of genes detected, not the complete transcription profile.
- The acceptance threshold depends on the number of HTOs (0.9^n), so multiplet-detection stringency changes with experimental design (number of hashtags).

## Evidence

- [readme] The package implements methods to fit regression mixture models for a probabilistic classification of cells, including multiplet detection.: "The package implements methods to fit regression mixture models for a probabilistic classification of cells, including multiplet detection."
- [abstract] Error probabilities for discarding uncertain droplets.: "provides error probabilities for droplet assignments that can be used to discard uncertain droplets"
- [intro] Genes-detected-per-droplet as covariate leverages the positive association with HTO counts.: "better results are often achieved if the total number of genes detected per droplet (not the complete transcription profile) is passed to the method along with the HTO counts to leverage the positive"
- [intro] The demuxmix package demultiplexes based on HTO counts using negative binomial regression mixture models.: "The *demuxmix* package implements a method to demultiplex droplets based on HTO counts using negative binomial regression mixture models."
