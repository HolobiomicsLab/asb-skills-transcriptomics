---
name: anndata-h5ad-data-handling
description: Use when loading, formatting, or saving single-cell RNA/ATAC data as
  AnnData .h5ad files to feed scBridge or read back its Prediction and Reliability
  outputs.
when_to_use_negative:
- Input data are raw peak-by-cell matrices that have not yet been transformed into
  a scATAC-seq gene activity matrix — scBridge's AnnData inputs require gene-level
  activity data
- The task involves protein (CITE-seq/ASAP-seq antibody) matrices outside the gene-expression/activity
  common-gene framework described for scBridge inputs
- Data are not single-cell matrices suited to AnnData (e.g., bulk RNA-seq or plain
  CSV feature tables without cell-level observations)
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2229
tools:
- name: scBridge
  role: Consumes source/target h5ad AnnData files and produces integrated AnnData
    outputs with Embedding, Prediction, and Reliability fields
  repo: https://github.com/XLearning-SCU/scBridge
- name: anndata
  role: Python library used to load the raw scATAC-seq gene activity matrix from its
    h5ad file and save normalized matrices as h5ad
- name: Signac
  role: Transforms scATAC-seq peak data into a scATAC-seq activity matrix before h5ad
    formatting
  repo: https://github.com/stuart-lab/signac
provenance:
  source_task_ids:
  - task_002
  - task_008
  - task_001
  source_papers:
  - doi: 10.1038/s41467-023-41795-5
    title: scBridge embraces cell heterogeneity in single-cell RNA-seq and ATAC-seq
      data integration
  source_content_hash: sha256:cd9b0c3acba9426c78dea24ef559218a4722be76584ec28748900ac6a4add61a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/XLearning-SCU/scBridge
  tools:
  - CUDA
  - PyTorch
  - Python
  - Signac
  - anndata
  - scBridge
  - scikit-learn
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# anndata-h5ad-data-handling

## Summary

Prepare and exchange source (scRNA-seq) and target (scATAC-seq) matrices as AnnData h5ad files with cell-type annotations in obs['CellType'], intersected to common genes, so they can be consumed and produced by scBridge's iterative integration pipeline.

## When to use

Use when loading, formatting, or saving single-cell RNA/ATAC data as AnnData .h5ad files to feed scBridge or read back its Prediction and Reliability outputs.

## When NOT to use

- Input data are raw peak-by-cell matrices that have not yet been transformed into a scATAC-seq gene activity matrix — scBridge's AnnData inputs require gene-level activity data
- The task involves protein (CITE-seq/ASAP-seq antibody) matrices outside the gene-expression/activity common-gene framework described for scBridge inputs
- Data are not single-cell matrices suited to AnnData (e.g., bulk RNA-seq or plain CSV feature tables without cell-level observations)

## Inputs

- Source scRNA-seq AnnData .h5ad with cell-type annotations in obs['CellType'] (e.g., SNARE-seq GSE126074 gene expression matrix)
- Target scATAC-seq AnnData .h5ad gene activity matrix (e.g., SNARE-seq GSE126074 gene activity matrix)
- scATAC-seq peak-by-cell matrix convertible to a gene activity matrix via Signac

## Outputs

- Integrated source and target AnnData .h5ad files with Embedding/X_umap, Prediction, and Reliability fields
- Normalized/preprocessed target gene activity matrix saved as target_processed.h5ad in AnnData format

## How to apply

Load each dataset with the anndata Python library as a source h5ad (scRNA-seq counts) and a target h5ad (scATAC-seq gene activity matrix), placing cell-type annotations in obs['CellType'] of the source; scATAC peak data must first be converted to a gene activity matrix with a tool such as Signac. Intersect the gene features of the count matrix and the activity matrix to retain only common genes before feeding data into scBridge. After training, read the transferred labels from .obs['Prediction'] and reliability scores from .obs['Reliability'] of the integrated target AnnData, and save integrated source/target files with Embedding/X_umap and Prediction/Reliability fields for downstream plotting.

## Related tools

- **scBridge** (Consumes source/target h5ad AnnData files and produces integrated AnnData outputs with Embedding, Prediction, and Reliability fields) — https://github.com/XLearning-SCU/scBridge
- **anndata** (Python library used to load the raw scATAC-seq gene activity matrix from its h5ad file and save normalized matrices as h5ad)
- **Signac** (Transforms scATAC-seq peak data into a scATAC-seq activity matrix before h5ad formatting) — https://github.com/stuart-lab/signac

## Examples

```
git clone https://github.com/XLearning-SCU/scBridge; python main.py --source_data source.h5ad --target_data target.h5ad --target_preprocess TFIDF
```

## Evaluation signals

- Both source and target files are valid, readable AnnData h5ad objects with matching gene dimensions equal to the number of common genes
- Source AnnData obs['CellType'] is populated for every cell, satisfying scBridge's annotation requirement
- Integrated target AnnData contains .obs['Prediction'] and .obs['Reliability'] columns populated for all target cells
- Label transfer accuracy computed from Prediction against target annotations reproduces reported values (e.g., 71.95% on SNARE-seq mouse cortex, 60.11% at the first iteration)
- UMAP plots colored by cell type, domain, and reliability can be rendered from Embedding/X_umap fields saved in the integrated AnnData files

## Limitations

- scATAC-seq peak data must first be transformed into a gene activity matrix (e.g., via Signac) — raw peak matrices are not accepted as h5ad inputs
- Only common genes shared between the scRNA-seq count matrix and the scATAC-seq gene activity matrix can be used, restricting the feature space
- TF-IDF preprocessing details for the target matrix are deferred to Supplementary Note 3 and not described in the main text

## Evidence

- [readme] scBridge accepts AnnData as inputs, including the source (scRNA-seq) and target (scATAC-seq) h5ad files. The source data should include the cell type annotation in obs.["CellType"].: "scBridge accepts AnnData ... as inputs, including the source (scRNA-seq) and target (scATAC-seq) h5ad files. The source data should include the cell type annotation in `obs.["CellType"]`"
- [readme] For scATAC-seq peak data, it needs to be first transformed into a scATAC-seq activity matrix using tools such as Signac: "For scATAC-seq peak data, it needs to be first transformed into a scATAC-seq activity matrix using tools such as Signac"
- [full_text] Load the raw scATAC-seq gene activity matrix from its h5ad file using the anndata Python library.: [evidence span withheld — anchor status: not_found]
- [full_text] Save the normalized gene activity matrix as target_processed.h5ad in AnnData format.: [evidence span withheld — anchor status: not_found]
