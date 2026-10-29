---
name: ligand-receptor-interaction-prediction-in-disease-skin
description: Use when comparing cell-cell communication between two disease states
  (e.g., lesional vs. nonlesional skin scRNA-seq) to identify significantly upregulated
  ligand-receptor interactions and shifted information flow.
when_to_use_negative:
- Data are spatially resolved and single-cell spatial mapping of communication is
  required — use SpatialCellChat (CellChat v3) instead, as the base workflow infers
  probabilities without spatial constraints
- The goal is predicting altered intracellular signaling downstream of receptors rather
  than ligand-receptor communication — tools like NicheNet address a different question
- No cell-group clustering is available — cell clustering is a pre-requisite for CellChat
  analysis
edam_operation: http://edamontology.org/operation_2437
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0602
tools:
- name: CellChat
  role: R package that infers ligand-receptor communication probabilities via Hill-function
    mass-action modeling and comparative information-flow analysis across conditions
  repo: https://github.com/sqjin/CellChat
- name: createCellChat
  role: Constructs the per-condition CellChat object from expression data and metadata
  repo: https://github.com/jinworks/CellChat
- name: computeCommunProb
  role: Computes ligand-receptor communication probabilities with trimean ensemble
    averaging and 100-permutation significance testing at p<0.05
  repo: https://github.com/jinworks/CellChat
- name: mergeCellChat
  role: Merges condition-specific CellChat objects for joint comparison and shared-manifold
    embedding
  repo: https://github.com/jinworks/CellChat
- name: netVisual_bubble
  role: Visualizes and ranks disease-enriched ligand-receptor interactions between
    matching cell populations
  repo: https://github.com/jinworks/CellChat
- name: CellChatDB
  role: Manually curated database of 2,021 validated molecular interactions (including
    multimeric complexes and cofactors) used as the prior interaction dictionary
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-021-21246-9
    title: Inference and analysis of cell-cell communication using CellChat
  source_content_hash: sha256:972da10fc8326563a66eef01d31698d2b743ba1f09f0e5361883b0f953ff52da
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sqjin/CellChat
  tools:
  - CellChat
  - CellChatDB
  - compareInteractions
  - computeCommunProb
  - computeCommunProbPathway
  - createCellChat
  - identifyOverExpressedGenes
  - mergeCellChat
  - netAnalysis_diff_signalingRole_scatter
  - netEmbedding
  - netVisual_bubble
  edam_operation: http://edamontology.org/operation_2437
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 21
---

# ligand-receptor-interaction-prediction-in-disease-skin

## Summary

Infers and ranks disease-altered ligand-receptor signaling between matched cell populations by building separate CellChat objects for each condition (e.g., lesional vs. nonlesional atopic-dermatitis skin), merging them, and quantifying differential information flow.

## When to use

Use when comparing cell-cell communication between two disease states (e.g., lesional vs. nonlesional skin scRNA-seq) to identify significantly upregulated ligand-receptor interactions and shifted information flow.

## When NOT to use

- Data are spatially resolved and single-cell spatial mapping of communication is required — use SpatialCellChat (CellChat v3) instead, as the base workflow infers probabilities without spatial constraints
- The goal is predicting altered intracellular signaling downstream of receptors rather than ligand-receptor communication — tools like NicheNet address a different question
- No cell-group clustering is available — cell clustering is a pre-requisite for CellChat analysis

## Inputs

- Processed scRNA-seq expression matrices for two conditions (e.g., lesional and nonlesional skin) with cell-type metadata (e.g., Inflam.FIB, Inflam.DC labels)
- CellChatDB ligand-receptor interaction database (2,021 validated molecular interactions)
- Seurat/SingleCellExperiment objects per condition

## Outputs

- Per-condition CellChat communication probability networks (pathway-aggregated)
- Differential information-flow ranking of up-/down-regulated signaling pathways (information_flow_ranking.csv)
- Comparative interaction count/strength totals (compareInteractions) and joint manifold embedding of both networks

## How to apply

