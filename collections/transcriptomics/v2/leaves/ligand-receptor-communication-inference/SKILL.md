---
name: ligand-receptor-communication-inference
description: Use when you have clustered scRNA-seq data (discrete cell groups or pseudotime-ordered
  continuous states) and need to predict significant ligand-receptor interactions
  and signaling pathways between cell populations.
when_to_use_negative:
- Input data has not been cell-clustered — cell clustering is a pre-requisite for
  CellChat analysis
- Input is spatial transcriptomics data requiring single-cell-resolution inference
  — use CellChat v3 (SpatialCellChat) instead
- You need validated ground-truth communication interactions — systematic evaluation
  of predicted networks is challenging due to the lack of ground truth
edam_operation: http://edamontology.org/operation_2436
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: CellChat
  role: Core R package for inferring, filtering, analyzing and visualizing ligand-receptor
    communication networks from scRNA-seq
  repo: https://github.com/sqjin/CellChat
- name: CellChat (jinworks/CellChat)
  role: Maintained CellChat v2/v3 repository; installed via devtools::install_github("jinworks/CellChat")
  repo: https://github.com/jinworks/CellChat
- name: CellChatDB
  role: Manually curated signaling interaction database (KEGG-derived) accounting
    for multimeric receptor complexes and cofactors
  repo: https://github.com/sqjin/CellChat
- name: Seurat
  role: Loading scRNA-seq count matrices and metadata, performing QC filters, and
    subclustering cell states (Louvain) prior to CellChat object construction
- name: CellChat Explorer
  role: Web-based interactive exploration of inferred cell-cell communication via
    the Cell-Cell Communication Atlas
provenance:
  source_task_ids:
  - task_002
  - task_003
  - task_001
  source_papers:
  - doi: 10.1038/s41467-021-21246-9
    title: Inference and analysis of cell-cell communication using CellChat
  source_content_hash: sha256:cd8deb53dbbe6951e97c9f79b679e010c14a21726cc20be5eb4a45cba6d0d2ce
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sqjin/CellChat
  tools:
  - CellChat
  - CellChat (jinworks/CellChat)
  - CellChat Explorer
  - CellChatDB
  - Seurat
  - computeCommunProb
  - createCellChat
  - filterCommunication
  - netAnalysis_computeCentrality
  edam_operation: http://edamontology.org/operation_2436
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 20
---

# ligand-receptor-communication-inference

## Summary

Infer, filter, and analyze cell-cell ligand-receptor communication networks from scRNA-seq using CellChat's mass-action/Hill-function model with the curated CellChatDB, then quantify signaling roles via centrality metrics and pattern recognition.

## When to use

Use when you have clustered scRNA-seq data (discrete cell groups or pseudotime-ordered continuous states) and need to predict significant ligand-receptor interactions and signaling pathways between cell populations.

## When NOT to use

- Input data has not been cell-clustered — cell clustering is a pre-requisite for CellChat analysis
- Input is spatial transcriptomics data requiring single-cell-resolution inference — use CellChat v3 (SpatialCellChat) instead
- You need validated ground-truth communication interactions — systematic evaluation of predicted networks is challenging due to the lack of ground truth

## Inputs

- Preprocessed scRNA-seq count matrix (e.g., 10X Genomics data from GEO accessions GSM3453537/GSM3453538 or GSE113854) with cell group/cluster metadata
- QC-filtered Seurat object meeting embryonic skin criteria (UMI 2500-50000, genes >= 1000, mitochondrial fraction < 20%)
- CellChatDB database (mouse or human, 2021 validated interactions accounting for heteromeric complexes and cofactors)

## Outputs

- Significant ligand-receptor interaction table (e.g., cellchat_e145_significant_interactions.csv) at p<0.05
- Pathway-level aggregated communication network (computeCommunProbPathway output)
- Centrality scores identifying dominant senders, receivers, mediators, and influencers per cell group
- Outgoing/incoming communication patterns (NMF-based, e.g., 5 outgoing and 5 incoming patterns)
- Circle/heatmap visualizations via netVisual_aggregate for selected signaling networks (e.g., Pros1-Axl, Edn3-Ednrb)

## How to apply

