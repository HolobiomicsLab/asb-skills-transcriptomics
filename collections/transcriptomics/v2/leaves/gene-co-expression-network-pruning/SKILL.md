---
name: gene-co-expression-network-pruning
description: Use when edge-weighted gene co-expression adjacency matrices (normal/cancer
  conditions per cancer type) are too dense for downstream fusion or community detection
  and must be pruned to their backbone.
when_to_use_negative:
- Input is raw TCGA gene expression data that has not yet been converted into edge-weighted
  co-expression adjacency matrices (use the network-building steps first)
- The goal is differential gene expression analysis with DESeq2 — this skill operates
  on network topology, not per-gene expression statistics
- Networks are unweighted or directed, since the disparity filter is designed for
  undirected weighted networks
edam_operation: http://edamontology.org/operation_3562
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3366
tools:
- name: Python
  role: Implements the 99th-percentile thresholding, permutation test, and disparity-filter
    pruning steps on the co-expression adjacency matrices
  repo: https://github.com/DEIB-GECO/GeneNetFusion
- name: Jupyter Notebook
  role: Pipeline.ipynb executes the complete pipeline including network construction
    and pruning before fusion
  repo: https://github.com/DEIB-GECO/GeneNetFusion/blob/master/Pipeline.ipynb
- name: PyGMQL
  role: Retrieves the TCGA gene expression data from which the networks are built
    prior to pruning
  repo: https://pygmql.readthedocs.io/en/latest/
provenance:
  source_task_ids:
  - task_007
  - task_001
  source_papers:
  - doi: 10.1038/s41540-021-00175-9
    title: Computational analysis of fused co-expression networks for the identification
      of candidate cancer gene biomarkers
  source_content_hash: sha256:594f1aee7a838c2a66f64968930e352e59b949444585d270b301fac1ff197106
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/DEIB-GECO/GeneNetFusion
  tools:
  - GMQL
  - Jupyter Notebook
  - PyGMQL
  - Python
  edam_operation: http://edamontology.org/operation_3562
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3366
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# gene-co-expression-network-pruning

## Summary

Prunes dense, edge-weighted gene co-expression networks (Euclidean-distance and Pearson-correlation based, built from TCGA RNA-Seq) to extract their multiscale backbone before normalization and SNF fusion, using 99th-percentile weight thresholding, permutation-tested Pearson filtering, and the disparity filter.

## When to use

Use when edge-weighted gene co-expression adjacency matrices (normal/cancer conditions per cancer type) are too dense for downstream fusion or community detection and must be pruned to their backbone.

## When NOT to use

- Input is raw TCGA gene expression data that has not yet been converted into edge-weighted co-expression adjacency matrices (use the network-building steps first)
- The goal is differential gene expression analysis with DESeq2 — this skill operates on network topology, not per-gene expression statistics
- Networks are unweighted or directed, since the disparity filter is designed for undirected weighted networks

## Inputs

- Edge-weighted adjacency matrices of gene co-expression networks (normal and cancer conditions per cancer type: KIRC, LIHC, PRAD)
- Euclidean-distance co-expression networks after 99th-percentile thresholding
- Pearson-correlation co-expression networks with permutation-test-derived significance p-values
- Merged/SNF-fused network adjacency matrices (e.g., fused_network_KIRC.csv)

## Outputs

- Pruned backbone edge lists exported to pruned_network_edge_lists.csv
- Per-network retained-edge fractions after disparity filtering
- Backbone-structured fused networks ready for Gephi modularity-based community extraction

## How to apply

Load the edge-weighted adjacency matrices from the GeneNetFusion repository (normal and cancer conditions, Euclidean-distance and Pearson-correlation based). Threshold Euclidean-distance networks by keeping only edges with a weight higher than the 99th percentile of the edge-weight distribution. Filter Pearson-correlation networks with a permutation test (10 random shuffles of the expression data) to find significant correlation thresholds, then prune with a percentile of the sorted Pearson p-values so the edge count matches the corresponding Euclidean network. After merging and SNF-fusing normal and cancer networks, apply the disparity filter: for each node, compute disparity-filter p-values by comparing its edge-weight distribution against a uniform null model, set the threshold at the 99th percentile of the overall p-value distribution, and retain only edges passing it — this extracts the backbone without destroying the network's major features, retaining on average about 4.9% of the Merged edges in the Fused networks. Export the pruned backbone edge lists (e.g., pruned_network_edge_lists.csv) and compute retained-edge fractions per network.

## Related tools

- **Python** (Implements the 99th-percentile thresholding, permutation test, and disparity-filter pruning steps on the co-expression adjacency matrices) — https://github.com/DEIB-GECO/GeneNetFusion
- **Jupyter Notebook** (Pipeline.ipynb executes the complete pipeline including network construction and pruning before fusion) — https://github.com/DEIB-GECO/GeneNetFusion/blob/master/Pipeline.ipynb
- **PyGMQL** (Retrieves the TCGA gene expression data from which the networks are built prior to pruning) — https://pygmql.readthedocs.io/en/latest/

## Examples

```
jupyter notebook Pipeline.ipynb  # executes preprocessing, matrix creation, fusion, and disparity-filter pruning for KIRC/LIHC/PRAD
```

## Evaluation signals

- In the final Fused networks, the disparity filter retains on average about 4.9% of the edges present in the Merged networks
- Euclidean-distance networks contain only edges with weight above the 99th percentile of the edge-weight distribution
- Pearson-correlation networks have edge counts fairly homogeneous with the corresponding Euclidean-distance networks after percentile pruning
- Fusion script runs successfully and produces fused-network adjacency matrix files (e.g., fused_network_KIRC.csv) formatted as weighted adjacency matrices
- Backbone extraction preserves the network's major features rather than disconnecting nodes

## Limitations

- The permutation test uses only 10 random shuffles of the expression data to determine Pearson correlation significance thresholds, which is a coarse significance estimate
- No software implementation or tool version is specified for the SNF algorithm used in the downstream fusion step
- The disparity filter requires weighted undirected networks; it cannot be applied directly to binary or directed graphs

## Evidence

- [methods] an additional pruning step, called disparity filter38; it extracts the backbone structure of an undirected weighted network: "an additional pruning step, called disparity filter38; it extracts the backbone structure of an undirected weighted network"
- [methods] we keep only the edges with a weight higher than the 99th percentile of the distribution of the network edge weights: "we keep only the edges with a weight higher than the 99th percentile of the distribution of the network edge weights"
- [results] The relevant edges selected with the disparity filter in the final (Fused) networks are on average about 4.9% of the edges in the Merged ones: "The relevant edges selected with the disparity filter in the final (Fused) networks are on average about 4.9% of the edges in the Merged ones"
- [methods] we perform a permutation test that allows detecting a significant threshold for the Pearson’s correlation values31: "we perform a permutation test that allows detecting a significant threshold for the Pearson’s correlation values31"
- [readme] The jupyter notebook [Pipeline.ipynb](https://github.com/DEIB-GECO/GeneNetFusion/blob/master/Pipeline.ipynb) contains the complete pipeline for the construction of the fused co-expression networks: "The jupyter notebook [Pipeline.ipynb](https://github.com/DEIB-GECO/GeneNetFusion/blob/master/Pipeline.ipynb) contains the complete pipeline for the construction of the fused co-expression networks"
