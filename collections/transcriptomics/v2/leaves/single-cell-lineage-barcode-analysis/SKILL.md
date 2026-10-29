---
name: single-cell-lineage-barcode-analysis
description: Use when you have paired single-cell lineage barcode (character matrix)
  and scRNA-seq expression data with per-cell type labels, and need to reconstruct
  and biologically validate a cell lineage tree by comparing barcode-defined subtrees
  against transcriptomic cell types.
when_to_use_negative:
- Lineage and expression data are unpaired (barcodes from different cells than the
  scRNA-seq profiles), since LinRace's asymmetric division model requires joint paired
  data
- The biological system has largely reversible cell state transitions, which LinRace's
  irreversible-transition assumption does not model
- The goal is de novo cell type annotation rather than lineage validation — this skill
  assumes cell type labels already exist
tools:
- name: LinRace
  role: Reconstructs the cell division tree from paired barcode and expression data
    via NJ_asym_Dif under the asymmetric cell division model; package also supports
    tree loading and subtree manipulation
  repo: github.com/ZhangLabGT/LinRace
- name: ape
  role: Part of the LinRace/ape R toolchain used to load the Newick tree and cut it
    at a fixed depth into subtrees
- name: ggplot2
  role: Visualization of cell-type composition across subtrees
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-023-44173-3
    title: 'LinRace: cell division history reconstruction of single cells using paired
      lineage barcode and gene expression data'
  source_content_hash: sha256:def7bacfd6bfbbffe5214d447cd0f6fcf9fc35fc8ba54d49e4d06422a34a0606
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - LinRace
  - R
  - ape
  - ggplot2
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# single-cell-lineage-barcode-analysis

## Summary

Reconstruct a cell division tree from paired lineage barcode and gene expression data with LinRace, then validate it by cutting the tree at a fixed depth and tabulating annotated cell-type composition within each subtree. Partial mixing of cell types across subtrees indicates consistency between transcriptome similarity and barcode similarity.

## When to use

Use when you have paired single-cell lineage barcode (character matrix) and scRNA-seq expression data with per-cell type labels, and need to reconstruct and biologically validate a cell lineage tree by comparing barcode-defined subtrees against transcriptomic cell types.

## When NOT to use

- Lineage and expression data are unpaired (barcodes from different cells than the scRNA-seq profiles), since LinRace's asymmetric division model requires joint paired data
- The biological system has largely reversible cell state transitions, which LinRace's irreversible-transition assumption does not model
- The goal is de novo cell type annotation rather than lineage validation — this skill assumes cell type labels already exist

## Inputs

- Lineage barcode character matrix (per-cell × per-site integer states, e.g., cm_ZF1_F3.txt read as a data.frame)
- Gene expression count matrix (scRNA-seq, e.g., zebrafish scGESTALT ZF1_F3_DrImpute counts)
- Newick-format lineage tree file (LinRace-reconstructed)
- Per-cell annotated cell type labels (e.g., 34 cell types for the 9707-cell mouse embryo dataset)
- Cut depth threshold (e.g., 26) and minimum subtree size (10 leaves)

## Outputs

- LinRace-reconstructed cell division tree (ape phylo object / Newick)
- cell_type_composition.csv: cell-type-by-subtree percentage table (rows = retained subtrees, columns = cell type percentages summing to 100 per subtree)
- Set of retained subtrees (each with ≥10 leaves after depth-26 cut)

## How to apply

Load the LinRace R package and build the lineage tree with NJ_asym_Dif, passing the barcode character matrix (e.g., as.data.frame(cm_ZF)), kmeans-derived cluster_ids from the top 50 PCs of expression data, the expression matrix, and state_lineages with max_Iter = 500. For validation on annotated datasets such as the mouse embryo data (9707 cells, 34 annotated cell types), cut the rooted tree at a chosen depth (e.g., depth 26) so every tip falls into exactly one subtree, and discard subtrees with fewer than ten leaves (~12 subtrees retained for embryo2). For each retained subtree, tabulate leaf counts per annotated cell type and normalize to percentages summing to 100 within the subtree. Mixed cell types within subtrees, and the same cell type appearing in different subtrees, reproduce the reported partial consistency between transcriptome similarity and barcode similarity, reflecting the asymmetric division model's state divergence and convergence.

## Related tools

- **LinRace** (Reconstructs the cell division tree from paired barcode and expression data via NJ_asym_Dif under the asymmetric cell division model; package also supports tree loading and subtree manipulation) — github.com/ZhangLabGT/LinRace
- **ape** (Part of the LinRace/ape R toolchain used to load the Newick tree and cut it at a fixed depth into subtrees)
- **ggplot2** (Visualization of cell-type composition across subtrees)

## Examples

```
tree_linrace <- NJ_asym_Dif(as.data.frame(cm_ZF),cluster_id,expr_ZF,state_lineages,max_Iter = 500)
```

## Evaluation signals

- Output cell_type_composition.csv is a valid CSV whose row count equals the number of retained subtrees (~12 for embryo2 after the depth-26 cut and ≥10-leaf filter)
- Each row's cell-type percentage values sum to 100 within every subtree
- Every tip cell of the tree appears in exactly one retained subtree after the depth cut (no leaves lost or double-counted beyond the <10-leaf discards)
- Retained subtrees each contain multiple cell types, and the same cell type appears across different subtrees, reproducing the reported partial consistency between transcriptome and barcode similarity
- Reconstructed tree topology is comparable to alternatives via normalized Nye similarity or normalized Robinson-Foulds distance against a reference tree

## Limitations

- LinRace assumes cell state transitions are irreversible; reversible transitions are not modeled
- LinRace assumes all cell states during development exist in current-day cells, and ancestral state reconstruction is restricted to cell states present in the scRNA-seq data — fast-changing systems may have uncaptured ancestral/intermediate states
- LinRace is most applicable to systems with asynchronous differentiation
- Results depend on lineage barcode quality (high diversity and coverage); raw matrix.mtx.gz files for the vignette may not be available due to GitHub LFS quota issues

## Evidence

- [methods] tree building with NJ_asym_Dif: "tree_linrace <- NJ_asym_Dif(as.data.frame(cm_ZF),cluster_id,expr_ZF,state_lineages,max_Iter = 500)"
- [discussion] asymmetric division model support: "LinRace adopts the asymmetric division model, which explains phenomena in real data, increases reconstruction accuracy, and allows for the estimation of ancestral cell states."
- [readme] method definition and integration of paired data: "LinRace is a method that integrates the lineage barcode and gene expression data using the asymmetric cell division model"
