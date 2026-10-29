---
name: lineage-tree-depth-partitioning
description: Use when you have a rooted lineage-reconstructed tree (e.g., LinRace
  output in Newick) plus per-leaf cell-type labels, and want to assess whether barcode-defined
  subtrees are pure or mixed with respect to transcriptome-defined cell types.
when_to_use_negative:
- The input is an unrooted or edge-length-free tree where a depth-based cut cannot
  assign every tip to exactly one subtree
- You need strict one-to-one correspondence between barcode subtrees and cell types;
  the LinRace asymmetric division model explicitly allows state divergence and state
  convergence, so subtree purity is not expected
- Cell-type labels are missing or unreliable for most leaves, making per-subtree composition
  percentages meaningless
edam_topics:
- http://edamontology.org/topic_3293
tools:
- name: LinRace
  role: R package providing the reconstructed lineage tree and the toolchain used
    to load and cut the tree (as in the mouse embryo2 vignette)
  repo: https://github.com/ZhangLabGT/LinRace
- name: ggplot2
  role: R plotting library used in the LinRace methods workflow for downstream visualization
    of the partitioning results
- name: R
  role: Environment in which the tree is loaded (ape/Newick), partitioned at depth
    26, and cell-type composition is tabulated
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-023-44173-3
    title: 'LinRace: cell division history reconstruction of single cells using paired
      lineage barcode and gene expression data'
  source_content_hash: sha256:da7232e62cb7674d301b92b60fe9d5326c9260ff7da3bebdea7fded32ae2fca4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ZhangLabGT/LinRace
  tools:
  - LinRace
  - R
  - ggplot2
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3293
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# lineage-tree-depth-partitioning

## Summary

Cut a reconstructed cell lineage tree (e.g., the LinRace tree of the early mouse embryo, 9707 cells / 34 cell types) at a fixed depth so every tip falls into exactly one subtree, then tabulate annotated cell-type composition per subtree to test consistency between barcode similarity and transcriptome similarity.

## When to use

Use when you have a rooted lineage-reconstructed tree (e.g., LinRace output in Newick) plus per-leaf cell-type labels, and want to assess whether barcode-defined subtrees are pure or mixed with respect to transcriptome-defined cell types.

## When NOT to use

- The input is an unrooted or edge-length-free tree where a depth-based cut cannot assign every tip to exactly one subtree
- You need strict one-to-one correspondence between barcode subtrees and cell types; the LinRace asymmetric division model explicitly allows state divergence and state convergence, so subtree purity is not expected
- Cell-type labels are missing or unreliable for most leaves, making per-subtree composition percentages meaningless

## Inputs

- Rooted lineage tree in Newick format (e.g., LinRace-reconstructed tree for the mouse embryo2 dataset, 9707 cells)
- Per-cell annotated cell-type label table (34 annotated cell types for the mouse embryo dataset)
- Depth cutoff for partitioning (e.g., depth 26)
- Minimum subtree size threshold (e.g., 10 leaves)

## Outputs

- cell_type_composition.csv — cell-type-by-subtree percentage table where each retained subtree's percentages sum to 100
- Set of retained subtrees after the min-leaf filter (~12 subtrees for embryo2)

## How to apply

Load the LinRace-reconstructed tree in Newick format and the per-cell annotated cell-type labels in R using the LinRace/ape toolchain (library(LinRace); the mouse embryo vignette uses the embryo2 dataset from M. Chan et al.). Cut the rooted tree at a chosen depth (here, depth 26) so that every tip falls into exactly one subtree. Discard subtrees with fewer than ten leaves (for the embryo2 dataset, ~12 subtrees are retained). For each retained subtree, tabulate leaf counts per annotated cell type and normalize to percentages summing to 100 within the subtree. The rationale is that if the reconstructed tree reflects both barcode and transcriptomic similarity, subtrees will each contain multiple cell types and the same cell type may span multiple subtrees — i.e., only partial consistency is expected. Write the result as a cell-type-by-subtree percentage table (cell_type_composition.csv).

## Related tools

- **LinRace** (R package providing the reconstructed lineage tree and the toolchain used to load and cut the tree (as in the mouse embryo2 vignette)) — https://github.com/ZhangLabGT/LinRace
- **ggplot2** (R plotting library used in the LinRace methods workflow for downstream visualization of the partitioning results)
- **R** (Environment in which the tree is loaded (ape/Newick), partitioned at depth 26, and cell-type composition is tabulated)

## Examples

```
devtools::install_github("https://github.com/ZhangLabGT/LinRace"); library(LinRace)
```

## Evaluation signals

- Every tip of the original tree appears in exactly one retained subtree after the depth-26 cut (no leaf lost or duplicated)
- Row count of cell_type_composition.csv equals the number of retained subtrees (≈12 rows for the embryo2 dataset)
- Each subtree's percentage columns sum to 100 within numerical tolerance
- All subtrees in the output have at least 10 leaves per the min-size filter
- Expected qualitative pattern: each retained subtree contains multiple cell types, and the same cell type appears in different subtrees (partial, not full, consistency)

## Limitations

- Consistency between transcriptome similarity and barcode similarity is only partial — subtrees mix cell types and cell types span subtrees
- LinRace assumes irreversible cell state transitions; reversible transitions are not modeled, which can affect how cell states distribute across subtrees
- LinRace assumes all cell states during development exist in current-day cells; ancestral/intermediate states absent from the scRNA-seq data cannot be reconstructed, limiting interpretation of subtree compositions
- GitHub README notes raw matrix files for the vignette may not be uploaded due to LFS quota; datasets must be fetched separately (e.g., GSE117542 for zebrafish; other datasets by request)

## Evidence

- [abstract] LinRace integrates lineage barcode and gene expression data using asymmetric cell division model: "which integrates lineage barcode and gene expression data using asymmetric cell division model and infers cell lineages and ancestral cell states using Neighbor-Joining and maximum-likelihood"
- [readme] README mouse embryo vignette: [evidence span withheld — anchor status: external_doc]
- [readme] README installation: [evidence span withheld — anchor status: external_doc]
- [discussion] LinRace asymmetric division model discussion: "Asymmetric divisions can cause one cell state to generate two distinct cell states, resulting in state divergence"
