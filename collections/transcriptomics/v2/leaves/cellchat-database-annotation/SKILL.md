---
name: cellchat-database-annotation
description: Use when scRNA-seq cell groups have been clustered and differentially
  expressed signaling genes identified, and you need to map over-expressed ligands/receptors
  onto a curated pathway-level interaction database before computing communication
  probabilities.
when_to_use_negative:
- When the study organism is not mouse (or has not been mapped to mouse orthologues,
  e.g. via biomaRt), since CellChatDB is curated for mouse signaling
- When you need ligand-receptor pairs absent from KEGG-based curation or literature-supported
  interactions — CellChatDB is manually curated and may lack novel or poorly annotated
  interactions
- When the input is spatial transcriptomics data requiring single-cell-resolution
  inference — use Spatial CellChat (CellChat v3) instead of standard CellChatDB annotation
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0128
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0602
tools:
- name: CellChat
  role: Core R package providing createCellChat, identifyOverExpressedGenes, identifyOverExpressedInteractions,
    computeCommunProb, computeCommunProbPathway and filterCommunication for database-annotated
    interaction inference
  repo: https://github.com/sqjin/CellChat
- name: CellChatDB
  role: Manually curated signaling interaction database (2,021 validated molecular
    interactions) against which over-expressed interactions are annotated
- name: CellChat Explorer
  role: Web-based browser for exploring CellChatDB and inferred communication outputs
- name: biomaRt
  role: Orthologue mapping tool to convert human gene identifiers to mouse for CellChatDB
    compatibility
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-021-21246-9
    title: Inference and analysis of cell-cell communication using CellChat
  source_content_hash: sha256:56a86d9e46f2ee52b821273b3e0e71fa902d034c9e496e982269fb4d179e13b8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sqjin/CellChat
  tools:
  - CellChat
  - CellChat Explorer
  - CellChatDB
  - Seurat
  - biomaRt
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0128
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7857142857142857
n_claims: 14
---

# cellchat-database-annotation

## Summary

Annotate inferred cell-cell communication against CellChatDB, a manually curated mouse signaling database of 2,021 validated molecular interactions that accounts for heteromeric ligand-receptor complexes and cofactors, to assign predicted interactions to signaling pathways.

## When to use

Use when scRNA-seq cell groups have been clustered and differentially expressed signaling genes identified, and you need to map over-expressed ligands/receptors onto a curated pathway-level interaction database before computing communication probabilities.

## When NOT to use

- When the study organism is not mouse (or has not been mapped to mouse orthologues, e.g. via biomaRt), since CellChatDB is curated for mouse signaling
- When you need ligand-receptor pairs absent from KEGG-based curation or literature-supported interactions — CellChatDB is manually curated and may lack novel or poorly annotated interactions
- When the input is spatial transcriptomics data requiring single-cell-resolution inference — use Spatial CellChat (CellChat v3) instead of standard CellChatDB annotation

## Inputs

- Seurat scRNA-seq count matrix with cell-level metadata and cluster labels (e.g. placode, pre-DC, DC1, DC2 groups from E14.5 embryonic skin)
- CellChatDB interaction database (secreted signaling, ECM-receptor, and cell-cell contact sub-databases)
- Over-expressed gene list per cell group from identifyOverExpressedGenes

## Outputs

- CellChat object with annotated ligand-receptor interactions and pathway assignments
- Significant ligand-receptor pair list at p<0.05 (e.g. 21 placode-to-dermal-condensate pairs spanning WNT, TGFbeta, SEMA3, PTN, PDGF, MK and FGF pathways)
- Pathway-level communication probability matrix from computeCommunProbPathway

## How to apply

After constructing the CellChat object with createCellChat on the Seurat count matrix and metadata, run identifyOverExpressedGenes (Wilcoxon rank sum test, significance level 0.05) per cell group, then run identifyOverExpressedInteractions against CellChatDB so that expressed ligands and receptors are matched to curated interactions rather than raw gene-pair guesses. CellChatDB (built on KEGG Pathway database entries) encodes multi-subunit complexes and cofactors, so a receptor is considered expressed only when all subunits are detected, avoiding the one-ligand/one-receptor simplification of earlier tools. Communication probabilities are then computed with computeCommunProb using the law-of-mass-action/Hill model and a permutation test (M=100 label shuffles, p<0.05), aggregated with computeCommunProbPathway, and filtered with filterCommunication. If CellChatDB is outdated, update it via updateCellChatDB; optionally restrict to secreted signaling or use biomaRt to map human genes to mouse orthologues. Outputs are pathway-level communication networks that feed netAnalysis_computeCentrality for sender/receiver/mediator/influencer classification.

## Related tools

- **CellChat** (Core R package providing createCellChat, identifyOverExpressedGenes, identifyOverExpressedInteractions, computeCommunProb, computeCommunProbPathway and filterCommunication for database-annotated interaction inference) — https://github.com/sqjin/CellChat
- **CellChatDB** (Manually curated signaling interaction database (2,021 validated molecular interactions) against which over-expressed interactions are annotated)
- **CellChat Explorer** (Web-based browser for exploring CellChatDB and inferred communication outputs)
- **biomaRt** (Orthologue mapping tool to convert human gene identifiers to mouse for CellChatDB compatibility)

## Examples

```
cellchat <- createCellChat(object, meta = meta, group.by = "cell_state"); cellchat@DB <- CellChatDB; cellchat <- identifyOverExpressedGenes(cellchat); cellchat <- identifyOverExpressedInteractions(cellchat); cellchat <- computeCommunProb(cellchat)
```

## Evaluation signals

- Significant interactions identified exclusively via the default permutation test (M=100 group label shuffles) at p<0.05
- Pathway-level aggregation recovers expected pathways — in the embryonic skin case, exactly 21 placode-to-DC ligand-receptor pairs spanning WNT, TGFbeta, SEMA3, PTN, PDGF, MK and FGF, with Pros1-Axl and Edn3-Ednrb called significant
- All receptor subunits of heteromeric complexes are expressed in the receiver population (CellChatDB accounts for multi-subunit composition), otherwise the interaction should not be retained
- Differential expression step reproduces Wilcoxon rank sum test results at significance level 0.05 across all cell groups before interaction annotation

## Limitations

- mRNA levels are used to approximate protein levels, so predicted ligand-receptor interactions may not reflect actual protein-level signaling
- Communication probability represents interaction strength only and is not exactly a probability
- No ground truth exists for systematic evaluation of predicted communication networks; similar methods tend to generate similar results regardless of accuracy
- Hill-function parameters Kh and n lack explicit biological connections and are used only as a nonlinear approximation of ligand-receptor interactions

## Evidence

- [intro] CellChatDB is a manually curated database accounting for ligand-receptor complex composition and cofactors, containing 2021 validated molecular interactions: "CellChatDB contains 2,021 validated molecular interactions, including 60% of paracrine/autocrine signaling interactions, 21% of extracellular matrix (ECM)-receptor interactions"
- [discussion] Differential expression precedes interaction identification at Wilcoxon 0.05: "we first identified differentially expressed signaling genes across all cell groups within a given scRNA-seq dataset, using the Wilcoxon rank sum test with the significance level of 0.05."
- [discussion] Permutation test parameters for significance calling: "The significant interactions between two cell groups are identified using a permutation test by randomly permuting the group labels of cells"
- [discussion] Law-of-mass-action model for communication probability: "We modeled ligand-receptor mediated signaling interactions using the law of mass action."
- [discussion] CellChatDB grounded in KEGG curation: [evidence span withheld — anchor status: not_found]
