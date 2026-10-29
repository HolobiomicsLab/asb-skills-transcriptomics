---
name: manifold-learning-of-signaling-networks
description: Use when comparing inferred signaling networks across two conditions
  (e.g., E13.5 vs E14.5 embryonic skin, lesional vs nonlesional AD skin) and you need
  to cluster pathways by functional similarity and rank up/downregulated signaling
  by differential information flow.
when_to_use_negative:
- Only a single condition/dataset is available — joint manifold learning and rankNet
  contrasts require at least two merged CellChat objects
- The two datasets use different ligand-receptor databases or non-comparable cell-group
  labels, so the shared pathway/cell-population space for the joint embedding is undefined
- The goal is de novo interaction inference rather than cross-condition comparison
  of already inferred networks
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_0128
- http://edamontology.org/topic_3170
tools:
- name: CellChat
  role: R package that infers communication networks per condition, merges them with
    mergeCellChat, and performs joint manifold learning via netClustering and netEmbedding
  repo: https://github.com/sqjin/CellChat
- name: createCellChat
  role: Builds the per-dataset CellChat object from count matrices and metadata before
    network inference
  repo: https://github.com/jinworks/CellChat
- name: computeCommunProb
  role: Infers ligand-receptor communication probabilities (Hill function, permutation
    test M=100, p<0.05) that feed the compared networks
  repo: https://github.com/jinworks/CellChat
- name: mergeCellChat
  role: Merges the per-condition CellChat objects into a single comparison object
    for joint embedding and rankNet contrasts
  repo: https://github.com/jinworks/CellChat
- name: netEmbedding
  role: Performs the manifold learning/UMAP embedding of the smoothed signaling-network
    similarity matrix
  repo: https://github.com/jinworks/CellChat
- name: netAnalysis_computeCentrality
  role: Computes out-degree, in-degree, flow betweenness, and information centrality
    for senders/receivers/mediators/influencers per condition
  repo: https://github.com/jinworks/CellChat
- name: umap
  role: Underlying UMAP implementation used for the low-dimensional projection of
    the smoothed similarity matrix
  repo: https://github.com/lmcinnes/umap
provenance:
  source_task_ids:
  - task_006
  - task_004
  source_papers:
  - doi: 10.1038/s41467-021-21246-9
    title: Inference and analysis of cell-cell communication using CellChat
  source_content_hash: sha256:cb519b34179099365603358c17884c9208cf1b63e1c1759f7c59e3f0e34df08a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sqjin/CellChat
  tools:
  - CellChat
  - compareInteractions
  - computeCommunProb
  - computeCommunProbPathway
  - createCellChat
  - identifyOverExpressedGenes
  - mergeCellChat
  - netAnalysis_computeCentrality
  - netAnalysis_diff_signalingRole_scatter
  - netEmbedding
  - netVisual_bubble
  - umap
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_0128
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5909090909090909
n_claims: 22
---

# manifold-learning-of-signaling-networks

## Summary

Jointly embed inferred cell-cell communication networks from two or more CellChat conditions on a shared low-dimensional manifold (SNN similarity + smoothing + UMAP) so conserved and context-specific signaling pathways can be classified and quantitatively contrasted by information-flow changes.

## When to use

Use when comparing inferred signaling networks across two conditions (e.g., E13.5 vs E14.5 embryonic skin, lesional vs nonlesional AD skin) and you need to cluster pathways by functional similarity and rank up/downregulated signaling by differential information flow.

## When NOT to use

- Only a single condition/dataset is available — joint manifold learning and rankNet contrasts require at least two merged CellChat objects
- The two datasets use different ligand-receptor databases or non-comparable cell-group labels, so the shared pathway/cell-population space for the joint embedding is undefined
- The goal is de novo interaction inference rather than cross-condition comparison of already inferred networks

## Inputs

- Per-condition scRNA-seq count matrices with cell-group metadata (e.g., E13.5 and E14.5 embryonic skin, or lesional/nonlesional human skin)
- Per-condition CellChat objects with inferred pathway-level communication probability networks
- CellChatDB interaction database (mouse or human, using the appropriate species DB)
- Set of signaling pathways shared across common cell populations between conditions

## Outputs

- Joint UMAP embedding coordinates of signaling pathways across conditions
- Pathway clustering assignments on the shared manifold (netClustering)
- Information-flow classification table (turn-on / increase / decrease / turn-off per pathway)
- Merged CellChat comparison object (e.g., embryonic_skin_cellchat_comparison.rds) and ranked differential interaction tables (e.g., information_flow_ranking.csv)

## How to apply

