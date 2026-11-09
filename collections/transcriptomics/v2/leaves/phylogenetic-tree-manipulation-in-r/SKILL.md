---
name: phylogenetic-tree-manipulation-in-r
description: Use when a reconstructed lineage or phylogenetic tree (e.g., Newick format
  from LinRace) must be partitioned at a specified depth and tip-level annotations
  (e.g., 34 cell types over 9707 mouse embryo cells) summarized per subtree in R.
when_to_use_negative:
- You need to evaluate tree reconstruction accuracy itself (use Nye similarity or
  Robinson-Foulds distance against a ground-truth tree instead of subtree composition
  analysis)
- The cell annotations are missing or not mappable to tree tip labels, so per-subtree
  cell-type tabulation is impossible
- Your goal is comparing alternative reconstruction methods (e.g., LinRace vs DCLEAR
  vs Cassiopeia) on topology, not analyzing composition of a single tree
edam_operation: http://edamontology.org/operation_0325
edam_topics:
- http://edamontology.org/topic_0084
- http://edamontology.org/topic_3170
tools:
- name: LinRace
  role: R package providing the reconstructed lineage tree and vignette workflow (e.g.,
    Mouse_test.Rmd for the embryo2 dataset) used for tree loading and subtree analysis
  repo: https://github.com/ZhangLabGT/LinRace
- name: R
  role: Language/runtime for tree parsing, depth-based cutting, leaf tabulation, and
    CSV export
- name: ggplot2
  role: Visualization of per-subtree cell type composition percentages
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-023-44173-3
    title: 'LinRace: cell division history reconstruction of single cells using paired
      lineage barcode and gene expression data'
  source_content_hash: sha256:bdf379f1bb3a2a6f524b6a682d73ef3812684ab684d8c52694d5990bc8e86fb0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ZhangLabGT/LinRace
  tools:
  - LinRace
  - R
  - ggplot2
  edam_operation: http://edamontology.org/operation_0325
  edam_topics:
  - http://edamontology.org/topic_0084
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# phylogenetic-tree-manipulation-in-r

## Summary

Manipulate rooted phylogenetic/lineage trees in R (loading Newick-format trees, cutting them at a given depth, extracting subtrees, and tabulating tip-level metadata per subtree) using the LinRace and ape toolchains. This enables downstream analyses such as checking whether barcode-reconstructed subtrees are consistent with transcriptome-derived cell type labels.

## When to use

Use when a reconstructed lineage or phylogenetic tree (e.g., Newick format from LinRace) must be partitioned at a specified depth and tip-level annotations (e.g., 34 cell types over 9707 mouse embryo cells) summarized per subtree in R.

## When NOT to use

- You need to evaluate tree reconstruction accuracy itself (use Nye similarity or Robinson-Foulds distance against a ground-truth tree instead of subtree composition analysis)
- The cell annotations are missing or not mappable to tree tip labels, so per-subtree cell-type tabulation is impossible
- Your goal is comparing alternative reconstruction methods (e.g., LinRace vs DCLEAR vs Cassiopeia) on topology, not analyzing composition of a single tree

## Inputs

- Reconstructed lineage/phylogenetic tree in Newick format (e.g., LinRace tree_linrace output)
- Per-cell annotation table mapping leaf/tip cell IDs to cell type labels (e.g., 34 annotated cell types for the mouse embryo dataset)
- Cut depth parameter (e.g., depth 26) and minimum subtree size threshold (e.g., 10 leaves)

## Outputs

- Set of subtrees obtained by cutting the rooted tree at the chosen depth
- cell_type_composition.csv: cell-type-by-subtree percentage table where each row is a retained subtree and per-subtree percentages sum to 100

## How to apply

Load the reconstructed tree (e.g., LinRace output from NJ_asym_Dif or the mouse embryo vignette tree) into R via library(LinRace) and ape, which supplies Newick parsing and rooting utilities. Cut the rooted tree at a chosen depth (e.g., depth 26 for the LinRace embryo2 tree) so every tip cell falls into exactly one subtree. Filter out subtrees with fewer than ten leaves (~12 subtrees retained for the embryo2 dataset of 9707 cells with 34 annotated cell types) to avoid noise from tiny clades. For each retained subtree, tabulate leaf counts per annotated cell type and normalize to percentages summing to 100 within the subtree; optional ggplot2 visualization supports interpretation. Rationale: if transcriptome similarity tracks barcode similarity partially, each subtree should be a mixture of cell types and the same cell type should recur across subtrees, so the percentage table directly tests that hypothesis.

## Related tools

- **LinRace** (R package providing the reconstructed lineage tree and vignette workflow (e.g., Mouse_test.Rmd for the embryo2 dataset) used for tree loading and subtree analysis) — https://github.com/ZhangLabGT/LinRace
- **R** (Language/runtime for tree parsing, depth-based cutting, leaf tabulation, and CSV export)
- **ggplot2** (Visualization of per-subtree cell type composition percentages)

## Examples

```
library(LinRace); tree <- read.tree('tree_linrace.newick'); subtrees <- tree_subset_at_depth(tree, depth=26); comps <- tapply(tips, subtree_id, function(x) 100*prop.table(table(cell_types[x]))); write.csv(comps, 'cell_type_composition.csv')
```

## Evaluation signals

- Output is a valid CSV whose row count equals the number of retained subtrees (~12 for embryo2 at depth 26 with min 10 leaves)
- Each per-subtree set of cell type percentage columns sums to 100 within rounding error
- Every tip of the original tree appears in exactly one subtree after the depth-26 cut (a partition, no cell lost or duplicated)
- Subtrees contain multiple cell types and the same cell type appears across different subtrees, matching the reported partial consistency between transcriptome and barcode similarity
- All discarded subtrees have fewer than ten leaves

## Limitations

- Results depend on the quality of the reconstructed tree; LinRace's own accuracy depends on lineage barcode diversity and coverage, and current barcode technology limits tree quality
- Cell type labels reflect terminal states only; LinRace assumes all cell states during development exist in current-day cells and models only irreversible state transitions
- Small subtrees (<10 leaves) are discarded, so composition analysis excludes rare clades
- Raw data for the vignette may not be present in the repo (GitHub LFS quota issue); datasets must be retrieved separately (e.g., GSE117542 for zebrafish, request for others)

## Evidence

- [results] Cutting the LinRace-reconstructed mouse embryo tree at depth 26 yields subtrees that each consist of multiple cell types, and the same cell type appears in different subtrees, showing the partial consistency between transcriptome similarity and barcode similarity.: [evidence span withheld — anchor status: not_found]
- [methods] R library loading and tree building context: "tree_linrace <- NJ_asym_Dif(as.data.frame(cm_ZF),cluster_id,expr_ZF,state_lineages,max_Iter = 500)"
- [readme] README vignette on mouse embryo dataset: [evidence span withheld — anchor status: external_doc]
- [discussion] Discussion limitation on state assumptions: "LinRace assumes that cell state transitions are irreversible, which is generally true under homeostatic conditions. However, reversible state transitions are also known to exist."
