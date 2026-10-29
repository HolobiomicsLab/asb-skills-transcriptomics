---
name: permutation-testing-of-cell-group-interactions
description: Use when you have a CellChat object with overexpressed signaling genes
  across cell groups and need to determine which predicted ligand-receptor interactions
  are statistically significant rather than chance-level.
when_to_use_negative:
- When the input is spatial transcriptomics data requiring single-cell-resolution
  inference — use Spatial CellChat (CellChat v3) instead, since v1/v2 permutation
  testing was developed for discrete cell groups in scRNA-seq
- When ground-truth validation of communication networks is required — the article
  notes systematic evaluation is impossible due to lack of ground truth, so the permutation
  test only controls label-randomization error, not biological accuracy
- When proteomics-level ligand-receptor evidence is needed — mRNA levels are used
  to approximate protein levels, so a significant interaction is not direct evidence
  of protein-level signaling
edam_operation: http://edamontology.org/operation_3656
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0602
tools:
- name: CellChat
  role: R package implementing computeCommunProb (Hill-function mass action model),
    the label-shuffling permutation test (M=100, p<0.05), filterCommunication, computeCommunProbPathway,
    and netAnalysis_computeCentrality
  repo: https://github.com/sqjin/CellChat
- name: createCellChat
  role: Constructs the CellChat object from filtered scRNA-seq counts and metadata
    with the mouse CellChatDB in secreted-signaling label-free mode prior to the permutation
    test
  repo: https://github.com/jinworks/CellChat
- name: computeCommunProb
  role: Computes ligand-receptor communication probabilities using the Hill function
    mass action model; the function whose permutation-based significance testing is
    the core of this skill
  repo: https://github.com/jinworks/CellChat
- name: CellChatDB
  role: Manually curated ligand-receptor/cofactor database (2021 interactions, KEGG-derived)
    supplying the interaction space tested by the permutation procedure
  repo: https://github.com/sqjin/CellChat
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-021-21246-9
    title: Inference and analysis of cell-cell communication using CellChat
  source_content_hash: sha256:74999d444ef1863d83930b0397de36930dc1460e314a42bc2c488f3ae0a6a09f
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
  edam_operation: http://edamontology.org/operation_3656
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6428571428571429
n_claims: 14
---

# Permutation testing of cell-group interactions

## Summary

Identify statistically significant ligand-receptor interactions between cell groups or cell states by randomly permuting cell group labels and recomputing communication probabilities, keeping interactions with p < 0.05. This is CellChat's null-model approach for filtering inferred cell-cell communication networks in scRNA-seq data.

## When to use

Use when you have a CellChat object with overexpressed signaling genes across cell groups and need to determine which predicted ligand-receptor interactions are statistically significant rather than chance-level.

## When NOT to use

- When the input is spatial transcriptomics data requiring single-cell-resolution inference — use Spatial CellChat (CellChat v3) instead, since v1/v2 permutation testing was developed for discrete cell groups in scRNA-seq
- When ground-truth validation of communication networks is required — the article notes systematic evaluation is impossible due to lack of ground truth, so the permutation test only controls label-randomization error, not biological accuracy
- When proteomics-level ligand-receptor evidence is needed — mRNA levels are used to approximate protein levels, so a significant interaction is not direct evidence of protein-level signaling

## Inputs

- CellChat object built from QC-filtered scRNA-seq count matrix with cell group labels (e.g., E14.5 embryonic mouse skin counts from GSM3453537/GSM3453538; QC: UMI 2500-50000, genes >= 1000, mitochondrial fraction < 20%)
- Curated ligand-receptor interaction database (CellChatDB, mouse, 2021 validated interactions, secreted-signaling subset for label-free mode)
- Cell metadata with group/pseudotime assignments

## Outputs

- Table of significant ligand-receptor interactions with communication probabilities and permutation p-values (e.g., 88 significant interactions across 22 pathways for E14.5 embryonic skin, written to cellchat_e145_significant_interactions.csv)
- Pathway-level aggregated communication network (computeCommunProbPathway)
- Centrality scores identifying dominant senders, receivers, mediators, and influencers (netAnalysis_computeCentrality)

## How to apply

