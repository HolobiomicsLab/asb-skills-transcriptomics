---
name: gene-expression-data-separation-and-granulation
description: Use when input is a continuous gene-expression microarray or RNA-seq
  matrix to be clustered for trend-preserving (order-preserving) biclusters, and many
  genes are inactive under some conditions, so raw-value biclustering would be distorted.
when_to_use_negative:
- Input is already discrete/integer data — use BCUnibicD (parallel UniBic for discrete
  data) instead of re-granulating
- The analysis requires preserving expression magnitudes or fold-changes; separation
  zeroes out all entries within ail±d, discarding value information
- Dataset has no meaningful up-/down-regulation structure relative to the row middle
  element, forcing the fallback q = 0.5 setting that bypasses separation
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: runibic
  role: Parallel C++11/R implementation of UniBic whose preprocessing (data separation,
    granulation, index-matrix construction via unisort/discretize) is applied to the
    expression matrix before bicluster seeding
  repo: github.com/athril/runibic
- name: biclust
  role: R package wrapper through which BCUnibic/BCUnibicD are invoked on the preprocessed
    matrix
- name: R
  role: Environment in which runibic is installed via devtools and the preprocessing–biclustering
    workflow is executed
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/srep23466
    title: 'UniBic: Sequential row-based biclustering algorithm for analysis of gene
      expression data'
  source_content_hash: sha256:abc16fd9e431354161b50e0ad22399de742e8be33762d96af748fbb2f2186ff6
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - UniBic
  - biclust
  - runibic
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# Gene-expression data separation and granulation

## Summary

UniBic's two-stage preprocessing that converts a noisy continuous gene-expression matrix A into an integer index-ready matrix A'' — first zeroing out non-differentially expressed entries (data separation with q = 15/m), then mapping remaining up-/down-regulated entries into r = 15 integer intervals (data granulation).

## When to use

Use when input is a continuous gene-expression microarray or RNA-seq matrix to be clustered for trend-preserving (order-preserving) biclusters, and many genes are inactive under some conditions, so raw-value biclustering would be distorted.

## When NOT to use

- Input is already discrete/integer data — use BCUnibicD (parallel UniBic for discrete data) instead of re-granulating
- The analysis requires preserving expression magnitudes or fold-changes; separation zeroes out all entries within ail±d, discarding value information
- Dataset has no meaningful up-/down-regulation structure relative to the row middle element, forcing the fallback q = 0.5 setting that bypasses separation

## Inputs

- Continuous gene-expression matrix A (e.g. GEO microarray datasets such as GDS181, GDS589, GDS1406, GDS1451, GDS1490, GDS2520, GDS3715, GDS3716)
- Percentage parameter q (default 15/m; q = 0.5 for data without separation preprocessing)
- Granulation interval parameter r (user-specifiable; e.g. r = 15)

## Outputs

- Data-separated matrix A' (rows ordered, non-regulated entries zeroed)
- Integer granulated matrix A'' with up-regulated entries as positive integers i and down-regulated entries as negative integers −i over r intervals
- Index matrix Y derived from A'' via unisort (row-wise increasing-order ranks)

## How to apply

Apply data separation first: order each row of the input matrix A, set the percentage parameter q to 15/m (default), compute the middle element ail with l = m/2 and d = min{ail−ais, ait−ail}, keep only entries beyond ail±d as up-/down-regulated values, and zero out all other entries to form A'. Then apply data granulation: partition the up-regulated decreasingly ordered entries in each row of A' into r (user-specifiable, e.g. r = 15) intervals mapped to positive integers i, and the down-regulated increasingly ordered entries into r intervals mapped to negative integers −i, yielding integer matrix A''. The rationale is to alleviate adverse impacts of entries whose genes are not activated under all conditions before the index matrix Y is built and LCS-based bicluster seeding begins in runibic.

## Related tools

- **runibic** (Parallel C++11/R implementation of UniBic whose preprocessing (data separation, granulation, index-matrix construction via unisort/discretize) is applied to the expression matrix before bicluster seeding) — github.com/athril/runibic
- **biclust** (R package wrapper through which BCUnibic/BCUnibicD are invoked on the preprocessed matrix)
- **R** (Environment in which runibic is installed via devtools and the preprocessing–biclustering workflow is executed)

## Examples

```
library(runibic); res <- biclust(method=BCUnibic(), BicatYeast)
```

## Evaluation signals

- After separation, A' contains zeros exactly for entries falling within ail±d of each row and nonzero values only beyond that interval
- A'' is an integer matrix whose positive entries come from up-regulated decreasingly ordered partitions (1..r) and negative entries from down-regulated increasing partitions (−1..−r)
- unisort on A'' yields an index matrix whose row ranks are consistent with the increasing order of each granulated row
- Downstream sanity check: at q = 15/m and r = 15 on the eight GEO datasets, UniBic should recover 151 total significant biclusters (62 GO-enriched), matching the reported result

## Limitations

- Identification of general trend-preserving biclusters remains a challenging problem even after this preprocessing
- Granulation discards magnitude information; entries within ail±d are zeroed, so biclusters found reflect trends, not expression levels
- q and r must be chosen per dataset (q = 15/m default; r user-specified); no automatic parameter tuning is described

## Evidence

- [methods] Data separation: "we chose a percentage parameter q with the default value set to 15/m (the value q = 0.5 is specially provided for data without separation preprocessing)"
- [methods] Data granulation: "we further preprocessed the input data by equally partitioning all the up-regulated decreasingly ordered entries in each row of A’ into r (a parameter which may be pre-specified by user) intervals"
- [methods] Data Preprocessing: "it is better to preprocess the input data to alleviate the adverse impacts to data entries since corresponding genes are not activated under all conditions"
- [readme] Index matrix construction: "unisort - returns matrix of indexes based on the increasing order in each row"
- [readme] Discrete vs continuous handling: "BCUnibic`/`runibic` - parallel UniBic for continuous data"
