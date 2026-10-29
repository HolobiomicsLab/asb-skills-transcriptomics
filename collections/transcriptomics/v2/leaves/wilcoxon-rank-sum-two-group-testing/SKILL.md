---
name: wilcoxon-rank-sum-two-group-testing
description: Use when a single-cell gene expression matrix has cells split into exactly
  two user-defined groups and in-session differential expression markers are needed
  to order a heatmap.
when_to_use_negative:
- The contrast involves more than two groups without a defined pairwise split — the
  workflow defines a two-group Wilcoxon rank-sum test between selected cell groups.
- The task is only visualizing or clustering a pre-existing marker gene list with
  no two-group differential-expression contrast to compute.
- The input is not per-cell gene expression with cell-group labels (e.g. bulk expression
  with one value per condition has no per-cell rank-sum structure).
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: CellexalVR
  role: Runs the in-session Wilcox differential-expression calculation (implemented
    in C++ for speed) and renders the clustered heatmap of the top 250 marker genes
  repo: https://github.com/sonejilab/cellexalvr
- name: cellexalvrR
  role: R package providing the back-end functions that perform the in-session differential-expression
    calculations, invoked via Rscript by CellexalVR
  repo: https://www.github.com/sonejilab/cellexalvrr
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.isci.2021.103251
    title: 'CellexalVR: A virtual reality platform to visualize and analyze single-cell
      omics data'
  source_content_hash: sha256:a92bb6f936b965db24da8bfcfdbf9185bcc3d65e4c4d06719b9ed3db71bbc20f
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
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# wilcoxon-rank-sum-two-group-testing

## Summary

Identifies differentially expressed marker genes between two user-defined cell groups by running a per-gene two-group Wilcoxon rank-sum test (implemented in C++ for speed in CellexalVR), Benjamini-Hochberg FDR correction, and ranking the top 250 marker genes for heatmap hierarchical clustering.

## When to use

Use when a single-cell gene expression matrix has cells split into exactly two user-defined groups and in-session differential expression markers are needed to order a heatmap.

## When NOT to use

- The contrast involves more than two groups without a defined pairwise split — the workflow defines a two-group Wilcoxon rank-sum test between selected cell groups.
- The task is only visualizing or clustering a pre-existing marker gene list with no two-group differential-expression contrast to compute.
- The input is not per-cell gene expression with cell-group labels (e.g. bulk expression with one value per condition has no per-cell rank-sum structure).

## Inputs

- Gene expression matrix containing all non-zero expression values of all cells and genes
- Two user-defined cell-group labels (e.g. selections made in CellexalVR)
- R installation (>= R 4.0.0) with the cellexalvrR package for back-end calculations

## Outputs

- Table of per-gene Wilcoxon p-values and Benjamini-Hochberg FDR values in [0, 1]
- Ranked list of top N (default 250) marker genes
- Hierarchically clustered expression matrix (genes and cells) rendered as a heatmap image

## How to apply

Load the gene expression matrix containing all non-zero expression values of all cells and genes together with the two user-defined cell-group labels. For every gene, run a two-group Wilcoxon rank-sum test between the selected cell groups; CellexalVR implements this Wilcox test in C++ for speed, while in R the equivalent is wilcox.test or a vectorized C++-backed equivalent. Correct the per-gene p-values for multiple testing using Benjamini-Hochberg FDR adjustment. Rank genes by significance, keep the top N marker genes (default N = 250), and perform hierarchical clustering on those marker genes and the cells to order the matrix before rendering the clustered expression matrix as a heatmap image. The accompanying R package cellexalvrR provides these back-end in-session calculation functions, invoked via Rscript by the CellexalVR application.

## Related tools

- **CellexalVR** (Runs the in-session Wilcox differential-expression calculation (implemented in C++ for speed) and renders the clustered heatmap of the top 250 marker genes) — https://github.com/sonejilab/cellexalvr
- **cellexalvrR** (R package providing the back-end functions that perform the in-session differential-expression calculations, invoked via Rscript by CellexalVR) — https://www.github.com/sonejilab/cellexalvrr

## Examples

```
p <- sapply(genes, function(g) wilcox.test(expr[g, grp1], expr[g, grp2])$p.value)
```

## Evaluation signals

- The heatmap image output file exists and is non-empty.
- The differential-expression table contains a p-value column and an FDR column, both with values in [0, 1].
- The number of retained marker genes equals the configured N (default 250).
- Top-ranked marker genes match the paper's reported Wilcox-test plus hierarchical-clustering behavior (top 250 genes by default).
- Per-gene p-values reproduce a reference wilcox.test run on the same two cell groups (rank-sum statistic is invariant to monotone expression scaling).

## Limitations

- Backend calculations such as identifying differentially expressed genes take longer to complete depending on how many cells/groups are selected, so large selections slow the in-session computation.
- The Wilcox test operates on all non-zero expression values loaded for the selected groups; the runtime scales with the number of genes and cells in the matrix.
- The default keeps only the top 250 marker genes for clustering, so weaker markers beyond N are discarded from the heatmap.

## Evidence

- [discussion] Wilcox test implemented in C++: "Differentially expressed genes for heatmaps are identified using the Wilcox test (implemented in C++ for speed)"
- [discussion] hierarchical clustering of top N marker genes: "clustering is performed using hierarchical clustering for the top N marker genes (default is 250)"
- [discussion] runtime scaling of DE calculations: "backend calculations such as identifying differentially expressed genes will take longer to complete depending on how many cells/groups are selected"
- [readme] cellexalvrR provides in-session calculations: [evidence span withheld — anchor status: external_doc]
