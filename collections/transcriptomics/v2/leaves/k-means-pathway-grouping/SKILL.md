---
name: k-means-pathway-grouping
description: Use when a CellChat object with an inferred communication probability
  network is available and signaling pathways must be classified into functionally
  coherent groups, e.g. to reproduce CellChat's four-group pathway classification
  on the wound skin dataset.
when_to_use_negative:
- The analysis goal is inferring communication probabilities between cell groups rather
  than classifying already-inferred pathways (use computeCommunProb instead)
- Only one or very few signaling pathways are present, so similarity-network grouping
  is under-determined
- Cell clustering of the scRNA-seq data has not been done — cell clustering is a pre-requisite
  for CellChat analysis
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: CellChat
  role: Provides computeNetSimilarity (functional network-Jaccard similarity), computeNetSimilarityPairwise
    (cross-condition), netClustering (k-means pathway grouping on the smoothed SNN
    similarity network), and netEmbedding (UMAP embedding of the similarity matrix)
  repo: https://github.com/sqjin/CellChat
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-021-21246-9
    title: Inference and analysis of cell-cell communication using CellChat
  source_content_hash: sha256:4a9cb884cf84e1e0442fcb7b3b6a02d9c08964ab344acd0c09db6e3407f134c7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sqjin/CellChat
  tools:
  - CellChat
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# k-means-pathway-grouping

## Summary

Groups inferred signaling pathways into functional classes by computing functional (network Jaccard overlap) and structural (topological dissimilarity) similarity between pathways, then assigning groups with k-means on the smoothed shared-nearest-neighbor similarity network, as implemented in CellChat's netClustering/computeNetSimilarity functions.

## When to use

Use when a CellChat object with an inferred communication probability network is available and signaling pathways must be classified into functionally coherent groups, e.g. to reproduce CellChat's four-group pathway classification on the wound skin dataset.

## When NOT to use

- The analysis goal is inferring communication probabilities between cell groups rather than classifying already-inferred pathways (use computeCommunProb instead)
- Only one or very few signaling pathways are present, so similarity-network grouping is under-determined
- Cell clustering of the scRNA-seq data has not been done — cell clustering is a pre-requisite for CellChat analysis

## Inputs

- CellChat object containing the inferred communication probability network (cell-group by signaling-pathway probability summaries)
- List of inferred signaling pathway names from the CellChat communication network

## Outputs

- Signaling-pathway functional group assignments from k-means clustering
- UMAP coordinates of the signaling-pathway manifold (from netEmbedding on the smoothed similarity matrix)
- pattern_analysis_results.rds containing W/H pattern matrices, pathway group assignments, and UMAP coordinates

## How to apply

From the CellChat object (output of createCellChat/computeCommunProb), compute functional similarity between signaling pathways as network Jaccard overlap with computeNetSimilarity (use computeNetSimilarityPairwise when comparing across conditions), alongside structural similarity based on topological dissimilarity. Build the shared-nearest-neighbor similarity network from these measures, then smooth the similarity matrix to reduce noise before clustering. Assign pathways to functional groups with netClustering, which runs k-means on the smoothed network; optionally embed the smoothed similarity matrix with netEmbedding (UMAP, with configurable n_neighbors and min_dist) to visualize the signaling-pathway manifold. Write the k-means pathway group assignments and UMAP coordinates to pattern_analysis_results.rds alongside the NMF W/H pattern matrices for downstream reporting.

## Related tools

- **CellChat** (Provides computeNetSimilarity (functional network-Jaccard similarity), computeNetSimilarityPairwise (cross-condition), netClustering (k-means pathway grouping on the smoothed SNN similarity network), and netEmbedding (UMAP embedding of the similarity matrix)) — https://github.com/sqjin/CellChat

## Examples

```
devtools::install_github("jinworks/CellChat"); cellchat <- netClustering(cellchat, k = 4)
```

## Evaluation signals

- Recovered k-means pathway groups reproduce the paper's reported four groups of pathways identified by functional similarity grouping in the wound skin dataset
- Similarity matrix values are non-negative and symmetric before smoothing; smoothing does not alter pathway count
- Every signaling pathway is assigned to exactly one functional group; no unassigned pathways remain
- UMAP coordinates from netEmbedding place pathways from the same k-means group in proximity on the manifold, consistent with the published figures
- For cross-condition analyses, computeNetSimilarityPairwise yields comparable similarity scores across the two CellChat objects

## Limitations

- CellChatDB and hence pathway set depends on the database version; updateCellChatDB should be used to stay current
- Overlap-based similarity comparison has a limited assumption: similar methods can generate similar results regardless of accuracy
- The inferred similarity structure depends on the quality of the upstream communication probability network, and communication probability only represents interaction strength, not a true probability

## Evidence

- [other] CellChat groups signaling pathways by defining functional or structural similarity measures: [evidence span withheld — anchor status: not_found]
- [abstract] Through manifold learning and quantitative contrasts, CellChat classifies signaling pathways: "Through manifold learning and quantitative contrasts, CellChat classifies signaling pathways and delineates conserved and context-specific pathways across different datasets."
- [discussion] we performed uniform manifold approximation and projection (UMAP) on the smoothed similarity matrix: "Finally, we performed uniform manifold approximation and projection (UMAP)92 on the smoothed similarity matrix."
- [methods] devtools::install_github("jinworks/CellChat"): "CellChat v1/v2 R package can be easily installed from Github using devtools: devtools::install_github("jinworks/CellChat")"
