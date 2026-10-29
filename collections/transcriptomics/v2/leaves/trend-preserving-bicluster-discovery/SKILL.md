---
name: trend-preserving-bicluster-discovery
description: Use when the input is a continuous gene expression or synthetic numeric
  matrix and the goal is to detect trend-preserving biclusters whose rows maintain
  relative order of values across a subset of columns.
when_to_use_negative:
- When the target biclusters are narrow — the article reports BicSPAM outperformed
  UniBic specifically on narrow biclusters
- When a constant-value (not trend/order-preserving) biclustering criterion is required
  — UniBic is designed for trend-preserving patterns
- When the identification of general (non-trend-preserving) biclusters is required
  — the paper notes this remains a challenging unsolved problem
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: UniBic
  role: Sequential row-based biclustering algorithm that finds LCS seeds in an index
    matrix to discover trend-preserving biclusters
  repo: http://sourceforge.net/projects/unibic
- name: runibic
  role: Parallel C++11 R implementation of UniBic; provides BCUnibic/BCUnibicD and
    LCS helper functions (pairwiseLCS, calculateLCS, backtrackLCS, unisort, cluster,
    discretize)
  repo: https://github.com/athril/runibic
- name: biclust
  role: Provides the Biclust wrapper class for UniBic results, the BicatYeast sample
    dataset, and drawHeatmap/parallelCoordinates visualization
- name: QUBIC
  role: Comparison biclustering algorithm loaded for benchmarking against UniBic
provenance:
  source_task_ids:
  - task_002
  - task_004
  source_papers:
  - doi: 10.1038/srep23466
    title: 'UniBic: Sequential row-based biclustering algorithm for analysis of gene
      expression data'
  source_content_hash: sha256:c546a05e0abcb8d88d6f132d692de448fb2c5a5277aa8545a1469a5a0cfc07a3
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - QUBIC
  - R
  - UniBic
  - biclust
  - runibic
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 16
---

# trend-preserving-bicluster-discovery

## Summary

Discovery of trend-preserving (order-preserving) biclusters in noisy gene expression matrices by applying the longest common subsequence (LCS) framework to pairs of rows of an index matrix derived from the input data, as implemented in UniBic and its parallel R port runibic.

## When to use

Use when the input is a continuous gene expression or synthetic numeric matrix and the goal is to detect trend-preserving biclusters whose rows maintain relative order of values across a subset of columns.

## When NOT to use

- When the target biclusters are narrow — the article reports BicSPAM outperformed UniBic specifically on narrow biclusters
- When a constant-value (not trend/order-preserving) biclustering criterion is required — UniBic is designed for trend-preserving patterns
- When the identification of general (non-trend-preserving) biclusters is required — the paper notes this remains a challenging unsolved problem

## Inputs

- Continuous gene expression data matrix (e.g. BicatYeast from the biclust package, airway SummarizedExperiment, GDS589 from GEO)
- Synthetic Type I test matrices with implanted trend-preserving biclusters (from the UniBic SourceForge deposit)
- Discrete/integer matrix (handled by BCUnibicD)

## Outputs

- Biclust result object (res@Number of biclusters, row/column membership per bicluster), e.g. saved as bicatyeast_unibic_biclust_result.rds
- Heatmap and parallel-coordinates visualizations per bicluster (drawHeatmap, parallelCoordinates)
- Jaccard-based relevance S(D,G) and recovery S(G,D) scores when benchmarking against genuine implanted biclusters

## How to apply

Derive an index matrix Y from the input matrix A, then optionally preprocess by Data Separation (percentage parameter q, default 15/m; set q=0.5 to skip separation preprocessing) and Data Granulation into r intervals (r = m). Partition Y into k subsets computed from the significance level (default 0.05), apply the LCS algorithm to each pair of rows in each subset to find significant longest common subsequences that seed candidate biclusters, then develop strict order-preserving biclusters and extend them by adding columns with error rate r ≤ 0.3. Compute significance for each extended trend-preserving bicluster, keep those with p < 0.05, order them decreasingly by significance, and output the requested number (default o=100). Run the parallel implementation via runibic in R, wrapping results in the Biclust object model for inspection and visualization with drawHeatmap and parallelCoordinates.

## Related tools

- **UniBic** (Sequential row-based biclustering algorithm that finds LCS seeds in an index matrix to discover trend-preserving biclusters) — http://sourceforge.net/projects/unibic
- **runibic** (Parallel C++11 R implementation of UniBic; provides BCUnibic/BCUnibicD and LCS helper functions (pairwiseLCS, calculateLCS, backtrackLCS, unisort, cluster, discretize)) — https://github.com/athril/runibic
- **biclust** (Provides the Biclust wrapper class for UniBic results, the BicatYeast sample dataset, and drawHeatmap/parallelCoordinates visualization)
- **QUBIC** (Comparison biclustering algorithm loaded for benchmarking against UniBic)

## Examples

```
library(runibic); library(biclust); data(BicatYeast); res <- biclust(method=BCUnibic(), BicatYeast); res@Number
```

## Evaluation signals

- On BicatYeast, the returned Biclust object satisfies res@Number == 92 biclusters
- Benchmarked on Type I synthetic matrices with q=0.5 and r=m, UniBic reproduces relevance ≈ 0.65 (next best BicSPAM 0.33) and recovery ≈ 0.69 (next best OPSM 0.39)
- All reported biclusters have p-values < 0.05 significance and are ordered decreasingly by significance
- Output count matches the user-requested number of biclusters (default o=100) and each is trend-preserving (order-consistent across its columns)
- LCS helper functions behave correctly: backtrackLCS recovers an LCS consistent with the dynamic programming matrix computed by pairwiseLCS/calculateLCS

## Limitations

- Identification of general trend-preserving biclusters remains a challenging problem
- Without data separation/granulation preprocessing (q=0.5), results depend on the significance threshold; preprocessing is recommended to alleviate adverse impacts on data entries where genes are not activated under all conditions
- The article discussion lacks versions, parameter values, dataset accessions, and software availability details, so exact reproduction may require the SourceForge deposit and runibic vignette
- Column extension tolerates an error rate r ≤ 0.3, so approximately (not strictly) trend-preserving biclusters are output in step 5

## Evidence

- [abstract] LCS framework for bicluster seeding: "The basic idea is to apply the longest common subsequence (LCS) framework to selected pairs of rows in an index matrix derived from an input data matrix to locate a seed for each bicluster to be"
- [methods] parameter q=0.5 skips separation preprocessing: "we chose a percentage parameter q with the default value set to 15/m (the value q = 0.5 is specially provided for data without separation preprocessing)"
- [methods] column extension error rate: "we extend it by first repeatedly adding new columns one at a time with an error rate r ≤ 0.3 until none is available."
- [methods] significance and ordering of output biclusters: "We calculate the significance value for those trend-preserving biclusters obtained in step 5. Those with p-values less than 0.05 are decreasingly ordered in their significance."
- [readme] README example invocation on BicatYeast: "res <- biclust(method=BCUnibic(),BicatYeast)"
