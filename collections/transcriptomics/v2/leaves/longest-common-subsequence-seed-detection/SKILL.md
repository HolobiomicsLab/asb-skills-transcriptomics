---
name: longest-common-subsequence-seed-detection
description: Use when clustering continuous gene expression microarray or RNA-seq
  matrix data to discover trend-preserving (order-preserving) biclusters hidden in
  complex, noisy data, e.g. GEO GDS datasets or BicatYeast.
when_to_use_negative:
- The goal is to find narrow biclusters specifically — the article reports BicSPAM
  was somewhat better at that task
- The input data are already discretized/integer-level and the target is not trend
  (order) preservation — other biclustering formulations may be more appropriate
- Only a single row (not a pair of rows of an index matrix) can be compared — the
  LCS seeding framework requires row-pair subsequences
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: runibic
  role: Parallel C++11 R/Bioconductor implementation of UniBic providing the LCS-based
    seed detection (BCUnibic/runibic, BCUnibicD for discrete data, pairwiseLCS, backtrackLCS,
    cluster, unisort)
  repo: github.com/athril/runibic
- name: biclust
  role: Biclust package wrapper used to invoke BCUnibic and to provide the BicatYeast
    sample dataset
- name: R
  role: Runtime environment for runibic and biclust (installed via devtools::install_github("athril/runibic"))
- name: UniBic
  role: Original sequential algorithm whose Step 3 applies the LCS algorithm to each
    pair of rows in each of the k subsets of the index matrix Y
  repo: sourceforge.net/projects/unibic
provenance:
  source_task_ids:
  - task_005
  - task_004
  source_papers:
  - doi: 10.1038/srep23466
    title: 'UniBic: Sequential row-based biclustering algorithm for analysis of gene
      expression data'
  source_content_hash: sha256:a2b91bdccc5e33cfe1220ef0564bb04a751d62d7916296369fa6ecfebfbc287b
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
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
yes_rate: 0.4444444444444444
n_claims: 9
---

# longest-common-subsequence-seed-detection

## Summary

Locates trend-preserving bicluster seeds in a gene expression matrix by applying the longest common subsequence (LCS) algorithm to selected pairs of rows of an index matrix derived from the input data, as implemented in the UniBic algorithm and its parallel runibic package.

## When to use

Use when clustering continuous gene expression microarray or RNA-seq matrix data to discover trend-preserving (order-preserving) biclusters hidden in complex, noisy data, e.g. GEO GDS datasets or BicatYeast.

## When NOT to use

- The goal is to find narrow biclusters specifically — the article reports BicSPAM was somewhat better at that task
- The input data are already discretized/integer-level and the target is not trend (order) preservation — other biclustering formulations may be more appropriate
- Only a single row (not a pair of rows of an index matrix) can be compared — the LCS seeding framework requires row-pair subsequences

## Inputs

- Gene expression data matrix A (rows = genes, columns = conditions), e.g. BicatYeast from the biclust R package or GEO datasets such as GDS589
- Index matrix Y derived from A (optionally after data separation and data granulation preprocessing)
- Parameter significance level (default 0.05) used to compute the number k of row partitions

## Outputs

- Set of significant trend-preserving biclusters (rows of genes with concordant column-wise expression trends), p < 0.05
- Biclust result object (e.g. res@Number biclusters, visualizable with drawHeatmap and parallelCoordinates)
- Longest common subsequences (seeds) computed between pairs of rows of the index matrix

## How to apply

First derive an index matrix Y from the input matrix A (after optional data separation with percentage parameter q = 15/m and data granulation into r=15 up/down-regulated intervals). Then compute an integer k from the significance of the to-be-identified biclusters (default 0.05) and equally partition Y into k row subsets. Apply the LCS algorithm to each pair of rows within each subset to find all significant longest common subsequences, breaking ties by giving the smaller column index higher rank. Start from the longest seed at the front of the seed list and develop strict order-preserving biclusters, then extend them to approximately trend-preserving biclusters by adding columns one at a time with error rate ≤ 0.3 and rows with error rate ≤ 0.15. Finally retain biclusters whose significance p-values are less than 0.05, ordered decreasingly by significance. The rationale is that finding the LCS between two rows of the index matrix transforms the discovery of trend-preserving biclusters into a simple subsequence problem.

## Related tools

- **runibic** (Parallel C++11 R/Bioconductor implementation of UniBic providing the LCS-based seed detection (BCUnibic/runibic, BCUnibicD for discrete data, pairwiseLCS, backtrackLCS, cluster, unisort)) — github.com/athril/runibic
- **biclust** (Biclust package wrapper used to invoke BCUnibic and to provide the BicatYeast sample dataset)
- **R** (Runtime environment for runibic and biclust (installed via devtools::install_github("athril/runibic")))
- **UniBic** (Original sequential algorithm whose Step 3 applies the LCS algorithm to each pair of rows in each of the k subsets of the index matrix Y) — sourceforge.net/projects/unibic

## Examples

```
library(runibic); library(biclust); data(BicatYeast); res <- biclust(method=BCUnibic(), BicatYeast); res@Number
```

## Evaluation signals

- Applying BCUnibic to the BicatYeast dataset from the biclust package returns res@Number = 92 biclusters (as shown in the runibic vignette)
- Summed significant bicluster count across the eight GEO datasets (GDS181, GDS589, GDS1406, GDS1451, GDS1490, GDS2520, GDS3715, GDS3716) with q = 15/m and r = 15 equals 151
- All retained biclusters have significance p-values less than 0.05 and are ordered decreasingly in significance
- Column extension respects the error rate ≤ 0.3 and row extension the error rate ≤ 0.15 thresholds
- Index matrix Y is partitioned into exactly k subsets, where k is computed from the significance level (default 0.05)

## Limitations

- Identification of general trend-preserving biclusters remains a challenging problem
- The original article's discussion provides no code/URL, no dataset accessions, and no version/parameter details for UniBic (recovered from the repo and runibic instead)
- UniBic outperformed previous biclustering algorithms except BicSPAM on narrow biclusters
- Ties in the index matrix must be broken by giving the smaller column index higher rank; deviating from this changes seeds

## Evidence

- [methods] LCS applied to row pairs of index matrix: "we designed a novel algorithm by applying the LCS algorithm to selected pairs of rows of an index matrix derived from the input data matrix."
- [methods] Step 3 Application of LCS: "Apply the LCS algorithm to each pair of rows in each of the k subsets of Y to find all the significant longest common subsequences."
- [methods] Step 2 partition: "We calculate an integer k based on the significance (default set to 0.05) of the to-be-identified trend-preserving biclusters"
- [methods] Column extension error rate: "we extend it by first repeatedly adding new columns one at a time with an error rate r ≤ 0.3 until none is available."
- [readme] runibic package description: "This package contains implementation of UniBic biclustering algorithm for gene expression data [Wang2016]
The algorithm tries to locate trend-preserving biclusters within complex and noisy data."
- [readme] BicatYeast example invocation: "res <- biclust(method=BCUnibic(),BicatYeast)"
- [discussion] Transformation rationale: "provides a transformation from the problem of discovering trend-preserving biclusters in a background matrix to a simple problem of finding the longest common subsequence"
