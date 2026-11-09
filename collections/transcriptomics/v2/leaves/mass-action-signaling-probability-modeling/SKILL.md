---
name: mass-action-signaling-probability-modeling
description: Use when inferring cell-state specific communication probabilities from
  scRNA-seq cell groups or continuous pseudotime states (e.g., E14.5 embryonic skin)
  and statistically testing which L-R pairs are significant.
when_to_use_negative:
- Input is spatial transcriptomics data requiring single-cell resolution communication
  — use Spatial CellChat (CellChat v3) instead
- No cell clustering or pseudotime state labels exist yet — cell clustering is a pre-requisite
  for CellChat analysis
- The target signaling events depend on protein-level abundance rather than mRNA proxies
  — CellChat approximates protein level with mRNA
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0202
tools:
- name: CellChat
  role: R package implementing the law-of-mass-action Hill function model, permutation
    testing, and centrality analysis of inferred communication networks
  repo: https://github.com/sqjin/CellChat
- name: CellChatDB
  role: Manually curated ligand-receptor/cofactor database (2021 validated molecular
    interactions) supplying the L-R pairs modeled, including multi-subunit receptor
    complexes
  repo: https://github.com/sqjin/CellChat
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-021-21246-9
    title: Inference and analysis of cell-cell communication using CellChat
  source_content_hash: sha256:e66702c76d8ca831fb6061499d26b2784eef2c67564e039bd2d61942da0fb3ac
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sqjin/CellChat
  tools:
  - CellChat
  - CellChatDB
  - computeCommunProb
  - createCellChat
  - filterCommunication
  - netAnalysis_computeCentrality
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0202
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7647058823529411
n_claims: 17
---

# mass-action-signaling-probability-modeling

## Summary

Model ligand-receptor mediated signaling in CellChat using a Hill function under the law of mass action, with trimean-based ensemble average expression and a permutation test (M=100, p<0.05) to call significant cell-cell communications.

## When to use

Use when inferring cell-state specific communication probabilities from scRNA-seq cell groups or continuous pseudotime states (e.g., E14.5 embryonic skin) and statistically testing which L-R pairs are significant.

## When NOT to use

- Input is spatial transcriptomics data requiring single-cell resolution communication — use Spatial CellChat (CellChat v3) instead
- No cell clustering or pseudotime state labels exist yet — cell clustering is a pre-requisite for CellChat analysis
- The target signaling events depend on protein-level abundance rather than mRNA proxies — CellChat approximates protein level with mRNA

## Inputs

- Filtered scRNA-seq count matrix (e.g., GEO GSM3453537/GSM3453538 E14.5 embryonic mouse skin 10X counts)
- Cell metadata with cell group labels or continuous pseudotime state annotations
- Mouse CellChatDB (secreted-signaling subset for label-free mode)

## Outputs

- Significant ligand-receptor interaction table with communication probabilities (e.g., cellchat_e145_significant_interactions.csv)
- Pathway-level aggregated communication network (computeCommunProbPathway output)
- Centrality scores for dominant senders, receivers, mediators, and influencers (netAnalysis_computeCentrality output)

## How to apply

Build a CellChat object with createCellChat from the filtered count matrix (e.g., embryonic skin QC: UMI 2500-50000, genes >= 1000, mitochondrial fraction < 20%) using the mouse CellChatDB in secreted-signaling (label-free) mode. Identify overexpressed signaling genes with the Wilcoxon rank sum test at significance level 0.05 via identifyOverExpressedGenes and identifyOverExpressedInteractions. Compute communication probability with computeCommunProb, modeling ligand-receptor interactions with the law of mass action where ensemble average expression of signaling genes per cell group is the statistically robust trimean of quartiles, and receptor/agonist effects are handled via multi-subunit complex composition. Call significant interactions using a permutation test that randomly shuffles cell group labels with M=100 permutations at p<0.05. Then filter with filterCommunication, aggregate to pathway level with computeCommunProbPathway, and compute signaling roles with netAnalysis_computeCentrality for sender/receiver/mediator/influencer assignment.

## Related tools

- **CellChat** (R package implementing the law-of-mass-action Hill function model, permutation testing, and centrality analysis of inferred communication networks) — https://github.com/sqjin/CellChat
- **CellChatDB** (Manually curated ligand-receptor/cofactor database (2021 validated molecular interactions) supplying the L-R pairs modeled, including multi-subunit receptor complexes) — https://github.com/sqjin/CellChat

## Examples

```
cellchat <- createCellChat(object = data.input, meta = meta, group.by = "labels"); cellchat <- computeCommunProb(cellchat, type = "trimeanMean", nboot = 100, population.size = FALSE)
```

## Evaluation signals

- Reproduction of the E14.5 embryonic skin reference result: 88 significant L-R interactions across 22 signaling pathways, including WNT, ncWNT, TGFβ, PDGF, NGF, FGF, and SEMA3
- Wnt6-Fzd10/Lrp6 and Wnt6-Fzd2/Lrp6 emerge as dominant canonical WNT pairs and FIB-E-derived ncWNT signaling is present
- All significant interactions pass the permutation test with M=100 label shufflings at p<0.05
- Only signaling genes passing the Wilcoxon rank sum test at significance level 0.05 contribute interactions
- Signaling role metrics (out-degree, in-degree, flow betweenness, information centrality) are computed for every cell group/state in the network

## Limitations

- Communication probability only represents interaction strength and is not exactly a probability
- Ligand-receptor interactions and mass action happen at the protein level, but mRNA levels are used to approximate protein abundance
- Hill function parameters Kh and n lack explicit biological connections; the Hill function is only a nonlinear approximation
- No ground truth exists for systematic evaluation of predicted cell-cell communication networks

## Evidence

- [discussion] law of mass action modeling: "We modeled ligand-receptor mediated signaling interactions using the law of mass action."
- [discussion] trimean ensemble average expression: "we calculated the ensemble average expression of signaling genes in a given cell group using a statistically robust mean method"
- [discussion] permutation test for significant interactions: "The significant interactions between two cell groups are identified using a permutation test by randomly permuting the group labels of cells"
- [methods] createCellChat function signature: "createCellChat(object, meta, group.by, datatype, coordinates, spatial.factors, assay, do.sparse)"
- [intro] CellChat infers communication with mass action models: "CellChat infers cell-state specific signaling communications within a given scRNA-seq data using mass action models"
