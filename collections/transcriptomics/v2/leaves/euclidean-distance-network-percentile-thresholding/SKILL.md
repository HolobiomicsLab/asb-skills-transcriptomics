---
name: euclidean-distance-network-percentile-thresholding
description: Use when building condition-specific (normal vs cancer) gene co-expression
  networks from TCGA RNA-Seq expression data and you need a distribution-based, top-percentile
  sparsification of Euclidean-distance similarity edges before network fusion.
when_to_use_negative:
- You are filtering Pearson-correlation co-expression networks — these use a permutation
  test (10 random shuffles) plus a p-value percentile threshold instead of the Euclidean
  99th-percentile rule
- You need the complete dense similarity matrix or a fixed absolute weight cutoff
  rather than a distribution-relative (top ~1%) edge set
- Your goal is a single co-expression network rather than multiple similarity-measure
  networks that will be merged and SNF-fused per cancer type
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: Python
  role: Implements the Euclidean-distance network computation and the 99th-percentile
    edge filtering inside creation_matrices.py / Pipeline.ipynb
  repo: https://github.com/DEIB-GECO/GeneNetFusion
- name: Jupyter Notebook
  role: Pipeline.ipynb executes the complete pipeline step that thresholds the Euclidean
    networks at the 99th percentile
  repo: https://github.com/DEIB-GECO/GeneNetFusion/blob/master/Pipeline.ipynb
- name: PyGMQL
  role: Retrieves and pre-processes the TCGA gene expression data (zFPKM > -3.0 filter)
    that the Euclidean networks are built from
  repo: https://pygmql.readthedocs.io/en/latest/
- name: GMQL
  role: Query language used to extract the TCGA RNA-Seq expression datasets per cancer
    type and condition
  repo: http://gmql.eu
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41540-021-00175-9
    title: Computational analysis of fused co-expression networks for the identification
      of candidate cancer gene biomarkers
  source_content_hash: sha256:5fc6bb7f2df1c2778ca8a4d03f492b1beea33933e3463464dd2c9f7b28fec837
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/DEIB-GECO/GeneNetFusion
  tools:
  - GMQL
  - Jupyter Notebook
  - PyGMQL
  - Python
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# Euclidean-distance-network-percentile-thresholding

## Summary

Filters condition-specific Euclidean-distance gene co-expression networks by retaining only edges whose weight exceeds the 99th percentile of the network's edge-weight distribution, producing a sparse per-condition network that is combined with the Pearson-correlation network and fused via SNF for KIRC, LIHC and PRAD.

## When to use

Use when building condition-specific (normal vs cancer) gene co-expression networks from TCGA RNA-Seq expression data and you need a distribution-based, top-percentile sparsification of Euclidean-distance similarity edges before network fusion.

## When NOT to use

- You are filtering Pearson-correlation co-expression networks — these use a permutation test (10 random shuffles) plus a p-value percentile threshold instead of the Euclidean 99th-percentile rule
- You need the complete dense similarity matrix or a fixed absolute weight cutoff rather than a distribution-relative (top ~1%) edge set
- Your goal is a single co-expression network rather than multiple similarity-measure networks that will be merged and SNF-fused per cancer type

## Inputs

- TCGA RNA-Seq gene expression data per cancer type and condition (normal/cancer), protein-coding and lncRNA genes with mean zFPKM > -3.0, retrieved via GMQL/PyGMQL
- Pairwise Euclidean-distance adjacency matrix per condition (KIRC, LIHC, PRAD; normal and cancer)

## Outputs

- Per-condition Euclidean co-expression adjacency matrices retaining only edges above the 99th percentile of the edge-weight distribution
- Sparse condition-specific networks that feed the merged (Euclidean + Pearson) sample networks and the final SNF-fused adjacency matrix (e.g. fused_network_KIRC.csv)

## How to apply

After pre-processing TCGA RNA-Seq data with GMQL/PyGMQL (protein-coding and long non-coding genes with mean zFPKM higher than -3.0 in both normal and cancer conditions), compute the Euclidean-distance co-expression network separately for the normal and the cancer condition of each cancer type (KIRC, LIHC, PRAD). Sort the network edge-weight distribution and keep only edges with a weight higher than the 99th percentile, i.e. retain roughly the top 1% most similar gene pairs; this is a per-network adaptive threshold, not a fixed weight cutoff. Apply the same 99th-percentile logic per condition so that the resulting Euclidean edge sets serve as the size reference for pruning the Pearson-correlation networks, keeping edge numbers fairly homogeneous across similarity measures. Normalize the logarithm of each filtered Euclidean adjacency matrix by its maximum logarithmic value, then sum it with the absolute-valued Pearson adjacency matrix per condition before SNF fusion (KNN kernels, iterative update until relative difference <10^-5).

## Related tools

- **Python** (Implements the Euclidean-distance network computation and the 99th-percentile edge filtering inside creation_matrices.py / Pipeline.ipynb) — https://github.com/DEIB-GECO/GeneNetFusion
- **Jupyter Notebook** (Pipeline.ipynb executes the complete pipeline step that thresholds the Euclidean networks at the 99th percentile) — https://github.com/DEIB-GECO/GeneNetFusion/blob/master/Pipeline.ipynb
- **PyGMQL** (Retrieves and pre-processes the TCGA gene expression data (zFPKM > -3.0 filter) that the Euclidean networks are built from) — https://pygmql.readthedocs.io/en/latest/
- **GMQL** (Query language used to extract the TCGA RNA-Seq expression datasets per cancer type and condition) — http://gmql.eu

## Examples

```
pip install -r requirements.txt && python creation_matrices.py
```

## Evaluation signals

- For each condition-specific Euclidean network, recompute the 99th percentile of the edge-weight distribution and verify that every retained edge weight is strictly higher than it (≈1% of gene pairs kept)
- Edge counts of the filtered Euclidean networks and the permutation/pruned Pearson networks are fairly homogeneous across the similarity measures within a condition
- The pipeline runs end-to-end and writes fused-network adjacency matrix files (e.g. fused_network_KIRC.csv) that exist and are formatted as weighted adjacency matrices
- Separate filtered networks are produced for both the normal and the cancer condition of each cancer type (KIRC, LIHC, PRAD)
- The log-normalized Euclidean adjacency (divided by its maximum logarithmic value) sums with the absolute-valued Pearson adjacency to yield one normal and one cancer sample network per cancer type

## Limitations

- The threshold is relative to each network's own edge-weight distribution, so the absolute weight cutoff differs across cancer types and conditions
- Only ~1% of edges survive, and a later disparity filter on the fused networks keeps on average about 4.9% of the edges in the Merged networks, so the backbone is highly sparse
- Pre-processing restricts genes to protein-coding and long non-coding RNAs with mean zFPKM > -3.0, so the network topology depends on these selection choices

## Evidence

- [methods] 99th percentile thresholding of Euclidean distance edge weights: "we keep only the edges with a weight higher than the 99th percentile of the distribution of the network edge weights"
- [results] Condition-specific Euclidean network filtering: "edges in the Euclidean distance networks for the normal and cancer condition are those after their 99th percentile thresholding"
- [results] Rationale: homogeneous edge counts across similarity measures: "filtered using the permutation method and a p-value percentile threshold such that their number is fairly homogeneous across the similarity measures"
- [readme] Repository implementation of the pipeline steps: "This repository contains all data, scripts and results related to the LIHC tumor analysis."