After creating a CellChat object (e.g., with createCellChat on filtered embryonic skin scRNA-seq counts using the mouse CellChatDB in secreted-signaling label-free mode) and running identifyOverExpressedGenes/identifyOverExpressedInteractions, call computeCommunProb to model ligand-receptor communication probability with the Hill function under the law of mass action, where ensemble average expression of signaling genes in each cell group is computed as a statistically robust trimean of quartiles. Significance is then assessed via a permutation test that randomly permutes (shuffles) the cell group labels of cells and recomputes communication probabilities M times (M=100 by default), retaining interactions at p < 0.05. Downstream, use filterCommunication to prune the network, computeCommunProbPathway to aggregate L-R pairs to the pathway level, and netAnalysis_computeCentrality to assign sender/receiver/mediator/influencer roles along pseudotime trajectories. The rationale is that shuffling group labels destroys genuine cell-type-restricted communication structure, so the observed communication probability must exceed the label-permuted null distribution to be considered biologically meaningful.

## Related tools

- **CellChat** (R package implementing computeCommunProb (Hill-function mass action model), the label-shuffling permutation test (M=100, p<0.05), filterCommunication, computeCommunProbPathway, and netAnalysis_computeCentrality) — https://github.com/sqjin/CellChat
- **createCellChat** (Constructs the CellChat object from filtered scRNA-seq counts and metadata with the mouse CellChatDB in secreted-signaling label-free mode prior to the permutation test) — https://github.com/jinworks/CellChat
- **computeCommunProb** (Computes ligand-receptor communication probabilities using the Hill function mass action model; the function whose permutation-based significance testing is the core of this skill) — https://github.com/jinworks/CellChat
- **CellChatDB** (Manually curated ligand-receptor/cofactor database (2021 interactions, KEGG-derived) supplying the interaction space tested by the permutation procedure) — https://github.com/sqjin/CellChat

## Examples

```
cellchat <- createCellChat(data, meta = meta, group.by = "celltype"); cellchat@DB <- CellChatDB$secreted; cellchat <- identifyOverExpressedGenes(cellchat); cellchat <- computeCommunProb(cellchat, type = "trimean", nboot = 100, population.size = TRUE)
```

## Evaluation signals

- Reproduction of the E14.5 embryonic skin result: exactly 88 significant ligand-receptor interactions across 22 signaling pathways, including WNT, ncWNT, TGFβ, PDGF, NGF, FGF, and SEMA3
- Dominant canonical WNT pairs identified as Wnt6-Fzd10/Lrp6 and Wnt6-Fzd2/Lrp6, with FIB-E-derived ncWNT signaling present
- Significance table exported correctly to cellchat_e145_significant_interactions.csv with p-values all < 0.05 from the M=100 label-permutation null
- Permutation parameters match the default: cell group labels randomly permuted M=100 times and interactions retained only at p < 0.05
- Network passes through filterCommunication and aggregates cleanly to the pathway level via computeCommunProbPathway without orphan interactions

## Limitations

- Communication probability only represents interaction strength and is not exactly a probability
- Ligand-receptor interactions happen at the protein level but mRNA levels are used as a proxy; lack of single-cell proteomics makes comprehensive modeling challenging
- Hill-function parameters Kh and n lack explicit biological connections; the Hill function is only a nonlinear approximation of ligand-receptor interactions
- No ground truth exists for systematic evaluation of predicted cell-cell communication networks, so permutation significance cannot be equated with accuracy
- Different ligand-receptor databases across methods (CellChat vs. SingleCellSignalR, iTALK, NicheNet, CellPhoneDB) confound cross-method comparison, and similar methods can generate similar overlaps regardless of accuracy

## Evidence

- [discussion] Permutation test definition: "The significant interactions between two cell groups are identified using a permutation test by randomly permuting the group labels of cells"
- [discussion] Mass action model: "We modeled ligand-receptor mediated signaling interactions using the law of mass action."
- [discussion] Trimean ensemble averaging: "we calculated the ensemble average expression of signaling genes in a given cell group using a statistically robust mean method"
- [discussion] Wilcoxon overexpression screen: "we first identified differentially expressed signaling genes across all cell groups within a given scRNA-seq dataset, using the Wilcoxon rank sum test with the significance level of 0.05."
- [discussion] Embryonic skin QC thresholds: "we removed the cells with the amount of UMI count less than 2500 and greater than 50000, as well as the cells with the number of genes less than 1000 and the fraction of mitochondrial counts greater"
- [full_text] Permutation count and threshold: [evidence span withheld — anchor status: not_found]