Build separate CellChat objects per condition with createCellChat and identify overexpressed signaling genes per cell group using a Wilcoxon rank sum test at 0.05 significance (identifyOverExpressedGenes). Infer communication probabilities with computeCommunProb, which models ligand-receptor interactions with a Hill function under the law of mass action, uses trimean ensemble averaging of expression, and applies a permutation test with 100 group-label shuffles at p<0.05; then aggregate to pathway level with computeCommunProbPathway. Merge the two objects with mergeCellChat, quantify total interaction number and strength per condition with compareInteractions, and compute differential information flow to rank up- and down-regulated pathways; jointly embed both networks via SNN-based similarity smoothing followed by UMAP (netEmbedding, netClustering). Finally, extract and rank disease-enriched ligand-receptor interactions between matching cell populations with netVisual_bubble and netAnalysis_diff_signalingRole_scatter, saving the ranked table (e.g., information_flow_ranking.csv).

## Related tools

- **CellChat** (R package that infers ligand-receptor communication probabilities via Hill-function mass-action modeling and comparative information-flow analysis across conditions) — https://github.com/sqjin/CellChat
- **createCellChat** (Constructs the per-condition CellChat object from expression data and metadata) — https://github.com/jinworks/CellChat
- **computeCommunProb** (Computes ligand-receptor communication probabilities with trimean ensemble averaging and 100-permutation significance testing at p<0.05) — https://github.com/jinworks/CellChat
- **mergeCellChat** (Merges condition-specific CellChat objects for joint comparison and shared-manifold embedding) — https://github.com/jinworks/CellChat
- **netVisual_bubble** (Visualizes and ranks disease-enriched ligand-receptor interactions between matching cell populations) — https://github.com/jinworks/CellChat
- **CellChatDB** (Manually curated database of 2,021 validated molecular interactions (including multimeric complexes and cofactors) used as the prior interaction dictionary)

## Examples

```
cellchat <- createCellChat(data, meta=meta, group.by="celltype"); cellchat <- identifyOverExpressedGenes(cellchat); cellchat <- computeCommunProb(cellchat); cellchat <- computeCommunProbPathway(cellchat); merged <- mergeCellChat(list(nl=cellchatNL, ls=cellchatLS))
```

## Evaluation signals

- The ranked information-flow table reproduces the reference ordering, with CCL19-CCR7 signaling from Inflam.FIB to Inflam.DC emerging as the most significantly upregulated disease-specific interaction
- Permutation-test p-values for retained interactions are p<0.05 with 100 group-label shuffles; overexpressed signaling genes pass Wilcoxon rank sum test at 0.05
- compareInteractions returns consistent total interaction number and strength per condition, and differential information flow correctly partitions pathways into up- vs. down-regulated sets
- The joint UMAP embedding of both networks (SNN similarity smoothing) yields sensible clustering of signaling groups across conditions
- Output matches the paper's reported comparative figures (output_matches_reference check)

## Limitations

- Communication probability is not exactly a probability — it represents interaction strength only
- mRNA levels are used to approximate protein levels; comprehensive modeling of ligand-receptor interactions remains challenging due to lack of single-cell proteomics
- Hill-function parameters Kh and n lack explicit biological connections in the current model
- No ground truth is available for systematic evaluation of predicted cell-cell communication networks

## Evidence

- [other] CCL19-CCR7 as the most significant signaling, contributing to the communication from Inflam.FIB to Inflam.DC: [evidence span withheld — anchor status: not_found]
- [discussion] Wilcoxon rank sum test with the significance level of 0.05: "we first identified differentially expressed signaling genes across all cell groups within a given scRNA-seq dataset, using the Wilcoxon rank sum test with the significance level of 0.05."
- [discussion] permutation test by randomly permuting the group labels of cells: "The significant interactions between two cell groups are identified using a permutation test by randomly permuting the group labels of cells"
- [intro] CellChatDB contains 2,021 validated molecular interactions: "CellChatDB contains 2,021 validated molecular interactions, including 60% of paracrine/autocrine signaling interactions, 21% of extracellular matrix (ECM)-receptor interactions and 19% of cell–cell"
- [discussion] The communication probability here only represents the interaction strength and is not exactly a probability.: "The communication probability here only represents the interaction strength and is not exactly a probability."
- [readme] README install instruction for the actively maintained CellChat repository: "CellChat v1/v2 R package can be easily installed from Github using devtools: devtools::install_github("jinworks/CellChat")"
