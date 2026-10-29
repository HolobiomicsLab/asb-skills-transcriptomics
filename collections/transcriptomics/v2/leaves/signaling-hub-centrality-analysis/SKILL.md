---
name: signaling-hub-centrality-analysis
description: Use when a CellChat communication network has been computed (significant
  L-R pairs at p<0.05) and you need to rank cell groups as dominant senders/receivers/mediators/influencers
  or pinpoint the hub cell state.
when_to_use_negative:
- No clustering exists yet — cell clustering is a pre-requisite for CellChat centrality
  analysis, so raw unannotated scRNA-seq must be clustered/annotated first (e.g. placode,
  pre-DC, DC1, DC2 states)
- You need interaction-level confidence/ground-truth validation rather than network
  ranking — systematic evaluation of predicted cell-cell communication networks is
  challenging due to the lack of ground truth
- The input is spatial transcriptomics requiring single-cell-resolution inference
  — use Spatial CellChat (CellChat v3) instead
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: CellChat
  role: 'Core package: computeCommunProb infers the communication network; netAnalysis_computeCentrality
    computes out-degree, in-degree, flow betweenness and information centrality to
    identify dominant senders/receivers/mediators/influencers'
  repo: https://github.com/sqjin/CellChat
- name: Seurat
  role: Load and QC the scRNA-seq data, subcluster the dermal compartment (Louvain,
    cell types with abundance > 5%) and provide the cell-group metadata fed into createCellChat
- name: UMAP
  role: Manifold learning via SNN similarity network construction, similarity smoothing
    and UMAP for visualizing signaling similarity between groups
provenance:
  source_task_ids:
  - task_003
  - task_001
  source_papers:
  - doi: 10.1038/s41467-021-21246-9
    title: Inference and analysis of cell-cell communication using CellChat
  source_content_hash: sha256:a3b15eaf15b7324685f4565bd70ee6620b1b4107db5d7beca0fa308e974a70c5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sqjin/CellChat
  tools:
  - CellChat
  - Seurat
  - UMAP
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# signaling-hub-centrality-analysis

## Summary

Identify dominant senders, receivers, mediators and influencers in a CellChat-inferred ligand-receptor communication network using weighted-directed graph centrality metrics (out-degree, in-degree, flow betweenness, information centrality), as demonstrated by the placode hub in E14.5 embryonic skin (44 secreted / 19 received significant pairs).

## When to use

Use when a CellChat communication network has been computed (significant L-R pairs at p<0.05) and you need to rank cell groups as dominant senders/receivers/mediators/influencers or pinpoint the hub cell state.

## When NOT to use

- No clustering exists yet — cell clustering is a pre-requisite for CellChat centrality analysis, so raw unannotated scRNA-seq must be clustered/annotated first (e.g. placode, pre-DC, DC1, DC2 states)
- You need interaction-level confidence/ground-truth validation rather than network ranking — systematic evaluation of predicted cell-cell communication networks is challenging due to the lack of ground truth
- The input is spatial transcriptomics requiring single-cell-resolution inference — use Spatial CellChat (CellChat v3) instead

## Inputs

- CellChat object with computed communication probability network (output of computeCommunProb, computeCommunProbPathway, filterCommunication)
- scRNA-seq count matrix with cell-group metadata (e.g. Seurat object, E14.5 embryonic skin or day-12 wound data)
- CellChatDB ligand-receptor database (mouse, label-based mode)

## Outputs

- Per-cell-group centrality scores (out-degree, in-degree, flow betweenness, information centrality) ranking senders/receivers/mediators/influencers
- Hub statistics: number of secreted and received significant ligand-receptor pairs per group (e.g. placode: 44 outgoing / 19 incoming)
- Enumerated hub-to-target ligand-receptor pair lists by signaling pathway (e.g. 21 placode-to-DC pairs across 7 pathways)
- e14_5_placode_cellchat_results.csv tabulated hub statistics

## How to apply

After building a CellChat object (createCellChat against CellChatDB) and inferring significant interactions with identifyOverExpressedGenes (Wilcoxon rank sum, 0.05), computeCommunProb (Hill/law-of-mass-action model, permutation test with M=100 label shuffles, p<0.05), computeCommunProbPathway and filterCommunication, call netAnalysis_computeCentrality to compute out-degree, in-degree, flow betweenness and information centrality on the weighted-directed network. Out-degree ranks dominant senders, in-degree ranks dominant receivers, and flow betweenness / information centrality identify mediators and influencers. Extract per-group sender/receiver statistics, e.g. for the E14.5 embryonic skin states (placode, pre-DC, DC1, DC2), confirming the placode secretes via 44 and receives via 19 significant ligand-receptor pairs. Ensemble-averaged expression of signaling genes uses the statistically robust trimean of quartiles. Tabulate hub statistics and hub-to-target pairs (e.g. 21 placode-to-DC pairs across WNT, TGFbeta, SEMA3, PTN, PDGF, MK, FGF) for downstream validation.

## Related tools

- **CellChat** (Core package: computeCommunProb infers the communication network; netAnalysis_computeCentrality computes out-degree, in-degree, flow betweenness and information centrality to identify dominant senders/receivers/mediators/influencers) — https://github.com/sqjin/CellChat
- **Seurat** (Load and QC the scRNA-seq data, subcluster the dermal compartment (Louvain, cell types with abundance > 5%) and provide the cell-group metadata fed into createCellChat)
- **UMAP** (Manifold learning via SNN similarity network construction, similarity smoothing and UMAP for visualizing signaling similarity between groups)

## Examples

```
cellchat <- netAnalysis_computeCentrality(cellchat, slot.name = "netP")
```

## Evaluation signals

- Placode hub statistics match reported values: secretes via 44 and receives via 19 significant ligand-receptor pairs
- Exactly 21 placode-to-DC ligand-receptor pairs spanning the seven listed pathways (WNT, TGFbeta, SEMA3, PTN, PDGF, MK, FGF) are recovered
- Pros1-Axl and Edn3-Ednrb are called as significant placode-to-dermal-condensate interactions at p<0.05
- Significant interactions pass the default permutation test (M=100 group-label shuffles, p<0.05) after filterCommunication
- For day-12 wound data as a reference run: 60 significant L-R pairs across 25 pathways with 5 outgoing and 5 incoming NMF communication patterns

## Limitations

- Communication probability only represents interaction strength and is not exactly a probability
- Ligand-receptor interactions and law-of-mass-action signaling happen at the protein level, but mRNA levels are used to approximate protein levels; single-cell proteomic information is technically difficult to capture
- Systematic evaluation of predicted cell-cell communication networks is challenging due to the lack of ground truth, and overlap-based comparison of similar methods can generate similar results regardless of accuracy

## Evidence

- [discussion] centrality metrics identify dominant senders/receivers: "we used measures in weighted-directed networks, including out-degree, in-degree, flow betweenesss and information centrality, to respectively identify dominant senders, receivers, mediators and"
- [discussion] permutation test identifies significant interactions: "The significant interactions between two cell groups are identified using a permutation test by randomly permuting the group labels of cells"
- [discussion] ensemble average expression via trimean: "we calculated the ensemble average expression of signaling genes in a given cell group using a statistically robust mean method"
- [discussion] communication probability is not a probability: "The communication probability here only represents the interaction strength and is not exactly a probability."
- [abstract] CellChat predicts signaling inputs and outputs: "CellChat predicts major signaling inputs and outputs for cells and how those cells and signals coordinate for functions using network analysis and pattern recognition approaches."
