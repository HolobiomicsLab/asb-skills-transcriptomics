---
name: highly-variable-gene-selection-filter-genes-dispersion
description: Use when a raw 10X-style count matrix (matrix.mtx, genes.tsv, barcodes.tsv)
  must be reduced to a manageable set of informative genes (e.g. top 1000 HVGs) before
  normalization, log1p, and downstream dimensionality reduction or imputation.
when_to_use_negative:
- When the goal is imputation on the Klein dataset, where the article instead selects
  the top 5000 highly variable genes using the Seurat package (klein_filterHVG.R),
  not Scanpy filter_genes_dispersion
- When the expression matrix is already normalized and log1p-transformed — the 'cell_ranger'
  dispersion call here expects un-logged, per-cell-normalized counts (log=False)
- When feature selection is not needed, e.g. the input is already a reduced or pre-filtered
  feature table
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: Scanpy
  role: Used to load the 10X count matrix and to perform the highly-variable-gene
    feature selection via sc.pp.filter_genes_dispersion (flavor='cell_ranger', n_top_genes=1000,
    log=False)
  repo: https://github.com/theislab/scanpy
- name: Seurat
  role: Alternative feature-selection path used in the Klein imputation example to
    select the top 5000 highly variable genes
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1007/s11704-022-2011-y
    title: 'AE-TPGG: a novel autoencoder-based approach for single-cell RNA-seq data
      imputation and dimensionality reduction'
  source_content_hash: sha256:a1e4023a08d70ca739d7b620f600e59d80d520eac4d287b4e275876c1e6972cf
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
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# Highly-variable gene selection with Scanpy filter_genes_dispersion

## Summary

Selects the top highly variable genes from a raw scRNA-seq count matrix using Scanpy's sc.pp.filter_genes_dispersion with the 'cell_ranger' flavor, as done in the AE-TPGG pipeline to reduce the 68K PBMC matrix to its 1000 most informative genes before dimensionality reduction.

## When to use

Use when a raw 10X-style count matrix (matrix.mtx, genes.tsv, barcodes.tsv) must be reduced to a manageable set of informative genes (e.g. top 1000 HVGs) before normalization, log1p, and downstream dimensionality reduction or imputation.

## When NOT to use

- When the goal is imputation on the Klein dataset, where the article instead selects the top 5000 highly variable genes using the Seurat package (klein_filterHVG.R), not Scanpy filter_genes_dispersion
- When the expression matrix is already normalized and log1p-transformed — the 'cell_ranger' dispersion call here expects un-logged, per-cell-normalized counts (log=False)
- When feature selection is not needed, e.g. the input is already a reduced or pre-filtered feature table

## Inputs

- 10X 10XGenomics-style count matrix: matrix.mtx plus genes.tsv and barcodes.tsv (loaded via Scanpy sc.read(...).T with names assigned from the TSVs)
- AnnData object with raw integer UMI counts in .X
- Optional target number of highly variable genes (e.g. n_top_genes=1000 for 68K PBMC)

## Outputs

- filter_result object from sc.pp.filter_genes_dispersion, whose gene_subset gives the indices of the selected top 1000 highly variable genes
- Gene-subsetted, per-cell-normalized, log1p-transformed AnnData (adata[:, filter_result.gene_subset]) ready for AE-TPGG

## How to apply

First make a temporary copy of the AnnData object and normalize per cell with sc.pp.normalize_per_cell so dispersion is computed on UMI-normalized values. Then call sc.pp.filter_genes_dispersion(adata_temp.X, flavor='cell_ranger', n_top_genes=1000, log=False) on the un-logged normalized data; log=False is important because the data have not yet been log-transformed at this stage. The returned filter_result.gene_subset indexes the retained genes. After feature selection, preprocess the FULL matrix with sc.pp.normalize_per_cell and sc.pp.log1p (yielding the semi-continuous normalized form the AE-TPGG two-part-generalized-gamma model expects), and only then subset to adata[:, filter_result.gene_subset]. Note the paper contrasts two variants: top 1000 HVGs via Scanpy for the 68K PBMC dimensionality-reduction example, and top 5000 HVGs via Seurat for the Klein imputation example.

## Related tools

- **Scanpy** (Used to load the 10X count matrix and to perform the highly-variable-gene feature selection via sc.pp.filter_genes_dispersion (flavor='cell_ranger', n_top_genes=1000, log=False)) — https://github.com/theislab/scanpy
- **Seurat** (Alternative feature-selection path used in the Klein imputation example to select the top 5000 highly variable genes)

## Examples

```
adata_temp = adata; sc.pp.normalize_per_cell(adata_temp); filter_result = sc.pp.filter_genes_dispersion(adata_temp.X, flavor='cell_ranger', n_top_genes=1000, log=False)
```

## Evaluation signals

- filter_result.gene_subset has length exactly n_top_genes (1000 for the 68K PBMC example)
- The temporary copy used for dispersion was per-cell-normalized (sc.pp.normalize_per_cell applied before filtering) and NOT log1p-transformed (log=False)
- Subsetting (adata[:, filter_result.gene_subset]) is applied AFTER the full matrix has been normalized and log1p-transformed, not before
- Downstream AE-TPGG run consumes the filtered matrix and produces adata.obsm['X_ae_tpgg'] with one row per cell, confirming the gene subset was compatible with the model

## Limitations

- The 'cell_ranger' flavor with log=False assumes the temporary copy is only per-cell-normalized; applying the filter to raw counts or to already-logged data would change the dispersion estimates
- The paper uses different HVG counts and tools per task (top 1000 via Scanpy for 68K PBMC dimensionality reduction vs top 5000 via Seurat for Klein imputation), so the parameters are example-specific, not universal

## Evidence

- [readme] Step 2. Feature selection. Select the top 1000 high variable genes by using the Scanpy package.: "Step 2. Feature selection. Select the top 1000 high variable genes by using the Scanpy package."
- [readme] filter_genes_dispersion call with parameters: "filter_result = sc.pp.filter_genes_dispersion(adata_temp.X, flavor='cell_ranger', n_top_genes=1000, log=False)"
