---
name: scrna-seq-expression-matrix-loading
description: Use when raw scRNA-seq counts stored in an h5 file (with true labels)
  must be loaded via the scGCL --dataset argument and pre-processed with the >1% non-zero
  QC filter, size-factor log-normalization, and top-2048 HVG selection before graph
  construction.
when_to_use_negative:
- Input is already normalized or already restricted to highly variable genes — re-applying
  the >1% non-zero filter and HVG selection would further discard genes/cells.
- Data are not single-cell RNA-seq counts (e.g. bulk RNA-seq or other assays) — the
  >1% non-zero filter and the downstream ZINB dropout model assume single-cell sparsity.
- The expression data and labels are not packaged in an h5 file — the scGCL dataset
  argument expects 'A h5 file' containing the matrix, true labels, and other information.
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: scGCL
  role: Pipeline whose --dataset CLI argument loads the h5 scRNA-seq expression matrix
    and applies the Section 2.1 QC filter before graph-based imputation
  repo: https://github.com/zehaoxiong123/scGCL
- name: scanpy
  role: Performs highly-variable-gene selection of the top t=2048 HVGs from the size-factor
    log-normalized matrix
- name: Python 3.8.8
  role: Runtime environment in which the scGCL main.py entry point and scanpy pre-processing
    are executed
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad098
    title: 'scGCL: an imputation method for scRNA-seq data based on graph contrastive
      learning'
  source_content_hash: sha256:95d09662bd89966a46e36be8451760b4194da046410b75ad75fcc5385842ed05
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zehaoxiong123/scGCL
  tools:
  - Python
  - Python 3.8.8
  - scGCL
  - scanpy
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7333333333333333
n_claims: 15
---

# scrna-seq-expression-matrix-loading

## Summary

Loads a raw scRNA-seq count matrix from an h5 dataset into the scGCL imputation pipeline and pre-processes it — QC filtering, size-factor log-normalization, and scanpy-based selection of the top 2048 highly variable genes — so the matrix is ready for graph construction.

## When to use

Use when raw scRNA-seq counts stored in an h5 file (with true labels) must be loaded via the scGCL --dataset argument and pre-processed with the >1% non-zero QC filter, size-factor log-normalization, and top-2048 HVG selection before graph construction.

## When NOT to use

- Input is already normalized or already restricted to highly variable genes — re-applying the >1% non-zero filter and HVG selection would further discard genes/cells.
- Data are not single-cell RNA-seq counts (e.g. bulk RNA-seq or other assays) — the >1% non-zero filter and the downstream ZINB dropout model assume single-cell sparsity.
- The expression data and labels are not packaged in an h5 file — the scGCL dataset argument expects 'A h5 file' containing the matrix, true labels, and other information.

## Inputs

- Raw scRNA-seq expression matrix stored as an h5 file (contains expression values, true labels, and other information), passed via the scGCL --dataset argument
- Dataset name string selecting the h5 file (e.g. adam)

## Outputs

- QC-filtered count matrix retaining only genes with non-zero expression in >1% of cells and cells with non-zero expression in >1% of genes
- Size-factor log-normalized h5 expression matrix restricted to the top 2048 highly variable genes, for use in downstream graph construction

## How to apply

Load the h5 dataset containing the scRNA-seq expression matrix and associated true labels using the dataset argument of the scGCL CLI (e.g. --dataset adam). Apply the Section 2.1 quality-control filter, retaining only genes with non-zero expression in more than 1% of cells and cells with non-zero expression in more than 1% of genes, following the scGNN reference strategy. Apply size-factor log normalization N(X_i,j) to the filtered count matrix to correct for library-size differences between cells. Use scanpy's highly-variable-gene selection to select the top t=2048 highly variable genes from the normalized matrix, then save the normalized, HVG-restricted matrix as an h5 file for subsequent graph construction. The paper reports no numeric acceptance threshold for this step, so validate structurally (correct 2048-gene shape after filtering and normalization).

## Related tools

- **scGCL** (Pipeline whose --dataset CLI argument loads the h5 scRNA-seq expression matrix and applies the Section 2.1 QC filter before graph-based imputation) — https://github.com/zehaoxiong123/scGCL
- **scanpy** (Performs highly-variable-gene selection of the top t=2048 HVGs from the size-factor log-normalized matrix)
- **Python 3.8.8** (Runtime environment in which the scGCL main.py entry point and scanpy pre-processing are executed)

## Examples

```
python main.py --dataset adam --epochs 300
```

## Evaluation signals

- The output h5 file exists and its expression matrix contains exactly the 2048 selected highly variable genes.
- Removed rows/columns correspond exactly to the >1% non-zero filter: genes with non-zero expression in ≤1% of cells and cells with non-zero expression in ≤1% of genes are gone.
- Matrix values are size-factor log-normalized (library-size corrected), not raw integer counts.
- True labels remain present in the h5 file so downstream node/clustering/similarity tasks can consume it.
- The README invocation 'python main.py --dataset adam --epochs 300' loads the matrix without errors.

## Limitations

- The paper reports no numeric acceptance threshold for the filter/normalization step, so acceptance is judged only by structural checks (2048 HVGs present after the >1% non-zero filter and size-factor log normalization).
- The >1% non-zero cutoff follows the scGNN reference strategy and may drop biologically meaningful low-detection genes or rare cell populations.
- The scGCL CLI dataset argument expects an h5 file containing the expression matrix, true labels, and other information; other input formats are not supported.

## Evidence

- [readme] A h5 file. Contains a matrix of scRNA-seq expression values,true labels, and other information.: "dataset | A h5 file. Contains a matrix of scRNA-seq expression values,true labels, and other information."
- [methods] $ python main.py --dataset adam --epochs 300: "You can run the scGCL from the command line:"
- [abstract] we propose a single-cell Graph Contrastive Learning method for scRNA-seq data imputation, named scGCL: "we propose a single-cell Graph Contrastive Learning method for scRNA-seq data imputation, named scGCL"
