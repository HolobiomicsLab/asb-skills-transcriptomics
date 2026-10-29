---
name: gene-detection-threshold-filtering
description: Use when a raw digital gene expression matrix (e.g., Cell Ranger output
  UMI counts) has not been filtered or normalized, signaled by indication='none' in
  scTITANS_full, and a cleaned, normalized matrix is needed before trajectory inference.
when_to_use_negative:
- The matrix has already been gene- and cell-filtered (indication='filtered') — only
  normalization is then needed.
- The matrix is already filtered and normalized (indication='normalized') — applying
  this skill would double-filter and re-normalize, biasing downstream pseudotime estimation.
- The user has completed preprocessing and trajectory inference and should use scTITANS
  'partial' mode instead of the full preprocessing path.
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scTITANS
  role: Pipeline that applies the gene/cell detection-threshold filtering as its preprocessing
    stage in 'full' mode when indication='none'
  repo: https://github.com/ZJUFanLab/scTITANS
- name: scran
  role: R package that normalizes the filtered matrix using the default pool-and-deconvolute
    normalization algorithm
- name: Cell Ranger
  role: Upstream source of the raw digital gene expression matrices this filtering
    is applied to
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.csbj.2021.07.016
    title: Identify differential genes and cell subclusters from time-series scRNA-seq
      data using scTITANS
  source_content_hash: sha256:feb68ba6781aa568a18b5fdbfe95ac615888e4fdd48f888c0a9837f348cf4bbb
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
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# gene-detection-threshold-filtering

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Preprocessing skill from the scTITANS pipeline that removes uninformative genes and cells from a raw UMI count matrix (genes as rows, cells as columns) using explicit detection thresholds, then hands the filtered matrix to scran pool-and-deconvolute normalization.

## When to use

Use when a raw digital gene expression matrix (e.g., Cell Ranger output UMI counts) has not been filtered or normalized, signaled by indication='none' in scTITANS_full, and a cleaned, normalized matrix is needed before trajectory inference.

## When NOT to use

- The matrix has already been gene- and cell-filtered (indication='filtered') — only normalization is then needed.
- The matrix is already filtered and normalized (indication='normalized') — applying this skill would double-filter and re-normalize, biasing downstream pseudotime estimation.
- The user has completed preprocessing and trajectory inference and should use scTITANS 'partial' mode instead of the full preprocessing path.

## Inputs

- Raw digital gene expression matrix (UMI counts per gene per cell; row genes, column cells), e.g., from Cell Ranger (10X Genomics)
- indication parameter set to 'none' (matrix is neither filtered nor normalized)

## Outputs

- Filtered and scran-normalized expression matrix written to normalized_expression_matrix.csv
- Matrix containing only genes passing the ≥2 transcripts in ≥10 cells detection rule and only cells within the 5th–95th percentile gene-count range and ≤10% mitochondrial UMI fraction

## How to apply

Load the raw digital gene expression matrix (UMI counts per gene per cell) with pandas from the user-supplied file. Apply gene filtering: drop genes without any counts in any cell, then retain only genes detected — defined as present with two or more transcripts — in at least 10 cells. Apply cell filtering: discard cells whose number of detected genes falls outside the 5th and 95th percentiles, and remove cells with more than 10% of their UMIs assigned to mitochondrial genes. Normalize the filtered matrix with the scran R package using its default pool-and-deconvolute implementation (via rpy2 or an equivalent Python scran implementation). Write the result as normalized_expression_matrix.csv; in the scTITANS_full workflow this corresponds to calling with indication='none' so the tool performs filtering plus normalization internally.

## Related tools

- **scTITANS** (Pipeline that applies the gene/cell detection-threshold filtering as its preprocessing stage in 'full' mode when indication='none') — https://github.com/ZJUFanLab/scTITANS
- **scran** (R package that normalizes the filtered matrix using the default pool-and-deconvolute normalization algorithm)
- **Cell Ranger** (Upstream source of the raw digital gene expression matrices this filtering is applied to)

## Examples

```
result = scTITANS_full(data_file,"none","none","none","F","","","none",0.01)
```

## Evaluation signals

- No gene in the output matrix has zero counts across all cells; every retained gene has ≥2 transcripts in ≥10 cells.
- Every retained cell's detected-gene count lies between the 5th and 95th percentiles of the original distribution.
- Every retained cell has ≤10% of its UMIs assigned to mitochondrial genes.
- normalized_expression_matrix.csv exists and parses as a valid matrix with genes as rows and cells as columns.
- Qualitative QC only — the paper reports no numeric accuracy criterion for this preprocessing stage.

## Limitations

- This skill is only correct for a truly raw matrix; mislabeling an already-filtered or normalized matrix as 'none' would apply thresholds twice.
- The ≥10-cell detection rule, 5th–95th percentile cell window, and 10% mitochondrial UMI cutoff are fixed in the paper; no software version numbers for scran or scTITANS are stated.
- Validation at this stage is qualitative; the paper reports no numeric accuracy criterion for filtering/normalization.

## Evidence
