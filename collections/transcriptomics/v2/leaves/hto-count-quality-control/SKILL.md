---
name: hto-count-quality-control
description: 'Use when raw (HTOs x droplets) hashtag-oligonucleotide count data must
  be prepared before negative binomial mixture fitting: initial labeling of positive
  vs negative droplets and IQR-based outlier flagging are needed.'
when_to_use_negative:
- Droplets already have final demultiplexing assignments or error probabilities —
  this skill only supplies initial labels and outlier flags, not the classification
  itself
- Data are not HTO oligonucleotide counts (e.g. genes x droplets RNA count matrix
  alone, or SNP/genotype-based demultiplexing) — the k-means/IQR logic is defined
  per HTO
- No per-droplet number of detected genes is available — the IQR outlier rule on detected
  genes cannot be applied and only the HTO-count threshold remains
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: demuxmix
  role: R/Bioconductor package whose preprocessing (Section 2.1.1) performs the k-means
    initialization and IQR outlier flagging on HTO counts and detected genes
  repo: https://doi.org/doi:10.18129/B9.bioc.demuxmix
- name: R
  role: Implementation language; k-means clustering (k = 2) on log-transformed per-HTO
    counts and Q3 + 1.5*IQR threshold computation are carried out in R
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btad481
    title: 'demuxmix: demultiplexing oligonucleotide-barcoded single-cell RNA sequencing
      data with regression mixture models'
  source_content_hash: sha256:43f3cd5ea60bb98148beb553d4dbec737e70bd779ff5fcd20be5eed514547443
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
n_claims: 6
---

# hto-count-quality-control

## Summary

Preprocessing step from demuxmix (Section 2.1.1) that derives initial positive/negative class labels per HTO via k-means (k = 2) on log-transformed HTO counts, and flags droplets as outliers using Q3 + 1.5*IQR thresholds on HTO counts and on the per-droplet number of detected genes.

## When to use

Use when raw (HTOs x droplets) hashtag-oligonucleotide count data must be prepared before negative binomial mixture fitting: initial labeling of positive vs negative droplets and IQR-based outlier flagging are needed.

## When NOT to use

- Droplets already have final demultiplexing assignments or error probabilities — this skill only supplies initial labels and outlier flags, not the classification itself
- Data are not HTO oligonucleotide counts (e.g. genes x droplets RNA count matrix alone, or SNP/genotype-based demultiplexing) — the k-means/IQR logic is defined per HTO
- No per-droplet number of detected genes is available — the IQR outlier rule on detected genes cannot be applied and only the HTO-count threshold remains

## Inputs

- (HTOs x droplets) hashtag oligonucleotide (HTO) count matrix
- Per-droplet number of detected genes vector
- SingleCellExperiment/DelayedArray-backed R objects

## Outputs

- hto_preprocessing_summary.csv with a per-droplet initial cluster assignment (positive/negative per HTO) and an outlier flag
- Set of outlier droplets (excluded from model fitting, still to be classified)

## How to apply

Load the (HTOs x droplets) count matrix and the per-droplet number of detected genes via SingleCellExperiment/DelayedArray interfaces in R. For each HTO separately, log-transform its count vector and run k-means with k = 2; take the cluster with the larger mean HTO count as the positive cluster containing droplets tagged by that HTO. Compute outlier thresholds as the third quartile plus 1.5 times the interquartile range, separately for HTO counts and for the number of detected genes per droplet. Flag droplets exceeding either threshold as outliers: they are excluded from model fitting but are still classified afterwards. Write the per-droplet table (initial cluster assignment plus outlier flag) to hto_preprocessing_summary.csv.

## Related tools

- **demuxmix** (R/Bioconductor package whose preprocessing (Section 2.1.1) performs the k-means initialization and IQR outlier flagging on HTO counts and detected genes) — https://doi.org/doi:10.18129/B9.bioc.demuxmix
- **R** (Implementation language; k-means clustering (k = 2) on log-transformed per-HTO counts and Q3 + 1.5*IQR threshold computation are carried out in R)

## Examples

```
km <- kmeans(log(hto_counts[hto, ]), centers = 2); thr <- quantile(hto_counts[hto, ], 0.75) + 1.5 * IQR(hto_counts[hto, ])
```

## Evaluation signals

- Exactly two k-means clusters are produced for each log-transformed HTO count vector
- hto_preprocessing_summary.csv contains a cluster assignment and an outlier flag for every droplet (no missing rows or NA flags)
- Every droplet flagged as an outlier exceeds the Q3 + 1.5*IQR threshold on HTO counts or on the number of detected genes
- For each HTO, the positive cluster is the one with the larger mean HTO count
- Outlier droplets are excluded from model fitting but still present in the final classification output

## Limitations

- The paper reports no quantitative acceptance metric beyond the rule-based QC check (two clusters present; flagged droplets exceed a Q3 + 1.5*IQR threshold)
- Outlier-flagged droplets are excluded from model fitting, so model parameters are estimated without them; they are only classified post hoc
- K-means initialization is per HTO on log-transformed counts, so clusters reflect each HTO's count distribution individually and can differ across HTOs

## Evidence

- [methods] k-means initialization: [evidence span withheld — anchor status: not_found]
- [methods] IQR outlier thresholds: [evidence span withheld — anchor status: not_found]
- [readme] package scope and QC: "Demultiplexing error rates can be estimated, and methods for quality control are provided."
- [intro] HTO-based demultiplexing: "The *demuxmix* package implements a method to demultiplex droplets based on HTO counts using negative binomial regression mixture models."
- [intro] HTO count matrix input: "These oligonucleotides are termed hashtag oligonucleotides (HTOs) and are sequenced together with the RNA molecules of the cells resulting in an (HTOs x droplets) count matrix in addition to the"
