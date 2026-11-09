---
name: single-cell-differential-expression-testing
description: Use when a user defines two or more cell groups over a single-cell gene
  expression matrix and marker genes must be ranked by statistical significance to
  build a clustered heatmap or DE table.
when_to_use_negative:
- The task is general-purpose marker discovery in R/Python with standard scRNA-seq
  pipelines (Seurat/Scanpy/monocle) rather than the in-session CellexalVR heatmap
  workflow
- Only one cell group is defined — a two-group Wilcoxon rank-sum comparison cannot
  be performed
- The input is raw single-cell data that has not yet been processed and converted
  to CellexalVR input files using the cellexalvrR R package (as_cellexalvrR + export2cellexalvr)
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: CellexalVR
  role: VR platform whose in-session heatmap workflow triggers this differential expression
    calculation on the selected cell groups
  repo: https://github.com/sonejilab/cellexalvr
- name: cellexalvrR
  role: R package providing the back-end functions that perform in-session calculations,
    including data preparation and DE gene identification for heatmaps
  repo: https://www.github.com/sonejilab/cellexalvrr
- name: R
  role: Runtime environment (R 4.0.0+) in which the C++-backed Wilcox test backend
    is invoked via Rscript during the CellexalVR session
  repo: https://cran.r-project.org/src/base/R-4/
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.isci.2021.103251
    title: 'CellexalVR: A virtual reality platform to visualize and analyze single-cell
      omics data'
  source_content_hash: sha256:32fe9a1bff375f155e376349f040abc6f56d6cc69c119943c3cb8249863fe5f5
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
yes_rate: 0.8333333333333334
n_claims: 6
---

# single-cell-differential-expression-testing

## Summary

Identifies differentially expressed genes between two user-defined cell groups of a single-cell expression matrix using the Wilcoxon rank-sum test (implemented in C++ in CellexalVR for speed) with Benjamini-Hochberg FDR correction, ranking markers for heatmap rendering and hierarchical clustering.

## When to use

Use when a user defines two or more cell groups over a single-cell gene expression matrix and marker genes must be ranked by statistical significance to build a clustered heatmap or DE table.

## When NOT to use

- The task is general-purpose marker discovery in R/Python with standard scRNA-seq pipelines (Seurat/Scanpy/monocle) rather than the in-session CellexalVR heatmap workflow
- Only one cell group is defined — a two-group Wilcoxon rank-sum comparison cannot be performed
- The input is raw single-cell data that has not yet been processed and converted to CellexalVR input files using the cellexalvrR R package (as_cellexalvrR + export2cellexalvr)

## Inputs

- Gene expression matrix with all non-zero expression values of all cells and genes (from processed scRNA-seq data, e.g. a cellexalvrR object exported via export2cellexalvr)
- Two user-defined cell-group labels over the cells

## Outputs

- Heatmap image of the hierarchically clustered expression matrix for the top marker genes (default top 250)
- Differential expression table with a per-gene p-value column and an FDR column (values in [0, 1])

## How to apply

Load the gene expression matrix containing all non-zero expression values for all cells and genes, together with the two user-defined cell-group labels. For every gene, run a two-group Wilcoxon rank-sum test between the selected cell groups; CellexalVR implements this Wilcox test in C++ for speed, while in R use wilcox.test or a vectorized C++-backed equivalent. Correct per-gene p-values for multiple testing with Benjamini-Hochberg FDR adjustment. Rank genes by significance, keep the top N marker genes (default N = 250), and perform hierarchical clustering on those marker genes (and cells) to order the matrix. Render the clustered expression matrix as a heatmap image and write out a table of per-gene p-values and FDRs. Note that runtime scales with the number of cells/groups selected, so large datasets will take longer to complete.

## Related tools

- **CellexalVR** (VR platform whose in-session heatmap workflow triggers this differential expression calculation on the selected cell groups) — https://github.com/sonejilab/cellexalvr
- **cellexalvrR** (R package providing the back-end functions that perform in-session calculations, including data preparation and DE gene identification for heatmaps) — https://www.github.com/sonejilab/cellexalvrr
- **R** (Runtime environment (R 4.0.0+) in which the C++-backed Wilcox test backend is invoked via Rscript during the CellexalVR session) — https://cran.r-project.org/src/base/R-4/

## Examples

```
wilcox.test(expr[gene, cells_group1], expr[gene, cells_group2]); fdr <- p.adjust(pvals, method = "BH")
```

## Evaluation signals

- The heatmap image file exists after the session completes
- The DE output table contains a p-value column and an FDR column with all values in [0, 1]
- The number of ranked marker genes kept for clustering equals the default N = 250 (or the user-specified N)
- The rendered heatmap shows the expression matrix ordered by hierarchical clustering of the marker genes and cells
- Runtime scales with the number of cells/groups selected — unexpectedly instant completion on a large selection suggests the backend Rscript call failed (check RscriptFilePath in Config/config.xml)

## Limitations

- Backend calculations such as identifying differentially expressed genes will take longer to complete depending on how many cells/groups are selected
- The Wilcox test is implemented in C++ for speed within CellexalVR's Unity (C#) host, which requires Rscript and the cellexalvrR package to be correctly configured (RscriptFilePath in Config/config.xml)

## Evidence

- [discussion] Differentially expressed genes for heatmaps identified using Wilcox test implemented in C++: "Differentially expressed genes for heatmaps are identified using the Wilcox test (implemented in C++ for speed)"
- [discussion] Hierarchical clustering of top N marker genes (default 250): "clustering is performed using hierarchical clustering for the top N marker genes (default is 250)"
- [readme] cellexalvrR provides back-end functions for in-session calculations: [evidence span withheld — anchor status: not_found]
- [discussion] Backend DE calculation runtime scales with selected cells/groups: "backend calculations such as identifying differentially expressed genes will take longer to complete depending on how many cells/groups are selected"
