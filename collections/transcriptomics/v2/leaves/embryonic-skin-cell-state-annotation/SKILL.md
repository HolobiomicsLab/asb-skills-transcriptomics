---
name: embryonic-skin-cell-state-annotation
description: Use when scRNA-seq count matrices from E14.5 embryonic skin must be split
  into spatially colocalized dermal states (placode, pre-DC, DC1, DC2) prior to cell-cell
  communication inference.
when_to_use_negative:
- Input is already a curated cell-state annotated object or a CellChat object — re-subclustering
  would destroy existing group labels
- Dataset contains only a handful of cells per dermal state, so Louvain subclustering
  restricted to cell types with abundance > 5% cannot resolve states
- The goal is de novo tissue annotation of non-skin tissue; the QC thresholds (UMI
  2500–50000, mito < 20%) are embryonic-skin specific
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: Seurat
  role: Load count matrix and metadata, run per-cell QC filtering, Louvain subclustering
    and UMAP of the dermal compartment
- name: CellChat
  role: Consumes the annotated cell groups via createCellChat and identifyOverExpressedGenes
    (Wilcoxon, 0.05) to infer placode-to-DC communication
  repo: https://github.com/sqjin/CellChat
- name: UMAP
  role: Visualize the subclustered dermal cell states in 2D
  repo: https://github.com/lmcinnes/umap
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-021-21246-9
    title: Inference and analysis of cell-cell communication using CellChat
  source_content_hash: sha256:ff180183618563415a2bd3a065aeb2252f6bea97e176243d919f2c26413de9bc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sqjin/CellChat
  tools:
  - CellChat
  - Seurat
  - UMAP
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# embryonic-skin-cell-state-annotation

## Summary

QC-filter, subcluster, and annotate the E14.5 embryonic skin dermal compartment into placode, pre-DC, DC1 and DC2 states so that downstream CellChat communication analysis operates on biologically meaningful cell groups.

## When to use

Use when scRNA-seq count matrices from E14.5 embryonic skin must be split into spatially colocalized dermal states (placode, pre-DC, DC1, DC2) prior to cell-cell communication inference.

## When NOT to use

- Input is already a curated cell-state annotated object or a CellChat object — re-subclustering would destroy existing group labels
- Dataset contains only a handful of cells per dermal state, so Louvain subclustering restricted to cell types with abundance > 5% cannot resolve states
- The goal is de novo tissue annotation of non-skin tissue; the QC thresholds (UMI 2500–50000, mito < 20%) are embryonic-skin specific

## Inputs

- E14.5 embryonic skin scRNA-seq count matrix (e.g. 10X Genomics output)
- Per-cell metadata (UMI counts, gene counts, mitochondrial fraction, cell type labels)
- Canonical marker gene panel for placode, pre-DC, DC1 and DC2 annotation

## Outputs

- QC-filtered Seurat object with placode, pre-DC, DC1 and DC2 state annotations
- Cell group label table feeding createCellChat grouping
- UMAP embedding of the subclustered dermal compartment

## How to apply

Load the E14.5 scRNA-seq count matrix and metadata in Seurat and apply embryonic skin QC thresholds: UMI count 2500–50000, genes ≥ 1000, mitochondrial fraction < 20%; also remove red blood cells and any contaminated groups flagged during subclustering (e.g. CD3D-expressing T cells, or immune cells lacking DC markers within the DC compartment). Subcluster the dermal cell compartment with Louvain community detection restricted to cell types with abundance > 5%, then assign each state as placode, pre-DC, DC1 or DC2 from canonical marker expression. Use the annotated cell group labels directly as the grouping input for CellChat's createCellChat and identifyOverExpressedGenes (Wilcoxon rank sum test, significance level 0.05). Rationale: CellChat requires pre-clustered, biologically annotated cell groups, and contaminated subclusters would distort ligand-receptor communication probabilities computed under the mass-action/Hill model.

## Related tools

- **Seurat** (Load count matrix and metadata, run per-cell QC filtering, Louvain subclustering and UMAP of the dermal compartment)
- **CellChat** (Consumes the annotated cell groups via createCellChat and identifyOverExpressedGenes (Wilcoxon, 0.05) to infer placode-to-DC communication) — https://github.com/sqjin/CellChat
- **UMAP** (Visualize the subclustered dermal cell states in 2D) — https://github.com/lmcinnes/umap

## Examples

```
library(Seurat); obj <- subset(skin_obj, nCount_RNA >= 2500 & nCount_RNA <= 50000 & nFeature_RNA >= 1000 & percent.mt < 20); skin_obj <- FindClusters(skin_obj, resolution = res)
```

## Evaluation signals

- All retained cells fall within QC bounds: UMI 2500–50000, genes ≥ 1000, mitochondrial fraction < 20%, with red blood cells and contaminated non-DC immune groups removed
- Subclustering yields exactly the four colocalized states (placode, pre-DC, DC1, DC2), each backed by canonical marker expression
- Only cell types with abundance > 5% enter Louvain subclustering, matching the published dermal-compartment strategy
- Downstream CellChat run on these labels recovers placode as dominant hub secreting via 44 and receiving via 19 ligand-receptor pairs, with 21 placode-to-DC pairs spanning WNT, TGFbeta, SEMA3, PTN, PDGF, MK and FGF pathways
- Marker-based annotations are consistent with Pros1-Axl and Edn3-Ednrb emerging as significant placode-to-dermal-condensate interactions at p<0.05

## Limitations

- Subclustering strategy excludes cell types with abundance ≤ 5%, so rare dermal states cannot be resolved as separate groups
- Contaminated subclusters (CD3D+ T cells inside FIB, immune cells lacking DC markers inside DC) must be manually identified and removed or they corrupt downstream communication inference
- QC thresholds are dataset-specific to embryonic skin and were not validated for adult or wound skin samples

## Evidence

- [discussion] Subclustering restricted to abundant cell types: "we performed subclustering analysis on the cell types, whose abundance in the dataset was greater than 5% using the Louvain community detection method."
- [discussion] Embryonic skin QC thresholds: "we removed the cells with the amount of UMI count less than 2500 and greater than 50000, as well as the cells with the number of genes less than 1000 and the fraction of mitochondrial counts greater"
- [discussion] Contaminated immune group removal: "This contaminated immune cell group was also removed for further analysis."
- [discussion] CellChat requires pre-clustered groups: "Cell clustering is a pre-requisite for cell–cell communication analysis with CellChat and other tools, such as CellPhoneDB, iTALK and SingleCellSignalR."
- [readme] CellChat installation from GitHub: "CellChat v1/v2 R package can be easily installed from Github using devtools: devtools::install_github("jinworks/CellChat")"
