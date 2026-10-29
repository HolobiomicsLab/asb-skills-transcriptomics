---
name: benjamini-hochberg-fdr-correction
description: Use when you have raw per-gene p-values from a two-group differential
  expression test (e.g. a Wilcoxon rank-sum test between user-defined cell groups)
  and need FDR-corrected values before ranking marker genes.
when_to_use_negative:
- No multiple testing context exists — e.g. a single pre-specified gene rather than
  genome/transcriptome-wide testing
- You only need uncorrected p-values to report the test itself rather than to rank/select
  marker genes
- The downstream tool already performs its own multiple-testing correction internally
  (re-adjusting would double-correct)
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: CellexalVR
  role: VR platform that triggers the in-session differential expression + heatmap
    calculation on user-defined cell groups
  repo: https://github.com/sonejilab/cellexalvr
- name: cellexalvrR
  role: R package providing the back-end in-session calculations, including the Wilcox
    test whose p-values are FDR-adjusted
  repo: https://www.github.com/sonejilab/cellexalvrr
- name: R
  role: Environment for running wilcox.test/p.adjust (BH method) or the vectorized
    C++-backed equivalent
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.isci.2021.103251
    title: 'CellexalVR: A virtual reality platform to visualize and analyze single-cell
      omics data'
  source_content_hash: sha256:2a5f72fcabdea3f5dc032b8d8b727c1e8ed8a2e0eb4adce7b7e9c97f8d8df9d6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sonejilab/cellexalvr
  tools:
  - CellexalVR
  - R
  - cellexalvrR
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# benjamini-hochberg-fdr-correction

## Summary

Applies Benjamini-Hochberg FDR adjustment to per-gene p-values from the two-group Wilcoxon rank-sum (Wilcox) test that CellexalVR uses to identify differentially expressed genes for its in-session heatmaps, so marker gene ranking controls the false discovery rate.

## When to use

Use when you have raw per-gene p-values from a two-group differential expression test (e.g. a Wilcoxon rank-sum test between user-defined cell groups) and need FDR-corrected values before ranking marker genes.

## When NOT to use

- No multiple testing context exists — e.g. a single pre-specified gene rather than genome/transcriptome-wide testing
- You only need uncorrected p-values to report the test itself rather than to rank/select marker genes
- The downstream tool already performs its own multiple-testing correction internally (re-adjusting would double-correct)

## Inputs

- Vector of raw per-gene p-values from a two-group Wilcoxon rank-sum (Wilcox) test
- Gene expression matrix containing all non-zero expression values of all cells and genes
- Two (or more) user-defined cell-group labels

## Outputs

- FDR-adjusted p-values (Benjamini-Hochberg) for every gene
- Ranked table of top N (default 250) marker genes with p-value and FDR columns
- Clustered expression matrix rendered as a heatmap image

## How to apply

Run the two-group Wilcoxon rank-sum test for every gene between the selected cell groups (in CellexalVR this Wilcox test is implemented in C++ for speed, via the cellexalvrR backend functions that perform in-session calculations). Correct the resulting per-gene p-values for multiple testing using Benjamini-Hochberg FDR adjustment (in R, p.adjust(pvals, method = 'BH')). Rank genes by significance (e.g. adjusted p-value/FDR) and keep the top N marker genes, N = 250 by default, for subsequent hierarchical clustering of the expression matrix. Render the clustered expression matrix as a heatmap and write out a table containing per-gene p-value and FDR columns.

## Related tools

- **CellexalVR** (VR platform that triggers the in-session differential expression + heatmap calculation on user-defined cell groups) — https://github.com/sonejilab/cellexalvr
- **cellexalvrR** (R package providing the back-end in-session calculations, including the Wilcox test whose p-values are FDR-adjusted) — https://www.github.com/sonejilab/cellexalvrr
- **R** (Environment for running wilcox.test/p.adjust (BH method) or the vectorized C++-backed equivalent)

## Examples

```
pvals <- apply(expr, 1, function(x) wilcox.test(x[grp1], x[grp2])$p.value); fdr <- p.adjust(pvals, method = "BH")
```

## Evaluation signals

- The DE output table contains both a p-value column and an FDR column, each with values in [0, 1]
- FDR values are monotone non-decreasing with the raw p-values (BH adjustment preserves p-value ordering)
- The heatmap image file exists and corresponds to hierarchical clustering of the top N marker genes (default N = 250)
- Top-ranked marker genes match the paper's reported Wilcox-test plus hierarchical-clustering behavior
- Every tested gene receives a corrected value (number of FDR entries equals number of tested genes)

## Limitations

- Backend calculations such as identifying differentially expressed genes take longer to complete depending on how many cells/groups are selected, so FDR correction can be slow for large datasets
- The paper does not state an explicit FDR significance cutoff; genes are selected by ranking (top 250 default) rather than by an adjusted-p threshold
- R scripting must be configured for CellexalVR (Rscript path in Config/config.xml; R >= 4.0.0 per the README)

## Evidence

- [discussion] Differentially expressed genes for heatmaps identified using Wilcox test implemented in C++: "Differentially expressed genes for heatmaps are identified using the Wilcox test (implemented in C++ for speed)"
- [discussion] Hierarchical clustering of top N marker genes (default 250): "clustering is performed using hierarchical clustering for the top N marker genes (default is 250)"
- [readme] cellexalvrR provides back-end functions that perform in-session calculations: [evidence span withheld — anchor status: external_doc]
