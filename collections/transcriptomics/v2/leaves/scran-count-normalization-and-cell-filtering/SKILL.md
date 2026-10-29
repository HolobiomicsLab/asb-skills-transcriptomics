---
name: scran-count-normalization-and-cell-filtering
description: Use when a raw UMI count matrix (e.g., from Cell Ranger or Fluidigm C1
  scRNA-seq) must be cleaned and normalized before trajectory inference, time-series
  differential expression, or cell-subcluster identification.
when_to_use_negative:
- The matrix has already been filtered and normalized — rerunning the filters would
  double-filter cells; pass indication='normalized' to scTITANS instead of reprocessing.
- The analysis goal is single-cell data from platforms not compatible with UMI-based
  scran normalization assumptions (the article applies this only to UMI matrices from
  Fluidigm C1 and Cell Ranger).
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scran
  role: R package used to normalize the filtered count matrix with the default pool-and-deconvolute
    normalization algorithm
  repo: https://github.com/ZJUFanLab/scTITANS
- name: scTITANS
  role: Downstream tool whose 'full' mode performs this filtering and normalization
    (indication='none') before trajectory inference and DEG identification
  repo: https://github.com/ZJUFanLab/scTITANS
- name: Cell Ranger
  role: Upstream tool producing the raw digital gene expression matrices that serve
    as input to the filtering/normalization step
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.csbj.2021.07.016
    title: Identify differential genes and cell subclusters from time-series scRNA-seq
      data using scTITANS
  source_content_hash: sha256:7e80c3bc04e4af28452f34cc41f20c1fd33e836f4895d05601c8d67c2acfcd1c
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/ZJUFanLab/scTITANS
  repo_url: https://github.com/ZJUFanLab/scTITANS
  tools:
  - Cell Ranger
  - scTITANS
  - scran
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# scran-count-normalization-and-cell-filtering

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Preprocessing pipeline from the scTITANS workflow that filters genes and cells from a raw UMI count matrix and normalizes it with scran's pool-and-deconvolute algorithm, producing a cleaned matrix ready for trajectory inference and time-dependent covariate analysis.

## When to use

Use when a raw UMI count matrix (e.g., from Cell Ranger or Fluidigm C1 scRNA-seq) must be cleaned and normalized before trajectory inference, time-series differential expression, or cell-subcluster identification.

## When NOT to use

- The matrix has already been filtered and normalized — rerunning the filters would double-filter cells; pass indication='normalized' to scTITANS instead of reprocessing.
- The analysis goal is single-cell data from platforms not compatible with UMI-based scran normalization assumptions (the article applies this only to UMI matrices from Fluidigm C1 and Cell Ranger).

## Inputs

- Raw gene expression count matrix (rows = genes, columns = cells), e.g., from Cell Ranger or Fluidigm C1 UMI data
- Optional cell metadata file (rows = cells)
- Status indicator for the matrix ('none' = raw, 'filtered', 'normalized')

## Outputs

- Filtered count matrix with zero-count genes removed, low-quality cells (outside 5th–95th percentile of detected genes or >10% mitochondrial UMIs) discarded
- Normalized expression matrix (scran pool-and-deconvolute normalization)

## How to apply

Start from the raw gene-by-cell count matrix (genes in rows, cells in columns). First filter genes with zero counts in all cells, and define a gene as detected when it has two or more transcripts in at least 10 cells. Then discard cells whose number of detected genes falls outside the 5th–95th percentile and filter cells with more than 10% of their UMIs assigned to mitochondrial genes. Finally normalize the filtered matrix with the scran R package using the default implementation of the pool-and-deconvolute normalization algorithm. When passing the matrix onward, record its status: scTITANS accepts 'none' for raw data, 'filtered' for filtered data, or 'normalized' for filtered-and-normalized data, so the indication parameter must match the preprocessing actually performed.

## Related tools

- **scran** (R package used to normalize the filtered count matrix with the default pool-and-deconvolute normalization algorithm) — https://github.com/ZJUFanLab/scTITANS
- **scTITANS** (Downstream tool whose 'full' mode performs this filtering and normalization (indication='none') before trajectory inference and DEG identification) — https://github.com/ZJUFanLab/scTITANS
- **Cell Ranger** (Upstream tool producing the raw digital gene expression matrices that serve as input to the filtering/normalization step)

## Examples

```
result = scTITANS_full(data_file,"none","none","none","F","","","none",0.01)
```

## Evaluation signals

- No genes with zero counts across all cells remain in the filtered matrix.
- Remaining genes are detected (≥2 transcripts) in at least 10 cells.
- No retained cell falls outside the 5th–95th percentile of the detected-genes distribution, and no cell has >10% of its UMIs assigned to mitochondrial genes.
- The normalized matrix passes scran's pool-and-deconvolute normalization without error and the matrix-status indication ('none'/'filtered'/'normalized') matches the actual preprocessing state.
- On the HSMM reference dataset (GSE52529, 271 cells), the preprocessed matrix supports a downstream scTITANS_full run at fdr_level=0.01 that returns SigGenes.fdr0.01.txt and SigClusters.fdr0.01.txt without error.

## Limitations

- The 5th–95th percentile cell filter and the ≥10-cell/≥2-transcript gene rule are dataset-agnostic defaults; datasets with unusual cell-size distributions (e.g., heterogeneous mixes like MRD with low-count cell types such as RPE/margin/periocular mesenchyme/lens epithelial cells) may need type-aware filtering.
- Software version numbers for scran (and scTITANS/Monocle3) are not stated in the article, so exact reproduction requires checking the repository.
- scTITANS_full in 'full' mode assumes the matrix is raw UMI counts when indication='none'; providing already-normalized data with the wrong indication will re-process incorrectly.

## Evidence
