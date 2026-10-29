---
name: weighted-vs-unweighted-artifact-comparison-visualization
description: Use when metacell-level (rather than single-cell-level) differential
  expression, differential abundance, or dimensionality reduction is planned and metacells
  have unequal sizes, to quantify and visualize the type I/II errors introduced by
  disregarding metacell size weights.
when_to_use_negative:
- When metacells are constructed with a fixed target size so all metacells contain
  approximately the same number of cells — weighting then has little effect
- When the analysis goal is single-cell-level differential expression, where the single-cell
  reference itself is the output and no metacell weighting is involved
- When building or quality-controlling metacells (purity, compactness, separation,
  INV) rather than performing weighted downstream analyses
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: MetacellAnalysisToolkit
  role: Provides the CD34_sc/CD34_mc example Seurat objects, the cell-to-metacell
    membership table, and the vignette pipeline for weighted vs unweighted metacell-level
    analysis
  repo: https://github.com/GfellerLab/MetacellAnalysisToolkit
- name: Seurat
  role: Runs metacell-level differential expression, PCA, and clustering on the aggregated
    metacell expression matrix in both weighting modes
- name: knitr
  role: Vignette engine used to render the comparison figures (rmarkdown/md_document
    output)
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s44320-024-00045-6
    title: Building and analyzing metacells in single-cell genomics data
  source_content_hash: sha256:5aed0c6242e89020598308998a0ca914147e3fc9efdd5d52d52aeb57288055f9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/GfellerLab/MetacellAnalysisToolkit
  tools:
  - MetacellAnalysisToolkit
  - Seurat
  - knitr
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# weighted-vs-unweighted-artifact-comparison-visualization

## Summary

Reproduces and visualizes the downstream-analysis artifacts that arise when metacell size (number of constituent single cells) is ignored in metacell-level differential expression, differential abundance, and PCA/clustering analyses. It contrasts unweighted vs size-weighted results against a single-cell-level reference to expose type I (Gene B) and type II (Gene C) errors, as in Fig 7B.

## When to use

Use when metacell-level (rather than single-cell-level) differential expression, differential abundance, or dimensionality reduction is planned and metacells have unequal sizes, to quantify and visualize the type I/II errors introduced by disregarding metacell size weights.

## When NOT to use

- When metacells are constructed with a fixed target size so all metacells contain approximately the same number of cells — weighting then has little effect
- When the analysis goal is single-cell-level differential expression, where the single-cell reference itself is the output and no metacell weighting is involved
- When building or quality-controlling metacells (purity, compactness, separation, INV) rather than performing weighted downstream analyses

## Inputs

- Single-cell Seurat object (e.g. MetacellAnalysisToolkit::CD34_sc) with cell-type annotations and normalization
- Metacell Seurat object (e.g. MetacellAnalysisToolkit::CD34_mc) with aggregated expression matrix
- Cell-to-metacell membership table (mc_data@misc$cell_membership$membership)
- Condition labels for the two comparison conditions

## Outputs

- Unweighted vs weighted metacell-level differential expression results per cell type (log fold-changes, p-values)
- Unweighted vs weighted cell-type differential abundance estimates
- PCA/clustering embeddings of the metacell expression matrix with and without size weights
- Type I/II error quantification against the single-cell-level reference
- Four comparison panels (Fig 7A-D equivalent) rendered as PNGs in './MetacellAnalysisToolkit_vignette_files/'

## How to apply

Load the single-cell and metacell Seurat objects plus the cell-to-metacell membership table shipped with MetacellAnalysisToolkit (e.g. CD34_sc and CD34_mc). Run metacell-level differential expression per cell type on the aggregated metacell expression matrix with Seurat in two modes: unweighted and weighted by metacell size (number of constituent single cells). Compute cell-type differential abundance at the metacell level under the same two weighting schemes, and perform PCA and clustering on the metacell expression matrix with and without size weights. Compare every unweighted and weighted metacell-level result against the single-cell-level reference to quantify type I/II differential-expression errors and distorted abundance estimates, then render the four comparison panels (Fig 7A-D equivalent) as PNG figures in the vignette figure directory (fig.path = './MetacellAnalysisToolkit_vignette_files/'). The rationale is that a metacell aggregates a variable number of cells, so treating all metacells as equal-weight observations distorts condition-level mean comparisons.

## Related tools

- **MetacellAnalysisToolkit** (Provides the CD34_sc/CD34_mc example Seurat objects, the cell-to-metacell membership table, and the vignette pipeline for weighted vs unweighted metacell-level analysis) — https://github.com/GfellerLab/MetacellAnalysisToolkit
- **Seurat** (Runs metacell-level differential expression, PCA, and clustering on the aggregated metacell expression matrix in both weighting modes)
- **knitr** (Vignette engine used to render the comparison figures (rmarkdown/md_document output))

## Examples

```
library(MetacellAnalysisToolkit); sc_data <- CD34_sc; mc_data <- CD34_mc; membership <- mc_data@misc$cell_membership$membership
```

## Evaluation signals

- Weighted metacell-level differential expression and differential abundance reproduce the single-cell-level reference results (acceptance condition)
- Unweighted analyses display the reported artifacts: spurious differential expression calls (type I error, Gene B) and missed calls (type II error, Gene C) as in Fig 7B
- Unweighted cell-type abundance estimates are visibly distorted relative to the single-cell reference in the rendered comparison figures
- All four comparison panels (Fig 7A-D equivalent) are rendered as PNG files in './MetacellAnalysisToolkit_vignette_files/'
- Each metacell's weight equals its number of constituent single cells as recorded in the membership table

## Limitations

- Artifacts depend on metacell size heterogeneity; if all metacells are of similar size, weighted and unweighted results will agree and no error signal is demonstrable
- Comparison is only meaningful against a single-cell-level reference computed on the same data and annotations
- The MetacellAnalysisToolkit README ships no changelog; tool/API behavior may differ across versions

## Evidence

- [results] Disregarding metacell size when comparing gene mean expression between conditions creates artifacts that can lead to type I errors (Gene B) or type II errors (Gene C), as shown in Fig 7B.: [evidence span withheld — anchor status: not_found]
- [methods] mc_data <- MetacellAnalysisToolkit::CD34_mc: "mc_data <- MetacellAnalysisToolkit::CD34_mc"
- [abstract] a fully integrated pipeline to rapidly build, visualize and evaluate metacells with different methods: "a fully integrated pipeline to rapidly build, visualize and evaluate metacells with different methods"
- [readme] Toolkit for metacell analysis. It consists of the `MATK` command line tool to easily identify metacells: "Toolkit for metacell analysis. It consists of the `MATK` command line tool to easily identify metacells with either [SEACells](https://github.com/dpeerlab/SEACells)"
- [methods] sc_data <- NormalizeData(sc_data, normalization.method = "LogNormalize"): "sc_data <- NormalizeData(sc_data, normalization.method = "LogNormalize")"
