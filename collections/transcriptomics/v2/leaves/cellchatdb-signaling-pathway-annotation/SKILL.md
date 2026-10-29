---
name: cellchatdb-signaling-pathway-annotation
description: Use when a scRNA-seq dataset has already been clustered into cell groups
  and you need significant ligand-receptor pairs classified into curated signaling
  pathways (e.g., reproducing the 60 pairs / 25 pathways day-12 wound result).
when_to_use_negative:
- When ligand-receptor pairs must be scored against a database other than CellChatDB
  (e.g., NicheNet or SingleCellSignalR priors) — different ligand-receptor databases
  across methods confound cross-method comparison.
- When input data lack cell clustering — CellChat cell-cell communication analysis
  requires pre-defined cell groups.
- When single-cell proteomic validation of signaling is required — CellChat approximates
  protein-level signaling from mRNA, which is not exactly a probability.
edam_operation: http://edamontology.org/operation_2436
edam_topics:
- http://edamontology.org/topic_0602
- http://edamontology.org/topic_3170
tools:
- name: CellChat
  role: R package providing createCellChat, identifyOverExpressedGenes, computeCommunProb,
    filterCommunication and computeCommunProbPathway for pathway-level annotation
    of cell-cell communication
  repo: https://github.com/sqjin/CellChat
- name: CellChatDB
  role: Manually curated database of 2,021 validated molecular interactions (multi-subunit
    complexes and cofactors) used to classify inferred pairs into signaling pathways
- name: KEGG Pathway database
  role: Source of the majority of manually curated ligand-receptor interactions underlying
    CellChatDB pathway annotations
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-021-21246-9
    title: Inference and analysis of cell-cell communication using CellChat
  source_content_hash: sha256:f619b8f2d94d7038be0f1417020f7e20657b6d096097b8f8820f3627a36e09c6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sqjin/CellChat
  tools:
  - CellChat
  - CellChatDB
  - KEGG Pathway database
  edam_operation: http://edamontology.org/operation_2436
  edam_topics:
  - http://edamontology.org/topic_0602
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# cellchatdb-signaling-pathway-annotation

## Summary

Annotates inferred ligand-receptor communications into functionally related signaling pathways using the manually curated CellChatDB (2,021 validated interactions), then aggregates pair-level communication probabilities to pathway level with computeCommunProbPathway. This yields interpretable pathway-level networks such as TGFβ, ncWNT, TNF and SPP1 in the day-12 mouse skin wound dataset.

## When to use

Use when a scRNA-seq dataset has already been clustered into cell groups and you need significant ligand-receptor pairs classified into curated signaling pathways (e.g., reproducing the 60 pairs / 25 pathways day-12 wound result).

## When NOT to use

- When ligand-receptor pairs must be scored against a database other than CellChatDB (e.g., NicheNet or SingleCellSignalR priors) — different ligand-receptor databases across methods confound cross-method comparison.
- When input data lack cell clustering — CellChat cell-cell communication analysis requires pre-defined cell groups.
- When single-cell proteomic validation of signaling is required — CellChat approximates protein-level signaling from mRNA, which is not exactly a probability.

## Inputs

- Preprocessed scRNA-seq expression matrix with cell-group metadata (e.g., GSE113854 day-12 mouse skin wound data, red blood cells excluded, 21,557 cells retained)
- Cell cluster labels / cell group assignments
- CellChatDB (mouse, label-based mode; updateable via updateCellChatDB)

## Outputs

- List of significant ligand-receptor pairs (60 pairs among 25 cell groups for the day-12 wound dataset)
- Pathway-level aggregated communication network (25 signaling pathways including TGFβ, ncWNT, TNF, SPP1, PTN, PDGF, CXCL, CCL, MIF)
- Pathway-level communication probability matrix

## How to apply

Construct a CellChat object with createCellChat using the mouse CellChatDB in label-based mode. Run identifyOverExpressedGenes (Wilcoxon rank sum test, significance level 0.05) followed by identifyOverExpressedInteractions, then infer pair-level communication probabilities with computeCommunProb, which models ligand-receptor interactions via a Hill function under the law of mass action with a default permutation test (nboot=100) that randomly permutes cell group labels. Filter with filterCommunication and aggregate significant pairs into pathways with computeCommunProbPathway. CellChatDB's curation matters because it accounts for multi-subunit ligand-receptor complexes and cofactors, classifying interactions into paracrine/autocrine signaling (60%), ECM-receptor (21%) and cell-cell contact (19%) categories drawn largely from KEGG.

## Related tools

- **CellChat** (R package providing createCellChat, identifyOverExpressedGenes, computeCommunProb, filterCommunication and computeCommunProbPathway for pathway-level annotation of cell-cell communication) — https://github.com/sqjin/CellChat
- **CellChatDB** (Manually curated database of 2,021 validated molecular interactions (multi-subunit complexes and cofactors) used to classify inferred pairs into signaling pathways)
- **KEGG Pathway database** (Source of the majority of manually curated ligand-receptor interactions underlying CellChatDB pathway annotations)

## Examples

```
cellchat <- createCellChat(object, meta = meta, group.by = "labels"); cellchat@DB <- CellChatDB.mouse; cellchat <- identifyOverExpressedGenes(cellchat); cellchat <- computeCommunProb(cellchat); cellchat <- filterCommunication(cellchat, min.cells = 10); cellchat <- computeCommunProbPathway(cellchat)
```

## Evaluation signals

- The day-12 wound network yields exactly 60 significant ligand-receptor pairs across 25 cell groups.
- The inferred pairs aggregate into 25 signaling pathways, with TGFβ, ncWNT, TNF and SPP1 among them.
- identifyOverExpressedGenes used the Wilcoxon rank sum test at significance level 0.05 before interaction inference.
- computeCommunProb used the default permutation test (nboot=100) with randomly permuted cell group labels and p<0.05 significance.
- filterCommunication removes non-significant pairs before pathway aggregation via computeCommunProbPathway.

## Limitations

- Communication probability represents interaction strength only and is not exactly a probability.
- mRNA levels are used to approximate protein levels; comprehensive modeling of ligand-receptor interactions remains challenging due to lack of single-cell proteomics.
- Hill function parameters Kh and n lack explicit biological connections and serve only as a nonlinear approximation of ligand-receptor interactions.
- No ground truth exists for systematic evaluation of predicted cell-cell communication networks.

## Evidence

- [intro] CellChatDB contains 2,021 validated molecular interactions, including 60% of paracrine/autocrine signaling interactions, 21% of extracellular matrix (ECM)-receptor interactions and 19% of cell–cell: "CellChatDB contains 2,021 validated molecular interactions, including 60% of paracrine/autocrine signaling interactions, 21% of extracellular matrix (ECM)-receptor interactions and 19% of cell–cell"
- [discussion] We modeled ligand-receptor mediated signaling interactions using the law of mass action.: "We modeled ligand-receptor mediated signaling interactions using the law of mass action."
- [discussion] The significant interactions between two cell groups are identified using a permutation test by randomly permuting the group labels of cells: "The significant interactions between two cell groups are identified using a permutation test by randomly permuting the group labels of cells"
- [discussion] we excluded red blood cells and used the remaining 21,557 cells.: "we excluded red blood cells and used the remaining 21,557 cells."
- [methods] A function named `updateCellChatDB` is also provided for easily updating CellChatDB.: "A function named `updateCellChatDB` is also provided for easily updating CellChatDB."
