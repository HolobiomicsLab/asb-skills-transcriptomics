---
name: canonical-wnt-signaling-pathway-annotation
description: Use when CellChat-predicted signaling pathways for scRNA-seq cell groups
  include WNT, and the canonical WNT ligand-receptor pairs and pathway-level aggregates
  must be extracted, ranked by communication probability, and validated against known
  dominant pairs.
when_to_use_negative:
- Input is spatial transcriptomics data where single-cell-resolution inference via
  CellChat v3 (SpatialCellChat) is required instead of the label-free scRNA-seq workflow
- The dataset lacks cell group/cluster annotations, since CellChat's permutation test
  operates on shuffled cell group labels and communication inference requires pre-defined
  groups
- Non-mouse data without prior gene-to-mouse-orthologue mapping (e.g., via biomaRt),
  since the mouse CellChatDB cannot be applied directly
edam_operation: http://edamontology.org/operation_3929
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: CellChat
  role: R package that infers WNT ligand-receptor communication probabilities and
    aggregates them to the signaling-pathway level
  repo: https://github.com/sqjin/CellChat
- name: CellChatDB
  role: Manually curated mouse ligand-receptor database (2,021 validated interactions)
    from which WNT pathway pairs are drawn
  repo: https://github.com/sqjin/CellChat
- name: computeCommunProb
  role: Computes WNT ligand-receptor communication probability via the Hill-function
    mass action model and significance via permutation test
  repo: https://github.com/sqjin/CellChat
- name: computeCommunProbPathway
  role: Aggregates individual L-R interactions (e.g., Wnt6-Fzd10/Lrp6, Wnt6-Fzd2/Lrp6)
    to the canonical WNT pathway level
  repo: https://github.com/sqjin/CellChat
- name: biomaRt
  role: Maps human gene identifiers to mouse orthologues so human datasets can use
    the mouse CellChatDB
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-021-21246-9
    title: Inference and analysis of cell-cell communication using CellChat
  source_content_hash: sha256:3f3f94a7f47f464bd8b3c99f71fe4436576f9dcf2c577592f216854741ae5efc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sqjin/CellChat
  tools:
  - CellChat
  - CellChatDB
  - biomaRt
  - computeCommunProb
  - computeCommunProbPathway
  - createCellChat
  - filterCommunication
  - netAnalysis_computeCentrality
  edam_operation: http://edamontology.org/operation_3929
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 15
---

# canonical-wnt-signaling-pathway-annotation

## Summary

Annotates and validates canonical WNT signaling calls within CellChat cell-cell communication inference, identifying dominant WNT ligand-receptor pairs (e.g., Wnt6-Fzd10/Lrp6, Wnt6-Fzd2/Lrp6) and distinguishing them from non-canonical WNT (ncWNT) signaling along pseudotemporal embryonic skin trajectories.

## When to use

Use when CellChat-predicted signaling pathways for scRNA-seq cell groups include WNT, and the canonical WNT ligand-receptor pairs and pathway-level aggregates must be extracted, ranked by communication probability, and validated against known dominant pairs.

## When NOT to use

- Input is spatial transcriptomics data where single-cell-resolution inference via CellChat v3 (SpatialCellChat) is required instead of the label-free scRNA-seq workflow
- The dataset lacks cell group/cluster annotations, since CellChat's permutation test operates on shuffled cell group labels and communication inference requires pre-defined groups
- Non-mouse data without prior gene-to-mouse-orthologue mapping (e.g., via biomaRt), since the mouse CellChatDB cannot be applied directly

## Inputs

- QC-filtered scRNA-seq count matrix (e.g., GSM3453537, GSM3453538 from GEO) with cell group/cluster metadata
- Mouse CellChatDB (secreted-signaling / label-free mode)
- Pseudotemporal cell-state labels for continuous dermal and epidermal trajectories

## Outputs

- Pathway-level WNT communication probability matrix (output of computeCommunProbPathway)
- Significant ligand-receptor interaction table including canonical WNT pairs (e.g., cellchat_e145_significant_interactions.csv)
- Annotated set of signaling pathways (e.g., 88 interactions across 22 pathways) with WNT/ncWNT labels

