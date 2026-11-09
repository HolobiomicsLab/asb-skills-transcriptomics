---
name: hierarchical-clustering-of-marker-genes
description: Use when generating an in-session CellexalVR heatmap after a user defines
  two or more cell groups, and the top-ranked marker genes and cells need to be ordered
  by hierarchical clustering for visualization.
when_to_use_negative:
- When no two or more user-defined cell groups exist in the session — the two-group
  Wilcox test requires group labels to compare
- When the goal is marker discovery outside a CellexalVR in-session heatmap (e.g.
  full published DE analysis in Seurat/Scanpy), since this skill implements CellexalVR's
  specific Wilcox + top-250 heatmap workflow
- When the input data has not been converted to CellexalVR input files via the cellexalvrR
  package (as_cellexalvrR / export2cellexalvr)
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0091
tools:
- name: CellexalVR
  role: VR platform whose in-session heatmap workflow triggers the differential expression
    (Wilcox test in C++) and hierarchical clustering of top marker genes
  repo: https://github.com/sonejilab/cellexalvr
- name: cellexalvrR
  role: Accompanying R package providing back-end functions that perform the in-session
    calculations, including DE gene identification for heatmaps
  repo: https://github.com/sonejilab/cellexalvrR
- name: R
  role: Runtime (R 4.0.0+) in which cellexalvrR executes the back-end Wilcox test
    and clustering via Rscript, as configured in Config/config.xml
  repo: https://cran.r-project.org/src/base/R-4/
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.isci.2021.103251
    title: 'CellexalVR: A virtual reality platform to visualize and analyze single-cell
      omics data'
  source_content_hash: sha256:da0d0c55a429cbe1605aed39373afb487e096d163da928c98d8ec52cf890e345
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sonejilab/cellexalvr
  tools:
  - CellexalVR
  - R
  - cellexalvrR
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# hierarchical-clustering-of-marker-genes

## Summary

In CellexalVR, in-session heatmaps order the expression matrix by hierarchically clustering the top N differentially expressed marker genes (default N = 250) and the selected cells, after genes are ranked by a Wilcox test with FDR correction. This yields a clustered heatmap image that visually groups co-expressed genes and similar cells across user-defined cell groups.

## When to use

Use when generating an in-session CellexalVR heatmap after a user defines two or more cell groups, and the top-ranked marker genes and cells need to be ordered by hierarchical clustering for visualization.

## When NOT to use

- When no two or more user-defined cell groups exist in the session — the two-group Wilcox test requires group labels to compare
- When the goal is marker discovery outside a CellexalVR in-session heatmap (e.g. full published DE analysis in Seurat/Scanpy), since this skill implements CellexalVR's specific Wilcox + top-250 heatmap workflow
- When the input data has not been converted to CellexalVR input files via the cellexalvrR package (as_cellexalvrR / export2cellexalvr)

## Inputs

- Gene expression matrix of all non-zero expression values for all cells and genes
- User-defined cell-group labels for two or more selected cell groups (from the in-session CellexalVR selection)

## Outputs

- Clustered heatmap image file with rows (genes) and columns (cells) ordered by hierarchical clustering
- Table of per-gene p-values and Benjamini-Hochberg FDR values for the tested genes
- Ranked list of top N (default 250) marker genes

## How to apply

Load the gene expression matrix containing all non-zero expression values for all cells and genes, together with the two user-defined cell-group labels. For every gene, run a two-group Wilcoxon rank-sum test between the selected groups (in CellexalVR this Wilcox test is implemented in C++ for speed; in R use wilcox.test or a vectorized C++-backed equivalent), then correct per-gene p-values with Benjamini-Hochberg FDR adjustment. Rank genes by significance and keep the top N marker genes, defaulting to N = 250 as the paper specifies. Perform hierarchical clustering on those top N marker genes (and on the cells) to order the matrix rows and columns, then render the clustered matrix as a heatmap image. Write out a table of per-gene p-values and FDRs alongside the image, and note that backend calculations take longer as more cells/groups are selected.

## Related tools

- **CellexalVR** (VR platform whose in-session heatmap workflow triggers the differential expression (Wilcox test in C++) and hierarchical clustering of top marker genes) — https://github.com/sonejilab/cellexalvr
- **cellexalvrR** (Accompanying R package providing back-end functions that perform the in-session calculations, including DE gene identification for heatmaps) — https://github.com/sonejilab/cellexalvrR
- **R** (Runtime (R 4.0.0+) in which cellexalvrR executes the back-end Wilcox test and clustering via Rscript, as configured in Config/config.xml) — https://cran.r-project.org/src/base/R-4/

## Examples

```
devtools::install_github("sonejilab/cellexalvrR"); library(cellexalvrR)  # R package backing the in-session Wilcox test and hierarchical clustering of top 250 marker genes
```

## Evaluation signals

- The heatmap image file exists and is non-empty after the session's backend calculation completes
- The DE table contains a p-value column and an FDR column with all values in [0, 1]
- The number of retained marker genes equals the configured N (default 250), ranked by adjusted p-value significance
- The rendered heatmap row/column order reflects the hierarchical clustering of the top marker genes and cells rather than the original input order
- The top-ranked marker genes match the expected Wilcox/hierarchical-clustering behavior reported in the paper

## Limitations

- Backend calculations such as identifying differentially expressed genes take longer to complete depending on how many cells/groups are selected, so large selections slow heatmap generation
- The Wilcox test and clustering run via the R backend (Rscript.exe path must be configured in Config/config.xml), so R and the cellexalvrR package must be installed correctly for in-session heatmaps to work
- Generating HTML reports of the session requires pandoc to be installed

## Evidence

- [discussion] Differentially expressed genes for heatmaps are identified using the Wilcox test (implemented in C++ for speed): "Differentially expressed genes for heatmaps are identified using the Wilcox test (implemented in C++ for speed)"
- [discussion] Hierarchical clustering of top N marker genes (default 250): "clustering is performed using hierarchical clustering for the top N marker genes (default is 250)"
- [discussion] Backend calculations scale with cells/groups: "backend calculations such as identifying differentially expressed genes will take longer to complete depending on how many cells/groups are selected"
- [readme] cellexalvrR provides back-end in-session calculations: [evidence span withheld — anchor status: external_doc]
