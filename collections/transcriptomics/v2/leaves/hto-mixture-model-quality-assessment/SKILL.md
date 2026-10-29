---
name: hto-mixture-model-quality-assessment
description: Use when fitted demuxmix HTO mixture models are available and you need
  a per-HTO quality metric (overlap score in (0, 1]) to judge whether negative and
  positive droplet components are adequately separated.
when_to_use_negative:
- The input is raw HTO counts only and no mixture model has been fitted — fit the
  demuxmix regression mixture models first (this skill assesses model quality, not
  classification itself)
- The goal is per-droplet classification or multiplet detection rather than per-HTO
  data quality assessment
- The fitted model object lacks component parameters (e.g. it is not a DemuxmixResult),
  so negative/positive component parameters cannot be extracted for the overlap computation
tools:
- name: demuxmix
  role: R/Bioconductor package that fits the negative binomial regression mixture
    models to HTO counts and exposes the fitted components and error probabilities
    used to compute the overlap score
  repo: https://doi.org/doi:10.18129/B9.bioc.demuxmix
- name: R
  role: Runtime environment for loading the DemuxmixResult object, extracting component
    parameters, and computing the overlap scores
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioinformatics/btad481
    title: 'demuxmix: demultiplexing oligonucleotide-barcoded single-cell RNA sequencing
      data with regression mixture models'
  source_content_hash: sha256:b5cb19c7fe904cab54a56e2d8b3f1f1f06ab276faf4f2f81059d30bad5be8b4e
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
yes_rate: 0.8
n_claims: 10
---

# hto-mixture-model-quality-assessment

## Summary

Computes demuxmix's overlap score (ovs), the shared probability mass of the negative and positive mixture components fitted to a hashtag oligonucleotide (HTO) count distribution, as a per-HTO data quality metric quantifying how separable the two droplet classes are.

## When to use

Use when fitted demuxmix HTO mixture models are available and you need a per-HTO quality metric (overlap score in (0, 1]) to judge whether negative and positive droplet components are adequately separated.

## When NOT to use

- The input is raw HTO counts only and no mixture model has been fitted — fit the demuxmix regression mixture models first (this skill assesses model quality, not classification itself)
- The goal is per-droplet classification or multiplet detection rather than per-HTO data quality assessment
- The fitted model object lacks component parameters (e.g. it is not a DemuxmixResult), so negative/positive component parameters cannot be extracted for the overlap computation

## Inputs

- DemuxmixResult object: fitted HTO mixture model (negative binomial regression or naive components) from the demuxmix package
- Per-droplet genes-detected vector (total number of genes detected per droplet)
- HTOs x droplets count matrix from the pooled, HTO-labeled scRNA-seq experiment

## Outputs

- overlap_scores.csv: per-HTO table of standardized overlap scores in (0, 1]
- Per-HTO separability assessment of negative vs positive droplet components

## How to apply

Load the fitted HTO mixture model (DemuxmixResult object) and the per-droplet genes-detected vector into R with the demuxmix package. For each HTO, extract the posterior component membership probabilities and the parameters of the negative and positive components from the fitted mixture model. Compute the overlap of the two components as their shared probability mass over the HTO count support per Equation (5); for regression-type models weight by the mean number of detected genes per droplet, since demuxmix leverages the positive association between detected genes and HTO counts, while naive-model overlaps omit this weighting. Standardize each overlap to lie in (0, 1] so scores are comparable across experiments, then assemble a per-HTO table (overlap_scores.csv). Small overlap values indicate well-separated negative and positive droplet classes and hence reliable demultiplexing for that HTO.

## Related tools

- **demuxmix** (R/Bioconductor package that fits the negative binomial regression mixture models to HTO counts and exposes the fitted components and error probabilities used to compute the overlap score) — https://doi.org/doi:10.18129/B9.bioc.demuxmix
- **R** (Runtime environment for loading the DemuxmixResult object, extracting component parameters, and computing the overlap scores)

## Examples

```
library(demuxmix)  # load fitted DemuxmixResult; for each HTO, compute ovs as shared negative/positive component mass per Equation (5) and write overlap_scores.csv
```

## Evaluation signals

- Every computed overlap score falls in (0, 1]
- Regression-model overlaps use weighted mean detected-gene values, while naive-model overlaps omit the weighting (per-model-type check)
- overlap_scores.csv contains exactly one row per HTO with a valid standardized score
- HTOs whose fitted mixture model shows poor negative/positive separation should have higher overlap values than well-separated HTOs in the same experiment

## Limitations

- The overlap score is a per-HTO quality metric of component separability; it does not by itself reassign or discard droplets — uncertain droplets must still be discarded using the assignment error probabilities
- The weighting by mean detected genes applies only to regression-type models; applying it to naive models (or omitting it for regression models) is inconsistent with the method
- The quality metric assumes the HTO count distribution is adequately described by the fitted mixture; a poor model fit invalidates the overlap score

## Evidence

- [abstract] Abstract statement on error probabilities for QC: "provides error probabilities for droplet assignments that can be used to discard uncertain droplets"
- [intro] Intro statement on negative binomial regression mixture models over HTO counts: "The *demuxmix* package implements a method to demultiplex droplets based on HTO counts using negative binomial regression mixture models."
- [intro] Intro statement on leveraging detected-gene association: "better results are often achieved if the total number of genes detected per droplet (not the complete transcription profile) is passed to the method along with the HTO counts"
- [readme] README statement on error-rate estimation and QC methods: "Demultiplexing error rates can be estimated, and methods for quality control are provided."
