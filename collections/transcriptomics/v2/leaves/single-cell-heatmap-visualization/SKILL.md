---
name: single-cell-heatmap-visualization
description: Use when two or more cell groups have been defined in-session from single-cell
  expression data and a ranked marker-gene heatmap with per-gene Wilcoxon p-values
  and FDRs is required to compare groups.
when_to_use_negative:
- Only a single cell group is defined — the two-group Wilcoxon rank-sum test requires
  at least two groups to compare
- The goal is pseudotime trajectory analysis rather than group comparison — CellexalVR
  handles that separately by partitioning selected cells into four clusters for a
  slingshot analysis
- Expression data has not yet been converted to CellexalVR input files via the cellexalvrR
  package (as_cellexalvrR / export2cellexalvr)
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: CellexalVR
  role: VR platform whose in-session heatmap feature triggers the Wilcoxon DE calculation
    and hierarchical clustering via the R backend
  repo: https://github.com/sonejilab/cellexalvr
- name: cellexalvrR
  role: R package providing the back-end functions that perform in-session calculations,
    including differential expression for heatmaps
  repo: https://github.com/sonejilab/cellexalvrR
- name: R
  role: Runtime environment (R >= 4.0.0 with Rscript.exe path configured) in which
    the cellexalvrR back-end calculations execute
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.isci.2021.103251
    title: 'CellexalVR: A virtual reality platform to visualize and analyze single-cell
      omics data'
  source_content_hash: sha256:e16a6009d450442a8943880ad6feb1977b82a3dcd561f0b74fb0cc096bac44a2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sonejilab/cellexalvr
  tools:
  - CellexalVR
  - R
  - cellexalvrR
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# single-cell-heatmap-visualization

## Summary

Identifies differentially expressed genes between user-defined cell groups with a Wilcoxon rank-sum test (implemented in C++ for speed), adjusts p-values with Benjamini-Hochberg FDR, and renders the top 250 marker genes as a hierarchically clustered heatmap — the in-session heatmap workflow of CellexalVR.

## When to use

Use when two or more cell groups have been defined in-session from single-cell expression data and a ranked marker-gene heatmap with per-gene Wilcoxon p-values and FDRs is required to compare groups.

## When NOT to use

- Only a single cell group is defined — the two-group Wilcoxon rank-sum test requires at least two groups to compare
- The goal is pseudotime trajectory analysis rather than group comparison — CellexalVR handles that separately by partitioning selected cells into four clusters for a slingshot analysis
- Expression data has not yet been converted to CellexalVR input files via the cellexalvrR package (as_cellexalvrR / export2cellexalvr)

## Inputs

- Gene expression matrix containing all non-zero expression values of all cells and genes
- Two or more user-defined cell-group labels assigned to each cell

## Outputs

- Heatmap image of the hierarchically clustered top-N marker gene expression matrix
- Table of per-gene Wilcoxon p-values and Benjamini-Hochberg FDR values for the top-ranked marker genes

## How to apply

Load the gene expression matrix (all non-zero expression values of all cells and genes) together with the user-defined cell-group labels. For every gene, run a two-group Wilcoxon rank-sum test between the selected cell groups — CellexalVR implements this Wilcox test in C++ for speed; in R use wilcox.test or a vectorized C++-backed equivalent. Correct per-gene p-values for multiple testing using Benjamini-Hochberg FDR adjustment. Rank genes by significance, keep the top N marker genes (default N = 250), and perform hierarchical clustering on those marker genes (and the cells) to order the matrix before rendering the heatmap image and a table of per-gene p-values and FDRs.

## Related tools

- **CellexalVR** (VR platform whose in-session heatmap feature triggers the Wilcoxon DE calculation and hierarchical clustering via the R backend) — https://github.com/sonejilab/cellexalvr
- **cellexalvrR** (R package providing the back-end functions that perform in-session calculations, including differential expression for heatmaps) — https://github.com/sonejilab/cellexalvrR
- **R** (Runtime environment (R >= 4.0.0 with Rscript.exe path configured) in which the cellexalvrR back-end calculations execute)

## Examples

```
devtools::install_github("sonejilab/cellexalvrR"); library(cellexalvrR)
```

## Evaluation signals

- The heatmap image file exists on disk after the session's backend calculation completes
- The DE table contains a p-value column and an FDR column with all values in [0, 1]
- The heatmap orders exactly the top N marker genes (default N = 250) selected by Wilcoxon significance
- Top-ranked marker genes match the paper's reported Wilcox-test + hierarchical-clustering behavior
- Backend runtime scales with the number of cells/groups selected — verify the calculation completes (it takes longer for large datasets)

## Limitations

- Backend calculations such as identifying differentially expressed genes take longer to complete depending on how many cells/groups are selected, so large datasets will be slow
- The workflow depends on the cellexalvrR R package and a configured Rscript.exe path; the HTML report additionally requires pandoc
- Only the top N marker genes (default 250) are clustered and displayed; lower-ranked genes are dropped from the heatmap

## Evidence

- [discussion] Differentially expressed genes for heatmaps identified using Wilcox test implemented in C++: "Differentially expressed genes for heatmaps are identified using the Wilcox test (implemented in C++ for speed)"
- [discussion] Hierarchical clustering of top N marker genes (default 250): "clustering is performed using hierarchical clustering for the top N marker genes (default is 250)"
- [discussion] Backend DE calculation time scales with selection size: "backend calculations such as identifying differentially expressed genes will take longer to complete depending on how many cells/groups are selected"
- [readme] cellexalvrR provides back-end in-session calculations: [evidence span withheld — anchor status: external_doc]
