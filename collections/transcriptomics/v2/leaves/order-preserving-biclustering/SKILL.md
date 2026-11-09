---
name: order-preserving-biclustering
description: Use when the goal is to find trend-preserving (order-preserving) biclusters
  in noisy, complex gene expression microarray data, where rows share a consistent
  upward/downward trend rather than identical absolute values.
when_to_use_negative:
- The analysis goal is finding biclusters with constant or additive/multiplicative
  coherent values rather than consistent row-wise ordering/trends
- The input is already a discrete or binary matrix — in that case use BCUnibicD (the
  discrete-data variant) instead of the continuous-data path with separation/granulation
  preprocessing
- Narrow biclusters are the primary target — BicSPAM was reported to be somewhat better
  at finding narrow biclusters, the task for which it was specifically designed
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3518
tools:
- name: runibic
  role: Parallel C++11 R implementation of UniBic; provides BCUnibic/BCUnibicD biclustering
    functions plus pairwiseLCS, calculateLCS, backtrackLCS, cluster, unisort, and
    discretize; invoked through the Biclust package wrapper
  repo: github.com/athril/runibic
- name: biclust
  role: R package whose wrapper interface (biclust(method=BCUnibic(), data)) is used
    to run UniBic and visualize results with drawHeatmap and parallelCoordinates
- name: UniBic
  role: Original sequential row-based biclustering algorithm whose six-step pipeline
    (index matrix creation, partition, LCS application, strict OP bicluster development,
    extension to approximately trend-preserving biclusters, significance output) defines
    the method
  repo: sourceforge.net/projects/unibic
- name: R
  role: Execution environment for runibic and biclust; installation via devtools::install_github
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/srep23466
    title: 'UniBic: Sequential row-based biclustering algorithm for analysis of gene
      expression data'
  source_content_hash: sha256:00681369f9fb72c2db9c1d2b433674376ad4082a583bd65bbbd06937a094c681
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - UniBic
  - biclust
  - runibic
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3518
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# order-preserving-biclustering

## Summary

UniBic-style trend-preserving biclustering: preprocess a gene expression matrix (data separation + granulation), build a row-wise index matrix, seed biclusters via longest common subsequences between row pairs, and extend seeds into approximately trend-preserving biclusters with significance filtering (p < 0.05).

## When to use

Use when the goal is to find trend-preserving (order-preserving) biclusters in noisy, complex gene expression microarray data, where rows share a consistent upward/downward trend rather than identical absolute values.

## When NOT to use

- The analysis goal is finding biclusters with constant or additive/multiplicative coherent values rather than consistent row-wise ordering/trends
- The input is already a discrete or binary matrix — in that case use BCUnibicD (the discrete-data variant) instead of the continuous-data path with separation/granulation preprocessing
- Narrow biclusters are the primary target — BicSPAM was reported to be somewhat better at finding narrow biclusters, the task for which it was specifically designed

## Inputs

- Gene expression microarray matrix (e.g. GEO GDS datasets such as GDS181, GDS589, GDS1406, GDS1451, GDS1490, GDS2520, GDS3715, GDS3716) as a numeric matrix or SummarizedExperiment
- Parameters q (default 15/m, or 0.5 for data without separation preprocessing) and r (interval count, e.g. 15) for preprocessing
- Significance level for index-matrix partitioning and bicluster filtering (default 0.05)

## Outputs

- Set of trend-preserving biclusters (row/column index sets) with p-values < 0.05, decreasingly ordered by significance
- Per-dataset bicluster counts (e.g. UniBic discovered 151 biclusters total across the eight GEO datasets, 62 of them GO-enriched)
- Heatmap and parallel-coordinate plots of individual biclusters (via biclust drawHeatmap/parallelCoordinates)

## How to apply

