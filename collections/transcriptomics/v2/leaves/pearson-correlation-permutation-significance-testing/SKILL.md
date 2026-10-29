---
name: pearson-correlation-permutation-significance-testing
description: Use when building gene co-expression networks from RNA-Seq expression
  matrices where Pearson-correlation edges must be filtered for statistical significance
  before fusing with other similarity-based networks (e.g., Euclidean-distance networks).
when_to_use_negative:
- The input is a Euclidean-distance co-expression network — that network is filtered
  by 99th-percentile edge-weight thresholding, not by the permutation test
- Edges have already been filtered and a fused network exists — the next step is the
  disparity filter backbone extraction, not this significance test
- Only a single similarity measure is being used with no need for edge-count parity
  across networks, where a simpler correlation cutoff may suffice
edam_operation: http://edamontology.org/operation_3658
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: Python
  role: Implements the permutation test, p-value percentile pruning, and adjacency-matrix
    construction in the pipeline scripts (creation_matrices.py, fusion_networks.py)
  repo: https://github.com/DEIB-GECO/GeneNetFusion
- name: Jupyter Notebook
  role: Pipeline.ipynb contains the complete pipeline step in which Pearson networks
    are computed and significance-filtered
  repo: https://github.com/DEIB-GECO/GeneNetFusion/blob/master/Pipeline.ipynb
- name: PyGMQL
  role: Used (with GMQL) to retrieve the TCGA gene expression data that feed the correlation
    networks
  repo: https://pygmql.readthedocs.io/en/latest/
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41540-021-00175-9
    title: Computational analysis of fused co-expression networks for the identification
      of candidate cancer gene biomarkers
  source_content_hash: sha256:7a9264b38cc2e2b6e6e8a7a6557be5f923469707d77f79ef549ff5dfcd79b0b8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/DEIB-GECO/GeneNetFusion
  tools:
  - GMQL
  - Jupyter Notebook
  - PyGMQL
  - Python
  edam_operation: http://edamontology.org/operation_3658
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# pearson-correlation-permutation-significance-testing

## Summary

A statistical edge-filtering procedure for Pearson-correlation gene co-expression networks: a permutation test on randomly shuffled TCGA expression data detects a significant correlation threshold, and a p-value percentile cut further prunes edges so edge counts are fairly homogeneous across similarity measures (e.g., matched to the 99th-percentile-filtered Euclidean-distance network).

## When to use

Use when building gene co-expression networks from RNA-Seq expression matrices where Pearson-correlation edges must be filtered for statistical significance before fusing with other similarity-based networks (e.g., Euclidean-distance networks).

## When NOT to use

- The input is a Euclidean-distance co-expression network — that network is filtered by 99th-percentile edge-weight thresholding, not by the permutation test
- Edges have already been filtered and a fused network exists — the next step is the disparity filter backbone extraction, not this significance test
- Only a single similarity measure is being used with no need for edge-count parity across networks, where a simpler correlation cutoff may suffice

## Inputs

- Pre-processed TCGA RNA-Seq gene expression matrix (protein-coding and lncRNA genes, mean zFPKM > -3.0) for a cancer type, separately for normal and cancer conditions
- Condition-specific Pearson-correlation adjacency matrices
- Edge count of the corresponding 99th-percentile-thresholded Euclidean-distance network (target for edge-count parity)

## Outputs

- Filtered Pearson-correlation adjacency matrices (significant edges only) for normal and cancer conditions
- Permutation-derived significance threshold for Pearson correlation values
- Filtered matrices ready to be combined (absolute values summed with normalized log-Euclidean adjacency) into per-condition sample networks

## How to apply

Starting from the pre-processed TCGA RNA-Seq expression data (protein-coding and long non-coding genes with mean zFPKM higher than -3.0), compute the Pearson-correlation co-expression network for the normal and cancer conditions of each cancer type (KIRC, LIHC, PRAD). Run a permutation test with 10 random shuffles of the expression data to detect a significant threshold for the Pearson correlation values. Then prune the remaining edges using a percentile threshold on the sorted Pearson p-values, choosing the percentile so the resulting edge count matches (is fairly homogeneous with) the corresponding Euclidean-distance network, which is thresholded at its 99th percentile of edge weights. The rationale is that percentile-based filtering alone is not meaningful for correlation values, so the permutation test provides an empirical significance threshold while the p-value percentile enforces parity across similarity measures before the networks are summed into per-condition sample networks and fused with SNF. Implementation lives in the repository's creation_matrices.py / fusion_networks.py steps of Pipeline.ipynb.

## Related tools

- **Python** (Implements the permutation test, p-value percentile pruning, and adjacency-matrix construction in the pipeline scripts (creation_matrices.py, fusion_networks.py)) — https://github.com/DEIB-GECO/GeneNetFusion
- **Jupyter Notebook** (Pipeline.ipynb contains the complete pipeline step in which Pearson networks are computed and significance-filtered) — https://github.com/DEIB-GECO/GeneNetFusion/blob/master/Pipeline.ipynb
- **PyGMQL** (Used (with GMQL) to retrieve the TCGA gene expression data that feed the correlation networks) — https://pygmql.readthedocs.io/en/latest/

## Examples

```
pip install -r requirements.txt && jupyter notebook Pipeline.ipynb
```

## Evaluation signals

- The number of retained Pearson edges is fairly homogeneous with (matches) the edge count of the corresponding 99th-percentile-filtered Euclidean-distance network
- All retained Pearson edges have correlation values above the threshold detected by the permutation test with 10 random shuffles
- Downstream steps succeed: the normal and cancer filtered matrices can be summed into per-condition sample networks and the SNF fusion converges (relative difference between consecutive W iterations < 10^-5)
- The fusion output (e.g., fused_network_KIRC.csv) exists and is formatted as a weighted adjacency matrix
- The pipeline runs end-to-end via pip install -r requirements.txt followed by Pipeline.ipynb

## Limitations

- The permutation test uses only 10 random shuffles, which limits the granularity of the empirical significance threshold
- The p-value percentile cut is set reactively to match the Euclidean network's edge count, so significance is partially calibrated to another similarity measure rather than to an absolute criterion
- No software implementation or version is specified in the article for parts of the pipeline, so exact reproducibility of the permutation procedure depends on the repository scripts

## Evidence

- [methods] we perform a permutation test that allows detecting a significant threshold for the Pearson’s correlation values31: "we perform a permutation test that allows detecting a significant threshold for the Pearson’s correlation values31"
- [results] filtered using the permutation method and a p-value percentile threshold such that their number is fairly homogeneous across the similarity measures: "filtered using the permutation method and a p-value percentile threshold such that their number is fairly homogeneous across the similarity measures"
- [results] edges in the Euclidean distance networks for the normal and cancer condition are those after their 99th percentile thresholding: "edges in the Euclidean distance networks for the normal and cancer condition are those after their 99th percentile thresholding"
- [readme] The jupyter notebook [Pipeline.ipynb](https://github.com/DEIB-GECO/GeneNetFusion/blob/master/Pipeline.ipynb) contains the complete pipeline for the construction of the fused co-expression networks: "contains the complete pipeline for the construction of the fused co-expression networks"
