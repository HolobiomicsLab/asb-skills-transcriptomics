---
name: information-flow-quantification-of-signaling-pathways
description: Use when two or more scRNA-seq conditions (e.g. lesional vs nonlesional
  AD skin, E13.5 vs E14.5 embryonic skin) each have a CellChat object and you need
  to rank which signaling pathways change their information flow between conditions.
when_to_use_negative:
- You have only one condition — information-flow contrasts require at least two CellChat
  objects to merge; use single-dataset centrality analysis instead
- Cell groups are not shared/matched across conditions and there are fewer than the
  common populations needed for joint manifold learning (e.g. the E13.5/E14.5 comparison
  relies on 11 common cell populations)
- The question is about spatially proximal communication at single-cell resolution
  — use CellChat v3 (SpatialCellChat) rather than this condition-contrast workflow
edam_operation: http://edamontology.org/operation_0282
tools:
- name: CellChat
  role: Infers pathway-level communication probabilities, merges condition objects
    (mergeCellChat, compareInteractions), and performs joint manifold learning and
    differential signaling-role analysis (netEmbedding, netClustering, netVisual_bubble,
    netAnalysis_diff_signalingRole_scatter)
  repo: https://github.com/sqjin/CellChat
provenance:
  source_task_ids:
  - task_004
  - task_006
  source_papers:
  - doi: 10.1038/s41467-021-21246-9
    title: Inference and analysis of cell-cell communication using CellChat
  source_content_hash: sha256:388bc24cc28f1936ff4e735c7541c686bc79a7e86c9dd1acd95ad70d5a418c55
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
  - netAnalysis_diff_signalingRole_scatter
  - netEmbedding
  - netVisual_bubble
  edam_operation: http://edamontology.org/operation_0282
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8125
n_claims: 16
---

# information-flow-quantification-of-signaling-pathways

## Summary

Quantify and compare signaling-pathway information flow between conditions by aggregating CellChat-inferred ligand-receptor communications to the pathway level, merging condition-specific CellChat objects, and ranking up/down-regulated pathways via differential interaction counts/strength and joint manifold learning (SNN smoothing + UMAP).

## When to use

Use when two or more scRNA-seq conditions (e.g. lesional vs nonlesional AD skin, E13.5 vs E14.5 embryonic skin) each have a CellChat object and you need to rank which signaling pathways change their information flow between conditions.

## When NOT to use

- You have only one condition — information-flow contrasts require at least two CellChat objects to merge; use single-dataset centrality analysis instead
- Cell groups are not shared/matched across conditions and there are fewer than the common populations needed for joint manifold learning (e.g. the E13.5/E14.5 comparison relies on 11 common cell populations)
- The question is about spatially proximal communication at single-cell resolution — use CellChat v3 (SpatialCellChat) rather than this condition-contrast workflow

## Inputs

- Condition-specific scRNA-seq count matrices with cell-type/group metadata (e.g. lesional vs nonlesional AD skin; E13.5 vs E14.5 embryonic skin from the CellChat figshare deposit)
- CellChat objects per condition with inferred communication probabilities (computeCommunProb / computeCommunProbPathway outputs)
- Organism-specific CellChatDB (human or mouse)

## Outputs

- Ranked differential information-flow table (up/down-regulated pathways, e.g. information_flow_ranking.csv)
- Joint pathway embedding coordinates from SNN smoothing + UMAP over pathways shared across common cell populations
- Information-flow classification table (turn-on / increase / decrease / turn-off) saved as e.g. embryonic_skin_cellchat_comparison.rds
- Differential ligand-receptor bubble plots and signaling-role scatter plots

## How to apply

For each condition, build a CellChat object with createCellChat, run identifyOverExpressedGenes (Wilcoxon rank sum test, significance level 0.05), then computeCommunProb with the Hill-function mass-action model, trimean ensemble averaging, and the permutation test (M=100 group-label shuffles, p<0.05); aggregate to pathways with computeCommunProbPathway. Merge the objects with mergeCellChat and quantify total interaction number and strength per condition via compareInteractions. Compute differential information flow (interaction counts and strength) to classify pathways as turned on, increased, decreased, or turned off (e.g. TNF turn-on and WNT/GALECTIN/KIT increase vs PTN/MK decrease at E14.5; CCL19-CCR7 upregulated lesional). Jointly embed both networks on a shared manifold via SNN-based similarity construction, similarity smoothing, and UMAP (netEmbedding, netClustering) to group conserved vs context-specific pathways, and extract disease-enriched ligand-receptor pairs with netVisual_bubble and netAnalysis_diff_signalingRole_scatter.

## Related tools

- **CellChat** (Infers pathway-level communication probabilities, merges condition objects (mergeCellChat, compareInteractions), and performs joint manifold learning and differential signaling-role analysis (netEmbedding, netClustering, netVisual_bubble, netAnalysis_diff_signalingRole_scatter)) — https://github.com/sqjin/CellChat

## Examples

```
cc.merged <- mergeCellChat(list(NL=cc.nl, LS=cc.ls), add.names=c("NL","LS")); netAnalysis_diff_signalingRole_scatter(cc.merged)
```

## Evaluation signals

- The information-flow ranking reproduces the paper's ordering, with CCL19-CCR7 signaling from Inflam.FIB to Inflam.DC as the most significantly upregulated disease-specific interaction
- The joint embedding reproduces reported pathway groupings, with WNT and KIT at large Euclidean distances between E13.5 and E14.5
- The classification table recovers turn-on TNF, increased WNT/GALECTIN/KIT, decreased PTN/MK, and turn-off NT/TWEAK pathways
- All significant interactions pass the permutation test with M=100 label shuffles at p<0.05; overexpressed genes pass the Wilcoxon test at significance level 0.05
- compareInteractions returns finite total interaction counts and strengths for each condition before differential ranking

## Limitations

- The communication probability only represents interaction strength and is not exactly a probability
- Systematic evaluation of predicted cell-cell communication networks is challenging due to the lack of ground truth
- mRNA levels are used to approximate protein levels, so comprehensive modeling of ligand-receptor interactions remains challenging
- The original CellChat repository (sqjin/CellChat) is deprecated; users should migrate to jinworks/CellChat (v2/v3)

## Evidence

- [abstract] CellChat classifies signaling pathways and delineates conserved and context-specific pathways across different datasets.: "Through manifold learning and quantitative contrasts, CellChat classifies signaling pathways and delineates conserved and context-specific pathways across different datasets."
- [discussion] permutation test quote: "The significant interactions between two cell groups are identified using a permutation test by randomly permuting the group labels of cells"
- [discussion] centrality metrics quote: "we used measures in weighted-directed networks, including out-degree, in-degree, flow betweenesss and information centrality, to respectively identify dominant senders, receivers, mediators and"
- [discussion] UMAP manifold learning quote: "Finally, we performed uniform manifold approximation and projection (UMAP)92 on the smoothed similarity matrix."
- [readme] deprecation README quote: [evidence span withheld — anchor status: external_doc]
