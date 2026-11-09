---
name: single-cell-demultiplexing-preprocessing
description: Use when preparing an (HTOs x droplets) count matrix plus per-droplet
  detected-gene counts for regression mixture model demultiplexing, and initial class
  labels or outlier flags are needed before model fitting.
when_to_use_negative:
- Input lacks an HTO count matrix (e.g. only gene-by-droplet RNA counts), so no per-HTO
  labeling exists to cluster
- Goal is the final probabilistic demultiplexing/classification itself rather than
  initialization and QC of its inputs
- Droplets have already been classified and the task is to discard uncertain assignments
  using error probabilities
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: demuxmix
  role: R/Bioconductor package providing SingleCellExperiment/DelayedArray interfaces
    for loading HTO counts and implementing the regression mixture model whose fitting
    this preprocessing initializes
- name: R
  role: Execution environment for the k-means clustering and IQR-based outlier flagging
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btad481
    title: 'demuxmix: demultiplexing oligonucleotide-barcoded single-cell RNA sequencing
      data with regression mixture models'
  source_content_hash: sha256:2d78a00be00eaeb3cf5e9fd099563c43f3a8a4f75ec2924a6ee8207356c6c17c
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - demuxmix
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# single-cell-demultiplexing-preprocessing

## Summary

Preprocessing step for HTO-based demultiplexing as implemented in demuxmix: per-HTO k-means (k = 2) clustering of log-transformed HTO counts to obtain initial positive/negative class labels, plus Q3 + 1.5*IQR outlier flagging on HTO counts and detected genes to exclude noisy droplets from model fitting.

## When to use

Use when preparing an (HTOs x droplets) count matrix plus per-droplet detected-gene counts for regression mixture model demultiplexing, and initial class labels or outlier flags are needed before model fitting.

## When NOT to use

- Input lacks an HTO count matrix (e.g. only gene-by-droplet RNA counts), so no per-HTO labeling exists to cluster
- Goal is the final probabilistic demultiplexing/classification itself rather than initialization and QC of its inputs
- Droplets have already been classified and the task is to discard uncertain assignments using error probabilities

## Inputs

- (HTOs x droplets) HTO count matrix
- Per-droplet number of detected genes
- SingleCellExperiment / DelayedArray object holding HTO and RNA count data

## Outputs

- Per-droplet table with initial cluster assignment (positive/negative per HTO) and outlier flag
- hto_preprocessing_summary.csv

## How to apply

Load the (HTOs x droplets) HTO count matrix and the per-droplet number of detected genes into R via the SingleCellExperiment/DelayedArray interfaces provided by the demuxmix package. For each individual hashtag oligonucleotide (HTO), log-transform its count vector and run k-means clustering with k = 2; take the cluster with the larger mean HTO count as the positive cluster of droplets tagged by that HTO. Separately for HTO counts and for detected genes, compute outlier thresholds as the third quartile plus 1.5 times the interquartile range, and flag droplets exceeding either threshold as outliers. Outlier droplets are excluded from model fitting but are still to be classified downstream. The rationale is that the mixture model needs reasonable starting labels, and the positive association between detected genes and HTO counts means outliers on either dimension distort the fit.

## Related tools

- **demuxmix** (R/Bioconductor package providing SingleCellExperiment/DelayedArray interfaces for loading HTO counts and implementing the regression mixture model whose fitting this preprocessing initializes)
- **R** (Execution environment for the k-means clustering and IQR-based outlier flagging)

## Examples

```
BiocManager::install("demuxmix"); library(demuxmix)  # then load HTO counts and genes-detected via SingleCellExperiment before per-HTO k-means (k=2) and Q3+1.5*IQR flagging
```

## Evaluation signals

- Output contains a cluster assignment and an outlier flag for every droplet
- Exactly two k-means clusters are produced per HTO
- The positive cluster is the one with the larger mean HTO count
- Every flagged outlier droplet exceeds the Q3 + 1.5*IQR threshold on HTO counts or on detected genes
- The paper reports no quantitative acceptance metric beyond this rule-based QC check

## Limitations

- The paper reports no quantitative acceptance metric beyond the rule-based QC check on cluster assignments and outlier flags
- Outlier droplets are excluded from model fitting yet must still be classified, so the preprocessing alone does not assign them
- The initialization relies on k-means with k = 2 per individual HTO, which assumes a separable positive/negative structure in log-transformed counts for each HTO

## Evidence

- [methods] Initial class labels via k-means with k=2 on log-transformed HTO counts: [evidence span withheld — anchor status: not_found]
- [methods] Outlier threshold definition: [evidence span withheld — anchor status: not_found]
- [abstract] Gene/HTO association rationale: "demuxmix utilizes the positive association between detected genes in the RNA library and HTO counts to explain parts of the variance in the HTO data"
- [readme] Package interface and purpose: "The package implements methods to fit regression mixture models for a probabilistic classification of cells, including multiplet detection."
- [intro] Data format produced by HTO sequencing: "sequenced together with the RNA molecules of the cells resulting in an (HTOs x droplets) count matrix in addition to the"
