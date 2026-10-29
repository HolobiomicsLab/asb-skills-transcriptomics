---
name: scrna-seq-expression-profile-analysis
description: Use when you have a precomputed neighbour-distance matrix from a large
  scRNA-seq dataset and need to estimate the size k of a potential rare cell cluster
  from distance-derivative statistics.
when_to_use_negative:
- Input is raw expression counts or a feature table rather than a computed cell-to-neighbour
  distance matrix
- The goal is comprehensive clustering of major cell types (e.g. via Seurat or Scater)
  rather than detecting a single rare/minor cluster
- The dataset is small enough that exhaustive rare-cell methods like RaceID are computationally
  feasible and a full k-scan is unnecessary
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: GapClust
  role: Implements the ∆∆D second-order derivative distance statistics and the skewness-based
    candidate-k detection from the neighbour-distance matrix
  repo: github.com/fabotao/GapClust
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-021-24489-8
    title: GapClust is a light-weight approach distinguishing rare cells from voluminous
      single cell expression profiles
  source_content_hash: sha256:232a537296d50540081b0f8ec5e340e3d0999846fee6cf2ca167e9831a09dd97
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GapClust
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# scRNA-seq-expression-profile-analysis

## Summary

Identifies candidate rare-cluster sizes k in single-cell RNA-seq data by computing the skewness of second-order neighbour-distance derivatives ∆∆D<m,k> across cells, flagging k values whose skewness exceeds a threshold of 2, as implemented in GapClust.

## When to use

Use when you have a precomputed neighbour-distance matrix from a large scRNA-seq dataset and need to estimate the size k of a potential rare cell cluster from distance-derivative statistics.

## When NOT to use

- Input is raw expression counts or a feature table rather than a computed cell-to-neighbour distance matrix
- The goal is comprehensive clustering of major cell types (e.g. via Seurat or Scater) rather than detecting a single rare/minor cluster
- The dataset is small enough that exhaustive rare-cell methods like RaceID are computationally feasible and a full k-scan is unnecessary

## Inputs

- Precomputed neighbour-distance matrix for all cells (from upstream distance-computation step)
- Candidate cluster size range K (maximum k to scan)
- Skewness threshold (default 2)

## Outputs

- delta2d_skewness.csv containing per-k skewness values and candidate-cluster flags
- List of candidate k values with skewness<k> > 2

## How to apply

Load the precomputed neighbour-distance matrix for all cells and, for each cell, sort distances to its nearest neighbours. Compute the first-order derivative ∆D as successive differences of the sorted distances, then the second-order derivative ∆∆D as successive differences of ∆D, which captures the inter-cluster gap between a minor cluster and its neighbours. Apply outlier filtering and averaging rules to the ∆∆D values so sporadic distance spikes do not dominate the signal. Aggregate the filtered ∆∆D statistics per candidate cluster size k (k ranging from 1 to K) and compute skewness<k> of the distribution; accept k as a rare-cluster candidate only when skewness<k> > 2. Write per-k skewness values and flags to delta2d_skewness.csv.

## Related tools

- **GapClust** (Implements the ∆∆D second-order derivative distance statistics and the skewness-based candidate-k detection from the neighbour-distance matrix) — github.com/fabotao/GapClust

## Examples

```
devtools::install_github("fabotao/GapClust")
```

## Evaluation signals

- delta2d_skewness.csv exists and contains a skewness column
- Only rows with skewness<k> > 2 are flagged as rare-cluster candidates
- Per-cell ∆∆D values pass the outlier filtering/averaging step before aggregation (no sporadic distance spikes dominating the signal)
- Skewness is computed over the full candidate range k = 1..K
- Identified candidate k corresponds to a biologically plausible rare-cluster size consistent with the inter-cluster gap in the distance matrix

## Limitations

- Candidate k detection depends on a clear inter-cluster gap; the design assumes cells in a minor cluster have distances to outside neighbours far greater than to inside neighbours
- Sporadic distance spikes can dominate the ∆∆D signal unless outlier filtering and averaging are applied
- The article section reports no explicit parameter settings for GapClust runs (e.g. K, skewness threshold), which must be chosen by the practitioner

## Evidence

- [intro] A second-order derivative type distance score capturing the variation of distance changes is proposed to utilise the gap information.: [evidence span withheld — anchor status: not_found]
- [discussion] Candidate k flagged by skewness threshold: "Candidate k can be identified if the skewness<k> > 2."
- [intro] GapClust design based on inter-cluster gap observation: "for a particular data point in a minor cluster Cn of size n, then the distance to its nearest neighbours outside Cn is far greater than to"
- [readme] GapClust is lightweight and efficient for rare cells: "GapClust takes advantage of the gap between minor cluster and neighbouring abundant cluster to let rare cells within minor cluster stand out"
