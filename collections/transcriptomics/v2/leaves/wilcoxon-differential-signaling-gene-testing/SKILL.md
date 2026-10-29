---
name: wilcoxon-differential-signaling-gene-testing
description: 'Use when preparing scRNA-seq data for CellChat communication inference:
  clustered cell groups exist and over-expressed signaling genes must be identified
  before computing ligand-receptor communication probabilities.'
when_to_use_negative:
- Cell groups are not yet defined (cell clustering is a pre-requisite for CellChat
  analysis)
- The goal is differential expression of all genes, not specifically signaling genes
  curated in a ligand-receptor database
- Data are spatial transcriptomics at single-cell resolution where Spatial CellChat
  (CellChat v3) with spatial constraints is more appropriate
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0602
tools:
- name: CellChat
  role: R package providing identifyOverExpressedGenes and identifyOverExpressedInteractions
    for the Wilcoxon-based differential signaling gene testing and downstream communication
    probability inference
  repo: https://github.com/sqjin/CellChat
- name: CellChatDB
  role: Manually curated database of signaling molecule interactions (KEGG-based,
    multimeric complexes and cofactors) defining which genes are tested
  repo: https://github.com/sqjin/CellChat
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-021-21246-9
    title: Inference and analysis of cell-cell communication using CellChat
  source_content_hash: sha256:0632f7fc350aea1c3dd3facb095754096965d3c9de9d471f0ed703e7f6a35fdc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sqjin/CellChat
  tools:
  - CellChat
  - CellChatDB
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# wilcoxon-differential-signaling-gene-testing

## Summary

Identify differentially expressed signaling genes (ligands, receptors, cofactors) across cell groups in an scRNA-seq dataset using the Wilcoxon rank sum test at significance level 0.05, as the first statistical filter in the CellChat cell-cell communication inference pipeline prior to ligand-receptor interaction inference.

## When to use

Use when preparing scRNA-seq data for CellChat communication inference: clustered cell groups exist and over-expressed signaling genes must be identified before computing ligand-receptor communication probabilities.

## When NOT to use

- Cell groups are not yet defined (cell clustering is a pre-requisite for CellChat analysis)
- The goal is differential expression of all genes, not specifically signaling genes curated in a ligand-receptor database
- Data are spatial transcriptomics at single-cell resolution where Spatial CellChat (CellChat v3) with spatial constraints is more appropriate

## Inputs

- CellChat object constructed with createCellChat from a preprocessed scRNA-seq dataset (e.g., Seurat/SingleCellExpression object) with cell group labels
- Curated ligand-receptor interaction database (CellChatDB, mouse label-based mode; 2,021 validated molecular interactions)

## Outputs

- Table of over-expressed signaling genes per cell group (Wilcoxon rank sum test, p < 0.05)
- Filtered set of candidate ligand-receptor pairs per cell-group pair (from identifyOverExpressedInteractions), feeding computeCommunProb

## How to apply

Run identifyOverExpressedGenes on the CellChat object to test each signaling gene (from the curated CellChatDB, which contains 2,021 validated molecular interactions) for differential expression across all cell groups using the Wilcoxon rank sum test with significance level 0.05. The test flags which ligands, receptors, and cofactors are plausibly active in each cell group, restricting the search space for downstream interaction inference. Then call identifyOverExpressedInteractions to keep only ligand-receptor pairs whose partners passed the expression test. This filtering step precedes computeCommunProb, which models ligand-receptor interactions via a Hill function under the law of mass action with a permutation test (nboot=100) that randomly permutes cell group labels. As a reference scale, applying this workflow to the day-12 mouse skin wound dataset (GSE113854, 21,557 cells, red blood cells excluded) yields 60 significant ligand-receptor pairs aggregated into 25 signaling pathways.

## Related tools

- **CellChat** (R package providing identifyOverExpressedGenes and identifyOverExpressedInteractions for the Wilcoxon-based differential signaling gene testing and downstream communication probability inference) — https://github.com/sqjin/CellChat
- **CellChatDB** (Manually curated database of signaling molecule interactions (KEGG-based, multimeric complexes and cofactors) defining which genes are tested) — https://github.com/sqjin/CellChat

## Examples

```
cellchat <- createCellChat(object=data, meta=meta, group.by="labels"); cellchat@DB <- CellChatDB; cellchat <- identifyOverExpressedGenes(cellchat); cellchat <- identifyOverExpressedInteractions(cellchat)
```

## Evaluation signals

- Over-expressed gene table contains per-group Wilcoxon rank sum test statistics with significance level 0.05 applied
- Downstream filtered network reproduces the reference: 60 significant ligand-receptor pairs across 25 signaling pathways (TGFβ, ncWNT, TNF, SPP1, PTN, PDGF, CXCL, CCL, MIF, etc.) on the day-12 mouse skin wound dataset (GSE113854, 21,557 cells)
- No candidate interactions are inferred for ligand-receptor pairs whose genes failed the 0.05 significance filter
- Permutation test (nboot=100, cell group labels randomly permuted) yields p < 0.05 for all retained interactions

## Limitations

- mRNA levels are used to approximate protein levels, whereas ligand-receptor interactions occur at the protein level; single-cell proteomic information is technically difficult to capture
- No ground truth exists for systematic evaluation of predicted cell-cell communication networks
- Different ligand-receptor databases across methods confound cross-method performance comparison

## Evidence

- [discussion] Wilcoxon differential signaling gene identification at 0.05: "we first identified differentially expressed signaling genes across all cell groups within a given scRNA-seq dataset, using the Wilcoxon rank sum test with the significance level of 0.05."
- [discussion] Permutation test for significant interactions: "The significant interactions between two cell groups are identified using a permutation test by randomly permuting the group labels of cells"
- [intro] CellChatDB content: "CellChatDB contains 2,021 validated molecular interactions, including 60% of paracrine/autocrine signaling interactions"
- [discussion] Reference result on day-12 skin wound data: "we excluded red blood cells and used the remaining 21,557 cells."
- [readme] install CellChat: "CellChat v1/v2 R package can be easily installed from Github using devtools: devtools::install_github("jinworks/CellChat")"