First preprocess the input matrix A: apply data separation by ordering each row, setting the percentage parameter q = 15/m and keeping entries beyond ail±d as up-/down-regulated values while zeroing others; then apply data granulation by partitioning up-regulated decreasingly ordered entries into r = 15 intervals mapped to integers i and down-regulated entries into r = 15 intervals mapped to −i, yielding integer matrix A''. Build the index matrix Y from A'' (ties broken by giving the smaller column index higher rank), compute an integer k from the significance level (default 0.05), and partition Y into k row subsets. Apply the LCS algorithm to each pair of rows in each subset to find significant longest common subsequences as seeds; develop strict order-preserving biclusters from the longest seeds, then extend them by repeatedly adding columns with error rate ≤ 0.3 and rows (original or negative) with error rate ≤ 0.15. Finally compute significance values and retain trend-preserving biclusters with p-values less than 0.05, decreasingly ordered by significance. The rationale is that discovering trend-preserving biclusters reduces to finding LCSs between rows of an index matrix, which locates a seed for each significant bicluster hidden in the background matrix.

## Related tools

- **runibic** (Parallel C++11 R implementation of UniBic; provides BCUnibic/BCUnibicD biclustering functions plus pairwiseLCS, calculateLCS, backtrackLCS, cluster, unisort, and discretize; invoked through the Biclust package wrapper) — github.com/athril/runibic
- **biclust** (R package whose wrapper interface (biclust(method=BCUnibic(), data)) is used to run UniBic and visualize results with drawHeatmap and parallelCoordinates)
- **UniBic** (Original sequential row-based biclustering algorithm whose six-step pipeline (index matrix creation, partition, LCS application, strict OP bicluster development, extension to approximately trend-preserving biclusters, significance output) defines the method) — sourceforge.net/projects/unibic
- **R** (Execution environment for runibic and biclust; installation via devtools::install_github)

## Examples

```
library(runibic); library(biclust); data(BicatYeast); res <- biclust(method=BCUnibic(), BicatYeast); drawHeatmap(BicatYeast, res, 1)
```

## Evaluation signals

- On the eight GEO real datasets (GDS181, GDS589, GDS1406, GDS1451, GDS1490, GDS2520, GDS3715, GDS3716) with q = 15/m and r = 15, the summed bicluster count should equal 151 for UniBic, matching the reported result
- Every retained bicluster must have a significance p-value less than 0.05
- Column extension must stop when no candidate column with error rate ≤ 0.3 remains; row extension must respect error rate ≤ 0.15 for original or negative rows
- Intermediate index matrix A'' must contain only integers from partitioning up-regulated entries into r intervals mapped to i and down-regulated entries into −i intervals
- Recovered biclusters should show consistent per-row ordering/trends; sanity-check a bicluster with drawHeatmap/parallelCoordinates in biclust

## Limitations

- Identification of general trend-preserving biclusters remains a challenging problem
- UniBic was somewhat outperformed by BicSPAM on narrow biclusters, the task BicSPAM was specifically designed for
- Preprocessing (separation with q = 15/m and granulation with r intervals) is needed to alleviate adverse impacts of entries whose corresponding genes are not activated under all conditions; skipping it changes behavior (q = 0.5 is specially provided for data without separation preprocessing)
- Tie handling in index matrix construction (giving the smaller column index higher rank) affects the derived ordering and hence seeds

## Evidence

- [abstract] The basic idea is to apply the longest common subsequence (LCS) framework to selected pairs of rows in an index matrix derived from an input data matrix to locate a seed for each bicluster to be: "The basic idea is to apply the longest common subsequence (LCS) framework to selected pairs of rows in an index matrix derived from an input data matrix to locate a seed for each bicluster to be"
- [methods] we extend it by first repeatedly adding new columns one at a time with an error rate r ≤ 0.3 until none is available.: "we extend it by first repeatedly adding new columns one at a time with an error rate r ≤ 0.3 until none is available."
- [methods] Those with p-values less than 0.05 are decreasingly ordered in their significance.: "We calculate the significance value for those trend-preserving biclusters obtained in step 5. Those with p-values less than 0.05 are decreasingly ordered in their significance."
- [readme] This package contains implementation of UniBic biclustering algorithm for gene expression data [Wang2016] The algorithm tries to locate trend-preserving biclusters within complex and noisy data.: "This package contains implementation of UniBic biclustering algorithm for gene expression data [Wang2016]
The algorithm tries to locate trend-preserving biclusters within complex and noisy data."
- [discussion] the identification of general trend-preserving biclusters remains a challenging problem: "the identification of general trend-preserving biclusters remains a challenging problem"
