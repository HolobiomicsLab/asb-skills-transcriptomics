---
name: skewness-based-rare-cluster-candidate-flagging
description: Use when running GapClust on a scRNA-seq neighbour-distance matrix and
  you need to determine which candidate cluster sizes k (1..K) correspond to a genuine
  inter-cluster gap harboring rare cells.
when_to_use_negative:
- Input is raw expression counts with no precomputed neighbour-distance matrix — the
  ΔΔD statistics require sorted nearest-neighbour distances as input.
- The goal is to characterise abundant/major cell types (use toolkits such as Seurat
  or Scater instead); this rule targets minor-cluster size detection via the inter-cluster
  gap.
- Candidate k must be selected by another criterion (e.g. user-specified ground-truth
  cluster size) rather than discovered from distance-gap skewness.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: GapClust
  role: Implements the ΔΔD second-order derivative distance statistics, per-k skewness
    computation, and the skewness<k> > 2 flagging rule
  repo: github.com/fabotao/GapClust
- name: e1071
  role: R package prerequisite (>= 1.7-0.1) supplying the skewness calculation used
    to score each candidate k
- name: rflann
  role: R package prerequisite (>= 1.8.4) used upstream for fast nearest-neighbour
    distance computation feeding the ΔΔD statistics
  repo: https://github.com/YeeJeremy/rflann
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-021-24489-8
    title: GapClust is a light-weight approach distinguishing rare cells from voluminous
      single cell expression profiles
  source_content_hash: sha256:8d27ba14afa0f5cc6647cf4eb385090e8154ca1bea631c51ccc001026354a8ab
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GapClust
  - e1071
  - rflann
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# skewness-based-rare-cluster-candidate-flagging

## Summary

GapClust's decision rule for flagging candidate rare-cluster sizes: it computes the skewness of the distribution of second-order derivative neighbour-distance statistics ΔΔD<m,k> per candidate k and accepts k as a rare-cluster candidate when skewness<k> > 2.

## When to use

Use when running GapClust on a scRNA-seq neighbour-distance matrix and you need to determine which candidate cluster sizes k (1..K) correspond to a genuine inter-cluster gap harboring rare cells.

## When NOT to use

- Input is raw expression counts with no precomputed neighbour-distance matrix — the ΔΔD statistics require sorted nearest-neighbour distances as input.
- The goal is to characterise abundant/major cell types (use toolkits such as Seurat or Scater instead); this rule targets minor-cluster size detection via the inter-cluster gap.
- Candidate k must be selected by another criterion (e.g. user-specified ground-truth cluster size) rather than discovered from distance-gap skewness.

## Inputs

- Precomputed cell-by-neighbour distance matrix (per-cell distances to nearest neighbours, output of GapClust's upstream distance-computation step)
- Range of candidate cluster sizes k = 1..K to scan

## Outputs

- delta2d_skewness.csv containing per-k skewness values and boolean rare-cluster-candidate flags (rows with skewness<k> > 2 flagged)

## How to apply

Load the precomputed neighbour-distance matrix and, for each cell, sort distances to its nearest neighbours. Compute the first-order derivative ΔD as successive differences of the sorted distances, then the second-order derivative ΔΔD from successive differences of ΔD; this score captures the variation of distance changes that reflects the inter-cluster gap surrounding a minor cluster. Apply the outlier filtering and averaging rules to ΔΔD values so sporadic distance spikes do not dominate the signal, then aggregate the filtered ΔΔD statistics per candidate cluster size k from 1 to K. Compute skewness<k> of the resulting distribution for each k (the e1071 R package provides skewness and is a stated prerequisite). Flag a candidate k as a rare-cluster candidate only when skewness<k> > 2, and write the per-k skewness values and flags to delta2d_skewness.csv.

## Related tools

- **GapClust** (Implements the ΔΔD second-order derivative distance statistics, per-k skewness computation, and the skewness<k> > 2 flagging rule) — github.com/fabotao/GapClust
- **e1071** (R package prerequisite (>= 1.7-0.1) supplying the skewness calculation used to score each candidate k)
- **rflann** (R package prerequisite (>= 1.8.4) used upstream for fast nearest-neighbour distance computation feeding the ΔΔD statistics) — https://github.com/YeeJeremy/rflann

## Examples

```
devtools::install_github("fabotao/GapClust")
```

## Evaluation signals

- delta2d_skewness.csv exists and contains a skewness column with one row per candidate k from 1 to K.
- Only rows with skewness<k> > 2 carry the candidate flag; verify no row at or below the threshold is flagged.
- ΔΔD values are computed as second-order differences: ΔD from successive differences of sorted neighbour distances, ΔΔD from successive differences of ΔD.
- Outlier filtering/averaging has been applied to ΔΔD before aggregation — no single sporadic distance spike dominates the per-k skewness.
- Flagged k values are consistent with known rare cluster sizes when re-running on datasets with validated rare populations (e.g. tuft cells in the intestine dataset).

## Limitations

- No parameter settings for GapClust runs (e.g. K, skewness threshold) are reported in the paper's discussion, so default thresholds may need tuning per dataset.
- The article notes candidate k is only identified when skewness<k> > 2; datasets without a sharp inter-cluster gap may yield no flagged candidate.
- After identifying rare cells, hidden clusters require rerunning GapClust with the identified rare cell types removed — a single pass does not enumerate all rare clusters.

## Evidence

- [discussion] Candidate k can be identified if the skewness<k> > 2.: "Candidate k can be identified if the skewness<k> > 2."
- [intro] A second-order derivative type distance score capturing the variation of distance changes is proposed to fully utilise this gap information.: "A second-order derivative type distance score capturing the variation of distance changes is proposed to fully utilise this gap information."
- [readme] GapClust takes advantage of the gap between minor cluster and neighbouring abundant cluster to let rare cells within minor cluster stand out through delicately designed statistics.: "GapClust takes advantage of the gap between minor cluster and neighbouring abundant cluster to let rare cells within minor cluster"
- [intro] The design of GapClust is inspired by the observation that for a particular data point in a minor cluster Cn of size n, then the distance to its nearest neighbours outside Cn is far greater than to: "for a particular data point in a minor cluster Cn of size n, then the distance to its nearest neighbours outside Cn is far greater than to"
