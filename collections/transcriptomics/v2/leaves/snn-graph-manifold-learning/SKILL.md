---
name: snn-graph-manifold-learning
description: Use when a CellChat object with an inferred communication probability
  network is available and you need to identify coordinated outgoing/incoming signaling
  patterns among cell groups or classify signaling pathways into functional groups.
when_to_use_negative:
- The input is raw scRNA-seq expression data without cell clustering or an inferred
  communication probability network — CellChat requires cell clustering and computeCommunProb
  results first.
- The goal is differential abundance or cell-state annotation rather than pattern
  recognition over an already-inferred intercellular communication network.
- Only a single ligand-receptor interaction pair is of interest — the SNN/NMF manifold
  machinery is designed for network- and pathway-level patterns, not individual interactions.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: CellChat
  role: R package providing identifyCommunicationPatterns (NMF pattern discovery),
    computeEigengap, computeNetSimilarity, netClustering, and netEmbedding (UMAP)
    for SNN-graph manifold learning over communication networks
  repo: https://github.com/sqjin/CellChat
- name: CellChat
  role: Current maintained version (v1/v2) installable via devtools::install_github("jinworks/CellChat")
  repo: https://github.com/jinworks/CellChat
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-021-21246-9
    title: Inference and analysis of cell-cell communication using CellChat
  source_content_hash: sha256:2fa8e5542b38b90ff592475bf303688935d795dcdb1398a63effc626be736774
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sqjin/CellChat
  tools:
  - CellChat
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# snn-graph-manifold-learning

## Summary

CellChat's manifold-learning approach that builds a shared-nearest-neighbor (SNN) similarity network over cell groups or signaling pathways, smooths the similarity matrix, embeds it with UMAP, and performs NMF to recover outgoing/incoming communication patterns and functional pathway groups.

## When to use

Use when a CellChat object with an inferred communication probability network is available and you need to identify coordinated outgoing/incoming signaling patterns among cell groups or classify signaling pathways into functional groups.

## When NOT to use

- The input is raw scRNA-seq expression data without cell clustering or an inferred communication probability network — CellChat requires cell clustering and computeCommunProb results first.
- The goal is differential abundance or cell-state annotation rather than pattern recognition over an already-inferred intercellular communication network.
- Only a single ligand-receptor interaction pair is of interest — the SNN/NMF manifold machinery is designed for network- and pathway-level patterns, not individual interactions.

## Inputs

- CellChat object with inferred communication probability network (output of createCellChat/computeCommunProb)
- Cell-group by signaling-pathway communication probability matrices for outgoing and incoming signaling
- Signaling pathway similarity matrices (functional/structural)

## Outputs

- NMF W/H pattern matrices for outgoing and incoming communication patterns
- k-means assignments of signaling pathways to functional groups
- UMAP coordinates of the signaling-pathway manifold
- pattern_analysis_results.rds file

## How to apply

From the CellChat object, extract cell-group by signaling-pathway probability summaries for outgoing and incoming signaling separately and run NMF with identifyCommunicationPatterns (pattern = 'outgoing' and 'incoming'), selecting the number of patterns via computeEigengap on the Laplacian of the consensus matrix after pruning elements < 0.3 for noise robustness; zero W/H elements < 0.5 for pattern-contribution (alluvial) summaries. For pathway-level grouping, compute functional similarity as network Jaccard overlap (computeNetSimilarity; computeNetSimilarityPairwise for cross-condition contrasts) alongside structural (topological dissimilarity) similarity, build the shared-nearest-neighbor similarity network, smooth it, and assign pathways to functional groups with netClustering via k-means. Finally, embed the smoothed similarity matrix with netEmbedding (UMAP, with configurable n_neighbors and min_dist) to obtain the signaling-pathway manifold, and write the W/H matrices, k-means group assignments, and UMAP coordinates to pattern_analysis_results.rds.

## Related tools

- **CellChat** (R package providing identifyCommunicationPatterns (NMF pattern discovery), computeEigengap, computeNetSimilarity, netClustering, and netEmbedding (UMAP) for SNN-graph manifold learning over communication networks) — https://github.com/sqjin/CellChat
- **CellChat** (Current maintained version (v1/v2) installable via devtools::install_github("jinworks/CellChat")) — https://github.com/jinworks/CellChat

## Examples

```
cellchat <- identifyCommunicationPatterns(cellchat, pattern = "outgoing")
```

## Evaluation signals

- Recovered outgoing and incoming pattern counts match those reported for the reference dataset (selected via computeEigengap on the pruned consensus matrix Laplacian)
- k-means pathway grouping via functional similarity reproduces the paper's four-group classification of signaling pathways in the wound skin dataset
- Consensus matrix elements < 0.3 are zeroed and W/H elements < 0.5 are zeroed before summaries/alluvial plots
- UMAP embedding of the smoothed similarity matrix places functionally similar signaling pathways (high Jaccard network overlap) near one another on the manifold
- pattern_analysis_results.rds contains W/H matrices, k-means group assignments, and UMAP coordinates with expected dimensions (cell groups x patterns and pathways x patterns)

## Limitations

- NMF and manifold-learning outputs depend on the quality of the upstream inferred communication probability, which is not exactly a probability and only represents interaction strength
- Pattern number selection relies on eigengap heuristics after consensus-matrix pruning, which can be sensitive to noise if the pruning threshold (0.3) is not appropriate for the dataset
- Different ligand-receptor databases across methods could confound cross-method comparison of recovered patterns

## Evidence

- [discussion] NMF pattern discovery via identifyCommunicationPatterns: "CellChat uses an unsupervised learning method non-negative matrix factorization that has been successfully applied in pattern recognition"
- [discussion] Consensus matrix pruning threshold: "We then pruned the consensus matrix by setting the elements to be zero if they are less than 0.3 to ensure better robustness to noise."
- [discussion] UMAP embedding of smoothed similarity matrix: "Finally, we performed uniform manifold approximation and projection (UMAP)92 on the smoothed similarity matrix."
- [discussion] W/H element thresholding for alluvial plots: "set the elements in W and H to be zero if they are less than 0.5"
- [readme] Tool availability: "CellChat v1/v2 R package can be easily installed from Github using devtools: devtools::install_github("jinworks/CellChat")"
