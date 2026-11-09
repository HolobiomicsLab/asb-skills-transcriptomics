---
name: time-series-scrna-seq-filtering-and-normalization
description: Use when a time-series scRNA-seq analysis starts from a raw digital gene
  expression matrix (genes × cells) and the matrix must be filtered and normalized
  prior to trajectory inference (e.g., Monocle3) and pseudotime-based DEG/subcluster
  identification.
when_to_use_negative:
- The expression matrix has already been normalized — indicate 'normalized' instead
  of rerunning scran normalization
- The matrix has already been filtered but not normalized — indicate 'filtered' rather
  than redoing gene/cell filtering
- The dataset is bulk RNA-seq or a feature/abundance table rather than a single-cell
  UMI count matrix (genes × cells)
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_2229
- http://edamontology.org/topic_3170
tools:
- name: scTITANS
  role: Downstream pipeline whose 'full' mode invokes this filtering and normalization;
    the 'indication' parameter records whether the matrix is raw ('none'), filtered,
    or filtered and normalized
  repo: https://github.com/ZJUFanLab/scTITANS
- name: scran
  role: Performs normalization of the filtered matrix using the default pool-and-deconvolute
    algorithm
- name: Cell Ranger
  role: Source of the raw digital gene expression matrices that this filtering/normalization
    step consumes
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.csbj.2021.07.016
    title: Identify differential genes and cell subclusters from time-series scRNA-seq
      data using scTITANS
  source_content_hash: sha256:1937795be15505765975481afef5384992cf5b98a9ef107b46a8113d9e61953a
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
  - Destiny
  - GPfates
  - Monocle
  - Monocle2
  - Mpath
  - SLICER
  - STEMNET
  - Slingshot
  - TSCAN
  - scTITANS
  - scran
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_2229
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 18
---

# time-series-scrna-seq-filtering-and-normalization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Preprocessing recipe from the scTITANS pipeline for raw single-cell UMI matrices (e.g., Cell Ranger output): filter genes detected in ≥10 cells with ≥2 transcripts, discard low/high-quality cells and high-mitochondrial cells, then normalize with scran's pool-and-deconvolute algorithm before trajectory inference.

## When to use

Use when a time-series scRNA-seq analysis starts from a raw digital gene expression matrix (genes × cells) and the matrix must be filtered and normalized prior to trajectory inference (e.g., Monocle3) and pseudotime-based DEG/subcluster identification.

## When NOT to use

- The expression matrix has already been normalized — indicate 'normalized' instead of rerunning scran normalization
- The matrix has already been filtered but not normalized — indicate 'filtered' rather than redoing gene/cell filtering
- The dataset is bulk RNA-seq or a feature/abundance table rather than a single-cell UMI count matrix (genes × cells)

## Inputs

- Raw digital gene expression matrix (genes × cells, UMI counts, e.g., from Cell Ranger 10X Genomics)
- Optional cell metadata file (row cells) with cluster/cell type information
- Optional gene metadata file with a gene_short_name header column

## Outputs

- Filtered gene expression matrix (genes detected in ≥10 cells with ≥2 transcripts; cells within the 5th–95th percentile gene count range and <10% mitochondrial UMIs)
- Normalized expression matrix (scran pool-and-deconvolute) ready for trajectory inference

## How to apply

Start from the raw UMI/digital gene expression matrix (e.g., Cell Ranger output). Apply gene filtering: genes without any counts in any cell are dropped, and a gene is considered detected only if present with two or more transcripts in at least 10 cells. Apply cell filtering: discard cells whose number of detected genes falls outside the 5th and 95th percentiles, and discard cells with more than 10% of their UMIs assigned to mitochondrial genes. Normalize the filtered matrix with the scran R package using its default pool-and-deconvolute normalization algorithm. Rationale: scTITANS performs trajectory inference and pseudotime correction of cell asynchrony downstream, which requires a consistently filtered and normalized matrix; the scTITANS 'indication' parameter ('none', 'filtered', or 'normalized') lets the tool know how far preprocessing has progressed.

## Related tools

- **scTITANS** (Downstream pipeline whose 'full' mode invokes this filtering and normalization; the 'indication' parameter records whether the matrix is raw ('none'), filtered, or filtered and normalized) — https://github.com/ZJUFanLab/scTITANS
- **scran** (Performs normalization of the filtered matrix using the default pool-and-deconvolute algorithm)
- **Cell Ranger** (Source of the raw digital gene expression matrices that this filtering/normalization step consumes)

## Examples

```
result = scTITANS_full(data_file,"none","none","none","F","","","none",0.01)
```

## Evaluation signals

- Filtered gene matrix retains only genes with ≥2 transcripts in ≥10 cells and drops genes with zero counts everywhere
- Retained cells all fall within the 5th–95th percentile range of detected gene counts
- No retained cell has >10% of its UMIs assigned to mitochondrial genes
- Normalized matrix is produced with scran's default pool-and-deconvolute implementation and downstream TI (e.g., Monocle3 pseudotime) runs without preprocessing errors
- When calling scTITANS_full, the indication parameter correctly reflects preprocessing state ('none' for raw, 'filtered', or 'normalized')

## Limitations

- Software version numbers for tools (e.g., scran, scTITANS) are not stated in the article
- The 5th/95th percentile and 10% mitochondrial UMI cutoffs are dataset-level defaults; extreme datasets may need dataset-specific thresholds
- Cell type annotation datasets can require additional filtering (e.g., MRD cell types dropped due to low cell counts), which this generic filtering step does not cover

## Evidence
