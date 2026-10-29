---
name: comparative-communication-analysis-across-conditions
description: Use when two or more scRNA-seq conditions (e.g. E13.5 vs E14.5 embryonic
  skin) have each been processed with CellChat and the goal is to identify which signaling
  pathways shift position in a joint pathway embedding or change information flow
  between conditions.
when_to_use_negative:
- Only a single condition is available — merging and joint embedding require at least
  two inferred networks; use per-dataset network analysis instead
- The datasets lack shared signaling pathways or common cell populations, so no joint
  manifold space over pathways can be constructed
- The goal is raw ligand-receptor pair inference or CellChatDB construction rather
  than cross-condition pathway-level comparison
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_3316
tools:
- name: CellChat
  role: 'Full pipeline: createCellChat, identifyOverExpressedGenes, computeCommunProb,
    computeCommunProbPathway, netAnalysis_computeCentrality, mergeCellChat, netClustering,
    netEmbedding, rankNet'
  repo: https://github.com/sqjin/CellChat
- name: UMAP
  role: Underlying manifold-learning method applied to the smoothed similarity matrix
    of signaling pathway networks
  repo: https://github.com/lmcinnes/umap
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-021-21246-9
    title: Inference and analysis of cell-cell communication using CellChat
  source_content_hash: sha256:71435c6e2c60989ceb3b1023f2c6250c2bca940a9097af2628a728bcc340a64f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sqjin/CellChat
  tools:
  - CellChat
  - UMAP
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_3316
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# comparative-communication-analysis-across-conditions

## Summary

Jointly compares CellChat-inferred cell-cell communication networks across two or more biological conditions by manifold-learning the signaling pathways in a shared space (SNN similarity + smoothing + UMAP) and quantitatively contrasting information flow, so that pathways can be classified as turned on/off, increased, or decreased between conditions.

## When to use

Use when two or more scRNA-seq conditions (e.g. E13.5 vs E14.5 embryonic skin) have each been processed with CellChat and the goal is to identify which signaling pathways shift position in a joint pathway embedding or change information flow between conditions.

## When NOT to use

- Only a single condition is available — merging and joint embedding require at least two inferred networks; use per-dataset network analysis instead
- The datasets lack shared signaling pathways or common cell populations, so no joint manifold space over pathways can be constructed
- The goal is raw ligand-receptor pair inference or CellChatDB construction rather than cross-condition pathway-level comparison

## Inputs

- Per-condition CellChat objects built from count matrices + cell metadata with createCellChat (mouse CellChatDB)
- Identified signaling pathways per condition (from computeCommunProbPathway) over common cell populations

## Outputs

- Merged CellChat comparison object (mergeCellChat), saved as RDS (e.g. embryonic_skin_cellchat_comparison.rds)
- Joint manifold-learning embedding coordinates of signaling pathways (SNN similarity + smoothing + UMAP via netClustering/netEmbedding)
- Information-flow classification table per pathway: turn-on / turn-off / increase / decrease between conditions

## How to apply

Build a CellChat object per condition: createCellChat with the species-specific CellChatDB, run identifyOverExpressedGenes (Wilcoxon rank sum test, significance level 0.05), computeCommunProb with the permutation test that randomly permutes group labels (M=100 permutations, p<0.05), and computeCommunProbPathway to aggregate interactions to signaling pathways; run netAnalysis_computeCentrality per object to compute out-degree, in-degree, flow betweenness and information centrality. Merge the objects with mergeCellChat over signaling pathways shared across common cell populations (e.g. the 11 shared populations in E13.5 vs E14.5 embryonic skin), then perform joint manifold learning — SNN-based similarity construction, similarity smoothing, and UMAP embedding via netClustering and netEmbedding — so conserved and divergent pathways separate by Euclidean distance. Finally use a rankNet-style quantitative contrast of information flow to classify each pathway as turn-off (e.g. NT, TWEAK), turn-on (e.g. TNF), increase (e.g. WNT, GALECTIN, KIT, IGF, VEGF), or decrease (e.g. PTN, MK) at the later stage. Save the merged comparison object, joint embedding coordinates, and classification table (e.g. embryonic_skin_cellchat_comparison.rds).

## Related tools

- **CellChat** (Full pipeline: createCellChat, identifyOverExpressedGenes, computeCommunProb, computeCommunProbPathway, netAnalysis_computeCentrality, mergeCellChat, netClustering, netEmbedding, rankNet) — https://github.com/sqjin/CellChat
- **UMAP** (Underlying manifold-learning method applied to the smoothed similarity matrix of signaling pathway networks) — https://github.com/lmcinnes/umap

## Examples

```
merged <- mergeCellChat(list(E13.5 = cellchat135, E14.5 = cellchat145), add.names = c("E13.5","E14.5")); merged <- netEmbedding(merged, type = "functional")
```

## Evaluation signals

- Joint embedding reproduces reported pathway groupings: WNT and KIT show large Euclidean distances between the E13.5 and E14.5 network positions
- rankNet-style information-flow contrast recovers turn-on TNF, increased WNT/GALECTIN/KIT, and decreased PTN/MK; NT and TWEAK turn off
- Both CellChat objects processed identically (Wilcoxon 0.05 overexpressed genes; permutation test M=100, p<0.05) before merging
- Comparison restricted to pathways shared across common cell populations (e.g. 11 common cell populations), so the embedding is built on a consistent pathway set

## Limitations

- Communication probability represents interaction strength and is not exactly a probability
- mRNA levels are used to approximate protein-level ligand-receptor signaling; no single-cell proteomic ground truth is available, making systematic evaluation of predicted networks difficult
- Similar methods tend to generate similar results regardless of accuracy, so overlap-based comparison across conditions should be interpreted cautiously

## Evidence

- [abstract] Through manifold learning and quantitative contrasts, CellChat classifies signaling pathways and delineates conserved and context-specific pathways across different datasets.: "Through manifold learning and quantitative contrasts, CellChat classifies signaling pathways and delineates conserved and context-specific pathways across different datasets."
- [discussion] Finally, we performed uniform manifold approximation and projection (UMAP)92 on the smoothed similarity matrix.: "Finally, we performed uniform manifold approximation and projection (UMAP)92 on the smoothed similarity matrix."
- [discussion] The significant interactions between two cell groups are identified using a permutation test by randomly permuting the group labels of cells: "The significant interactions between two cell groups are identified using a permutation test by randomly permuting the group labels of cells"
- [discussion] we used measures in weighted-directed networks, including out-degree, in-degree, flow betweenesss and information centrality, to respectively identify dominant senders, receivers, mediators and: "we used measures in weighted-directed networks, including out-degree, in-degree, flow betweenesss and information centrality"
- [methods] CellChat v1/v2 R package can be easily installed from Github using devtools: "CellChat v1/v2 R package can be easily installed from Github using devtools: devtools::install_github("jinworks/CellChat")"
