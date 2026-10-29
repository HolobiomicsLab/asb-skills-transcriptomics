---
name: weighted-network-edge-percentile-thresholding
description: Use when you have edge-weighted undirected co-expression networks (e.g.
  Euclidean distance or Pearson correlation based) that are too dense for downstream
  fusion or community detection and need data-driven edge pruning.
when_to_use_negative:
- When the input is a fused (SNF) network needing backbone extraction via the disparity
  filter rather than a raw edge-weight percentile cut
- When networks are unweighted or binary, since percentile thresholding of edge weights
  is undefined
- When the goal is Pearson correlation significance filtering, where the article prescribes
  the permutation test with a p-value percentile threshold instead of raw weight percentile
  thresholding
edam_operation: http://edamontology.org/operation_3562
edam_topics:
- http://edamontology.org/topic_0602
tools:
- name: Python
  role: Implements the edge-weight percentile thresholding and disparity-filter pruning
    steps of the pipeline
  repo: https://github.com/DEIB-GECO/GeneNetFusion
- name: Jupyter Notebook
  role: Pipeline.ipynb contains the complete pipeline including network construction
    and pruning before Gephi community extraction
  repo: https://github.com/DEIB-GECO/GeneNetFusion/blob/master/Pipeline.ipynb
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41540-021-00175-9
    title: Computational analysis of fused co-expression networks for the identification
      of candidate cancer gene biomarkers
  source_content_hash: sha256:9721407c51928488f314b758ae413e1792bd8e2f839e28eff1dfbbf92b4a2a43
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/DEIB-GECO/GeneNetFusion
  tools:
  - Jupyter Notebook
  - Python
  edam_operation: http://edamontology.org/operation_3562
  edam_topics:
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# weighted-network-edge-percentile-thresholding

## Summary

Prunes edge-weighted gene co-expression networks by keeping only edges whose weight (or disparity-filter p-value) exceeds a percentile threshold of the edge-weight distribution, e.g. the 99th percentile, to extract the network's backbone before normalization and fusion.

## When to use

Use when you have edge-weighted undirected co-expression networks (e.g. Euclidean distance or Pearson correlation based) that are too dense for downstream fusion or community detection and need data-driven edge pruning.

## When NOT to use

- When the input is a fused (SNF) network needing backbone extraction via the disparity filter rather than a raw edge-weight percentile cut
- When networks are unweighted or binary, since percentile thresholding of edge weights is undefined
- When the goal is Pearson correlation significance filtering, where the article prescribes the permutation test with a p-value percentile threshold instead of raw weight percentile thresholding

## Inputs

- Edge-weighted adjacency matrices of gene co-expression networks (normal and cancer conditions, Euclidean distance and Pearson correlation based)
- TCGA gene expression data per cancer type (KIRC, LIHC, PRAD) from GMQL/PyGMQL retrieval
- Percentile threshold (99th percentile of network edge weights) or disparity-filter p-value distribution

## Outputs

- Pruned backbone network edge lists (e.g. pruned_network_edge_lists.csv)
- Retained-edge fraction per pruned network (e.g. ~4.9% of Merged edges in Fused networks)
- Threshold values (99th percentile weight cut, permutation-test significance threshold)

## How to apply

For Euclidean distance-based networks built from TCGA gene expression data, keep only the edges with a weight higher than the 99th percentile of the distribution of the network edge weights, applied separately to the normal and cancer condition networks. For Pearson correlation-based networks, replace the raw percentile cut with a permutation test that detects a significant threshold for the correlation values, using a p-value percentile threshold chosen so the retained edge count is fairly homogeneous across similarity measures. After merging and SNF-fusing the condition-specific networks, apply the disparity filter as an additional pruning step that extracts the backbone structure of the undirected weighted network by comparing each node's edge-weight distribution against a uniform null model. Rationale: this preserves each network's major features while drastically reducing edge count — in the final Fused networks the disparity filter retains on average about 4.9% of the edges in the Merged networks. Implement the steps in Python/Jupyter (Pipeline.ipynb in the GeneNetFusion repository) and export pruned backbone edge lists (e.g. to pruned_network_edge_lists.csv).

## Related tools

- **Python** (Implements the edge-weight percentile thresholding and disparity-filter pruning steps of the pipeline) — https://github.com/DEIB-GECO/GeneNetFusion
- **Jupyter Notebook** (Pipeline.ipynb contains the complete pipeline including network construction and pruning before Gephi community extraction) — https://github.com/DEIB-GECO/GeneNetFusion/blob/master/Pipeline.ipynb

## Examples

```
python -c "import pandas as pd; w = pd.read_csv('edge_weights.csv')['weight']; thr = w.quantile(0.99); w[w > thr].to_csv('pruned_network_edge_lists.csv', index=False)"
```

## Evaluation signals

- For Euclidean distance networks, every retained edge weight is strictly higher than the 99th percentile of that network's edge-weight distribution
- After disparity filtering of Fused networks, the retained edges are on average about 4.9% of the edges in the Merged networks
- Retained edge counts via the permutation-test p-value percentile threshold are fairly homogeneous across similarity measures
- The pruned backbone preserves the network's major features (no collapse of connectivity or loss of condition-specific structure)
- Pruned edge lists export successfully in a consistent schema for downstream normalization, fusion, and Gephi community extraction

## Limitations

- The 99th percentile weight threshold is prescribed specifically for Euclidean distance networks; Pearson correlation networks require the permutation-test threshold instead
- Disparity-filter results are reported only as an aggregate (~4.9% of Merged edges retained in Fused networks), not per network or per cancer type
- The repository ships data, scripts and results only for the LIHC tumor analysis, so reproduction for KIRC/PRAD requires rerunning the pipeline on those datasets

## Evidence

- [methods] 99th percentile weight threshold on Euclidean networks: "we keep only the edges with a weight higher than the 99th percentile of the distribution of the network edge weights"
- [methods] disparity filter as additional pruning step: "an additional pruning step, called disparity filter38; it extracts the backbone structure of an undirected weighted network"
- [results] retained edge fraction in fused networks: "The relevant edges selected with the disparity filter in the final (Fused) networks are on average about 4.9% of the edges in the Merged ones"
- [results] Pearson network filtering via permutation test: "filtered using the permutation method and a p-value percentile threshold such that their number is fairly homogeneous across the similarity measures"
