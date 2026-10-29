---
name: second-order-derivative-statistic-computation
description: Use when you have a precomputed cell-to-neighbour distance matrix for
  a scRNA-seq dataset and need to derive ∆D/∆∆D statistics and per-k skewness to identify
  candidate rare cluster sizes.
when_to_use_negative:
- Input is raw expression counts without a precomputed neighbour-distance matrix —
  run the upstream distance-computation step first
- The goal is to characterise major/abundant cell types — GapClust's ∆∆D statistic
  is designed to exploit the gap around minor clusters only
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: GapClust
  role: Implements the ∆∆D second-order derivative statistic, per-k skewness computation,
    and skewness > 2 flagging of candidate k
  repo: github.com/fabotao/GapClust
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-021-24489-8
    title: GapClust is a light-weight approach distinguishing rare cells from voluminous
      single cell expression profiles
  source_content_hash: sha256:fd46a21272fe77eb45609879299ac5a97e4a825dd2d31ce98b61d05d26d88c7d
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
yes_rate: 0.9
n_claims: 10
---

# second-order-derivative-statistic-computation

## Summary

Computes the second-order derivative distance statistics (∆∆D) over sorted nearest-neighbour distances and aggregates their per-k skewness so GapClust can flag candidate rare-cluster sizes k when skewness > 2.

## When to use

Use when you have a precomputed cell-to-neighbour distance matrix for a scRNA-seq dataset and need to derive ∆D/∆∆D statistics and per-k skewness to identify candidate rare cluster sizes.

## When NOT to use

- Input is raw expression counts without a precomputed neighbour-distance matrix — run the upstream distance-computation step first
- The goal is to characterise major/abundant cell types — GapClust's ∆∆D statistic is designed to exploit the gap around minor clusters only

## Inputs

- Precomputed neighbour-distance matrix for all cells (from the upstream distance-computation step)
- Parameter K (maximum number of nearest neighbours / candidate k range)

## Outputs

- delta2d_skewness.csv containing per-k skewness values and candidate flags (rows with skewness > 2 flagged)

## How to apply

Load the precomputed neighbour-distance matrix, sort each cell's distances to its nearest neighbours, and compute first-order derivative statistics ∆D as the successive differences of the sorted distances. Then compute the second-order derivative ∆∆D from successive differences of ∆D, capturing the variation of distance changes that reflects the inter-cluster gap. Apply the outlier filtering and averaging rules to the ∆∆D values so sporadic distance spikes do not dominate the signal. Aggregate the filtered ∆∆D statistics per candidate cluster size k (k from 1 to K) and compute skewness<k> of the distribution (e.g. with e1071, a package prerequisite of GapClust). A k is accepted as a rare-cluster candidate only if skewness<k> > 2. Write per-k skewness values and flags to delta2d_skewness.csv.

## Related tools

- **GapClust** (Implements the ∆∆D second-order derivative statistic, per-k skewness computation, and skewness > 2 flagging of candidate k) — github.com/fabotao/GapClust

## Examples

```
devtools::install_github("fabotao/GapClust")
```

## Evaluation signals

- Output table delta2d_skewness.csv contains a skewness column and per-k rows
- Only rows with skewness<k> > 2 are flagged as candidate rare-cluster sizes
- ∆D values equal successive differences of the sorted per-cell neighbour distances; ∆∆D equals successive differences of ∆D
- Filtered/averaged ∆∆D statistics show no single sporadic distance spike dominating the aggregated per-k signal
- Flagged candidate k values are consistent with known rare cluster sizes in validation data (e.g. tuft cells in the intestine dataset)

## Limitations

- Candidate k identification depends on the threshold skewness > 2; no alternative parameter settings (e.g. K, skewness threshold) are reported for all runs
- The statistic exploits the gap between a minor cluster and its neighbouring abundant cluster; clusters without a clear inter-cluster gap may not be flagged
- Sporadic distance spikes must be handled by the outlier filtering and averaging rules or they can dominate the ∆∆D signal

## Evidence

- [methods] GapClust calculates the skewness of the distribution of ∆∆D values across all cells as k varies from 1 to K, and candidate k are flagged when the skewness exceeds the threshold value 2.: [evidence span withheld — anchor status: not_found]
- [intro] A second-order derivative type distance score capturing the variation of distance changes is proposed to utilise the gap information.: [evidence span withheld — anchor status: not_found]
- [intro] the design of GapClust is inspired by the observation that for a particular data point in a minor cluster Cn of size n, then the distance to its nearest neighbours outside Cn is far greater than to: "for a particular data point in a minor cluster Cn of size n, then the distance to its nearest neighbours outside Cn is far greater than to"
- [discussion] Candidate k can be identified if the skewness<k> > 2.: "Candidate k can be identified if the skewness<k> > 2."
- [readme] e1071 >= 1.7-0.1 (README prerequisite used for skewness computation): "Seurat >=  3.1.0
rflann >=1.8.4 (GitHub: YeeJeremy/rflann)
irlba >=2.3.3
e1071 >= 1.7-0.1"
