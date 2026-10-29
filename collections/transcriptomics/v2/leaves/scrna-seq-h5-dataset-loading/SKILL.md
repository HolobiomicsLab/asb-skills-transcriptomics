---
name: scrna-seq-h5-dataset-loading
description: Use when your analysis starts from an h5 file containing an scRNA-seq
  expression matrix, true labels, and other information (e.g. the adam Alzheimer's
  dataset feeding scGCL), and you need it parsed into a cell-by-gene matrix plus reference
  labels.
when_to_use_negative:
- Input is an already-imputed expression table (e.g. scgcl_ad_imputed_expression.csv)
  rather than the raw h5 dataset — this loader is for the upstream h5 input only
- Data is stored in .h5ad, mtx, or FASTQ format rather than the h5 layout scGCL's
  --dataset argument expects
- You need to change cell or gene content during loading; the h5 matrix must pass
  through to imputation unaltered so ARI/NMI can be compared against the true labels
  in the file
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scGCL
  role: Pipeline whose main.py --dataset argument consumes the loaded h5 file for
    imputation and clustering
  repo: https://github.com/zehaoxiong123/scGCL
- name: Python
  role: Runtime (version 3.8.8) used to launch scGCL and parse the h5 expression matrix
    and labels
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btad098
    title: 'scGCL: an imputation method for scRNA-seq data based on graph contrastive
      learning'
  source_content_hash: sha256:15725862036154a66fcb82008bd8b86452b539f7fb07cb4f399c38bb04eee5d7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zehaoxiong123/scGCL
  tools:
  - PyTorch
  - Python
  - scGCL
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# scrna-seq-h5-dataset-loading

## Summary

Loads the adam h5 scRNA-seq dataset (Alzheimer's disease) that bundles the expression matrix, true cell labels, and accompanying information into the cell-by-gene structure required by scGCL for ZINB-based imputation and downstream clustering (ARI/NMI) evaluation.

## When to use

Use when your analysis starts from an h5 file containing an scRNA-seq expression matrix, true labels, and other information (e.g. the adam Alzheimer's dataset feeding scGCL), and you need it parsed into a cell-by-gene matrix plus reference labels.

## When NOT to use

- Input is an already-imputed expression table (e.g. scgcl_ad_imputed_expression.csv) rather than the raw h5 dataset — this loader is for the upstream h5 input only
- Data is stored in .h5ad, mtx, or FASTQ format rather than the h5 layout scGCL's --dataset argument expects
- You need to change cell or gene content during loading; the h5 matrix must pass through to imputation unaltered so ARI/NMI can be compared against the true labels in the file

## Inputs

- adam h5 file containing a matrix of scRNA-seq expression values, true labels, and other information
- Python environment pinned to Python 3.8.8, PyTorch 1.10.0, torch-geometric 2.2.0, torch-cluster 1.6.0, torch-scatter 2.0.9, torch-sparse 0.6.13, faiss-cpu 1.7.3

## Outputs

- Loaded cell-by-gene scRNA-seq expression matrix from the h5 file
- True cell label vector extracted from the h5 file, used as the reference partition for clustering metrics
- Dimensionality check confirming imputed output (e.g. scgcl_ad_imputed_expression.csv) matches the input h5 cell-by-gene dimensions

## How to apply

Load the adam h5 dataset, extracting three aligned components: the scRNA-seq expression matrix, the true cell labels, and accompanying metadata. Verify the expression matrix is cell-by-gene and that the label vector length matches the cell count before any processing, since downstream steps assume this correspondence. Pass the h5 path to scGCL via its --dataset argument (e.g. --dataset adam) and train for 300 epochs as in the published settings; the graph contrastive learner plus ZINB autoencoder will consume the loaded matrix to estimate dropout values. Retain the true labels from the h5 file unchanged so they can serve as the reference partition for clustering (ARI/NMI) after imputation. A correct load yields an imputed export (e.g. scgcl_ad_imputed_expression.csv) with exactly the same cell-by-gene dimensions as the input h5 expression matrix.

## Related tools

- **scGCL** (Pipeline whose main.py --dataset argument consumes the loaded h5 file for imputation and clustering) — https://github.com/zehaoxiong123/scGCL
- **Python** (Runtime (version 3.8.8) used to launch scGCL and parse the h5 expression matrix and labels)

## Examples

```
python main.py --dataset adam --epochs 300
```

## Evaluation signals

- Imputed matrix has the same cell-by-gene dimensions as the input h5 expression matrix
- True labels read from the h5 file exactly match the cell count and are used as the reference partition for ARI/NMI
- Computed clustering metrics reproduce the published AD-dataset values (ARI 0.62, NMI 0.70)
- AD-versus-control cell separation reproduces the ordering and separation shown in Supplementary Fig. S3a
- Loading runs in the documented environment (Python 3.8.8, PyTorch 1.10.0, torch-geometric 2.2.0, faiss-cpu 1.7.3)

## Limitations

- The h5 loader assumes scGCL's specific file layout (expression matrix + true labels + other information in one h5 file, as for the adam dataset); other formats are not supported by --dataset
- Run requires the pinned dependency stack (Python 3.8.8, PyTorch 1.10.0, torch-geometric 2.2.0, torch-scatter 2.0.9, torch-sparse 0.6.13, torch-cluster 1.6.0, faiss-cpu 1.7.3)
- Published protocol uses 300 training epochs; altering epochs changes the imputation feeding the ARI/NMI comparison

## Evidence

- [readme] A h5 file. Contains a matrix of scRNA-seq expression values,true labels, and other information.: "A h5 file. Contains a matrix of scRNA-seq expression values,true labels, and other information."
- [methods] python main.py --dataset adam --epochs 300: "You can run the scGCL from the command line: $ python main.py --dataset adam --epochs 300"
- [intro] scGCL can enhance the expression patterns of specific genes in Alzheimer's disease datasets: "we find that scGCL can enhance the expression patterns of specific genes in Alzheimer's disease datasets"
