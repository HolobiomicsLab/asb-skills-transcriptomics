---
name: cell-type-annotation-mapping
description: Use when you have a rooted lineage tree in Newick plus per-cell cell-type
  labels and want to quantify which cell types co-occur within subtrees (e.g., after
  cutting the LinRace embryo tree at depth 26).
when_to_use_negative:
- You need to reconstruct or compare full lineage trees rather than summarize cell-type
  composition of existing tree subtrees (use tree reconstruction/RF or Nye similarity
  workflows instead).
- Cell-type labels are not available for the tree tips, so no per-subtree composition
  table can be computed.
- The tree is unrooted or lacks branch-length information, making depth-based cutting
  ill-defined.
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_2229
- http://edamontology.org/topic_3170
tools:
- name: LinRace
  role: Provides the reconstructed lineage tree (NJ_asym_Dif output) and the R environment
    used to load, cut, and annotate the tree tips with cell-type labels
  repo: https://github.com/ZhangLabGT/LinRace
- name: R
  role: Analysis environment for loading the Newick tree, cutting at depth 26, filtering
    subtrees, and computing per-subtree cell-type percentages
- name: ggplot2
  role: Visualization of the cell-type composition of subtrees
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-023-44173-3
    title: 'LinRace: cell division history reconstruction of single cells using paired
      lineage barcode and gene expression data'
  source_content_hash: sha256:1b43f8fcbfc4cfbf1481e5ddba7252c2955e110b080887b96bcebf82f0ab41aa
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ZhangLabGT/LinRace
  tools:
  - LinRace
  - R
  - ggplot2
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_2229
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# cell-type-annotation-mapping

## Summary

Maps per-cell annotated cell-type labels onto leaves of a reconstructed lineage tree (e.g., LinRace's mouse embryo tree with 9707 cells and 34 annotated cell types), then tabulates per-subtree cell-type composition to test consistency between transcriptome similarity and barcode similarity.

## When to use

Use when you have a rooted lineage tree in Newick plus per-cell cell-type labels and want to quantify which cell types co-occur within subtrees (e.g., after cutting the LinRace embryo tree at depth 26).

## When NOT to use

- You need to reconstruct or compare full lineage trees rather than summarize cell-type composition of existing tree subtrees (use tree reconstruction/RF or Nye similarity workflows instead).
- Cell-type labels are not available for the tree tips, so no per-subtree composition table can be computed.
- The tree is unrooted or lacks branch-length information, making depth-based cutting ill-defined.

## Inputs

- LinRace-reconstructed lineage tree in Newick format (e.g., mouse embryo embryo2 dataset, 9707 cells)
- Per-cell annotated cell-type labels (34 annotated cell types for the mouse embryo dataset)
- Depth cutoff for cutting the rooted tree (e.g., depth 26)
- Minimum subtree size threshold (10 leaves)

## Outputs

- cell_type_composition.csv: cell-type-by-subtree percentage table where each subtree row sums to 100
- Set of retained subtrees (subtrees with >= 10 leaves after cutting at the depth cutoff)

## How to apply

Load the LinRace-reconstructed lineage tree in Newick format and the per-cell annotated cell-type labels in R using the LinRace/ape toolchain (library(LinRace)). Cut the rooted tree at a chosen depth (depth 26 for the mouse embryo dataset) so every tip falls into exactly one subtree, then discard subtrees with fewer than ten leaves (~12 subtrees are retained for the embryo2 dataset). For each retained subtree, tabulate leaf counts per annotated cell type and normalize to percentages summing to 100 within the subtree. Write the cell-type-by-subtree percentage table to cell_type_composition.csv. Interpret mixed composition across subtrees, and recurring cell types across subtrees, as partial consistency between transcriptome similarity and barcode similarity, consistent with LinRace's asymmetric division model where one cell state can generate two distinct states and distant branches can divide into the same state.

## Related tools

- **LinRace** (Provides the reconstructed lineage tree (NJ_asym_Dif output) and the R environment used to load, cut, and annotate the tree tips with cell-type labels) — https://github.com/ZhangLabGT/LinRace
- **R** (Analysis environment for loading the Newick tree, cutting at depth 26, filtering subtrees, and computing per-subtree cell-type percentages)
- **ggplot2** (Visualization of the cell-type composition of subtrees)

## Examples

```
library(LinRace); tree <- read.tree('tree_linrace.nwk'); subtrees <- treeSlice(tree, 26); comp <- t(sapply(subtrees, function(st) 100*table(labels[st$tip.label])/length(st$tip.label))); write.csv(comp, 'cell_type_composition.csv')
```

## Evaluation signals

- Output cell_type_composition.csv is a valid CSV whose row count equals the number of retained subtrees (~12 for the embryo2 dataset).
- Each subtree row's cell-type percentage columns sum exactly to 100.
- Every tree tip falls into exactly one subtree after cutting at depth 26 (no tips lost or double-assigned).
- All retained subtrees contain at least 10 leaves; subtrees below this cutoff are excluded.
- Retained subtrees each contain multiple cell types, and the same cell type appears across different subtrees, matching the reported finding of partial consistency.

## Limitations

- LinRace assumes irreversible cell state transitions; reversible transitions are not modeled, which can distort apparent cell-type composition of subtrees.
- Ancestral state reconstruction is restricted to cell states present in the scRNA-seq data; fast-changing systems may have uncaptured ancestral/intermediate states, so subtree compositions reflect only observed tip labels.
- LinRace is most applicable to systems with asynchronous differentiation, where most ancestral cell states exist in current-day cells; in other systems subtree cell-type mixes may be misleading.

## Evidence

- [results] Cutting the LinRace-reconstructed mouse embryo tree at depth 26 yields subtrees that each consist of multiple cell types, and the same cell type appears in different subtrees: [evidence span withheld — anchor status: not_found]
- [discussion] LinRace adopts the asymmetric division model: "Asymmetric divisions can cause one cell state to generate two distinct cell states, resulting in state divergence; and cells on distant lineage trees can also divide asymmetrically into the same cell"
- [readme] README mouse embryo vignette: [evidence span withheld — anchor status: external_doc]
