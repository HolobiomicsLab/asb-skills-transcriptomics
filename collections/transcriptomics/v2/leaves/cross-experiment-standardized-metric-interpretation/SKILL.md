---
name: cross-experiment-standardized-metric-interpretation
description: Use when a DemuxmixResult (fitted HTO mixture model) is available and
  you need a per-HTO data quality metric of droplet separability that can be compared
  across experiments or used for QC filtering.
when_to_use_negative:
- No mixture model has been fit yet — the raw (HTOs x droplets) count matrix must
  first be modeled with demuxmix before an overlap score can be computed
- A regression-type model was fit but the per-droplet genes-detected vector is unavailable,
  since the overlap weighting requires the mean number of detected genes
- The goal is demultiplexing cell assignments (rather than assessing model/component
  separability quality) — droplet classification with posterior probabilities is the
  appropriate operation instead
tools:
- name: demuxmix
  role: Fits the negative binomial regression mixture models on HTO counts and provides
    the DemuxmixResult object from which the negative/positive component overlap score
    is extracted and standardized
  repo: https://doi.org/doi:10.18129/B9.bioc.demuxmix
- name: R
  role: Runtime environment in which the demuxmix package is loaded and the overlap-score
    computation, standardization, and CSV export are performed
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioinformatics/btad481
    title: 'demuxmix: demultiplexing oligonucleotide-barcoded single-cell RNA sequencing
      data with regression mixture models'
  source_content_hash: sha256:0c0694367fdae14466d80d219b82c8ee3a7979a77afc3e2bee877455c111170d
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
yes_rate: 0.6
n_claims: 5
---

# cross-experiment-standardized-metric-interpretation

## Summary

Quantifies how well a fitted HTO demultiplexing model separates negative and positive droplets by computing the overlap score (ovs) — the shared probability mass of the negative and positive mixture components per Equation (5) — and standardizing each score to (0, 1] so values are comparable across experiments.

## When to use

Use when a DemuxmixResult (fitted HTO mixture model) is available and you need a per-HTO data quality metric of droplet separability that can be compared across experiments or used for QC filtering.

## When NOT to use

- No mixture model has been fit yet — the raw (HTOs x droplets) count matrix must first be modeled with demuxmix before an overlap score can be computed
- A regression-type model was fit but the per-droplet genes-detected vector is unavailable, since the overlap weighting requires the mean number of detected genes
- The goal is demultiplexing cell assignments (rather than assessing model/component separability quality) — droplet classification with posterior probabilities is the appropriate operation instead

## Inputs

- Fitted HTO mixture model (DemuxmixResult object) from the demuxmix package
- Per-droplet genes-detected vector (number of genes detected per droplet, used for regression-type models)
- Posterior component membership probabilities and negative/positive component parameters per HTO

## Outputs

- Per-HTO table of standardized overlap scores in (0, 1]
- overlap_scores.csv

## How to apply

Load the fitted HTO mixture model (DemuxmixResult object) and the per-droplet genes-detected vector in R using the demuxmix package. For each HTO, extract the posterior component membership probabilities and the parameters of the negative and positive components from the fitted mixture. Compute the overlap between the two components as their shared probability mass over the HTO count support per Equation (5); for regression-type models, weight by the mean number of detected genes (leveraging the positive association between genes detected per droplet and HTO counts), and omit the weighting for naive models. Standardize each overlap to lie in (0, 1] so scores are comparable across experiments, then assemble a per-HTO table and write it to overlap_scores.csv. Interpret higher overlap as poorer separability of negative and positive droplets for that HTO, signaling reduced demultiplexing quality.

## Related tools

- **demuxmix** (Fits the negative binomial regression mixture models on HTO counts and provides the DemuxmixResult object from which the negative/positive component overlap score is extracted and standardized) — https://doi.org/doi:10.18129/B9.bioc.demuxmix
- **R** (Runtime environment in which the demuxmix package is loaded and the overlap-score computation, standardization, and CSV export are performed)

## Examples

```
Rscript -e 'library(demuxmix); res <- demuxmix(htoCounts, genes = genesDetected); # extract per-HTO overlap of negative/positive components, standardize to (0,1], write overlap_scores.csv'
```

## Evaluation signals

- Every overlap score falls in (0, 1] after standardization
- Overlaps derived from regression-type models use the weighted mean number of detected genes, while naive-model overlaps omit the weighting
- Per-HTO table written to overlap_scores.csv contains one overlap value per HTO in the fitted model
- HTOs with high overlap (shared probability mass close to the standardized maximum) are flagged as having poorly separable negative and positive droplet components

## Limitations

- The overlap score is model-type dependent: naive and regression-type models differ in whether the mean detected-genes weighting is applied, so comparisons should respect which model was fit
- Score interpretability relies on the (0, 1] standardization step; skipping it undermines cross-experiment comparability
- For regression-type models the metric depends on the supplied genes-detected values, so low-quality RNA metadata can distort the overlap score

## Evidence

- [other] Overlap score computation from fitted mixture components: [evidence span withheld — anchor status: not_found]
- [intro] Regression models leverage genes detected per droplet: "better results are often achieved if the total number of genes detected per droplet (not the complete transcription profile) is passed to the method"
- [readme] demuxmix provides quality-control methods including error-rate estimation: "Demultiplexing error rates can be estimated, and methods for quality control are provided."
- [abstract] Error probabilities support QC-based filtering: "demuxmix's probabilistic classification framework provides error probabilities for droplet assignments that can be used to discard uncertain droplets"