Build a CellChat object with createCellChat using the mouse CellChatDB (secreted-signaling label-free mode or label-based mode), then run identifyOverExpressedGenes with the Wilcoxon rank sum test at significance level 0.05 and identifyOverExpressedInteractions. Compute communication probabilities with computeCommunProb, which models ligand-receptor interactions via a Hill function under the law of mass action using ensemble average expression computed as the trimean of quartiles; identify significant interactions by permuting cell group labels with the default M=100 permutation test at p<0.05. Filter with filterCommunication, aggregate interactions to pathway level with computeCommunProbPathway, and compute sender/receiver/mediator/influencer roles with netAnalysis_computeCentrality (out-degree, in-degree, flow betweenness, information centrality). Optionally decompose the network into outgoing/incoming patterns with identifyCommunicationPatterns using non-negative matrix factorization, setting W and H elements below 0.5 to zero. Validate against known counts (e.g., 88 L-R interactions across 22 pathways in E14.5 embryonic skin; 60 L-R pairs across 25 pathways in day-12 wound) and export tables such as cellchat_e145_significant_interactions.csv.

## Related tools

- **CellChat** (Core R package for inferring, filtering, analyzing and visualizing ligand-receptor communication networks from scRNA-seq) — https://github.com/sqjin/CellChat
- **CellChat (jinworks/CellChat)** (Maintained CellChat v2/v3 repository; installed via devtools::install_github("jinworks/CellChat")) — https://github.com/jinworks/CellChat
- **CellChatDB** (Manually curated signaling interaction database (KEGG-derived) accounting for multimeric receptor complexes and cofactors) — https://github.com/sqjin/CellChat
- **Seurat** (Loading scRNA-seq count matrices and metadata, performing QC filters, and subclustering cell states (Louvain) prior to CellChat object construction)
- **CellChat Explorer** (Web-based interactive exploration of inferred cell-cell communication via the Cell-Cell Communication Atlas)

## Examples

```
cellchat <- createCellChat(object, meta = meta, group.by = "labels"); cellchat <- identifyOverExpressedGenes(cellchat); cellchat <- computeCommunProb(cellchat); cellchat <- computeCommunProbPathway(cellchat); cellchat <- netAnalysis_computeCentrality(cellchat)
```

## Evaluation signals

- Reproduced day-12 wound network yields 60 significant ligand-receptor pairs across 25 signaling pathways and NMF pattern recognition returns 5 outgoing and 5 incoming patterns
- E14.5 embryonic skin run recovers 88 significant ligand-receptor interactions across 22 pathways, with Wnt6-Fzd10/Lrp6 and Wnt6-Fzd2/Lrp6 as dominant canonical WNT pairs and FIB-E-derived ncWNT signaling present
- Placode hub statistics match reported values: placode cells secrete via 44 and receive via 19 ligand-receptor pairs, with exactly 21 placode-to-DC pairs spanning WNT, TGFbeta, SEMA3, PTN, PDGF, MK and FGF pathways
- Pros1-Axl and Edn3-Ednrb are called as significant placode-to-dermal-condensate interactions at p<0.05
- filterCommunication removed insignificant interactions and the permutation test (M=100 label shuffles) produced p<0.05 significance calls for retained pairs

## Limitations

- Systematic evaluation of predicted cell-cell communication networks is challenging due to the lack of ground truth
- mRNA levels are used to approximate protein levels; comprehensive modeling of ligand-receptor interactions remains challenging due to lack of single-cell proteomics
- Mass action model parameters Kh and n lack explicit biological connections; the Hill function is only a nonlinear approximation
- Communication probability only represents interaction strength and is not exactly a probability
- Different ligand-receptor databases across methods confound cross-method performance comparison

## Evidence

- [discussion] We modeled ligand-receptor mediated signaling interactions using the law of mass action.: "We modeled ligand-receptor mediated signaling interactions using the law of mass action."
- [discussion] The significant interactions between two cell groups are identified using a permutation test by randomly permuting the group labels of cells: "The significant interactions between two cell groups are identified using a permutation test by randomly permuting the group labels of cells"
- [discussion] we used measures in weighted-directed networks, including out-degree, in-degree, flow betweenesss and information centrality, to respectively identify dominant senders, receivers, mediators and: "we used measures in weighted-directed networks, including out-degree, in-degree, flow betweenesss and information centrality, to respectively identify dominant senders, receivers, mediators and"
- [intro] CellChatDB contains 2,021 validated molecular interactions, including 60% of paracrine/autocrine signaling interactions, 21% of extracellular matrix (ECM)-receptor interactions: "CellChatDB contains 2,021 validated molecular interactions, including 60% of paracrine/autocrine signaling interactions, 21% of extracellular matrix (ECM)-receptor interactions"
- [methods] CellChat v1/v2 R package can be easily installed from Github using devtools: devtools::install_github("jinworks/CellChat"): "CellChat v1/v2 R package can be easily installed from Github using devtools: devtools::install_github("jinworks/CellChat")"
