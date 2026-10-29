---
name: scrnaseq-quality-control-filtering
description: Use when preprocessing 10X Genomics scRNA-seq count matrices (e.g., GEO
  samples GSM3453537/GSM3453538) before building a CellChat object, and cells must
  be filtered by UMI count, gene count, and mitochondrial fraction thresholds.
when_to_use_negative:
- The input is already a filtered/normalized expression matrix or a pre-built CellChat
  object, not raw cell-by-gene counts.
- The tissue or dataset has different expected mitochondrial or capture characteristics
  such that the embryonic-skin thresholds (UMI 2500–50000, genes >= 1000, mito < 20%)
  are biologically inappropriate.
- The goal is spatially resolved or proteomic data QC rather than scRNA-seq droplet
  data filtering.
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: CellChat
  role: Downstream R package whose createCellChat step consumes the QC-filtered count
    matrix; filtering quality directly shapes inferred ligand-receptor interactions
  repo: https://github.com/sqjin/CellChat
- name: createCellChat
  role: CellChat constructor function called on the QC-filtered data with mouse CellChatDB
    in secreted-signaling (label-free) mode
  repo: https://github.com/sqjin/CellChat
- name: updateCellChatDB
  role: Helper to keep the CellChatDB interaction database current before analysis
    of the filtered data
  repo: https://github.com/sqjin/CellChat
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-021-21246-9
    title: Inference and analysis of cell-cell communication using CellChat
  source_content_hash: sha256:b77f25d984950cbfb24f1a6ba7a078c6ebdf4c864b50a14d8ff66af6dad50dca
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sqjin/CellChat
  tools:
  - CellChat
  - computeCommunProb
  - createCellChat
  - filterCommunication
  - netAnalysis_computeCentrality
  - updateCellChatDB
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# scrnaseq-quality-control-filtering

## Summary

Applies the embryonic mouse skin scRNA-seq QC filters used before CellChat cell-cell communication inference — UMI count 2500–50000, number of genes >= 1000, and mitochondrial count fraction < 20% — so that downstream communication probabilities are computed on reliably captured cells.

## When to use

Use when preprocessing 10X Genomics scRNA-seq count matrices (e.g., GEO samples GSM3453537/GSM3453538) before building a CellChat object, and cells must be filtered by UMI count, gene count, and mitochondrial fraction thresholds.

## When NOT to use

- The input is already a filtered/normalized expression matrix or a pre-built CellChat object, not raw cell-by-gene counts.
- The tissue or dataset has different expected mitochondrial or capture characteristics such that the embryonic-skin thresholds (UMI 2500–50000, genes >= 1000, mito < 20%) are biologically inappropriate.
- The goal is spatially resolved or proteomic data QC rather than scRNA-seq droplet data filtering.

## Inputs

- Raw 10X Genomics scRNA-seq count matrices from GEO (e.g., GSM3453535–GSM3453538)
- Per-cell metadata (barcode-level UMI counts, detected gene counts, mitochondrial count fractions, cell group labels)

## Outputs

- Filtered count matrix with cells passing UMI 2500–50000, genes >= 1000, and mitochondrial fraction < 20%
- QC report of removed/retained cell counts per filter criterion, ready for createCellChat

## How to apply

Load raw count matrices from GEO (e.g., GSM3453537 and GSM3453538 for E14.5 embryonic mouse skin) and remove cells whose UMI count is less than 2500 or greater than 50000, cells with fewer than 1000 detected genes, and cells whose fraction of mitochondrial counts exceeds 20%. Apply analogous dataset-specific exclusions where warranted by the article's filter steps, such as removing red blood cells from the mouse skin wound dataset (21,557 of 21,819 cells retained) or dropping contaminated subclusters (CD3D-expressing T cell group from FIB subclustering; immune cell group lacking DC markers from DC subclustering). Feed the filtered matrix plus per-cell metadata (cell groups, pseudotime where applicable) into createCellChat so that identifyOverExpressedGenes, computeCommunProb, and downstream pathway aggregation operate on a cleaned cell population. Decide thresholds by the tissue's expected mitochondrial load — the 20% cutoff was chosen for embryonic skin — and document retained cell counts per filter step for reproducibility.

## Related tools

- **CellChat** (Downstream R package whose createCellChat step consumes the QC-filtered count matrix; filtering quality directly shapes inferred ligand-receptor interactions) — https://github.com/sqjin/CellChat
- **createCellChat** (CellChat constructor function called on the QC-filtered data with mouse CellChatDB in secreted-signaling (label-free) mode) — https://github.com/sqjin/CellChat
- **updateCellChatDB** (Helper to keep the CellChatDB interaction database current before analysis of the filtered data) — https://github.com/sqjin/CellChat

## Examples

```
cellchat <- createCellChat(object = filtered_counts, meta = cell_meta, group.by = "cell_group", datatype = "RNA")  # after filtering: UMI 2500-50000, genes >= 1000, mito < 20%
```

## Evaluation signals

- All retained cells satisfy UMI count 2500–50000, number of genes >= 1000, and mitochondrial count fraction < 20%.
- Downstream CellChat run on the filtered E14.5 embryonic skin data reproduces 88 significant ligand-receptor interactions across 22 signaling pathways, including WNT, ncWNT, TGFβ, PDGF, NGF, FGF, and SEMA3, with Wnt6-Fzd10/Lrp6 and Wnt6-Fzd2/Lrp6 as dominant canonical WNT pairs.
- Retention/removed counts per filter step are logged (e.g., red blood cell removal kept 21,557 of 21,829 cells in the wound dataset).
- The significant interaction table exported to cellchat_e145_significant_interactions.csv matches the reference finding, indicating the filter pipeline did not distort group composition.
- Contaminated populations flagged by marker checks (CD3D+ T cell group in FIB subclustering; immune group lacking DC markers) are excluded before communication inference.

## Limitations

- The cutoffs (UMI 2500–50000, genes >= 1000, mito < 20%) are embryonic-skin-specific; the article's other filter steps (contaminated cluster removal, consensus matrix pruning < 0.3, W/H matrix thresholding < 0.5) are dataset-specific decisions rather than universal QC rules.
- QC filtering alone does not guarantee correct communication inference: the article notes mRNA approximates protein levels and there is no ground truth for systematic evaluation of predicted cell-cell communication networks.
- Overly aggressive filtering changes cell group composition and thus permutation-test p-values (M=100, p<0.05), potentially altering which interactions are called significant.

## Evidence

- [discussion] Embryonic skin QC thresholds: "we removed the cells with the amount of UMI count less than 2500 and greater than 50000, as well as the cells with the number of genes less than 1000 and the fraction of mitochondrial counts greater"
- [discussion] Red blood cell removal from wound dataset: "we excluded red blood cells and used the remaining 21,557 cells."
- [discussion] Contaminated T cell removal in FIB subclustering: "This contaminated TC population was removed for further analysis."
- [discussion] No ground truth for communication prediction evaluation: "systematic evaluation of predicted cell–cell communication networks is challenging due to the lack of ground truth7."
- [methods] Downstream CellChat object creation from filtered data: "`createCellChat(object, meta, group.by, datatype, coordinates, spatial.factors, assay, do.sparse)`"
