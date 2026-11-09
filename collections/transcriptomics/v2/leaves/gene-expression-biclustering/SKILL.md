---
name: gene-expression-biclustering
description: Use when the input is a continuous (or discretized) gene expression matrix
  and the goal is to discover overlapping trend-preserving (order-preserving) biclusters
  and compare them against planted ground truth or other algorithms.
when_to_use_negative:
- The target biclusters are narrow constant-row/constant-column patterns specifically
  designed for BicSPAM, which outperformed UniBic on narrow biclusters
- The data are raw sequencing reads or unmatrixed measurements rather than a rows-by-conditions
  expression matrix
- A clustering that partitions all rows into disjoint groups is sufficient — UniBic
  seeks overlapping local trend-preserving submatrices, not global partitions
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: runibic
  role: Parallel C++11 UniBic implementation providing BCUnibic/BCUnibicD, pairwiseLCS,
    cluster, unisort and discretize functions called through the Biclust wrapper
  repo: https://github.com/athril/runibic
- name: biclust
  role: Provides the Biclust method wrapper (biclust(method=BCUnibic(), ...)) and
    the BicatYeast sample dataset
- name: QUBIC
  role: Comparison algorithm loaded alongside UniBic; its showinfo function extracts
    per-method bicluster statistics for benchmarking
- name: R
  role: Execution environment; runibic installed via devtools::install_github("athril/runibic")
provenance:
  source_task_ids:
  - task_003
  - task_008
  - task_004
  - task_002
  source_papers:
  - doi: 10.1038/srep23466
    title: 'UniBic: Sequential row-based biclustering algorithm for analysis of gene
      expression data'
  source_content_hash: sha256:80dfe5793e84069e8410a0a42068f716117120cfcc4d51bd2598124cb7460327
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/athril/runibic
  tools:
  - QUBIC
  - R
  - UniBic
  - biclust
  - runibic
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# gene-expression-biclustering

## Summary

Detects trend-preserving biclusters in gene expression matrices using UniBic, which finds LCS-based seeds between rows of an index matrix, then extends and filters biclusters by significance. The runibic R package provides a parallel C++11 implementation with a Biclust wrapper.

## When to use

Use when the input is a continuous (or discretized) gene expression matrix and the goal is to discover overlapping trend-preserving (order-preserving) biclusters and compare them against planted ground truth or other algorithms.

## When NOT to use

- The target biclusters are narrow constant-row/constant-column patterns specifically designed for BicSPAM, which outperformed UniBic on narrow biclusters
- The data are raw sequencing reads or unmatrixed measurements rather than a rows-by-conditions expression matrix
- A clustering that partitions all rows into disjoint groups is sufficient — UniBic seeks overlapping local trend-preserving submatrices, not global partitions

## Inputs

- Continuous gene expression matrix (e.g., BicatYeast from the biclust R package)
- SummarizedExperiment object (e.g., airway dataset)
- Discretized expression matrix (for BCUnibicD)
- Synthetic matrices with implanted ground-truth biclusters (UniBic SourceForge deposit)

## Outputs

- Biclust result object (res@Number biclusters with row/column memberships)
- Heatmaps and parallel-coordinate plots via drawHeatmap/parallelCoordinates
- relevance/recovery (Jaccard-based S(D,G), S(G,D)) score tables, e.g. overlap_scores.csv
- Side-by-side method comparison table, e.g. bicluster_comparison.csv

## How to apply

Load the expression matrix (e.g., BicatYeast from biclust, airway, or synthetic matrices from the UniBic SourceForge deposit) and run BCUnibic via the biclust wrapper with default significance 0.05; the algorithm creates an index matrix, partitions it into k subsets, and applies LCS to row pairs to seed each bicluster. Extend seeds by adding columns with error rate r ≤ 0.3, then rows with error rate ≤ 0.15, keeping trend-preserving biclusters with p-values < 0.05 (up to o=100, ordered by significance). For data without separation preprocessing set q=0.5 (the default q is 15/m); for discrete data use BCUnibicD. Validate discovered biclusters against planted truth by computing relevance S(D,G) and recovery S(G,D) (Jaccard-based), or compare across methods (BCCC, BCBimax, BCPlaid, BCQU, BCUnibic) using QUBIC::showinfo. A known reference result is 92 biclusters (res@Number) on BicatYeast.

## Related tools

- **runibic** (Parallel C++11 UniBic implementation providing BCUnibic/BCUnibicD, pairwiseLCS, cluster, unisort and discretize functions called through the Biclust wrapper) — https://github.com/athril/runibic
- **biclust** (Provides the Biclust method wrapper (biclust(method=BCUnibic(), ...)) and the BicatYeast sample dataset)
- **QUBIC** (Comparison algorithm loaded alongside UniBic; its showinfo function extracts per-method bicluster statistics for benchmarking)
- **R** (Execution environment; runibic installed via devtools::install_github("athril/runibic"))

## Examples

```
library(runibic); library(biclust); data(BicatYeast); res <- biclust(method=BCUnibic(), BicatYeast); res@Number
```

## Evaluation signals

- On BicatYeast with defaults, BCUnibic returns res@Number == 92 biclusters
- All output biclusters have p-values < 0.05, up to o=100, decreasingly ordered in significance
- On Type I synthetic matrices run with q=0.5, r=m, Jaccard relevance ≈ 0.65 and recovery ≈ 0.69, exceeding BicSPAM (0.33) and OPSM (0.39)
- Relevance/recovery scores remain stable as overlap of implanted biclusters increases from 0x0 to 9x9, confirming overlap robustness
- QUBIC::showinfo on the five Biclust objects (resCC, resBi, resPlaid, resQub, resUni) yields numeric bicluster statistics for every method in the comparison table

## Limitations

- Identification of general trend-preserving biclusters remains a challenging problem
- BicSPAM performs better on narrow biclusters, a task for which it was specifically designed
- Preprocessing (data separation with q, granulation into r intervals) affects results; skipping it requires q=0.5

## Evidence

- [methods] LCS seed framework: "we designed a novel algorithm by applying the LCS algorithm to selected pairs of rows of an index matrix derived from the input data matrix."
- [methods] significance and partitioning: "We calculate an integer k based on the significance (default set to 0.05) of the to-be-identified trend-preserving biclusters using the techniques developed in19."
- [methods] preprocessing parameter q: "we chose a percentage parameter q with the default value set to 15/m (the value q = 0.5 is specially provided for data without separation preprocessing)"
- [abstract] benchmark result: "UniBic, outperformed all previous biclustering algorithms in terms of commonly used evaluation scenarios except for BicSPAM on narrow biclusters."
- [readme] package scope: "This package contains implementation of UniBic biclustering algorithm for gene expression data"
- [readme] concrete invocation: "res <- biclust(method=BCUnibic(),BicatYeast)"
