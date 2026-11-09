---
name: single-cell-gene-and-cell-filtering
description: Use when the input is a raw (unfiltered, unnormalized) digital gene expression
  matrix (rows = genes, columns = cells), e.g. from Cell Ranger, that will feed scTITANS
  or a trajectory-based differential expression analysis.
when_to_use_negative:
- The matrix has already been filtered and/or normalized (e.g. scran-normalized input)
  — pass indication='filtered' or 'normalized' to scTITANS instead of 'none', otherwise
  filtering is reapplied
- The input is not a gene-by-cell single-cell count matrix (e.g. a feature table,
  bulk RNA-seq counts, or an already-differential-expression result) — the gene/cell
  percentile and mitochondrial-UMI cutoffs do not apply
- The downstream analysis requires retaining rare cell populations or lowly expressed
  genes (e.g. very small clusters like the MRD RPE/margin/periocular mesenchyme/lens
  epithelial cells that the study removed owing to low cell counts) — the default
  cutoffs will drop them
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: scTITANS
  role: Hosts the built-in preprocessing pipeline; its 'full' mode performs the gene
    filtering, cell filtering, and scran normalization when the matrix is passed with
    indication='none'
  repo: https://github.com/ZJUFanLab/scTITANS
- name: scran
  role: Normalizes the filtered matrix using the default implementation of the pool
    and deconvolute normalization algorithm
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.csbj.2021.07.016
    title: Identify differential genes and cell subclusters from time-series scRNA-seq
      data using scTITANS
  source_content_hash: sha256:200f9ea659d279482af20363fa2acb36f3077ec08d282aefcbaaee40d294f318
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
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# single-cell-gene-and-cell-filtering

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Quality-control step from the scTITANS pipeline that removes low-quality genes and cells from a raw single-cell U count matrix and normalizes the result with scran, so that downstream trajectory inference and pseudotime-based DEG calling operate on a cleaned matrix.

## When to use

Use when the input is a raw (unfiltered, unnormalized) digital gene expression matrix (rows = genes, columns = cells), e.g. from Cell Ranger, that will feed scTITANS or a trajectory-based differential expression analysis.

## When NOT to use

- The matrix has already been filtered and/or normalized (e.g. scran-normalized input) — pass indication='filtered' or 'normalized' to scTITANS instead of 'none', otherwise filtering is reapplied
- The input is not a gene-by-cell single-cell count matrix (e.g. a feature table, bulk RNA-seq counts, or an already-differential-expression result) — the gene/cell percentile and mitochondrial-UMI cutoffs do not apply
- The downstream analysis requires retaining rare cell populations or lowly expressed genes (e.g. very small clusters like the MRD RPE/margin/periocular mesenchyme/lens epithelial cells that the study removed owing to low cell counts) — the default cutoffs will drop them

## Inputs

- Raw digital gene expression matrix (rows = genes, columns = cells), e.g. raw UMI counts from Cell Ranger (10X Genomics)
- Optional: precomputed cell and gene metadata files (cell_metadata rows = cells; gene_metadata with a gene_short_name header) when supplying pre-filtered data to scTITANS

## Outputs

- Filtered gene-by-cell count matrix (genes detected in >=10 cells with >=2 transcripts; cells within the 5th-95th percentile of detected genes and <10% mitochondrial UMI retained)
- scran pool-and-deconvolute-normalized matrix ready for trajectory inference (Monocle3) and scTITANS DEG/subcluster identification

## How to apply

Apply two filters to the raw U count matrix: (1) gene filtering — drop genes without any counts in any cell, and define a gene as detected only if it is present with two or more transcripts in at least 10 cells; (2) cell filtering — discard cells whose number of detected genes falls outside the 5th and 95th percentiles, and remove cells with more than 10% of their UMIs assigned to mitochondrial genes. After filtering, normalize the matrix with the scran R package using the default pool-and-deconvolute normalization algorithm. In scTITANS' full mode this entire preprocessing is triggered by passing indication='none' for raw data (use 'filtered' if only gene/cell filtering was already done, or 'normalized' if the matrix is already filtered and normalized), so the pipeline knows which stage to resume from.

## Related tools

- **scTITANS** (Hosts the built-in preprocessing pipeline; its 'full' mode performs the gene filtering, cell filtering, and scran normalization when the matrix is passed with indication='none') — https://github.com/ZJUFanLab/scTITANS
- **scran** (Normalizes the filtered matrix using the default implementation of the pool and deconvolute normalization algorithm)

## Examples

```
result = scTITANS_full(data_file,"none","none","none","F","","","none",0.01)
```

## Evaluation signals

- Every retained gene has >=2 transcripts in at least 10 cells and no all-zero-count genes remain in the matrix
- Every retained cell has a detected-gene count inside the 5th-95th percentile range and <=10% of its UMIs assigned to mitochondrial genes
- The downstream scTITANS call consumes a scran-normalized matrix; in a scTITANS_full run with a raw matrix, indication must be 'none', and the run must emit SigGenes.fdr0.01.txt and SigClusters.fdr0.01.txt
- Filtering results are reproducible: re-running on the same raw matrix (e.g. the AD GSE98969 dataset, 8016 mouse CD45+ immune cells) should remove the same set of low-quality cells and genes before scran normalization

## Limitations

- Software version numbers for scTITANS, scran, and Monocle3 are not stated in the article, so exact reproducibility of filter behavior across versions is uncertain
- Cell filtering by 5th-95th percentile of detected genes and the 10% mitochondrial UMI cutoff can discard biologically meaningful rare cell types, as seen when RPE/margin/periocular mesenchyme/lens epithelial cells were removed from MRD owing to low cell counts
- The filters are designed for raw UMI count matrices; behavior on already-normalized or aggregated data is not defined by the paper

## Evidence
