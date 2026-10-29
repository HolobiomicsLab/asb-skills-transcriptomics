---
name: overlap-score-computation-from-mixture-components
description: 'Decision support for judging per-HTO demultiplexing quality after fitting
  demuxmix regression mixture models: needed when you must rank HTOs by how well their
  negative and positive droplet populations separate before accepting classifications.'
when_to_use_negative:
- The HTO mixture models have not yet been fitted — the overlap score requires the
  fitted negative and positive components of the DemuxmixResult, not raw HTO counts
  alone
- You need per-droplet classification decisions or error probabilities — this skill
  yields a per-HTO separability metric, not assignments for individual droplets
- The data are not oligonucleotide-barcoded (HTO-labeled) scRNA-seq droplets — the
  negative/positive component structure is specific to demuxmix's HTO mixture models
edam_operation: http://edamontology.org/operation_0336
edam_topics:
- http://edamontology.org/topic_3518
- http://edamontology.org/topic_2229
tools:
- name: demuxmix
  role: R/Bioconductor package whose fitted regression mixture models (negative and
    positive HTO components, posterior membership probabilities) supply the quantities
    from which the overlap score is computed
  repo: https://doi.org/doi:10.18129/B9.bioc.demuxmix
- name: R
  role: Execution environment for loading the DemuxmixResult object, extracting component
    parameters, and computing/standardizing the overlap values
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioinformatics/btad481
    title: 'demuxmix: demultiplexing oligonucleotide-barcoded single-cell RNA sequencing
      data with regression mixture models'
  source_content_hash: sha256:8ed3e23655ae1e9a652f2dc19c19a481e1d9a3fbcd691a470fed43d18fd0489f
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - demuxmix
  edam_operation: http://edamontology.org/operation_0336
  edam_topics:
  - http://edamontology.org/topic_3518
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# overlap-score-computation-from-mixture-components

## Summary

Computes the overlap score (ovs) for each hashtag oligonucleotide (HTO) as the shared probability mass of the negative and positive components of the fitted demuxmix mixture model, a per-HTO data-quality metric quantifying how separable the two droplet classes are.

## When to use

Decision support for judging per-HTO demultiplexing quality after fitting demuxmix regression mixture models: needed when you must rank HTOs by how well their negative and positive droplet populations separate before accepting classifications.

## When NOT to use

- The HTO mixture models have not yet been fitted — the overlap score requires the fitted negative and positive components of the DemuxmixResult, not raw HTO counts alone
- You need per-droplet classification decisions or error probabilities — this skill yields a per-HTO separability metric, not assignments for individual droplets
- The data are not oligonucleotide-barcoded (HTO-labeled) scRNA-seq droplets — the negative/positive component structure is specific to demuxmix's HTO mixture models

## Inputs

- Fitted HTO mixture model (DemuxmixResult object) from the demuxmix package in R
- Per-droplet number of genes detected vector (used to weight overlaps from regression-type models)
- HTO count matrix (HTOs x droplets) that the mixture models were fitted to

## Outputs

- Per-HTO overlap score table (e.g., overlap_scores.csv) with standardized overlap values in (0, 1]
- Per-HTO quality ranking identifying HTOs with poorly separated negative/positive components

## How to apply

Load the fitted HTO mixture model (DemuxmixResult object) and the per-droplet genes-detected vector into R using the demuxmix package. For each HTO, extract the posterior component membership probabilities and the parameters of the negative and positive components from the fitted mixture. Compute the overlap of the two components as their shared probability mass over the HTO count support per Equation (5): for regression-type models weight by the mean number of detected genes (leveraging the positive association between genes detected and HTO counts), and for naive models omit the weighting. Standardize each overlap to lie in (0, 1] so scores are comparable across experiments, then assemble a per-HTO table (e.g., overlap_scores.csv). Smaller overlap values indicate better-separated negative and positive droplet classes and hence a more reliable HTO.

## Related tools

- **demuxmix** (R/Bioconductor package whose fitted regression mixture models (negative and positive HTO components, posterior membership probabilities) supply the quantities from which the overlap score is computed) — https://doi.org/doi:10.18129/B9.bioc.demuxmix
- **R** (Execution environment for loading the DemuxmixResult object, extracting component parameters, and computing/standardizing the overlap values)

## Examples

```
library(demuxmix); dmr <- demuxmix(htoMatrix, numGenes = numGenes); ovs <- overlapScore(dmr); write.csv(ovs, "overlap_scores.csv")
```

## Evaluation signals

- Every computed overlap score falls in (0, 1] after standardization
- Regression-model overlaps are weighted by the mean number of detected genes, whereas naive-model overlaps omit the weighting — check the weighting convention matches the model type
- The output table contains exactly one overlap value per fitted HTO
- HTOs whose negative and positive components show large shared probability mass (high ovs) should coincide with HTOs flagged as unreliable by posterior-probability QC

## Limitations

- The overlap score is defined over the fitted negative and positive components only; it presumes a two-component mixture was successfully fitted for the HTO
- Scores are standardized to (0, 1] for cross-experiment comparability; raw shared-probability-mass values from different models or experiments are not directly comparable
- For regression-type models the overlap depends on the genes-detected weighting, so a poor genes-detected vector can distort the quality assessment

## Evidence

- [other] The overlap score (ovs) is obtained by calculating the shared probability mass of the negative and positive components of the fitted HTO mixture model: [evidence span withheld — anchor status: not_found]
- [abstract] demuxmix utilizes the positive association: "demuxmix utilizes the positive association between detected genes in the RNA library and HTO counts to explain parts of the variance in the HTO data resulting in improved droplet assignments"
- [abstract] provides error probabilities for droplet assignments: "provides error probabilities for droplet assignments that can be used to discard uncertain droplets"
- [readme] Demultiplexing error rates can be estimated: "The package implements methods to fit regression mixture models for a probabilistic classification of cells, including multiplet detection. Demultiplexing error rates can be estimated, and methods"
