---
name: single-cell-umi-count-matrix-filtering
description: Use when the input is a raw (unfiltered, unnormalized) gene-by-cell UMI
  count matrix, e.g. a Cell Ranger raw digital gene expression matrix, that must be
  cleaned and normalized before trajectory or downstream scRNA-seq analysis.
when_to_use_negative:
- The matrix has already been filtered by the user — pass indication='filtered' instead
  and skip this stage.
- The matrix has already been filtered and normalized — pass indication='normalized'
  and go directly to trajectory inference.
- The user has already completed preprocessing and trajectory inference — use scTITANS_partial
  mode rather than the full-mode preprocessing pipeline.
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scTITANS
  role: Full-mode wrapper whose 'indication' parameter triggers this filtering and
    normalization preprocessing stage on raw count matrices
  repo: https://github.com/ZJUFanLab/scTITANS
- name: scran
  role: R package used for normalization of the filtered matrix via the default pool-and-deconvolute
    algorithm
- name: Cell Ranger
  role: Typical upstream tool producing the raw digital gene expression matrices that
    this skill consumes
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.csbj.2021.07.016
    title: Identify differential genes and cell subclusters from time-series scRNA-seq
      data using scTITANS
  source_content_hash: sha256:8285dbef508944b50c59e5c03dd39b9481da40dadabd489ee9eac5cb2e071b94
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
yes_rate: 0.75
n_claims: 8
---

# single-cell-umi-count-matrix-filtering

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Preprocess a raw single-cell digital gene expression (UMI count) matrix by gene filtering, cell quality filtering, and scran pool-and-deconvolute normalization, as implemented in the scTITANS 'full' mode when the 'indication' parameter is set to 'none'.

## When to use

Use when the input is a raw (unfiltered, unnormalized) gene-by-cell UMI count matrix, e.g. a Cell Ranger raw digital gene expression matrix, that must be cleaned and normalized before trajectory or downstream scRNA-seq analysis.

## When NOT to use

- The matrix has already been filtered by the user — pass indication='filtered' instead and skip this stage.
- The matrix has already been filtered and normalized — pass indication='normalized' and go directly to trajectory inference.
- The user has already completed preprocessing and trajectory inference — use scTITANS_partial mode rather than the full-mode preprocessing pipeline.

## Inputs

- Raw digital gene expression matrix of UMI counts per gene per cell (rows = genes, columns = cells), e.g. a Cell Ranger output file loaded with pandas
- indication parameter of scTITANS_full set to 'none' (raw, unfiltered and unnormalized matrix)

## Outputs

- Filtered and scran-normalized expression matrix written to normalized_expression_matrix.csv

## How to apply

Load the raw digital gene expression matrix (rows = genes, columns = cells) from the input file with pandas. Filter genes first by removing genes without any counts in any cell, then keep only genes detected as present with two or more transcripts in at least 10 cells. Filter cells by discarding those whose number of detected genes falls outside the 5th and 95th percentiles, and removing cells with more than 10% of their UMIs assigned to mitochondrial genes. Normalize the filtered matrix with the scran R package using its default pool-and-deconvolute normalization algorithm (via rpy2 or an equivalent Python implementation), and write the result to normalized_expression_matrix.csv. This corresponds to the scTITANS full-mode preprocessing branch triggered when 'indication' is 'none'; if the user already filtered, use 'filtered', and if filtered and normalized, use 'normalized' to skip this stage.

## Related tools

- **scTITANS** (Full-mode wrapper whose 'indication' parameter triggers this filtering and normalization preprocessing stage on raw count matrices) — https://github.com/ZJUFanLab/scTITANS
- **scran** (R package used for normalization of the filtered matrix via the default pool-and-deconvolute algorithm)
- **Cell Ranger** (Typical upstream tool producing the raw digital gene expression matrices that this skill consumes)

## Examples

```
result = scTITANS_full(data_file,"none","none","none","F","","","none",0.01)
```

## Evaluation signals

- Output file normalized_expression_matrix.csv exists and parses as a valid gene-by-cell matrix.
- Every retained gene passes the detection rule: present with two or more transcripts in at least 10 cells (after removing all-zero genes).
- Every retained cell has a number of detected genes within the 5th-95th percentile range of the dataset.
- Every retained cell has at most 10% of its UMIs assigned to mitochondrial genes.
- Normalized values are consistent with scran pool-and-deconvolute size-factor scaling (qualitative QC; the paper reports no numeric accuracy criterion for this stage).

## Limitations

- Validation at this stage is a qualitative QC check; the paper reports no numeric accuracy criterion for filtering/normalization.
- Software version numbers for tools such as scran and scTITANS are not stated in the article, so exact reproducibility of the normalization may vary across versions.
- The percentile- and mitochondrial-threshold cutoffs (5th-95th percentile, 10% mitochondrial UMIs) are fixed defaults; datasets with different quality profiles may need care.
- If the matrix is already filtered or normalized, re-running this stage would double-process the data; the 'indication' parameter must be set accordingly.

## Evidence
