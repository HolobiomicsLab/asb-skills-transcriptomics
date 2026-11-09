---
name: highly-variable-gene-selection
description: Use when pre-processing raw scRNA-seq count matrices (h5 format) for
  scGCL-style imputation and you need to reduce the filtered, size-factor log-normalized
  matrix to a fixed set of 2048 highly variable genes prior to graph construction.
when_to_use_negative:
- The expression matrix has already been restricted to highly variable genes or contains
  fewer than 2048 genes
- The data are bulk RNA-seq or already-normalized matrices that have not passed the
  scGCL >1% non-zero QC filter, so scanpy HVG selection would be applied to inappropriate
  inputs
- The downstream method is not scGCL (imputation via graph contrastive learning with
  a ZINB autoencoder) and does not expect a fixed 2048-gene feature space
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scGCL
  role: Pipeline whose pre-processing stage performs the QC filter, size-factor log
    normalization, and scanpy-based selection of the top 2048 highly variable genes;
    invoked with --dataset <h5 file>
  repo: https://github.com/zehaoxiong123/scGCL
- name: scanpy
  role: Provides the highly-variable-gene selection routine used to pick the top t=2048
    HVGs from the normalized matrix
- name: Python
  role: Execution environment (Python 3.8.8) for the scGCL pre-processing script run
    as `python main.py --dataset ...`
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad098
    title: 'scGCL: an imputation method for scRNA-seq data based on graph contrastive
      learning'
  source_content_hash: sha256:d5a56407d5d4774042b5380d3ddc234380c7d006f99eda8851e68fc46423e1f6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zehaoxiong123/scGCL
  tools:
  - Python
  - scGCL
  - scanpy
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# highly-variable-gene-selection

## Summary

Reduces a normalized scRNA-seq expression matrix to the top t=2048 highly variable genes using scanpy, as the third pre-processing step of the scGCL imputation pipeline (after >1% non-zero filtering and size-factor log normalization) before graph construction.

## When to use

Use when pre-processing raw scRNA-seq count matrices (h5 format) for scGCL-style imputation and you need to reduce the filtered, size-factor log-normalized matrix to a fixed set of 2048 highly variable genes prior to graph construction.

## When NOT to use

- The expression matrix has already been restricted to highly variable genes or contains fewer than 2048 genes
- The data are bulk RNA-seq or already-normalized matrices that have not passed the scGCL >1% non-zero QC filter, so scanpy HVG selection would be applied to inappropriate inputs
- The downstream method is not scGCL (imputation via graph contrastive learning with a ZINB autoencoder) and does not expect a fixed 2048-gene feature space

## Inputs

- Raw scRNA-seq count matrix in h5 format (e.g. the adam Alzheimer's disease dataset) containing expression values and true labels
- Quality-control-filtered matrix (genes non-zero in >1% of cells, cells non-zero in >1% of genes)
- Size-factor log-normalized expression matrix

## Outputs

- Expression matrix restricted to the top 2048 highly variable genes, saved as an h5 file for downstream graph construction

## How to apply

Load the h5 scRNA-seq file (expression matrix plus true labels) via the scGCL pipeline's --dataset argument (e.g. adam). First apply the Section 2.1 QC filter, retaining only genes expressed in >1% of cells and cells expressing >1% of genes, following the scGNN reference strategy. Then apply size-factor log normalization N(X_i,j) to correct library-size differences between cells. Finally use scanpy's highly-variable-gene selection to keep the top t=2048 HVGs from the normalized matrix, and save the result as an h5 file for subsequent graph contrastive learning. The rationale is to retain the most informative genes while fixing the feature dimension for the ZINB autoencoder and graph model.

## Related tools

- **scGCL** (Pipeline whose pre-processing stage performs the QC filter, size-factor log normalization, and scanpy-based selection of the top 2048 highly variable genes; invoked with --dataset <h5 file>) — https://github.com/zehaoxiong123/scGCL
- **scanpy** (Provides the highly-variable-gene selection routine used to pick the top t=2048 HVGs from the normalized matrix)
- **Python** (Execution environment (Python 3.8.8) for the scGCL pre-processing script run as `python main.py --dataset ...`)

## Examples

```
import scanpy as sc; adata = sc.read_h5ad('adam.h5'); sc.pp.highly_variable_genes(adata, n_top_genes=2048)
```

## Evaluation signals

- The output h5 file exists and its expression matrix contains exactly the 2048 selected highly variable genes (the paper reports no numeric acceptance threshold for this step, so this structural check is the qualitative acceptance criterion)
- The retained genes are a subset of the matrix produced after the >1% non-zero QC filter, not of the raw matrix
- Values in the output matrix are size-factor log-normalized rather than raw counts
- Running the downstream scGCL command (python main.py --dataset <file> --epochs 300) accepts the h5 without errors, confirming schema compatibility (matrix + true labels)

## Limitations

- The paper reports no numeric acceptance threshold for the HVG selection step, so correctness is judged only by the structural check that 2048 genes remain
- The input must be an h5 file containing the expression matrix and true labels; other formats are not handled by the scGCL pipeline interface
- The 2048-gene top-HVG setting is tied to scGCL's design; other imputation methods may require a different t

## Evidence

- [methods] size-factor log normalization precedes HVG selection: [evidence span withheld — anchor status: not_found]
- [methods] scanpy selects top 2048 HVGs: [evidence span withheld — anchor status: not_found]
- [readme] input format is h5 with expression matrix and labels: "A h5 file. Contains a matrix of scRNA-seq expression values,true labels, and other information."
- [methods] QC filter threshold of >1% non-zero expression: [evidence span withheld — anchor status: not_found]
