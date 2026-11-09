---
name: kmeans-initial-clustering-of-hto-droplets
description: Use when preparing an (HTOs x droplets) count matrix for demuxmix demultiplexing
  and unsupervised initial class labels are needed to seed the mixture model for each
  HTO, or to flag outliers before model fitting.
when_to_use_negative:
- Input is a final demultiplexed classification rather than raw HTO counts — this
  skill only produces initialization labels and outlier flags, not the final demuxmix
  assignment.
- The data contain no hashtag oligonucleotide (HTO) counts (e.g., only a genes x droplets
  RNA matrix), since clustering is performed on log-transformed HTO counts per HTO.
- Droplets have already been excluded by upstream QC and you only need the final probabilistic
  classification including multiplet detection — go directly to the regression mixture
  model fitting step.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: demuxmix
  role: R/Bioconductor package whose preprocessing step performs the per-HTO k-means
    (k = 2) initialization and IQR-based outlier flagging before negative binomial
    regression mixture model fitting
  repo: https://doi.org/doi:10.18129/B9.bioc.demuxmix
- name: R
  role: Runtime environment in which the (HTOs x droplets) count matrix and per-droplet
    detected-gene counts are loaded via SingleCellExperiment/DelayedArray and k-means
    clustering is executed
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btad481
    title: 'demuxmix: demultiplexing oligonucleotide-barcoded single-cell RNA sequencing
      data with regression mixture models'
  source_content_hash: sha256:a75f53efc979cddf7d3540aaa4512195eba04b2c17bded71f3cc73869e2776ee
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - demuxmix
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 4
---

# k-means initial clustering of HTO droplets

## Summary

Initializes positive/negative droplet labels per hashtag oligonucleotide (HTO) by k-means clustering (k = 2) on log-transformed HTO counts, with Q3 + 1.5*IQR outlier flagging on HTO counts and detected genes, as the preprocessing step that seeds demuxmix's negative binomial regression mixture model fitting.

## When to use

Use when preparing an (HTOs x droplets) count matrix for demuxmix demultiplexing and unsupervised initial class labels are needed to seed the mixture model for each HTO, or to flag outliers before model fitting.

## When NOT to use

- Input is a final demultiplexed classification rather than raw HTO counts — this skill only produces initialization labels and outlier flags, not the final demuxmix assignment.
- The data contain no hashtag oligonucleotide (HTO) counts (e.g., only a genes x droplets RNA matrix), since clustering is performed on log-transformed HTO counts per HTO.
- Droplets have already been excluded by upstream QC and you only need the final probabilistic classification including multiplet detection — go directly to the regression mixture model fitting step.

## Inputs

- (HTOs x droplets) HTO count matrix (e.g., SingleCellExperiment/DelayedArray object loadable in R)
- Per-droplet number of detected genes from the (genes x droplets) RNA read count matrix

## Outputs

- hto_preprocessing_summary.csv: per-droplet table with initial k-means cluster assignment (positive/negative per HTO) and outlier flag (Q3 + 1.5*IQR rule on HTO counts or detected genes)
- Initial class labels used to initialize demuxmix mixture model fitting

## How to apply

Load the (HTOs x droplets) count matrix and the per-droplet number of detected genes into R via the SingleCellExperiment/DelayedArray interfaces of the demuxmix package. For each hashtag oligonucleotide (HTO) individually, log-transform its count vector across droplets and run k-means clustering with k = 2; take the cluster with the larger mean HTO count as the positive cluster containing droplets tagged by that HTO. Separately compute outlier thresholds as the third quartile plus 1.5 times the interquartile range (Q3 + 1.5*IQR), once for HTO counts and once for the number of detected genes per droplet; flag droplets exceeding either threshold as outliers. Flagged droplets are excluded from model fitting but still remain to be classified downstream. Write the per-droplet table containing the initial cluster assignment and the outlier flag to hto_preprocessing_summary.csv, which seeds the subsequent negative binomial regression mixture model fitting.

## Related tools

- **demuxmix** (R/Bioconductor package whose preprocessing step performs the per-HTO k-means (k = 2) initialization and IQR-based outlier flagging before negative binomial regression mixture model fitting) — https://doi.org/doi:10.18129/B9.bioc.demuxmix
- **R** (Runtime environment in which the (HTOs x droplets) count matrix and per-droplet detected-gene counts are loaded via SingleCellExperiment/DelayedArray and k-means clustering is executed)

## Examples

```
if (!require("BiocManager")) install.packages("BiocManager"); BiocManager::install("demuxmix"); library(demuxmix)
```

## Evaluation signals

- hto_preprocessing_summary.csv contains a cluster assignment and an outlier flag for every droplet (no missing rows).
- k-means produces exactly two clusters (k = 2) per HTO, and the cluster with the larger mean HTO count is labeled positive.
- Every flagged droplet exceeds the Q3 + 1.5*IQR threshold on HTO counts or on the number of detected genes per droplet.
- Flagged droplets are excluded from model fitting but are retained in the table for later classification.
- The pipeline is rule-based QC only: the paper reports no quantitative acceptance metric beyond these invariants.

## Limitations

- The paper reports no quantitative acceptance metric for the preprocessing beyond the rule-based Q3 + 1.5*IQR outlier check.
- Outlier-flagged droplets are excluded from model fitting but still need to be classified downstream, so this step alone does not finalize droplet labels.
- Initial labels are per-HTO k-means clusters on log-transformed counts only; the final demultiplexing quality depends on the subsequent regression mixture model, which additionally leverages the number of genes detected per droplet.

## Evidence

- [methods] k-means initialization and IQR outlier rule: [evidence span withheld — anchor status: not_found]
- [intro] package scope: "The *demuxmix* package implements a method to demultiplex droplets based on HTO counts using negative binomial regression mixture models."
- [intro] genes-detected covariate rationale: "better results are often achieved if the total number of genes detected per droplet (not the complete transcription profile) is passed to the method along with the HTO counts"
- [readme] README package description: "The package implements methods to fit regression mixture models for a probabilistic classification of cells, including multiplet detection."
