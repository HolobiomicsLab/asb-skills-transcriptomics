---
name: nmf-signaling-pattern-discovery
description: Use when a CellChat communication probability network (cell groups x
  signaling pathways) is available and the goal is to decompose sender/receiver signaling
  into outgoing and incoming patterns and group signaling pathways by functional or
  structural similarity.
when_to_use_negative:
- No CellChat object with an inferred communication probability network exists yet
  — run createCellChat, identifyOverExpressedGenes/Interactions, and computeCommunProb
  first.
- The analysis goal is inferring ligand-receptor communication probabilities or sender/receiver
  centrality roles, not decomposing signaling into patterns or grouping pathways.
- Input is raw scRNA-seq count data or a clustering-only object without per-pathway
  communication probabilities.
edam_operation: http://edamontology.org/operation_3637
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: CellChat
  role: Provides identifyCommunicationPatterns (NMF decomposition into outgoing/incoming
    patterns), computeEigengap, computeNetSimilarity/computeNetSimilarityPairwise,
    netClustering, and netEmbedding for pattern discovery and pathway grouping
  repo: https://github.com/sqjin/CellChat
- name: CellChatDB
  role: Manually curated ligand-receptor database whose pathways are the input signaling
    categories being decomposed and grouped
  repo: https://github.com/sqjin/CellChat
provenance:
  source_task_ids:
  - task_001
  - task_008
  source_papers:
  - doi: 10.1038/s41467-021-21246-9
    title: Inference and analysis of cell-cell communication using CellChat
  source_content_hash: sha256:ce6b8eb1e95abbece0681deac6d55329cd816577fbc4bcd8e95cd368fb6a0c6c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sqjin/CellChat
  tools:
  - CellChat
  - CellChatDB
  edam_operation: http://edamontology.org/operation_3637
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# NMF signaling-pattern discovery

## Summary

Discover coordinated outgoing and incoming cell-cell communication patterns by applying non-negative matrix factorization to the cell-group by signaling-pathway probability matrices inferred by CellChat, then embed and cluster pathways via manifold learning to reveal how signals coordinate across cell groups.

## When to use

Use when a CellChat communication probability network (cell groups x signaling pathways) is available and the goal is to decompose sender/receiver signaling into outgoing and incoming patterns and group signaling pathways by functional or structural similarity.

## When NOT to use

- No CellChat object with an inferred communication probability network exists yet — run createCellChat, identifyOverExpressedGenes/Interactions, and computeCommunProb first.
- The analysis goal is inferring ligand-receptor communication probabilities or sender/receiver centrality roles, not decomposing signaling into patterns or grouping pathways.
- Input is raw scRNA-seq count data or a clustering-only object without per-pathway communication probabilities.

## Inputs

- CellChat object with inferred communication probability network (upstream createCellChat/computeCommunProb/computeCommunProbPathway output, e.g. stored as .rds)
- Cell-group by signaling-pathway outgoing and incoming probability matrices derived from the inferred network

## Outputs

- W and H NMF pattern matrices for outgoing and incoming communication (thresholded at 0.5 for alluvial plots)
- Outgoing and incoming communication pattern counts/plots (e.g., 5 outgoing and 5 incoming patterns in the day-12 wound dataset)
- k-means signaling-pathway group assignments from functional similarity grouping (e.g., four pathway groups)
- UMAP coordinates of the signaling-pathway manifold (netEmbedding)
- pattern_analysis_results.rds containing W/H matrices, pathway groups, and UMAP coordinates

## How to apply

From the CellChat object with the inferred communication probability network (output of createCellChat/computeCommunProb/computeCommunProbPathway), extract the cell-group by signaling-pathway probability summaries for outgoing and incoming signaling separately and run identifyCommunicationPatterns with pattern='outgoing' and 'incoming'. Select the number of patterns by computing the eigengap on the Laplacian of the consensus matrix pruned by setting elements < 0.3 to zero for robustness to noise, and zero W/H elements < 0.5 for pattern-contribution (alluvial) summaries. In the mouse skin wound application this returned five outgoing and five incoming communication patterns. Separately, compute functional similarity between pathways as network Jaccard overlap with computeNetSimilarity (computeNetSimilarityPairwise for cross-dataset comparison) alongside structural (topological dissimilarity) similarity, build and smooth a shared-nearest-neighbor similarity network, cluster pathways with netClustering (k-means), and embed the smoothed similarity matrix with netEmbedding (UMAP, configurable n_neighbors and min_dist); functional similarity grouping identified four groups of pathways in the wound skin dataset. Write the W/H matrices, pathway group assignments, and UMAP coordinates to pattern_analysis_results.rds.

## Related tools

- **CellChat** (Provides identifyCommunicationPatterns (NMF decomposition into outgoing/incoming patterns), computeEigengap, computeNetSimilarity/computeNetSimilarityPairwise, netClustering, and netEmbedding for pattern discovery and pathway grouping) — https://github.com/sqjin/CellChat
- **CellChatDB** (Manually curated ligand-receptor database whose pathways are the input signaling categories being decomposed and grouped) — https://github.com/sqjin/CellChat

## Examples

```
cellchat <- identifyCommunicationPatterns(cellchat, pattern = c("outgoing","incoming")); sim <- computeNetSimilarity(cellchat); cellchat <- netEmbedding(cellchat, sim, umap.config = umap::umap.defaults)
```

## Evaluation signals

- On the day-12 mouse skin wound dataset (GSE113854), NMF should return 5 outgoing and 5 incoming communication patterns, matching the published figures.
- Functional similarity grouping should reproduce the four groups of signaling pathways reported for the wound skin dataset.
- Pruned consensus matrix contains no elements below 0.3; alluvial plot W/H matrices contain no elements below 0.5.
- Pattern matrices W (cell groups x patterns) and H (patterns x pathways) are non-negative and dimensions match the input outgoing/incoming probability matrices.
- Pathway group assignments and UMAP coordinates are consistent with the smoothed similarity matrix and are validated by expert comparison against the published figures.

## Limitations

- Choosing the number of patterns depends on the eigengap heuristic on the pruned consensus matrix and can be sensitive to noise; the 0.3 and 0.5 thresholds are dataset-specific choices.
- NMF pattern recognition is unsupervised, so pattern interpretation requires expert comparison against published/reference outputs; there is no ground truth for the predicted communication networks.
- For cross-condition pathway classification, structural/functional similarity measures must be computed pairwise (computeNetSimilarityPairwise) rather than within a single dataset.

## Evidence

- [discussion] CellChat uses an unsupervised learning method non-negative matrix factorization that has been successfully applied in pattern recognition: "CellChat uses an unsupervised learning method non-negative matrix factorization that has been successfully applied in pattern recognition"
- [discussion] set the elements in W and H to be zero if they are less than 0.5: "set the elements in W and H to be zero if they are less than 0.5"
- [discussion] We then pruned the consensus matrix by setting the elements to be zero if they are less than 0.3 to ensure better robustness to noise.: "We then pruned the consensus matrix by setting the elements to be zero if they are less than 0.3 to ensure better robustness to noise."
- [discussion] Finally, we performed uniform manifold approximation and projection (UMAP)92 on the smoothed similarity matrix.: "Finally, we performed uniform manifold approximation and projection (UMAP)92 on the smoothed similarity matrix."
- [other] CellChat groups signaling pathways by defining functional or structural similarity measures, and applying functional similarity grouping identified four groups of pathways in the wound skin dataset.: [evidence span withheld — anchor status: not_found]
