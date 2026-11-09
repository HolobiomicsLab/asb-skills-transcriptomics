---
name: gene-expression-matrix-normalization
description: Use when raw scRNA-seq count matrices (e.g., GSE65525 Buettner, GSE74547
  Kolodziejczyk, GSE75748 Chu-time) must be normalized before c-CSN/CNDM construction
  or before clustering comparisons against GEM.
when_to_use_negative:
- Input is already a c-CSN-transformed conditional network degree matrix (CNDM) —
  normalization of the GEM is upstream and the matrix no longer represents expression
  counts
- Task is differentiation-potency or trajectory-direction analysis (network flow entropy,
  NFE) rather than preparing an expression matrix for network construction or clustering
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: c-CSN
  role: 'Downstream consumer of the normalized GEM: constructs CCSNs and the CNDM
    used for clustering'
  repo: https://github.com/LinLi-0909/c-CSN
- name: SC3
  role: Applied downstream to both the normalized GEM and the CNDM to compute clustering
    ARI
- name: Seurat
  role: Clustering method applied to the normalized GEM and CNDM in the benchmark
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.gpb.2020.05.005
    title: 'c-CSN: Single-Cell RNA Sequencing Data Analysis by Conditional Cell-Specific
      Network'
  source_content_hash: sha256:bd13c40db1f1af5a055cc364463bea3cb513e021b3f756c27728af8cb841bc92
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/LinLi-0909/c-CSN
  tools:
  - SC3
  - Seurat
  - c-CSN
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# gene-expression-matrix-normalization

## Summary

Normalizes raw scRNA-seq gene expression matrices (GEMs) so they are suitable as inputs to c-CSN conditional cell-specific network construction and downstream clustering/ARI benchmarking, per the c-CSN pipeline of Li et al. (2021).

## When to use

Use when raw scRNA-seq count matrices (e.g., GSE65525 Buettner, GSE74547 Kolodziejczyk, GSE75748 Chu-time) must be normalized before c-CSN/CNDM construction or before clustering comparisons against GEM.

## When NOT to use

- Input is already a c-CSN-transformed conditional network degree matrix (CNDM) — normalization of the GEM is upstream and the matrix no longer represents expression counts
- Task is differentiation-potency or trajectory-direction analysis (network flow entropy, NFE) rather than preparing an expression matrix for network construction or clustering

## Inputs

- Raw scRNA-seq gene expression matrix (GEM), genes × cells, e.g. from Buettner (GSE65525), Kolodziejczyk (GSE74547), Chu-time (GSE75748)
- Gold-standard cell type labels with published cluster counts for each dataset

## Outputs

- Normalized gene expression matrix suitable for c-CSN/CCSN construction and for the GEM arm of the ARI clustering benchmark

## How to apply

Obtain the raw scRNA-seq GEM (genes × cells) together with gold-standard cell type labels, e.g. Buettner (GSE65525), Kolodziejczyk (GSE74547), or Chu-time (GSE75748), and normalize each gene expression matrix before any network step. Normalization matters because the c-CSN hypothesis test numerically estimates ρxy|z from a scatter diagram in 3D expression space, so unnormalized count scale would distort the conditional-independence statistic. The normalized GEM feeds construction of CCSNs per cell (conditional genes chosen as top-degree hub genes from the CSN) and aggregation into the m × n conditional network degree matrix (CNDM). The same normalized GEM serves as the baseline (GEM arm) in the ARI benchmark where K-means, hierarchical clustering, K-medoids, and SC3 are run with the published cell type counts as the number of clusters. Correct application is validated pairwise: the CNDM ARI must exceed the GEM ARI for the same method and dataset on all evaluated datasets.

## Related tools

- **c-CSN** (Downstream consumer of the normalized GEM: constructs CCSNs and the CNDM used for clustering) — https://github.com/LinLi-0909/c-CSN
- **SC3** (Applied downstream to both the normalized GEM and the CNDM to compute clustering ARI)
- **Seurat** (Clustering method applied to the normalized GEM and CNDM in the benchmark)

## Evaluation signals

- The normalized GEM produces, per method (K-means, hierarchical, K-medoids, SC3) and per dataset (Buettner, Kolodziejczyk, Chu-time), a CNDM ARI strictly greater than the corresponding GEM ARI
- ARI values are computed against gold-standard cell type labels using the published cell type counts as the number of clusters
- Downstream CNDM output has the expected m × n (genes × cells) shape of the conditional network degree matrix
- The GEM arm reproduces a valid ARI baseline (mirroring Table 1), i.e. clustering on the raw/normalized GEM is a non-degenerate comparison

## Limitations

- No parallel computation scheme is provided in the c-CSN pipeline despite the G-fold computational cost increase; the authors state it is desired
- c-CSN is not designed to construct causal gene association networks; directions of gene associations cannot be obtained
- Normalization choice is not specified per-method in the source; the paper only requires the GEM be normalized before the conditional-statistic estimation step

## Evidence

- [other] Workflow step 1 requires normalized GEM before CCSN construction: [evidence span withheld — anchor status: not_found]
- [methods] Conditional statistic is estimated from expression data, so scale matters: "We numerically estimate the value of ρxy|z by making a scatter diagram based on gene expression data"
- [intro] c-CSN transforms expression data into network data, motivating the normalized input: "c-CSN can transform the original gene expression data of each cell to the direct and robust gene–gene association data (or network data) of the same cell"
- [methods] Expected outcome of the downstream benchmark on all datasets: "CNDM performs obviously better than GEM on all datasets"
