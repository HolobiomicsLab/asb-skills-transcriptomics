---
name: semi-continuous-count-normalization-log1p
description: Use when raw scRNA-seq count data (e.g., 10X matrix.mtx/genes.tsv/barcodes.tsv)
  must be transformed into semi-continuous normalized form before feeding the AE-TPGG
  autoencoder or comparing with highly-variable-gene filtering.
when_to_use_negative:
- Data are already normalized and log1p-transformed — reapplying normalize_per_cell/log1p
  would double-transform; in that case call ae_tpgg with log1p=False as in the Klein
  imputation example
- The analysis target is the Klein imputation feature selection, which uses the Seurat
  package to select the top 5000 highly variable genes rather than Scanpy filter_genes_dispersion
- Input is not raw scRNA-seq count data (e.g., already-summarized feature tables or
  bulk RNA-seq matrices)
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Scanpy
  role: Loads the scRNA-seq data and provides sc.pp.normalize_per_cell, sc.pp.log1p,
    and sc.pp.filter_genes_dispersion (flavor='cell_ranger', n_top_genes=1000, log=False)
    for the semi-continuous transformation and feature selection
  repo: https://github.com/theislab/scanpy
- name: Seurat
  role: Used in the Klein imputation example to select the top 5000 highly variable
    genes before the same normalization/log1p preprocessing
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1007/s11704-022-2011-y
    title: 'AE-TPGG: a novel autoencoder-based approach for single-cell RNA-seq data
      imputation and dimensionality reduction'
  source_content_hash: sha256:5c62d7c56412495c1bdbc5780874375522ee711b771723ff21c0fc6855913b18
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/theislab/scanpy
  tools:
  - Keras
  - Python 3
  - Scanpy
  - Seurat
  - TensorFlow
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# semi-continuous-count-normalization-log1p

## Summary

Convert raw scRNA-seq UMI count matrices into the semi-continuous normalized form (per-cell total-UMI normalization followed by log1p transformation) that the AE-TPGG two-part-generalized-gamma model expects as input. This preprocessing captures the mixed discrete-continuous nature of gene expression data prior to feature selection, dimensionality reduction, or imputation.

## When to use

Use when raw scRNA-seq count data (e.g., 10X matrix.mtx/genes.tsv/barcodes.tsv) must be transformed into semi-continuous normalized form before feeding the AE-TPGG autoencoder or comparing with highly-variable-gene filtering.

## When NOT to use

- Data are already normalized and log1p-transformed — reapplying normalize_per_cell/log1p would double-transform; in that case call ae_tpgg with log1p=False as in the Klein imputation example
- The analysis target is the Klein imputation feature selection, which uses the Seurat package to select the top 5000 highly variable genes rather than Scanpy filter_genes_dispersion
- Input is not raw scRNA-seq count data (e.g., already-summarized feature tables or bulk RNA-seq matrices)

## Inputs

- Raw scRNA-seq count matrix loaded in Scanpy AnnData (e.g., 10X matrix.mtx transposed, with gene names from genes.tsv and barcodes from barcodes.tsv)
- Gene-expression CSV of a filtered dataset (e.g., klein_filtered_data.csv) for the imputation workflow
- Temporary per-cell-normalized copy used for filter_genes_dispersion feature selection

## Outputs

- Semi-continuous normalized AnnData (normalize_per_cell + log1p) subset to the top 1000 (or 5000 for Klein) highly variable genes
- filter_result.gene_subset boolean mask from sc.pp.filter_genes_dispersion
- Preprocessed matrix ready as input to ae_tpgg()

## How to apply

First perform feature selection on a temporary copy of the data: normalize per cell with sc.pp.normalize_per_cell, then run sc.pp.filter_genes_dispersion with flavor='cell_ranger', n_top_genes=1000, log=False to obtain filter_result.gene_subset (the Klein dataset example instead selects the top 5000 HVGs with Seurat). Then normalize the full matrix with sc.pp.normalize_per_cell followed by sc.pp.log1p to convert the original counts into the semi-continuous form the two-part-generalized-gamma distribution is designed to model. Subset the log-normalized matrix to the selected gene subset (adata[:, filter_result.gene_subset]) before running ae_tpgg. For the Klein imputation workflow, apply the same normalize_per_cell + log1p steps to the filtered CSV data, and pass log1p=False when calling ae_tpgg with copy=True, return_info=True so the model receives the already-transformed data.

## Related tools

- **Scanpy** (Loads the scRNA-seq data and provides sc.pp.normalize_per_cell, sc.pp.log1p, and sc.pp.filter_genes_dispersion (flavor='cell_ranger', n_top_genes=1000, log=False) for the semi-continuous transformation and feature selection) — https://github.com/theislab/scanpy
- **Seurat** (Used in the Klein imputation example to select the top 5000 highly variable genes before the same normalization/log1p preprocessing)

## Examples

```
sc.pp.normalize_per_cell(adata); sc.pp.log1p(adata); adata_filtered = adata[:, filter_result.gene_subset]
```

## Evaluation signals

- filter_genes_dispersion returns a filter_result whose gene_subset selects exactly 1000 genes (n_top_genes=1000) from the per-cell-normalized temporary copy
- The transformed matrix is in semi-continuous form: values are non-negative and log1p-shaped (many exact zeros plus small positive continuous values), not raw integer UMIs
- adata_filtered is subset to adata[:, filter_result.gene_subset] and can be passed to ae_tpgg() to yield adata.obsm['X_ae_tpgg']
- Downstream clustering metrics (ACC, ARI, NMI, F1) of the reduced data are competitive with the t-SNE baseline as reported in Figure 1/2 of the AE-TPGG example

## Limitations

- Feature selection uses a temporary per-cell-normalized copy with log=False and flavor='cell_ranger'; applying filter_genes_dispersion to log-transformed data instead changes the dispersion-based HVG set
- The normalization is tuned to semi-continuous normalized scRNA-seq counts as modeled by the two-part-generalized-gamma distribution; it is specific to the AE-TPGG workflow rather than a generic preprocessing step
- The 68K PBMC example selects the top 1000 HVGs via Scanpy, whereas the Klein example selects the top 5000 HVGs via Seurat — the two workflows are not interchangeable

## Evidence

- [readme] sc.pp.normalize_per_cell(adata)
sc.pp.log1p(adata)
adata_filtered = adata[:, filter_result.gene_subset]: "sc.pp.normalize_per_cell(adata)
sc.pp.log1p(adata)
adata_filtered = adata[:, filter_result.gene_subset]"
- [abstract] AE-TPGG takes mixed discrete-continuous random variables of scRNA-seq data into account using a two-part model: [evidence span withheld — anchor status: not_found]
