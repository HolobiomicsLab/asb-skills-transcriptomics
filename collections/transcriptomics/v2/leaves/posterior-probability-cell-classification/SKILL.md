---
name: posterior-probability-cell-classification
description: Use when demultiplexing pooled single-cell RNA-seq experiments whose
  cells were labeled with hashtag oligonucleotides (HTOs), and per-droplet sample
  assignments with error probabilities are required.
when_to_use_negative:
- Input is a single-sample experiment with no HTO labeling — there is no oligonucleotide
  count matrix to model
- Only the full transcription profile per cell is available but the total number of
  genes detected per droplet cannot be passed along, so the regression model's key
  covariate is missing
- The task is cell hashing demultiplexing based on something other than HTO oligonucleotide
  counts (e.g., genotype-based demultiplexing)
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_0621
- http://edamontology.org/topic_3170
tools:
- name: demuxmix
  role: R/Bioconductor package that preprocesses per-HTO counts, fits negative binomial
    (regression) mixture models via EM, performs model selection, and classifies droplets
    by posterior probability with multiplet detection
  repo: https://doi.org/doi:10.18129/B9.bioc.demuxmix
- name: R
  role: Runtime environment in which the demuxmix demultiplexing workflow is executed
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btad481
    title: 'demuxmix: demultiplexing oligonucleotide-barcoded single-cell RNA sequencing
      data with regression mixture models'
  source_content_hash: sha256:ae0c76bf3c80d057de5102bcf618f97ce8a3ae219a14ce18be7bb536c8edbced
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - demuxmix
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_0621
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# posterior-probability-cell-classification

## Summary

A probabilistic droplet-classification step from the demuxmix package that assigns each droplet to its sample of origin (or multiplet) via posterior probabilities from negative binomial regression mixture models, and discards uncertain droplets using an acceptance threshold.

## When to use

Use when demultiplexing pooled single-cell RNA-seq experiments whose cells were labeled with hashtag oligonucleotides (HTOs), and per-droplet sample assignments with error probabilities are required.

## When NOT to use

- Input is a single-sample experiment with no HTO labeling — there is no oligonucleotide count matrix to model
- Only the full transcription profile per cell is available but the total number of genes detected per droplet cannot be passed along, so the regression model's key covariate is missing
- The task is cell hashing demultiplexing based on something other than HTO oligonucleotide counts (e.g., genotype-based demultiplexing)

## Inputs

- (HTOs x droplets) HTO count matrix
- (genes x droplets) RNA read count matrix or the derived number of genes detected per droplet
- Number of HTOs (n) used to set the pAcpt acceptance threshold

## Outputs

- Per-droplet demultiplexing assignments to sample of origin or multiplet class
- Posterior probabilities / error probabilities for each droplet assignment
- Demultiplexing error-rate estimates (e.g., demuxmix_results.csv)

## How to apply

Load the (HTOs x droplets) HTO count matrix and the number of genes detected per droplet into R, then run demuxmix's per-HTO preprocessing followed by EM fitting of negative binomial mixture models per HTO — fitting both the naive model and the regression model that uses genes-detected-per-droplet as covariate. Select between the naive and regression models by minimizing expected classification error; the regression model exploits the positive association between detected genes and HTO counts to explain variance in HTO data. Classify each droplet to its sample of origin (or multiplet) by posterior probability, applying the acceptance threshold pAcpt (default 0.9^n, where n is the number of HTOs) to discard uncertain droplets. Write per-droplet assignments with posterior probabilities (e.g., to demuxmix_results.csv) so downstream users can filter by assignment error probability.

## Related tools

- **demuxmix** (R/Bioconductor package that preprocesses per-HTO counts, fits negative binomial (regression) mixture models via EM, performs model selection, and classifies droplets by posterior probability with multiplet detection) — https://doi.org/doi:10.18129/B9.bioc.demuxmix
- **R** (Runtime environment in which the demuxmix demultiplexing workflow is executed)

## Examples

```
if (!require("BiocManager")) install.packages("BiocManager"); BiocManager::install("demuxmix"); library(demuxmix)
```

## Evaluation signals

- Every droplet carries a sample-of-origin or multiplet label plus a posterior probability; uncertain droplets below pAcpt (default 0.9^n) are discarded rather than force-assigned
- Demultiplexing error rates estimated by the package are reported and within acceptable bounds for the experiment
- Fitting the regression model with genes-detected-per-droplet as covariate yields equal or lower expected classification error than the naive model after model selection
- Output table (e.g., demuxmix_results.csv) has one row per droplet with matching droplet identifiers to the input HTO count matrix
- Multiplet detections appear as a distinct class, not conflated with single-sample assignments

## Limitations

- Droplets whose assignment posterior probability falls below the pAcpt threshold are discarded, so low-quality or ambiguous droplets yield no assignment
- The regression model's benefit depends on being given the total number of genes detected per droplet; without it only the naive model is available
- Labeling cells with sample-specific hashtag oligonucleotides is an additional required experimental step before pooling

## Evidence

- [abstract] probabilistic classification with acceptance threshold: "provides error probabilities for droplet assignments that can be used to discard uncertain droplets"
- [intro] regression covariate rationale: "better results are often achieved if the total number of genes detected per droplet (not the complete transcription profile) is passed to the method along with the HTO counts"
- [methods] pipeline organization: "Demultiplexing oligonucleotide-labeled scRNA-seq data with demuxmix"
- [readme] package capability overview: "implements methods to fit regression mixture models for a probabilistic classification of cells, including multiplet detection. Demultiplexing error rates can be estimated"
- [readme] installation: "BiocManager::install("demuxmix")"
