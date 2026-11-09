---
name: marker-gene-based-cell-visualization
description: Use when IA-SVA (or similar SV analysis) has identified outlier cells
  via a surrogate variable, but standard all-gene visualizations (PCA, USVA, tSNE)
  fail to clearly separate those outlier cells in the embedding.
when_to_use_negative:
- When the surrogate-variable marker set has not yet been computed — this skill depends
  on an upstream find_markers step with the p-value and R-squared cutoffs already
  applied
- When the goal is de novo clustering or cell-type discovery rather than visualizing
  cells already flagged by a surrogate variable
- When the analysis target is differential expression or factor testing rather than
  2D embedding comparison
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0092
tools:
- name: Rtsne
  role: Computes the 2D tSNE embeddings (all-gene baseline and marker-gene projection)
    with set.seed(100) and dims = 2
- name: iasva
  role: Supplies the surrogate variable estimates (SV1), marker gene selection (find_markers),
    and the alpha-cell example dataset in package extdata
  repo: https://github.com/UcarLab/iasva
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41598-018-35365-9
    title: Detection of correlated hidden factors from single cell transcriptomes
      using Iteratively Adjusted-SVA (IA-SVA)
  source_content_hash: sha256:ecd2ef98746bbb8979630cc9ddc8de8aec1374380c923491464c20c2adf0cab0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/UcarLab/iasva
  tools:
  - Rtsne
  - iasva
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# marker-gene-based-cell-visualization

## Summary

Builds 2D tSNE embeddings from log-transformed IA-SVA-derived marker gene counts (adjusted p-value < 0.05, R-squared > 0.3) to make hidden heterogeneity — e.g. outlier alpha cells flagged by SV1 — visually separable where all-gene PCA/tSNE projections fail.

## When to use

Use when IA-SVA (or similar SV analysis) has identified outlier cells via a surrogate variable, but standard all-gene visualizations (PCA, USVA, tSNE) fail to clearly separate those outlier cells in the embedding.

## When NOT to use

- When the surrogate-variable marker set has not yet been computed — this skill depends on an upstream find_markers step with the p-value and R-squared cutoffs already applied
- When the goal is de novo clustering or cell-type discovery rather than visualizing cells already flagged by a surrogate variable
- When the analysis target is differential expression or factor testing rather than 2D embedding comparison

## Inputs

- Read count matrix for the cell population of interest (e.g. iasva_counts_test.Rds, 101 alpha cells, Fluidigm C1, Lawlor et al. human pancreatic islet data)
- Marker gene count matrix associated with the surrogate variable (from find_markers, adjusted p-value < 0.05 and R-squared > 0.3)
- Surrogate variable values (e.g. SV1) used to label outlier vs. remaining cells

## Outputs

- tsne_all_genes.csv — 2D tSNE coordinates (101 rows x 2 dims) from all genes
- tsne_marker_genes.csv — 2D tSNE coordinates (101 rows x 2 dims) from marker genes
- tsne_all_genes.png — all-gene embedding scatter plot with outliers colored red
- tsne_marker_genes.png — marker-gene embedding scatter plot with outliers colored red

## How to apply

Load the read count matrix (e.g. iasva_counts_test.Rds with 101 human alpha cells from the iasva package extdata) and log-transform as log(counts + 1). Load marker gene counts previously selected against the surrogate variable using the find_markers cutoffs (adjusted p-value < 0.05, R-squared > 0.3) and log-transform them identically. Run Rtsne (dims = 2, default settings) on the transposed log-count matrices with set.seed(100) for both the all-gene baseline and the marker-gene projection. Define cell labels by thresholding SV1 (e.g. at -0.1 to call six outlier cells) and color outlier cells red versus remaining cells blue in Dim1 vs Dim2 scatter plots. The rationale is that genes captured by IA-SVA for a hidden factor recapitulate the signal that all-gene embeddings dilute, improving separation of the affected cell group.

## Related tools

- **Rtsne** (Computes the 2D tSNE embeddings (all-gene baseline and marker-gene projection) with set.seed(100) and dims = 2)
- **iasva** (Supplies the surrogate variable estimates (SV1), marker gene selection (find_markers), and the alpha-cell example dataset in package extdata) — https://github.com/UcarLab/iasva

## Examples

```
set.seed(100); tsne_markers <- Rtsne(t(log(marker.counts + 1)), dims = 2); write.csv(tsne_markers$Y, "tsne_marker_genes.csv")
```

## Evaluation signals

- All four output files exist: tsne_all_genes.csv, tsne_marker_genes.csv, tsne_all_genes.png, tsne_marker_genes.png
- Both coordinate tables contain 101 rows with exactly 2 dimensions
- The all-gene tSNE plot fails to isolate the 6 outlier cells while the marker-gene tSNE plot separates them into a distinct cluster
- Cell type labeling matches thresholding SV1 at -0.1 (six outlier cells colored red)
- Both tSNE runs use set.seed(100) and identical default Rtsne settings so the baseline/projection contrast is reproducible

## Limitations

- All-gene embeddings (PCA, USVA, tSNE) may fail to clearly detect outlier alpha cells that IA-SVA identified via its surrogate variables, so the baseline visualization alone is insufficient
- The outlier/outlier-cell boundary depends on an arbitrary SV threshold (e.g. -0.1); users must select appropriate cutoffs, and IA-SVA's significance index caveats remain for determining the number of hidden factors
- Compromised alpha cells detected this way should be excluded from downstream analyses, so this visualization is a flagging/QC step, not a final cell-type assignment

## Evidence

- [methods] marker gene selection: "marker genes that are significantly associated with SV1 (multiple testing adjusted p-value < 0.05, default significance cutoff, and R-squared value > 0.3"
- [discussion] genes improve visualization: "genes captured by IA-SVA can be used to improve the visualization of single cells into their respective clusters"
- [methods] dataset provenance: "we use read counts of alpha cells from healthy (non-diabetic) subjects (n = 101)"
- [results] baseline visualization fails: "we applied recently developed visualization methods, Cell View27 and Spectral tSNE16"
- [readme] package availability: [evidence span withheld — anchor status: external_doc]
