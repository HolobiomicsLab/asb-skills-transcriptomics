---
name: scatac-seq-activity-matrix-preparation
description: Use when raw scATAC-seq peak data has already been converted to a gene
  activity matrix (e.g. via Signac) and must be TF-IDF transformed onto scRNA-seq
  common genes before scBridge label transfer.
when_to_use_negative:
- Input is still a peak-by-cell matrix rather than a gene activity matrix — it must
  first be transformed with a tool such as Signac
- The input is the annotated scRNA-seq source data — scBridge's default source preprocessing
  is 'Standard', not TFIDF
- The matrix has already been TF-IDF transformed and standardized for scBridge consumption
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3170
tools:
- name: scBridge
  role: Consumes the TF-IDF-processed target h5ad via the --target_preprocess argument
    (Default='TFIDF') as its target scATAC-seq input
  repo: https://github.com/XLearning-SCU/scBridge
- name: Signac
  role: Upstream tool for transforming scATAC-seq peak data into a scATAC-seq activity
    matrix before preprocessing
  repo: https://github.com/stuart-lab/signac
- name: Python
  role: Implementation language used with the anndata library to load, intersect features,
    TF-IDF transform, and save the h5ad target matrix
provenance:
  source_task_ids:
  - task_005
  - task_001
  source_papers:
  - doi: 10.1038/s41467-023-41795-5
    title: scBridge embraces cell heterogeneity in single-cell RNA-seq and ATAC-seq
      data integration
  source_content_hash: sha256:d71dccfb4bdfbb1bd6689634af7d478f49d3c7f392b49a3d61c95aebeb5a4f34
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/XLearning-SCU/scBridge
  tools:
  - CUDA
  - PyTorch
  - Python
  - Signac
  - scBridge
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# scatac-seq-activity-matrix-preparation

## Summary

Prepares the scATAC-seq gene activity matrix as the target input for scBridge by intersecting features with scRNA-seq common genes and applying TF-IDF followed by standardization, matching scBridge's default target_preprocess='TFIDF' strategy.

## When to use

Use when raw scATAC-seq peak data has already been converted to a gene activity matrix (e.g. via Signac) and must be TF-IDF transformed onto scRNA-seq common genes before scBridge label transfer.

## When NOT to use

- Input is still a peak-by-cell matrix rather than a gene activity matrix — it must first be transformed with a tool such as Signac
- The input is the annotated scRNA-seq source data — scBridge's default source preprocessing is 'Standard', not TFIDF
- The matrix has already been TF-IDF transformed and standardized for scBridge consumption

## Inputs

- Raw scATAC-seq gene activity matrix in h5ad (AnnData) format
- scRNA-seq count matrix (h5ad) used to define the set of common genes
- For peak-level scATAC-seq: peak-by-cell matrix requiring prior conversion to a gene activity matrix

## Outputs

- TF-IDF-then-Standard normalized scATAC-seq gene activity matrix saved as AnnData h5ad (target_processed.h5ad)
- Feature dimensionality equal to the number of scRNA-seq/scATAC-seq common genes

## How to apply

Load the raw scATAC-seq gene activity matrix from its h5ad file with the anndata Python library and intersect the gene features with the scRNA-seq count matrix so only common genes are retained. Apply TF-IDF, which is scBridge's default preprocessing for target data (--target_preprocess, Default='TFIDF'), while 'Standard' is the alternative. Standardize the TF-IDF-transformed matrix to zero means and unit variances, then save the result as an AnnData h5ad (e.g. target_processed.h5ad) whose feature dimension equals the number of common genes. In the scBridge workflow this matrix becomes the --target_data h5ad, whereas the scRNA-seq source keeps Standard preprocessing by default. For peak-level scATAC-seq data, first convert to an activity matrix using a tool such as Signac before applying this skill.

## Related tools

- **scBridge** (Consumes the TF-IDF-processed target h5ad via the --target_preprocess argument (Default='TFIDF') as its target scATAC-seq input) — https://github.com/XLearning-SCU/scBridge
- **Signac** (Upstream tool for transforming scATAC-seq peak data into a scATAC-seq activity matrix before preprocessing) — https://github.com/stuart-lab/signac
- **Python** (Implementation language used with the anndata library to load, intersect features, TF-IDF transform, and save the h5ad target matrix)

## Examples

```
python main.py --data_path=<folder> --source_data=<source>.h5ad --target_data=<target>.h5ad --target_preprocess=TFIDF --novel_type --umap_plot
```

## Evaluation signals

- Output is a readable AnnData h5ad file
- Feature dimension of the processed target matrix equals the number of common genes shared with the scRNA-seq count matrix
- Matrix values reflect the TF-IDF-then-Standardize pipeline, matching scBridge's target_preprocess='TFIDF' default
- The scBridge run with this target accepts the file without feature-mismatch errors against the source h5ad

## Limitations

- TF-IDF preprocessing details are discussed in Supplementary Note 3, which is not included in the provided document
- Common-gene intersection must be performed before feeding data into scBridge; differing gene sets between omics will break the input
- scBridge was developed for unpaired scRNA-seq/scATAC-seq integration; sparsity and distribution differences across omics are handled downstream, not by preprocessing alone

## Evidence

- [methods] TFIDF is scBridge default target preprocessing: "`--target_preprocess=["Standard", "TFIDF"]` (Default="TFIDF") the preprocessing strategy for target data"
- [readme] Peak data must first be converted to an activity matrix with Signac: "For scATAC-seq peak data, it needs to be first transformed into a scATAC-seq activity matrix using tools such as Signac"
- [discussion] Count normalization used in the paper's preprocessing: "we first normalized each cell by dividing its total number of read counts on all genes, and then multiplied them by 10,000"
- [methods] Source data uses Standard preprocessing by default, contrasting with TFIDF for target: "`--source_preprocess=["Standard", "TFIDF"]` (Default="Standard") the preprocessing strategy for source data"
