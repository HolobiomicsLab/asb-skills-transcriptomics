---
name: gene-cell-nonzero-fraction-filtering
description: Use when raw scRNA-seq count matrices (e.g. h5 expression files for the
  scGCL pipeline) must be quality-controlled before normalization and graph construction,
  and a scGNN-style >1% non-zero expression filter is the desired gene/cell exclusion
  criterion.
when_to_use_negative:
- Input is already normalized, HVG-restricted, or previously filtered — reapplying
  the >1% non-zero filter would double-filter and change the gene/cell set
- Input is not a raw scRNA-seq count matrix (e.g. bulk RNA-seq, proteomics, or feature
  tables), since the >1% gene/cell non-zero criterion is defined on cell-by-gene count
  data
- The downstream method requires the full unfiltered gene space (e.g. marker-gene
  analysis across all genes), since this filter removes low-detection genes before
  the 2048-HVG restriction
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scGCL
  role: Pipeline whose Section 2.1 pre-processing implements the >1% non-zero gene/cell
    filter, size-factor log-normalization, and h5 I/O
  repo: https://github.com/zehaoxiong123/scGCL
- name: scanpy
  role: Performs highly-variable-gene selection (top t=2048 HVGs) on the filtered,
    normalized matrix
- name: Python
  role: Execution environment for the scGCL pre-processing pipeline (Python 3.8.8)
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad098
    title: 'scGCL: an imputation method for scRNA-seq data based on graph contrastive
      learning'
  source_content_hash: sha256:8f249c2faeceb044982b446fb0b6df8e0242ab96a3397a4cc25b2266ed6dd7d9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zehaoxiong123/scGCL
  tools:
  - Python
  - scGCL
  - scanpy
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 4
---

# gene-cell-nonzero-fraction-filtering

## Summary

Quality-control filter from scGCL's first pre-processing step that retains only genes with non-zero expression in more than 1% of cells and cells with non-zero expression in more than 1% of genes, following the scGNN reference strategy, before size-factor log-normalization and top-2048 HVG selection.

## When to use

Use when raw scRNA-seq count matrices (e.g. h5 expression files for the scGCL pipeline) must be quality-controlled before normalization and graph construction, and a scGNN-style >1% non-zero expression filter is the desired gene/cell exclusion criterion.

## When NOT to use

- Input is already normalized, HVG-restricted, or previously filtered — reapplying the >1% non-zero filter would double-filter and change the gene/cell set
- Input is not a raw scRNA-seq count matrix (e.g. bulk RNA-seq, proteomics, or feature tables), since the >1% gene/cell non-zero criterion is defined on cell-by-gene count data
- The downstream method requires the full unfiltered gene space (e.g. marker-gene analysis across all genes), since this filter removes low-detection genes before the 2048-HVG restriction

## Inputs

- Raw scRNA-seq count matrix stored in an h5 file, with expression values, true labels, and other information (e.g. the 'adam' Alzheimer's disease dataset)

## Outputs

- Filtered count matrix retaining only genes with >1% non-zero expression across cells and cells with >1% non-zero expression across genes
- Size-factor log-normalized matrix restricted to the top 2048 highly variable genes (scanpy selection), saved as an h5 file for downstream graph construction

## How to apply

Load the h5 scRNA-seq file (e.g. via scGCL's --dataset argument, such as --dataset adam) containing the expression matrix and true labels. Apply the Section 2.1 filter to the raw count matrix: drop any gene whose non-zero expression values occur in no more than 1% of cells, and drop any cell whose non-zero expression values occur in no more than 1% of genes. The rationale, following scGNN, is to remove genes and cells that carry too little signal to support downstream graph construction and ZINB-based dropout imputation. After filtering, apply size-factor log normalization N(X_i,j) to correct library-size differences, then use scanpy's highly-variable-gene selection to keep the top t=2048 highly variable genes. Save the resulting normalized, HVG-restricted matrix back to an h5 file for the graph contrastive learning stages of scGCL.

## Related tools

- **scGCL** (Pipeline whose Section 2.1 pre-processing implements the >1% non-zero gene/cell filter, size-factor log-normalization, and h5 I/O) — https://github.com/zehaoxiong123/scGCL
- **scanpy** (Performs highly-variable-gene selection (top t=2048 HVGs) on the filtered, normalized matrix)
- **Python** (Execution environment for the scGCL pre-processing pipeline (Python 3.8.8))

## Examples

```
python main.py --dataset adam --epochs 300
```

## Evaluation signals

- Output h5 file exists and its expression matrix contains exactly the 2048 scanpy-selected highly variable genes
- Every retained gene has non-zero expression in more than 1% of cells in the input matrix
- Every retained cell has non-zero expression in more than 1% of genes in the input matrix
- Expression values in the output have been size-factor log-normalized N(X_i,j), i.e. library-size differences between cells are corrected
- Downstream scGCL run (e.g. python main.py --dataset adam --epochs 300) accepts the preprocessed file without errors

## Limitations

- The paper reports no numeric acceptance threshold for this step, so acceptance is judged only by the structural check that the output contains the 2048 selected HVGs after the >1% filter and normalization
- The fixed 1% non-zero threshold is inherited from the scGNN reference strategy and may be too permissive or too strict for datasets with different sparsity profiles
- Filtering plus the top-2048 HVG restriction discards low-detection genes, which limits later analyses needing the full gene space

## Evidence

- [methods] In the first pre-processing step, scGCL applies quality-control filtering that keeps only genes with non-zero expression in more than 1% of cells and cells with non-zero expression in more than 1% of genes, following the scGNN reference strategy.: [evidence span withheld — anchor status: not_found]
- [methods] Use scanpy's highly-variable-gene selection to select the top t=2048 highly variable genes from the normalized matrix.: [evidence span withheld — anchor status: not_found]
- [readme] A h5 file. Contains a matrix of scRNA-seq expression values,true labels, and other information.: "A h5 file. Contains a matrix of scRNA-seq expression values,true labels, and other information."
