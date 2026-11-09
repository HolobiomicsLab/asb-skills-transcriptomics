---
name: marker-gene-percentile-selection
description: Use when preparing positive/negative marker files for BIDCell training
  on spatial transcriptomics data (e.g., Xenium breast cancer), or when defining cell-type
  markers by expression percentile cutoffs from a single-cell reference.
when_to_use_negative:
- The spatial dataset already ships with curated marker files (manual curation is
  an accepted alternative, so re-deriving percentiles may be unnecessary).
- The tissue of interest has no matching single-cell reference in repositories such
  as TISCH2, Allen Brain Map, or the Human Cell Atlas, so percentile ranking cannot
  be grounded in cell-type-specific expression.
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_2229
tools:
- name: BIDCell
  role: Consumes the generated positive/negative marker files as biologically-informed
    supervision for segmentation training; marker count influences predicted cell
    size
  repo: https://github.com/SydneyBioX/BIDCell
- name: SydneyBioX/scClassify
  role: Source of example single-cell reference data that can be used to derive marker
    files
  repo: https://github.com/SydneyBioX/scClassify
- name: TISCH2
  role: Public repository for downloading single-cell reference data (e.g., sc_breast.csv)
    from which percentile markers are computed
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-023-44560-w
    title: 'BIDCell: Biologically-informed self-supervised learning for segmentation
      of subcellular spatial transcriptomics data'
  source_content_hash: sha256:e867e6bcc9d531ccbba2d07140552a6f0c15fa4d51119ef48c0a234deaaa9a70
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SydneyBioX/BIDCell
  tools:
  - BIDCell
  - SydneyBioX/scClassify
  - TISCH2
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# marker-gene-percentile-selection

## Summary

Derives positive and negative cell-type marker gene lists from a single-cell reference by selecting genes with expression in the highest and lowest 10 percentile per cell type, then removing positive markers shared by at least a third of cell types. These marker files drive BIDCell's biologically-informed loss functions and enable expression-purity/neighbour-contamination evaluation.

## When to use

Use when preparing positive/negative marker files for BIDCell training on spatial transcriptomics data (e.g., Xenium breast cancer), or when defining cell-type markers by expression percentile cutoffs from a single-cell reference.

## When NOT to use

- The spatial dataset already ships with curated marker files (manual curation is an accepted alternative, so re-deriving percentiles may be unnecessary).
- The tissue of interest has no matching single-cell reference in repositories such as TISCH2, Allen Brain Map, or the Human Cell Atlas, so percentile ranking cannot be grounded in cell-type-specific expression.

## Inputs

- Average-expression single-cell reference csv (e.g., sc_breast.csv) with cell-type average expression for all genes in the spatial dataset
- Choice of cell types to include for the tissue (e.g., breast cancer reference from TISCH2)

## Outputs

- Positive marker file (sc_breast_markers_pos.csv) listing top-10-percentile genes per cell type after removing markers common to >=1/3 of cell types
- Negative marker file (sc_breast_markers_neg.csv) listing bottom-10-percentile genes per cell type

## How to apply

Start from an average-expression single-cell reference csv (e.g., sc_breast.csv from TISCH2 for breast cancer) containing expression for all genes in the spatial dataset per cell type. For each cell type, define positive markers as genes with expression in the highest 10 percentile and negative markers as genes with expression in the lowest 10 percentile, writing them to sc_breast_markers_pos.csv and sc_breast_markers_neg.csv. Remove positive markers that are common to at least a third of the cell types in the dataset, as this was found appropriate across various datasets. Note that using a larger number of positive markers tends to increase the size of predicted cells, so the marker list size is a tunable decision point; only <1,000 genes are needed for segmentation, and manual curation or alternative marker-determination approaches are also acceptable.

## Related tools

- **BIDCell** (Consumes the generated positive/negative marker files as biologically-informed supervision for segmentation training; marker count influences predicted cell size) — https://github.com/SydneyBioX/BIDCell
- **SydneyBioX/scClassify** (Source of example single-cell reference data that can be used to derive marker files) — https://github.com/SydneyBioX/scClassify
- **TISCH2** (Public repository for downloading single-cell reference data (e.g., sc_breast.csv) from which percentile markers are computed)

## Examples

```
pos = ref.quantile(0.9, axis=1); markers_pos = {ct: ref.columns[ref.loc[ct] >= pos[ct]].tolist() for ct in ref.index}  # save to sc_breast_markers_pos.csv
```

## Evaluation signals

- Marker files contain, per cell type, exactly those genes with expression in the highest (positive) and lowest (negative) 10 percentile of the reference.
- Positive markers common to at least a third of cell types are excluded from the positive marker file.
- Total gene count across marker/reference selection stays under 1,000 genes, the amount needed for segmentation.
- Downstream BIDCell runs using the files produce cells of sensible size; abnormally large predicted cells suggest too many positive markers were retained.
- Markers support the expected purity metrics (positive/negative purity F1 scores and reduced percentage of B cells expressing T-cell markers CD4, CD8A, CD8B, CD3D, CD3E) in the Fig. 3 evaluation workflow.

## Limitations

- Using a larger number of positive markers tends to increase the size of predicted cells, so marker list length must be tuned.
- Removing positive markers common to at least a third of cell types is a heuristic found appropriate across various datasets but is not guaranteed optimal for every tissue.
- Manual curation and alternative marker-determination approaches may be needed when percentile ranking is insufficient.

## Evidence

- [readme] Percentile cutoff definition: "The positive and negative markers were those with expressions in the highest and lowest 10 percentile for each cell type of a tissue sample."
- [readme] Common-marker removal heuristic: "We found that removing positive markers that were common to at least a third of cell types in each dataset was appropriate across various datasets."
- [readme] Marker count effect and gene budget: "Using a larger number of positive markers tends to increase the size of predicted cells. Manual curation and alternative approaches to determine the marker genes can also be used."
- [readme] Gene budget for segmentation: "Only <1,000 genes are needed to perform segmentation. Specify a selection of genes in a file"
- [methods] Reference file provenance: "Please see the provided breast cancer single-cell reference and positive/negative marker files (`sc_breast.csv`, `sc_breast_markers_pos.csv`, and `sc_breast_markers_neg.csv`)"
