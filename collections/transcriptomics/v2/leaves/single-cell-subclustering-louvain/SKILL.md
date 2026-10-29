---
name: single-cell-subclustering-louvain
description: Use when a coarse cell-type annotation (e.g., 'dermal condensate') must
  be resolved into finer transcriptional states, and downstream tools like CellChat
  require state-resolved cell groups as input.
when_to_use_negative:
- The input cell type has abundance <= 5% of the dataset — Louvain subclustering on
  rare populations is underpowered and not supported by the article's workflow
- Contaminated or doublet-containing groups have not yet been removed (e.g., CD3D+
  T cells within FIB, or immune cells lacking DC markers), as these distort community
  structure
- The analysis goal is de novo clustering of the full dataset rather than resolving
  states within an already-annotated compartment
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: Seurat
  role: Loads count matrix and metadata, performs per-cell QC filtering, and hosts
    the subclustering and UMAP visualization of the dermal compartment
- name: CellChat
  role: Consumes the Louvain-derived subcluster annotations as cell-group labels for
    downstream communication inference (createCellChat group.by input)
  repo: https://github.com/sqjin/CellChat
- name: UMAP
  role: Projects the smoothed similarity matrix of subclusters for visual validation
    of state separation
  repo: https://github.com/lmcinnes/umap
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-021-21246-9
    title: Inference and analysis of cell-cell communication using CellChat
  source_content_hash: sha256:5781bf273cb440d45f14f0be60b7af08e6d8d6ec73941e887b3602d3323dde98
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
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# single-cell-subclustering-louvain

## Summary

Subclusters a defined single-cell compartment (e.g., dermal cells in E14.5 embryonic skin) using Louvain community detection restricted to cell types with abundance > 5%, then annotates resulting states (placode, pre-DC, DC1, DC2) from canonical markers prior to cell-cell communication analysis with CellChat.

## When to use

Use when a coarse cell-type annotation (e.g., 'dermal condensate') must be resolved into finer transcriptional states, and downstream tools like CellChat require state-resolved cell groups as input.

## When NOT to use

- The input cell type has abundance <= 5% of the dataset — Louvain subclustering on rare populations is underpowered and not supported by the article's workflow
- Contaminated or doublet-containing groups have not yet been removed (e.g., CD3D+ T cells within FIB, or immune cells lacking DC markers), as these distort community structure
- The analysis goal is de novo clustering of the full dataset rather than resolving states within an already-annotated compartment

## Inputs

- Seurat object or scRNA-seq count matrix with cell-level metadata (e.g., 10X Genomics output, GEO GSM3453535-GSM3453538)
- Pre-computed coarse cell-type annotations (e.g., dermal compartment labels)
- Canonical marker gene list for state annotation

## Outputs

- Fine-grained cell-state labels per cell (e.g., placode, pre-DC, DC1, DC2)
- Annotated Seurat object with subcluster identities and UMAP embedding
- Subcluster-level metadata suitable for createCellChat group.by input

## How to apply

After initial Seurat QC on the scRNA-seq count matrix (e.g., embryonic skin criteria: UMI count 2500-50000, genes >= 1000, mitochondrial fraction < 20%), remove flagged contaminated groups (e.g., red blood cells, CD3D-expressing T cells in FIB subclustering, immune cells lacking DC markers in DC subclustering). Then perform subclustering with Louvain community detection only on cell types whose abundance in the dataset is greater than 5%, as rare populations lack the statistical power for robust community detection. Annotate each derived subcluster (e.g., placode, pre-DC, DC1, DC2) using canonical marker genes, and verify by inspecting UMAP embeddings. These annotated subpopulations become the cell-group labels passed to createCellChat for downstream ligand-receptor inference.

## Related tools

- **Seurat** (Loads count matrix and metadata, performs per-cell QC filtering, and hosts the subclustering and UMAP visualization of the dermal compartment)
- **CellChat** (Consumes the Louvain-derived subcluster annotations as cell-group labels for downstream communication inference (createCellChat group.by input)) — https://github.com/sqjin/CellChat
- **UMAP** (Projects the smoothed similarity matrix of subclusters for visual validation of state separation) — https://github.com/lmcinnes/umap

## Examples

```
cellchat <- createCellChat(object = seurat_obj, meta = seurat_obj@meta.data, group.by = "subcluster_label")
```

## Evaluation signals

- All subclustered cell types have abundance > 5% in the parent dataset before Louvain detection
- Known contaminated groups (e.g., CD3D+ T cells, immune cells lacking DC markers) are excluded before clustering
- Each derived state is annotated by at least one canonical marker gene with detectable expression
- QC thresholds applied upstream match the article's criteria: UMI 2500-50000, genes >= 1000, mitochondrial fraction < 20%
- Subcluster labels are consistent across UMAP visualization and correspond to biologically distinct populations (placode vs. pre-DC vs. DC1 vs. DC2)

## Limitations

- Louvain subclustering is only applied to cell types with abundance > 5%; rarer populations are excluded and may harbor biologically relevant states
- Contaminated populations must be manually identified and removed (e.g., via marker inspection); automated detection of such groups is not part of the workflow
- Subcluster annotations depend on the quality and specificity of the canonical marker panel used

## Evidence

- [discussion] Subclustering restricted to cell types > 5% abundance: "we performed subclustering analysis on the cell types, whose abundance in the dataset was greater than 5% using the Louvain community detection method."
- [discussion] Embryonic skin QC thresholds: "we removed the cells with the amount of UMI count less than 2500 and greater than 50000, as well as the cells with the number of genes less than 1000 and the fraction of mitochondrial counts greater"
- [discussion] Removal of contaminated immune group from DC subclustering: "This contaminated immune cell group was also removed for further analysis."
- [discussion] Removal of contaminated CD3D T cell group: "This contaminated TC population was removed for further analysis."
