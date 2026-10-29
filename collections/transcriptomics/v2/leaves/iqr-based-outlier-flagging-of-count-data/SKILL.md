---
name: iqr-based-outlier-flagging-of-count-data
description: Use when initializing demuxmix model fitting on an (HTOs x droplets)
  count matrix and a per-droplet detected-gene vector, and droplets with extreme HTO
  counts or abnormally many detected genes must be flagged as outliers before mixture
  fitting.
when_to_use_negative:
- Input is not oligonucleotide-labeled scRNA-seq demultiplexing data (no HTO count
  matrix and no per-droplet detected-genes metric)
- The goal is downstream classification filtering by assignment error probability
  — that is the post-fitting step of discarding uncertain droplets, not this pre-fitting
  outlier flagging
- Droplets that fail the thresholds should be dropped entirely rather than retained
  for classification; demuxmix flags outliers as excluded from fitting but still classified
tools:
- name: demuxmix
  role: R/Bioconductor package whose preprocessing step performs the k-means initialization
    and IQR-based outlier flagging on HTO counts and detected genes
  repo: https://doi.org/doi:10.18129/B9.bioc.demuxmix
- name: R
  role: Runtime environment; count matrix loaded through SingleCellExperiment/DelayedArray
    interfaces
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btad481
    title: 'demuxmix: demultiplexing oligonucleotide-barcoded single-cell RNA sequencing
      data with regression mixture models'
  source_content_hash: sha256:8e5a73f8889c3eaaab0050f7cecd4c70d8c72ce21c827c0b0feb8d379a1a78d3
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - demuxmix
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 5
---

# iqr-based-outlier-flagging-of-count-data

## Summary

Rule-based outlier flagging used in demuxmix preprocessing: k-means (k=2) on log-transformed HTO counts yields initial positive/negative droplet classes, then droplets exceeding Q3 + 1.5*IQR on HTO counts or detected genes are flagged as outliers.

## When to use

Use when initializing demuxmix model fitting on an (HTOs x droplets) count matrix and a per-droplet detected-gene vector, and droplets with extreme HTO counts or abnormally many detected genes must be flagged as outliers before mixture fitting.

## When NOT to use

- Input is not oligonucleotide-labeled scRNA-seq demultiplexing data (no HTO count matrix and no per-droplet detected-genes metric)
- The goal is downstream classification filtering by assignment error probability — that is the post-fitting step of discarding uncertain droplets, not this pre-fitting outlier flagging
- Droplets that fail the thresholds should be dropped entirely rather than retained for classification; demuxmix flags outliers as excluded from fitting but still classified

## Inputs

- (HTOs x droplets) HTO count matrix (e.g., SingleCellExperiment/DelayedArray-backed)
- Per-droplet number of detected genes (RNA library metric, one value per droplet)

## Outputs

- Per-droplet table with initial k-means cluster assignment (positive/negative) per HTO
- Per-droplet outlier flag based on Q3 + 1.5*IQR thresholds on HTO counts and detected genes
- hto_preprocessing_summary.csv containing cluster assignment and outlier flag for every droplet

## How to apply

Load the (HTOs x droplets) count matrix and the per-droplet number of detected genes via the SingleCellExperiment/DelayedArray interfaces of the demuxmix R/Bioconductor package. For one hashtag oligonucleotide (HTO) at a time, log-transform its count vector and run k-means with k = 2; the cluster with the larger mean HTO count is the positive (tagged) cluster and the other the negative cluster, giving initial class labels for the regression mixture model. Independently compute outlier thresholds as the third quartile plus 1.5 times the interquartile range, once for the HTO counts and once for the number of detected genes per droplet. Flag any droplet exceeding either threshold as an outlier: such droplets are excluded from model fitting but still marked to be classified, so no droplet is silently dropped. Rationale: demuxmix leverages the positive association between detected genes and HTO counts, so droplets extreme on either axis distort the mixture fit and are quarantined before initialization.

## Related tools

- **demuxmix** (R/Bioconductor package whose preprocessing step performs the k-means initialization and IQR-based outlier flagging on HTO counts and detected genes) — https://doi.org/doi:10.18129/B9.bioc.demuxmix
- **R** (Runtime environment; count matrix loaded through SingleCellExperiment/DelayedArray interfaces)

## Examples

```
Rscript -e 'library(demuxmix); x <- log(hto_counts); km <- kmeans(x, centers=2); thr <- quantile(hto_counts, .75) + 1.5*IQR(hto_counts); outliers <- hto_counts > thr | genes_detected > quantile(genes_detected, .75) + 1.5*IQR(genes_detected)'
```

## Evaluation signals

- The output table contains a cluster assignment and an outlier flag for every droplet (no droplet is missing)
- Exactly two k-means clusters (k = 2) are produced per HTO on the log-transformed counts
- Every flagged droplet exceeds Q3 + 1.5*IQR on HTO counts or on detected genes; no unflagged droplet exceeds both thresholds
- The cluster with the larger mean HTO count is the one labeled positive for that HTO
- The per-droplet summary is written to hto_preprocessing_summary.csv with both required columns

## Limitations

- The paper reports no quantitative acceptance metric for this QC step beyond the rule-based check that flagged droplets exceed the Q3 + 1.5*IQR threshold
- This is a preprocessing/initialization heuristic; final demultiplexing quality depends on the subsequent negative binomial regression mixture fit, not on the k-means labels themselves
- Outlier droplets are excluded from model fitting, so very noisy libraries may push many droplets out of the fit and reduce information available for parameter estimation

## Evidence

- [abstract] demuxmix utilizes the positive association between detected genes in the RNA library and HTO counts to explain parts of the variance: "demuxmix utilizes the positive association between detected genes in the RNA library and HTO counts to explain parts of the variance"
- [intro] better results are often achieved if the total number of genes detected per droplet (not the complete transcription profile) is passed to the method along with the HTO counts: "better results are often achieved if the total number of genes detected per droplet (not the complete transcription profile) is passed to the method along with the HTO counts"
- [readme] Demultiplexing error rates can be estimated, and methods for quality control are provided.: "Demultiplexing error rates can be estimated, and methods for quality control are provided."
