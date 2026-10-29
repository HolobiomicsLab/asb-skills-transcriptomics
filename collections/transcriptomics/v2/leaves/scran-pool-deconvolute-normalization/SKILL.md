---
name: scran-pool-deconvolute-normalization
description: Use when a raw digital gene expression matrix (UMI counts, genes x cells,
  e.g. from Cell Ranger) has been gene/cell filtered but not normalized, signaled
  by 'indication' = 'none' or 'filtered' in scTITANS.
when_to_use_negative:
- The input matrix has already been filtered and normalized — scTITANS should be called
  with indication = 'normalized' and this skill must not re-normalize.
- Only trajectory inference has been done by the user — the scTITANS 'partial' mode
  (scTITANS_partial) is more suitable and normalization is not part of that path.
- Data are not UMI count data suited to scran's pool-and-deconvolute size factors
  (e.g. already TPM/FPKM-normalized matrices).
edam_operation: http://edamontology.org/operation_2930
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: scran
  role: R package providing the default pool-and-deconvolute normalization algorithm
    applied to the filtered UMI matrix
- name: scTITANS
  role: Pipeline that invokes the filtering and scran normalization stage in its 'full'
    mode before trajectory inference; driven by the 'indication' parameter
  repo: https://github.com/ZJUFanLab/scTITANS
provenance:
  source_task_ids:
  - task_001
  - task_004
  source_papers:
  - doi: 10.1016/j.csbj.2021.07.016
    title: Identify differential genes and cell subclusters from time-series scRNA-seq
      data using scTITANS
  source_content_hash: sha256:5e43174ed101fca0ee652dce277585bc535c472fd438bed4814a01d549490298
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
  - scTITANS
  - scran
  edam_operation: http://edamontology.org/operation_2930
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# scran-pool-deconvolute-normalization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Normalizes a filtered single-cell UMI count matrix with the scran R package's default pool-and-deconvolute size-factor algorithm, the normalization stage of the scTITANS preprocessing pipeline for time-series scRNA-seq data.

## When to use

Use when a raw digital gene expression matrix (UMI counts, genes x cells, e.g. from Cell Ranger) has been gene/cell filtered but not normalized, signaled by 'indication' = 'none' or 'filtered' in scTITANS.

## When NOT to use

- The input matrix has already been filtered and normalized — scTITANS should be called with indication = 'normalized' and this skill must not re-normalize.
- Only trajectory inference has been done by the user — the scTITANS 'partial' mode (scTITANS_partial) is more suitable and normalization is not part of that path.
- Data are not UMI count data suited to scran's pool-and-deconvolute size factors (e.g. already TPM/FPKM-normalized matrices).

## Inputs

- Raw digital gene expression matrix of UMI counts (rows = genes, columns = cells), e.g. from Cell Ranger
- scTITANS 'indication' parameter value ('none', 'filtered', or 'normalized') stating the matrix status

## Outputs

- Filtered and normalized expression matrix (e.g. normalized_expression_matrix.csv) for downstream scTITANS/Monocle3 analysis

## How to apply

Load the raw digital gene expression matrix (UMI counts per gene per cell) and first filter it: drop genes with no counts in any cell, keep genes with >=2 transcripts in at least 10 cells, discard cells outside the 5th-95th percentile of detected gene counts, and remove cells with >10% of UMIs assigned to mitochondrial genes. Then normalize the filtered matrix with the scran R package using its default implementation of the pool-and-deconvolute normalization algorithm (callable from Python via rpy2 or an equivalent Python scran implementation). This corresponds to the scTITANS 'full' mode behavior when the 'indication' parameter is 'none' (raw data) or 'filtered' (filtered but not normalized); only when indication is 'normalized' should normalization be skipped. Write the result as the normalized expression matrix (e.g. normalized_expression_matrix.csv) feeding into Monocle3 trajectory and pseudotime estimation in scTITANS.

## Related tools

- **scran** (R package providing the default pool-and-deconvolute normalization algorithm applied to the filtered UMI matrix)
- **scTITANS** (Pipeline that invokes the filtering and scran normalization stage in its 'full' mode before trajectory inference; driven by the 'indication' parameter) — https://github.com/ZJUFanLab/scTITANS

## Examples

```
library(scran); counts_norm <- computeSumFactors(counts_filtered); counts_norm <- log2(counts_norm + 1)
```

## Evaluation signals

- Output file (e.g. normalized_expression_matrix.csv) exists and parses as a valid gene x cell matrix.
- Retained genes satisfy the detection rule: >=2 transcripts present in at least 10 cells, and genes without any counts are removed.
- Retained cells fall within the 5th-95th percentile range of detected gene counts and have <=10% of UMIs assigned to mitochondrial genes.
- Normalization was skipped (not re-applied) when the scTITANS 'indication' parameter was set to 'normalized'.
- Qualitative QC only — the paper reports no numeric accuracy criterion for this preprocessing stage (downstream task validates DEG overlap percentages against Table S3/S4).

## Limitations

- Software version numbers for scran (and scTITANS/Monocle3) are not stated in the article or README, so exact reproducibility of the normalization is not pinned.
- The paper reports no numeric accuracy criterion for this preprocessing stage — validation is a qualitative QC check of filter thresholds.
- Cell filtering discards cells outside the 5th-95th percentile of detected genes, which may remove legitimate outlier cells with unusual gene counts.

## Evidence