## How to apply

Build a CellChat object from QC-filtered mouse scRNA-seq counts (embryonic skin filters: UMI 2500-50000, genes >= 1000, mitochondrial fraction < 20%) with the mouse CellChatDB in secreted-signaling (label-free) mode via createCellChat. After identifyOverExpressedGenes (Wilcoxon rank sum test, significance 0.05), compute ligand-receptor communication probabilities with computeCommunProb using the Hill-function mass action model with trimean-of-quartiles ensemble average expression, and call significant interactions via permutation test shuffling cell group labels with M=100 permutations at p<0.05. Aggregate to the signaling-pathway level with computeCommunProbPathway so canonical WNT pairs (e.g., Wnt6-Fzd10/Lrp6 and Wnt6-Fzd2/Lrp6) and FIB-E-derived ncWNT signaling can be distinguished within the pathway-level network. Validate the annotation by confirming the expected number of significant interactions (e.g., 88 L-R interactions across 22 pathways including WNT, ncWNT, TGFβ, PDGF, NGF, FGF, SEMA3 in the E14.5 skin trajectory) and export the significant interaction table to CSV.

## Related tools

- **CellChat** (R package that infers WNT ligand-receptor communication probabilities and aggregates them to the signaling-pathway level) — https://github.com/sqjin/CellChat
- **CellChatDB** (Manually curated mouse ligand-receptor database (2,021 validated interactions) from which WNT pathway pairs are drawn) — https://github.com/sqjin/CellChat
- **computeCommunProb** (Computes WNT ligand-receptor communication probability via the Hill-function mass action model and significance via permutation test) — https://github.com/sqjin/CellChat
- **computeCommunProbPathway** (Aggregates individual L-R interactions (e.g., Wnt6-Fzd10/Lrp6, Wnt6-Fzd2/Lrp6) to the canonical WNT pathway level) — https://github.com/sqjin/CellChat
- **biomaRt** (Maps human gene identifiers to mouse orthologues so human datasets can use the mouse CellChatDB)

## Examples

```
library(CellChat); cellchat <- createCellChat(object, meta=meta, group.by="cellstate"); cellchat <- computeCommunProb(cellchat); cellchat <- computeCommunProbPathway(cellchat)
```

## Evaluation signals

- Reproduced counts: 88 significant ligand-receptor interactions across 22 signaling pathways in the E14.5 embryonic skin pseudotemporal trajectory
- Wnt6-Fzd10/Lrp6 and Wnt6-Fzd2/Lrp6 emerge as the dominant canonical WNT pairs in the significant interaction table
- FIB-E-derived ncWNT signaling is present and separable from canonical WNT at the pathway level
- All retained WNT interactions pass the permutation test (M=100 group-label shuffles) at p<0.05
- Signaling genes identified by Wilcoxon rank sum test at significance level 0.05 include the WNT ligands/receptors used downstream

## Limitations

- mRNA levels are used to approximate protein levels, so inferred WNT signaling may not reflect protein-level ligand-receptor engagement
- There is no ground truth for systematic evaluation of the predicted WNT communication network
- The communication probability represents interaction strength and is not exactly a probability
- Hill-function parameters Kh and n lack explicit biological connections and are only a nonlinear approximation of ligand-receptor interactions

## Evidence

- [discussion] Modeling WNT communication via mass action: "We modeled ligand-receptor mediated signaling interactions using the law of mass action."
- [discussion] Permutation significance for WNT pairs: "The significant interactions between two cell groups are identified using a permutation test by randomly permuting the group labels of cells"
- [discussion] Trimean ensemble expression: "we calculated the ensemble average expression of signaling genes in a given cell group using a statistically robust mean method"
- [intro] CellChatDB scope: "CellChatDB contains 2,021 validated molecular interactions, including 60% of paracrine/autocrine signaling interactions"
- [intro] Handling continuous pseudotime states: "CellChat infers cell-state specific signaling communications within a given scRNA-seq data using mass action models"
- [methods] CellChat installation: "CellChat v1/v2 R package can be easily installed from Github using devtools: devtools::install_github("jinworks/CellChat")"
