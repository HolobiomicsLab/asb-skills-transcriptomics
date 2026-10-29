---
name: metacell-level-differential-abundance-analysis
description: Use when comparing cell-type abundance or gene mean expression between
  conditions using aggregated metacell profiles, and you must ensure metacell size
  weights are applied to avoid inflated type I/II errors.
when_to_use_negative:
- Your analysis operates directly on the single-cell expression matrix — metacell
  aggregation and weighting are not needed and add unnecessary indirection
- You are building or quality-controlling metacells themselves (purity, compactness,
  separation, INV) rather than using pre-built metacells for downstream comparison
- Metacells in your dataset are highly homogeneous in size (uniform cell counts),
  where weighting has little effect and single-cell-level analysis may be simpler
edam_operation: http://edamontology.org/operation_3656
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: MetacellAnalysisToolkit
  role: Provides the metacell Seurat objects (CD34_mc), single-cell data (CD34_sc),
    cell-to-metacell membership table, and vignette scaffolding for the weighted vs
    non-weighted downstream comparison
  repo: https://github.com/GfellerLab/MetacellAnalysisToolkit
- name: Seurat
  role: Runs metacell-level differential expression per cell type on the aggregated
    metacell expression matrix, and PCA/clustering on the metacell expression matrix
  repo: https://github.com/satijalab/seurat
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s44320-024-00045-6
    title: Building and analyzing metacells in single-cell genomics data
  source_content_hash: sha256:5a9de6160cf3d9264c5cf696896c2348aaefcd1d6ece06220906295b1dd87bf3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/GfellerLab/MetacellAnalysisToolkit
  tools:
  - MetacellAnalysisToolkit
  - Seurat
  edam_operation: http://edamontology.org/operation_3656
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# metacell-level-differential-abundance-analysis

## Summary

Perform differential abundance and differential expression at the metacell level with Seurat, running each test in two modes — unweighted and weighted by metacell size (number of constituent single cells) — and compare both against single-cell-level reference results to expose type I/II artifacts.

## When to use

Use when comparing cell-type abundance or gene mean expression between conditions using aggregated metacell profiles, and you must ensure metacell size weights are applied to avoid inflated type I/II errors.

## When NOT to use

- Your analysis operates directly on the single-cell expression matrix — metacell aggregation and weighting are not needed and add unnecessary indirection
- You are building or quality-controlling metacells themselves (purity, compactness, separation, INV) rather than using pre-built metacells for downstream comparison
- Metacells in your dataset are highly homogeneous in size (uniform cell counts), where weighting has little effect and single-cell-level analysis may be simpler

## Inputs

- Single-cell Seurat object (e.g. MetacellAnalysisToolkit::CD34_sc)
- Metacell Seurat object (e.g. MetacellAnalysisToolkit::CD34_mc)
- Cell-to-metacell membership table (mc_data@misc$cell_membership$membership)
- Cell type annotations for single cells (sc_data$celltype)

## Outputs

- Weighted vs unweighted metacell-level differential expression results per cell type
- Weighted vs unweighted metacell-level differential abundance estimates per cell type
- Four comparison panels (Fig 7A-D equivalent) as PNG figures in the vignette figure directory (fig.path = './MetacellAnalysisToolkit_vignette_files/')

## How to apply

Load the single-cell and metacell Seurat objects (e.g. the CD34_sc and CD34_mc datasets shipped with MetacellAnalysisToolkit) together with the cell-to-metacell membership table. Run metacell-level differential expression per cell type with Seurat on the aggregated metacell expression matrix in two modes: unweighted, and weighted by metacell size (the number of constituent single cells per metacell). Compute cell-type differential abundance at the metacell level under the same two weighting schemes. Optionally perform PCA and clustering on the metacell expression matrix with and without size weights. The rationale: disregarding metacell size when comparing gene mean expression between conditions creates artifacts that can yield type I errors (Gene B) or type II errors (Gene C), as shown in Fig 7B. Validate by checking that the weighted analyses reproduce the single-cell-level reference while the unweighted analyses show spurious differential calls and distorted abundance estimates in the rendered comparison figures (Fig 7A-D equivalents).

## Related tools

- **MetacellAnalysisToolkit** (Provides the metacell Seurat objects (CD34_mc), single-cell data (CD34_sc), cell-to-metacell membership table, and vignette scaffolding for the weighted vs non-weighted downstream comparison) — https://github.com/GfellerLab/MetacellAnalysisToolkit
- **Seurat** (Runs metacell-level differential expression per cell type on the aggregated metacell expression matrix, and PCA/clustering on the metacell expression matrix) — https://github.com/satijalab/seurat

## Examples

```
sc_data <- MetacellAnalysisToolkit::CD34_sc; mc_data <- MetacellAnalysisToolkit::CD34_mc; mc_data$purity <- mc_purity(membership = mc_data@misc$cell_membership$membership, annotation = sc_data$celltype)
```

## Evaluation signals

- Weighted differential expression/abundance results reproduce the single-cell-level reference results
- Unweighted analyses display the reported artifacts: spurious differential expression calls (type I error, e.g. Gene B) and missed calls (type II error, e.g. Gene C)
- Weighting mode matches metacell size = number of constituent single cells, verifiable from the membership table counts
- Four comparison panels (Fig 7A-D equivalent) render as PNG files in './MetacellAnalysisToolkit_vignette_files/'

## Limitations

- Disregarding metacell size inflates type I and type II errors in downstream differential expression, distorting gene mean expression comparisons between conditions
- Differential abundance estimates are likewise distorted when metacell weights are omitted, so unweighted metacell-level summaries should not be treated as equivalent to single-cell-level references
- Differential expression should be computed per cell type; a global comparison ignoring cell type identity mixes heterogeneous expression programs and obscures the weighted/unweighted artifact

## Evidence

- [results] Disregarding metacell size when comparing gene mean expression between conditions creates artifacts that can lead to type I errors (Gene B) or type II errors (Gene C), as shown in Fig 7B.: [evidence span withheld — anchor status: not_found]
- [methods] sc_data <- NormalizeData(sc_data, normalization.method = "LogNormalize"): "sc_data <- NormalizeData(sc_data, normalization.method = "LogNormalize")"
- [methods] mc_data <- MetacellAnalysisToolkit::CD34_mc: "mc_data <- MetacellAnalysisToolkit::CD34_mc"
- [methods] sc_data <- MetacellAnalysisToolkit::CD34_sc: "sc_data <- MetacellAnalysisToolkit::CD34_sc"
- [methods] the **purity** of a metacell is the proportion of the most abundant cell type within the metacell: "the **purity** of a metacell is the proportion of the most abundant cell type within the metacell"