Build a separate CellChat object per condition: createCellChat with the mouse CellChatDB, identifyOverExpressedGenes (Wilcoxon rank sum test, significance level 0.05), computeCommunProb with the Hill-function mass-action model, trimean ensemble averaging, and a permutation test (M=100 group-label shuffles, p<0.05), then aggregate to pathway level with computeCommunProbPathway. Compute signaling-role centralities (out-degree, in-degree, flow betweenness, information centrality) with netAnalysis_computeCentrality per dataset. Merge objects with mergeCellChat and run joint manifold learning over signaling pathways shared across common cell populations: construct an SNN-based similarity network, apply similarity smoothing, and embed with UMAP via netClustering and netEmbedding. Finally, use rankNet-style quantitative contrast of differential information flow (interaction counts and strength) to classify pathways as turned on (e.g., TNF), increased (WNT, GALECTIN, KIT, IGF, VEGF), decreased (PTN, MK), or turned off (NT, TWEAK), and inspect Euclidean distances in the joint embedding to identify pathways whose signaling networks diverge most between conditions.

## Related tools

- **CellChat** (R package that infers communication networks per condition, merges them with mergeCellChat, and performs joint manifold learning via netClustering and netEmbedding) — https://github.com/sqjin/CellChat
- **createCellChat** (Builds the per-dataset CellChat object from count matrices and metadata before network inference) — https://github.com/jinworks/CellChat
- **computeCommunProb** (Infers ligand-receptor communication probabilities (Hill function, permutation test M=100, p<0.05) that feed the compared networks) — https://github.com/jinworks/CellChat
- **mergeCellChat** (Merges the per-condition CellChat objects into a single comparison object for joint embedding and rankNet contrasts) — https://github.com/jinworks/CellChat
- **netEmbedding** (Performs the manifold learning/UMAP embedding of the smoothed signaling-network similarity matrix) — https://github.com/jinworks/CellChat
- **netAnalysis_computeCentrality** (Computes out-degree, in-degree, flow betweenness, and information centrality for senders/receivers/mediators/influencers per condition) — https://github.com/jinworks/CellChat
- **umap** (Underlying UMAP implementation used for the low-dimensional projection of the smoothed similarity matrix) — https://github.com/lmcinnes/umap

## Examples

```
cellchat <- createCellChat(data, meta=meta, group.by="labels"); cellchat <- computeCommunProb(cellchat); cellchat <- computeCommunProbPathway(cellchat); cellchat <- netAnalysis_computeCentrality(cellchat); merged <- mergeCellChat(list(E13.5=cellchat1, E14.5=cellchat2)); merged <- netEmbedding(merged); merged <- netClustering(merged)
```

## Evaluation signals

- Joint embedding reproduces the reported pathway groupings, with WNT and KIT at large Euclidean distances between E13.5 and E14.5
- Information-flow classification recovers turn-on TNF, increased WNT/GALECTIN/KIT (and IGF/VEGF), decreased PTN/MK, and turned-off NT/TWEAK signaling
- Disease-comparison ranking places CCL19-CCR7 from Inflam.FIB to Inflam.DC as the most significantly upregulated interaction (output_matches_reference against reported figures)
- Permutation-based significance (M=100 shuffles, p<0.05) was applied before pathway aggregation; centralities computed for all 11 common cell populations
- Merged object and embedding coordinates save/load cleanly (e.g., saveRDS of embryonic_skin_cellchat_comparison.rds) and contain the shared pathway set

## Limitations

- No ground truth exists for systematic evaluation of predicted communication networks; similar methods may generate similar overlap regardless of accuracy
- Communication probability represents interaction strength and is not exactly a probability; mRNA levels approximate protein-level ligand-receptor interactions
- Manifold-learning comparisons are confounded when different ligand-receptor databases are used across conditions

## Evidence

- [abstract] Manifold learning classifies conserved and context-specific pathways: "Through manifold learning and quantitative contrasts, CellChat classifies signaling pathways and delineates conserved and context-specific pathways across different datasets."
- [discussion] UMAP on smoothed similarity matrix: "Finally, we performed uniform manifold approximation and projection (UMAP)92 on the smoothed similarity matrix."
- [discussion] Permutation test with M=100 group label shuffling: "The significant interactions between two cell groups are identified using a permutation test by randomly permuting the group labels of cells"
- [discussion] Centrality measures identify dominant signaling roles: "we used measures in weighted-directed networks, including out-degree, in-degree, flow betweenesss and information centrality, to respectively identify dominant senders, receivers, mediators and"
- [readme] CellChat repository migrated to new location: [evidence span withheld — anchor status: external_doc]
