---
name: hto-count-log-transformation
description: 'Use when initializing demuxmix''s regression mixture model: per-HTO
  count vectors from an (HTOs x droplets) matrix must be log-transformed and split
  into two clusters to seed positive/negative droplet labels before model fitting.'
when_to_use_negative:
- Droplets already have reliable demultiplexing labels from another tool — this step
  only produces *initial* labels for mixture model initialization.
- Data are RNA gene-expression counts rather than HTO oligonucleotide counts; the
  k=2 clustering and Q3 + 1.5*IQR thresholds are calibrated for HTO distributions.
- A full demuxmix classification (error probabilities, multiplet detection) is the
  goal without needing to re-derive initialization labels — call demuxmix directly
  instead.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: demuxmix
  role: R/Bioconductor package providing the preprocessing implementation (log transform,
    per-HTO k-means initialization, IQR-based outlier flagging) for its negative binomial
    regression mixture model demultiplexer
  repo: https://doi.org/doi:10.18129/B9.bioc.demuxmix
- name: R
  role: runtime environment for demuxmix; k-means and quartile/IQR computations are
    executed in R
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btad481
    title: 'demuxmix: demultiplexing oligonucleotide-barcoded single-cell RNA sequencing
      data with regression mixture models'
  source_content_hash: sha256:53ad0e6a5364093ba0760b1053c15901167e1e526f7867da69d8f0ba1d6504f2
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - demuxmix
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# hto-count-log-transformation

## Summary

Preprocessing step from demuxmix (Section 2.1.1) that log-transforms each HTO's count vector, applies k-means (k = 2) per HTO to obtain initial positive/negative droplet class labels, and flags outliers via Q3 + 1.5*IQR thresholds on HTO counts and detected genes.

## When to use

Use when initializing demuxmix's regression mixture model: per-HTO count vectors from an (HTOs x droplets) matrix must be log-transformed and split into two clusters to seed positive/negative droplet labels before model fitting.

## When NOT to use

- Droplets already have reliable demultiplexing labels from another tool — this step only produces *initial* labels for mixture model initialization.
- Data are RNA gene-expression counts rather than HTO oligonucleotide counts; the k=2 clustering and Q3 + 1.5*IQR thresholds are calibrated for HTO distributions.
- A full demuxmix classification (error probabilities, multiplet detection) is the goal without needing to re-derive initialization labels — call demuxmix directly instead.

## Inputs

- (HTOs x droplets) HTO count matrix (SingleCellExperiment/DelayedArray-backed)
- per-droplet number of detected genes (from the (genes x droplets) RNA count matrix)

## Outputs

- per-droplet table with initial positive/negative cluster assignment and outlier flag (hto_preprocessing_summary.csv)
- set of outlier droplets excluded from model fitting but still to be classified

## How to apply

Load the (HTOs x droplets) HTO count matrix and the per-droplet number of detected genes via the SingleCellExperiment/DelayedArray interfaces of the demuxmix R/Bioconductor package. For each individual HTO, log-transform its count vector and run k-means with k = 2; take the cluster with the larger mean HTO count as the positive cluster (droplets tagged by that HTO). Independently compute outlier thresholds as the third quartile plus 1.5 times the interquartile range, separately for HTO counts and for detected genes per droplet. Droplets exceeding either threshold are flagged as outliers: they are excluded from mixture model fitting but still classified afterwards. The rationale for log transformation is that it renders per-HTO count distributions amenable to two-component clustering that separates tagged from untagged droplets, supplying the initial class labels the mixture model needs.

## Related tools

- **demuxmix** (R/Bioconductor package providing the preprocessing implementation (log transform, per-HTO k-means initialization, IQR-based outlier flagging) for its negative binomial regression mixture model demultiplexer) — https://doi.org/doi:10.18129/B9.bioc.demuxmix
- **R** (runtime environment for demuxmix; k-means and quartile/IQR computations are executed in R)

## Examples

```
km <- kmeans(log(hto_counts[, hto_id]), centers = 2); pos <- which(km$cluster == which.max(tapply(hto_counts[, hto_id], km$cluster, mean)))
```

## Evaluation signals

- Exactly two k-means clusters are produced for every log-transformed HTO count vector.
- The cluster with the larger mean HTO count is taken as the positive cluster for each HTO.
- Every droplet flagged as an outlier exceeds the Q3 + 1.5*IQR threshold on HTO counts OR on the number of detected genes.
- The output hto_preprocessing_summary.csv contains a cluster assignment and an outlier flag for every droplet (no missing rows).
- Outlier-flagged droplets are excluded from mixture model fitting but still appear in the final classification.

## Limitations

- The paper reports no quantitative acceptance metric for this step beyond the rule-based QC check that exactly two clusters form and flagged droplets exceed the Q3 + 1.5*IQR thresholds.
- Outlier droplets cannot contribute to mixture model fitting; they are only classified afterwards, so heavily contaminated HTOs may yield unreliable initial labels.
- Clustering and thresholding are performed separately for each individual HTO; no cross-HTO joint initialization is described.

## Evidence

- [methods] k-means on log-transformed counts seeds initial class labels: [evidence span withheld — anchor status: not_found]
- [methods] larger-mean cluster is positive: [evidence span withheld — anchor status: not_found]
- [other] IQR outlier threshold definition: [evidence span withheld — anchor status: not_found]
- [intro] detected-genes covariate rationale: "better results are often achieved if the total number of genes detected per droplet (not the complete transcription profile) is passed to the method along with the HTO counts"
- [readme] package scope (README): "The package implements methods to fit regression mixture models for a probabilistic classification of cells, including multiplet detection."
