---
name: mitochondrial-umi-quality-control
description: Use when preprocessing a raw single-cell UMI count matrix (e.g., from
  Cell Ranger) where low-quality cells enriched for mitochondrial transcripts must
  be removed before normalization and trajectory inference.
when_to_use_negative:
- The input matrix has already been filtered by the user — in scTITANS full mode this
  corresponds to setting 'indication' to 'filtered' or 'normalized', so QC should
  not be re-applied
- The data are not UMI-based raw counts (e.g., already-normalized expression values),
  where the 10% mitochondrial UMI threshold is not meaningful
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scTITANS
  role: Full-mode pipeline that applies the mitochondrial UMI cell filter (plus gene
    and percentile cell filters) when 'indication' is 'none' (raw matrix)
  repo: https://github.com/ZJUFanLab/scTITANS
- name: scran
  role: Normalizes the matrix downstream of the mitochondrial/percentile QC filtering
    using the default pool-and-deconvolute algorithm
- name: Cell Ranger
  role: Source of the raw digital gene expression matrices on which the QC filtering
    is performed
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.csbj.2021.07.016
    title: Identify differential genes and cell subclusters from time-series scRNA-seq
      data using scTITANS
  source_content_hash: sha256:bc04f90c3ddd035690d83d5fa0de818af54531127c05fe6bbc44d1993311f74a
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
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# mitochondrial-umi-quality-control

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A cell-level QC step in the scTITANS preprocessing workflow that discards cells with more than 10% of their UMIs assigned to mitochondrial genes, as part of filtering a raw digital gene expression matrix (UMI counts per gene per cell) prior to scran normalization.

## When to use

Use when preprocessing a raw single-cell UMI count matrix (e.g., from Cell Ranger) where low-quality cells enriched for mitochondrial transcripts must be removed before normalization and trajectory inference.

## When NOT to use

- The input matrix has already been filtered by the user — in scTITANS full mode this corresponds to setting 'indication' to 'filtered' or 'normalized', so QC should not be re-applied
- The data are not UMI-based raw counts (e.g., already-normalized expression values), where the 10% mitochondrial UMI threshold is not meaningful

## Inputs

- Raw digital gene expression matrix (UMI counts, rows = genes, columns = cells) from tools such as Cell Ranger (10X Genomics)
- Gene identifiers or gene metadata sufficient to identify mitochondrial genes

## Outputs

- Filtered expression matrix retaining only cells whose detected-gene count lies within the 5th-95th percentile range and whose mitochondrial UMI fraction is ≤ 10%
- Feeds into normalized_expression_matrix.csv after scran normalization

## How to apply

Load the raw digital gene expression matrix (genes as rows, cells as columns) with pandas. Apply the companion gene filter first (drop genes with no counts; keep genes with ≥2 transcripts in at least 10 cells), then the cell filter: discard cells whose number of detected genes falls outside the 5th and 95th percentiles, and remove cells with more than 10% of their UMIs mapped to mitochondrial genes. Identify mitochondrial genes from gene identifiers/annotation and compute per-cell mitochondrial UMI fraction; threshold at 0.10 as the discard cutoff. Rationale: high mitochondrial UMI fractions signal stressed or damaged cells and are filtered before the matrix is normalized with the scran pool-and-deconvolute algorithm. In the scTITANS full mode this QC runs automatically when the 'indication' parameter is 'none' (raw, unfiltered matrix).

## Related tools

- **scTITANS** (Full-mode pipeline that applies the mitochondrial UMI cell filter (plus gene and percentile cell filters) when 'indication' is 'none' (raw matrix)) — https://github.com/ZJUFanLab/scTITANS
- **scran** (Normalizes the matrix downstream of the mitochondrial/percentile QC filtering using the default pool-and-deconvolute algorithm)
- **Cell Ranger** (Source of the raw digital gene expression matrices on which the QC filtering is performed)

## Examples

```
result = scTITANS_full(data_file,"none","none","none","F","","","none",0.01)
```

## Evaluation signals

- No retained cell has more than 10% of its UMIs assigned to mitochondrial genes
- Retained cells' detected-gene counts all fall within the 5th-95th percentile range of the input matrix
- Retained genes satisfy the detection rule: no zero-count genes and ≥2 transcripts in at least 10 cells
- Output file normalized_expression_matrix.csv exists and is a valid matrix consistent with the filtering and scran normalization steps (qualitative QC; the paper reports no numeric accuracy criterion for this stage)

## Limitations

- The 10% mitochondrial UMI cutoff and 5th-95th percentile gene-count window are fixed defaults in scTITANS, with no user-adjustable thresholds described
- The paper reports no numeric accuracy criterion for this filtering stage — validation is a qualitative QC check
- Software version numbers for scTITANS, scran, and Monocle3 are not stated in the article

## Evidence
