---
name: adjacency-matrix-processing
description: Use when you have edge-weighted adjacency matrices of gene co-expression
  networks (e.g., built from TCGA expression with Euclidean distance or Pearson correlation)
  and need to prune them to a backbone before network fusion or community extraction.
when_to_use_negative:
- Input is still a raw TCGA gene expression matrix that has not been converted into
  co-expression adjacency matrices yet — use the preprocessing/creation_matrices stage
  first
- The analysis goal is differential expression or biomarker classification baselines
  rather than network structure — use DESeq2 (R/Bioconductor) instead
- The network is unweighted or directed — the disparity filter described here is defined
  for undirected weighted networks
edam_operation: http://edamontology.org/operation_0291
edam_topics:
- http://edamontology.org/topic_0085
- http://edamontology.org/topic_0203
tools:
- name: Python
  role: Implements the disparity-filter p-value computation, 99th-percentile thresholding,
    and export of pruned backbone edge lists from the edge-weighted adjacency matrices
  repo: https://github.com/DEIB-GECO/GeneNetFusion
- name: Jupyter Notebook
  role: Pipeline.ipynb contains the complete pipeline for constructing and pruning
    the fused co-expression networks
  repo: https://github.com/DEIB-GECO/GeneNetFusion/blob/master/Pipeline.ipynb
- name: Gephi
  role: Downstream consumer of the pruned fused network, used to extract gene communities
    with the modularity method
  repo: https://gephi.org
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41540-021-00175-9
    title: Computational analysis of fused co-expression networks for the identification
      of candidate cancer gene biomarkers
  source_content_hash: sha256:7978d7952ee55a086f2fa0019317f54421b736a55fcb28e325cb3e5fe9134015
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/DEIB-GECO/GeneNetFusion
  tools:
  - Gephi
  - Jupyter Notebook
  - Python
  edam_operation: http://edamontology.org/operation_0291
  edam_topics:
  - http://edamontology.org/topic_0085
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# adjacency-matrix-processing

## Summary

Prunes edge-weighted gene co-expression network adjacency matrices (Euclidean distance and Pearson correlation based, normal and cancer conditions) with the disparity filter to extract the multiscale backbone structure before normalization and fusion. In the GeneNetFusion pipeline this reduces the Fused networks to on average about 4.9% of the edges retained in the Merged networks.

## When to use

Use when you have edge-weighted adjacency matrices of gene co-expression networks (e.g., built from TCGA expression with Euclidean distance or Pearson correlation) and need to prune them to a backbone before network fusion or community extraction.

## When NOT to use

- Input is still a raw TCGA gene expression matrix that has not been converted into co-expression adjacency matrices yet — use the preprocessing/creation_matrices stage first
- The analysis goal is differential expression or biomarker classification baselines rather than network structure — use DESeq2 (R/Bioconductor) instead
- The network is unweighted or directed — the disparity filter described here is defined for undirected weighted networks

## Inputs

- Edge-weighted adjacency matrices of gene co-expression networks (normal and cancer conditions, Euclidean distance and Pearson correlation based)
- TCGA gene expression data retrieved through GMQL and PyGMQL (e.g., KIRC, LIHC, PRAD)
- GeneNetFusion repository data files (Datasets and Datasets_created folders)

## Outputs

- Pruned backbone edge lists exported to pruned_network_edge_lists.csv
- Retained-edge fraction per pruned network
- Pruned/fused networks formatted for Gephi (Graphs folder, LIHC fused network)

## How to apply

Load the edge-weighted adjacency matrices of the built gene co-expression networks (normal and cancer conditions, Euclidean distance and Pearson correlation based) from the deposited GeneNetFusion data. For each network node, compute disparity-filter p-values by comparing its edge-weight distribution against a uniform null model, then set the threshold at the 99th percentile of the overall p-value distribution of each network and retain only edges passing it — the disparity filter extracts the backbone structure of an undirected weighted network without destroying its major features. Before fusion, apply measure-specific pruning: keep only Euclidean distance network edges with weight higher than the 99th percentile of the edge-weight distribution, and filter Pearson correlation networks using the permutation method with a p-value percentile threshold so edge counts are fairly homogeneous across similarity measures. Compute and report the retained-edge fraction per pruned network (about 4.9% of Merged edges survive on average in the Fused networks), then export the pruned backbone edge lists to pruned_network_edge_lists.csv using Python/Jupyter Notebook scripts such as creation_matrices.py and fusion_networks.py.

## Related tools

- **Python** (Implements the disparity-filter p-value computation, 99th-percentile thresholding, and export of pruned backbone edge lists from the edge-weighted adjacency matrices) — https://github.com/DEIB-GECO/GeneNetFusion
- **Jupyter Notebook** (Pipeline.ipynb contains the complete pipeline for constructing and pruning the fused co-expression networks) — https://github.com/DEIB-GECO/GeneNetFusion/blob/master/Pipeline.ipynb
- **Gephi** (Downstream consumer of the pruned fused network, used to extract gene communities with the modularity method) — https://gephi.org

## Examples

```
python fusion_networks.py
```

## Evaluation signals

- In the final (Fused) networks, the disparity filter retains on average about 4.9% of the edges present in the Merged networks
- Euclidean distance networks are pruned by keeping only edges with weight higher than the 99th percentile of the network edge-weight distribution
- Filtered edge numbers are fairly homogeneous across the two similarity measures (Euclidean distance and Pearson correlation)
- The pruned backbone preserves the network's major features (no collapse of structure) before SNF-based fusion of normal and cancer networks
- Exported file pruned_network_edge_lists.csv contains backbone edge lists for each condition/similarity-measure network

## Limitations

- No software implementation or tool version is specified for the SNF algorithm used to obtain the final cancer-type-specific networks from the pruned condition-specific ones
- The p-value threshold is set at the 99th percentile of each network's overall p-value distribution — a data-dependent heuristic rather than a fixed significance level
- The public repository ships data, scripts, and results only for the LIHC tumor analysis; KIRC and PRAD results are reported in the paper but not fully deposited

## Evidence

- [methods] disparity filter extracts backbone: "an additional pruning step, called disparity filter38; it extracts the backbone structure of an undirected weighted network"
- [results] disparity filter retains ~4.9% of edges: "The relevant edges selected with the disparity filter in the final (Fused) networks are on average about 4.9% of the edges in the Merged ones"
- [methods] 99th percentile edge-weight cutoff: "we keep only the edges with a weight higher than the 99th percentile of the distribution of the network edge weights"
- [results] homogeneous edge counts across similarity measures: "filtered using the permutation method and a p-value percentile threshold such that their number is fairly homogeneous across the similarity measures"
- [readme] Pipeline notebook contains full network construction pipeline: "The jupyter notebook [Pipeline.ipynb](https://github.com/DEIB-GECO/GeneNetFusion/blob/master/Pipeline.ipynb) contains the complete pipeline for the construction of the fused co-expression networks"
