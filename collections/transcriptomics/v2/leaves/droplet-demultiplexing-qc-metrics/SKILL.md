---
name: droplet-demultiplexing-qc-metrics
description: Use when an (HTOs x droplets) count matrix has been demultiplexed with
  negative binomial regression mixture models and you need a per-HTO separability/QC
  metric or want to discard droplets with high assignment error probabilities.
when_to_use_negative:
- Input lacks HTO hashtag counts (e.g. genotype-based demultiplexing from SNPs rather
  than oligonucleotide-barcoded HTO data)
- The mixture model has not yet been fitted — the overlap score is derived from the
  fitted negative/positive components, not directly from raw HTO counts
- You need the full per-droplet transcription profile rather than the total genes-detected
  summary; demuxmix works with the number of detected genes passed alongside HTO counts
edam_operation: http://edamontology.org/operation_4513
edam_topics:
- http://edamontology.org/topic_3173
- http://edamontology.org/topic_0622
tools:
- name: demuxmix
  role: R/Bioconductor package that fits negative binomial regression mixture models
    to HTO counts, exposes posterior component membership probabilities and component
    parameters for overlap-score computation, and provides QC and error-rate estimation
  repo: https://doi.org/doi:10.18129/B9.bioc.demuxmix
- name: R
  role: Runtime environment in which the fitted DemuxmixResult object is loaded and
    the per-HTO overlap scores are computed
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioinformatics/btad481
    title: 'demuxmix: demultiplexing oligonucleotide-barcoded single-cell RNA sequencing
      data with regression mixture models'
  source_content_hash: sha256:cae467551fb0ac9c3254d16da3fd00a40ec11e5276aa62742a9ec76a2322f584
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - demuxmix
  edam_operation: http://edamontology.org/operation_4513
  edam_topics:
  - http://edamontology.org/topic_3173
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# droplet-demultiplexing-qc-metrics

## Summary

Computes data-quality metrics from fitted HTO demultiplexing models — notably the overlap score (ovs), the shared probability mass of the negative and positive mixture components per hashtag oligonucleotide — to quantify how separable negative and positive droplets are and to flag uncertain assignments.

## When to use

Use when an (HTOs x droplets) count matrix has been demultiplexed with negative binomial regression mixture models and you need a per-HTO separability/QC metric or want to discard droplets with high assignment error probabilities.

## When NOT to use

- Input lacks HTO hashtag counts (e.g. genotype-based demultiplexing from SNPs rather than oligonucleotide-barcoded HTO data)
- The mixture model has not yet been fitted — the overlap score is derived from the fitted negative/positive components, not directly from raw HTO counts
- You need the full per-droplet transcription profile rather than the total genes-detected summary; demuxmix works with the number of detected genes passed alongside HTO counts

## Inputs

- (HTOs x droplets) HTO count matrix sequenced together with the RNA molecules
- Per-droplet vector of the total number of genes detected in the RNA library (required for regression-type models, not the full transcription profile)
- Fitted demuxmix model (DemuxmixResult object) containing negative and positive mixture components per HTO

## Outputs

- Per-HTO table of standardized overlap scores (ovs), each in (0, 1], written to overlap_scores.csv
- Estimated demultiplexing error rates
- Per-droplet assignment error/posterior probabilities usable for discarding uncertain droplets

## How to apply

Load the fitted mixture model (DemuxmixResult object) and the per-droplet genes-detected vector in R using the demuxmix package. For each HTO, extract the posterior component membership probabilities and parameters of the negative and positive components and compute their shared probability mass over the HTO count support per Equation (5): for regression-type models weight by the mean number of detected genes (demuxmix leverages the positive association between genes detected per droplet and HTO counts), and omit this weighting for naive models. Standardize each overlap value to (0, 1] so scores are comparable across experiments, and assemble a per-HTO table (overlap_scores.csv). Use the model's probabilistic error estimates in the same pass to discard uncertain droplets and estimate demultiplexing error rates.

## Related tools

- **demuxmix** (R/Bioconductor package that fits negative binomial regression mixture models to HTO counts, exposes posterior component membership probabilities and component parameters for overlap-score computation, and provides QC and error-rate estimation) — https://doi.org/doi:10.18129/B9.bioc.demuxmix
- **R** (Runtime environment in which the fitted DemuxmixResult object is loaded and the per-HTO overlap scores are computed)

## Examples

```
if (!require("BiocManager")) install.packages("BiocManager"); BiocManager::install("demuxmix"); library(demuxmix)
```

## Evaluation signals

- Every computed overlap score falls in (0, 1] after standardization
- Regression-model overlaps are weighted by mean detected-gene values while naive-model overlaps omit the weighting
- The per-HTO table is complete: one ovs value per HTO in the input matrix, written to overlap_scores.csv
- Droplets flagged as uncertain by assignment error probabilities are excluded downstream, and estimated demultiplexing error rates match the fitted model's QC output

## Limitations

- Naive models cannot use the genes-detected weighting, so their overlap scores may be less separable than regression-type counterparts
- The overlap score requires a converged mixture fit; components that fail to separate cleanly produce ovs values that must be interpreted experiment-to-experiment (hence standardization to (0, 1])

## Evidence

- [abstract] demuxmix's probabilistic classification framework provides error probabilities for droplet assignments that can be used to discard uncertain droplets: "demuxmix's probabilistic classification framework provides error probabilities for droplet assignments that can be used to discard uncertain droplets"
- [intro] Genes-detected as covariate rationale: "better results are often achieved if the total number of genes detected per droplet (not the complete transcription profile) is passed to the method along with the HTO counts to leverage the positive"
- [readme] QC methods in package scope: "Demultiplexing error rates can be estimated, and methods for quality control are provided."
- [intro] Mixture model on HTO counts: "The *demuxmix* package implements a method to demultiplex droplets based on HTO counts using negative binomial regression mixture models."
