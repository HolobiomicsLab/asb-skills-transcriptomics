---
name: single-cell-neighbour-distance-processing
description: Use when a scRNA-seq rare-cell workflow needs candidate minor-cluster
  sizes derived from an inter-cluster gap signal, i.e. a precomputed cell-to-neighbour
  distance matrix is available and k must be estimated via ∆∆D skewness.
when_to_use_negative:
- Input is raw gene expression counts rather than a computed neighbour-distance matrix
  — the upstream distance-computation step must run first
- The goal is characterising major cell types, which toolkits such as Seurat and Scater
  handle comprehensively; this skill targets the minor-cluster gap signal only
edam_operation: http://edamontology.org/operation_3432
tools:
- name: GapClust
  role: Implements the ∆∆D second-order derivative statistics, the outlier filtering/averaging
    rules, per-k skewness computation, and the skewness > 2 candidate-k flagging
  repo: github.com/fabotao/GapClust
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-021-24489-8
    title: GapClust is a light-weight approach distinguishing rare cells from voluminous
      single cell expression profiles
  source_content_hash: sha256:faa1dbd029c06409bba278555603a6b399447f0405aa7c0df51c4d24f7ce243d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GapClust
  edam_operation: http://edamontology.org/operation_3432
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# single-cell-neighbour-distance-processing

## Summary

Computes first-order (∆D) and second-order (∆∆D) derivative statistics from the sorted nearest-neighbour distance matrix of single-cell profiles, filters and aggregates them per candidate cluster size k, and scores skewness<k> to flag candidate rare-cluster sizes in GapClust.

## When to use

Use when a scRNA-seq rare-cell workflow needs candidate minor-cluster sizes derived from an inter-cluster gap signal, i.e. a precomputed cell-to-neighbour distance matrix is available and k must be estimated via ∆∆D skewness.

## When NOT to use

- Input is raw gene expression counts rather than a computed neighbour-distance matrix — the upstream distance-computation step must run first
- The goal is characterising major cell types, which toolkits such as Seurat and Scater handle comprehensively; this skill targets the minor-cluster gap signal only

## Inputs

- precomputed nearest-neighbour distance matrix for all cells (output of the upstream distance-computation step)
- maximum neighbour order K defining the candidate cluster-size range 1..K

## Outputs

- delta2d_skewness.csv containing a per-k skewness column and flag rows for candidate k
- filtered/aggregated ∆∆D statistics per candidate cluster size k

## How to apply

Load the precomputed neighbour-distance matrix for all cells; for each cell sort distances to its nearest neighbours and compute the first-order derivatives ∆D as successive differences of the sorted distances. Compute the second-order derivatives ∆∆D as successive differences of ∆D, capturing the variation of distance changes that reflects the inter-cluster gap (for a cell in a minor cluster Cn of size n, distances to neighbours outside Cn are far greater than to neighbours inside it). Apply outlier filtering and averaging rules to the ∆∆D values so sporadic distance spikes do not dominate the signal, then aggregate the filtered ∆∆D statistics per candidate cluster size k (k = 1..K) and compute skewness<k> of the distribution. Write the per-k skewness values and flags to delta2d_skewness.csv; a candidate k is accepted as a rare-cluster candidate only if skewness<k> > 2.

## Related tools

- **GapClust** (Implements the ∆∆D second-order derivative statistics, the outlier filtering/averaging rules, per-k skewness computation, and the skewness > 2 candidate-k flagging) — github.com/fabotao/GapClust

## Examples

```
devtools::install_github("fabotao/GapClust")
```

## Evaluation signals

- Output table delta2d_skewness.csv contains a skewness column with one row per candidate k
- Only rows with skewness<k> > 2 are flagged as rare-cluster candidates
- ∆D values equal the successive differences of each cell's sorted neighbour distances, and ∆∆D the successive differences of ∆D
- Sporadic distance spikes are damped: filtered/averaged ∆∆D distributions should not be dominated by single-cell outliers
- Accepted candidate k should correspond to a plausible rare-cluster size consistent with the gap structure visible in the distance matrix

## Limitations

- Detection depends on the inter-cluster gap being visible in the neighbour-distance statistics; clusters without a clear gap may not be flagged
- Sporadic distance spikes can dominate the ∆∆D signal if outlier filtering and averaging rules are not applied
- The README and discussion section report no parameter settings (e.g. K, skewness threshold) for specific GapClust runs, so values must be chosen per dataset

## Evidence

- [methods] GapClust calculates the skewness of the distribution of ∆∆D<m,k> values across all cells as k varies from 1 to K, and candidate k are flagged when the skewness exceeds the threshold value 2.: "Candidate k can be identified if the skewness<k> > 2."
- [intro] Second-order derivative ∆∆D rationale: "A second-order derivative type distance score capturing the variation of distance changes is proposed to fully utilise this gap information."
- [intro] Design observation motivating ∆∆D: "the design of GapClust is inspired by the observation that for a particular data point in a minor cluster Cn of size n"
- [readme] GapClust statistics design: "GapClust takes advantage of the gap between minor cluster and neighbouring abundant cluster to let rare cells within minor cluster stand out"
