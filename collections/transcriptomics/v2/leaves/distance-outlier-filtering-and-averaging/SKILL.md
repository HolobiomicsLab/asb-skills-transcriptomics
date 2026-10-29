---
name: distance-outlier-filtering-and-averaging
description: Use when scRNA-seq neighbour-distance matrices produce sporadic distance
  spikes that would dominate ∆∆D statistics, and candidate rare-cluster sizes k must
  be scored via per-k skewness for GapClust-style rare cell detection.
when_to_use_negative:
- When the input is raw expression counts and no neighbour-distance matrix has been
  computed yet — the upstream distance-computation step must run first
- When the goal is characterising major cell types (e.g. with Seurat or Scater toolkits)
  rather than detecting rare cell populations
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: GapClust
  role: Implements the ∆∆D computation, outlier filtering and averaging, per-k skewness
    calculation, and the skewness<k> > 2 candidate-k flagging rule
  repo: github.com/fabotao/GapClust
- name: e1071
  role: R dependency (>= 1.7-0.1) used by GapClust; plausible source of the skewness
    computation, per the README prerequisites list
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-021-24489-8
    title: GapClust is a light-weight approach distinguishing rare cells from voluminous
      single cell expression profiles
  source_content_hash: sha256:324985c3d92d50d74d1185cbfd815dae79faf532e7c2d6831f8e2de56390c9dd
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GapClust
  - e1071
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# distance-outlier-filtering-and-averaging

## Summary

A GapClust filtering skill that tames sporadic nearest-neighbour distance spikes in the second-order derivative statistics ∆∆D<m,k> by applying outlier filtering and averaging rules before per-k aggregation, so the inter-cluster gap signal reflects genuine cluster structure rather than single-cell distance outliers.

## When to use

Use when scRNA-seq neighbour-distance matrices produce sporadic distance spikes that would dominate ∆∆D statistics, and candidate rare-cluster sizes k must be scored via per-k skewness for GapClust-style rare cell detection.

## When NOT to use

- When the input is raw expression counts and no neighbour-distance matrix has been computed yet — the upstream distance-computation step must run first
- When the goal is characterising major cell types (e.g. with Seurat or Scater toolkits) rather than detecting rare cell populations

## Inputs

- Precomputed neighbour-distance matrix for all cells (output of the upstream GapClust distance-computation step)
- Maximum neighbour order K defining the candidate cluster-size range (k = 1..K)

## Outputs

- delta2d_skewness.csv containing per-k skewness values with flag column for k where skewness<k> > 2
- Filtered/averaged ∆∆D statistics aggregated per candidate cluster size k

## How to apply

Load the precomputed neighbour-distance matrix produced by the upstream distance-computation step, then for each cell sort distances to its nearest neighbours and compute first-order differences ∆D and second-order differences ∆∆D, which capture the variation of distance changes reflecting the inter-cluster gap. Apply the outlier filtering and averaging rules to the ∆∆D values so that sporadic distance spikes do not dominate the signal — this is the rationale of the skill, grounded in the observation that for a cell in a minor cluster Cn of size n the distance to nearest neighbours outside Cn is far greater than to neighbours inside it. Aggregate the filtered ∆∆D statistics per candidate cluster size k (k ranging from 1 to K) and compute skewness<k> of the distribution. Flag a k as a rare-cluster candidate only when skewness<k> > 2, and write the per-k skewness values and flags to delta2d_skewness.csv.

## Related tools

- **GapClust** (Implements the ∆∆D computation, outlier filtering and averaging, per-k skewness calculation, and the skewness<k> > 2 candidate-k flagging rule) — github.com/fabotao/GapClust
- **e1071** (R dependency (>= 1.7-0.1) used by GapClust; plausible source of the skewness computation, per the README prerequisites list)

## Examples

```
devtools::install_github("fabotao/GapClust")  # then run GapClust on the precomputed neighbour-distance matrix to obtain delta2d_skewness.csv with skewness<k> flags
```

## Evaluation signals

- Output table delta2d_skewness.csv contains a skewness column and only rows with skewness<k> > 2 are flagged as candidate k
- Filtered ∆∆D distributions are not dominated by single-cell distance spikes (compare pre-/post-filter skewness stability)
- Per-k skewness values are computed over the full range k = 1..K with no missing k entries
- Downstream rare-cell calls match known rare populations (e.g. tuft cells in the intestine dataset, marker genes Dclk1 and Trpm5)

## Limitations

- No parameter settings for GapClust runs (e.g. K, skewness threshold) are reported in some article sections, so defaults must be assumed from the package
- Results for several real datasets are deferred to unlinked supplementary figures (Supplementary Figs. 14-22, Supplementary Table 5), limiting reproducibility checks
- Sporadic distance spikes must genuinely be outliers — if a large fraction of cells are rare, averaging may suppress the true inter-cluster gap signal

## Evidence

- [intro] A second-order derivative type distance score capturing the variation of distance changes is proposed to fully utilise this gap information.: "A second-order derivative type distance score capturing the variation of distance changes is proposed to fully utilise this gap information."
- [discussion] Candidate k threshold: "Candidate k can be identified if the skewness<k> > 2."
- [readme] GapClust design rationale: "GapClust takes advantage of the gap between minor cluster and neighbouring abundant cluster to let rare cells within minor cluster stand out"
- [readme] Installation command: "devtools::install_github("fabotao/GapClust")"
