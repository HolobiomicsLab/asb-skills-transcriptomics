---
name: anndata-scrnaseq-data-handling
description: Use when loading scRNA-seq count data (e.g. .h5ad files) into AnnData
  objects for model-based integration, or when batch and cell type information must
  be exposed in specific adata.obs columns for tools like scETM.
when_to_use_negative:
- Input is already a pathways-by-genes matrix or other feature table rather than a
  cells-by-genes scRNA-seq count matrix
- Data is not single-cell transcriptomic (e.g. bulk RNA-seq or proteomics) and cannot
  be represented as an AnnData cells-by-genes object with batch annotations
- Batch and cell type metadata cannot be mapped to adata.obs columns and the downstream
  tool offers no batch_col/cell_type_col override
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scETM
  role: Downstream consumer of the AnnData objects; requires cells-by-genes matrix
    with batch_indices in adata.obs and provides prepare_for_transfer for unaligned
    datasets
  repo: github.com/hui2000ji/scETM
- name: AnnData
  role: File format and in-memory container for the cells-by-genes matrix with obs-based
    annotations
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-021-25534-2
    title: Learning interpretable cellular and gene signature embeddings from single-cell
      transcriptomic data
  source_content_hash: sha256:0a7a447e6fc8a535ef087252e015e09187dabf3576ffefcfc757d04a059c1d8f
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - AnnData
  - scETM
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# anndata-scrnaseq-data-handling

## Summary

Prepare single-cell RNA-seq datasets as AnnData cells-by-genes matrices with batch indices and cell type annotations so downstream methods such as scETM can consume them. This data-handling step underpins training, transfer learning, and embedding extraction in the scETM integrative analysis workflow.

## When to use

Use when loading scRNA-seq count data (e.g. .h5ad files) into AnnData objects for model-based integration, or when batch and cell type information must be exposed in specific adata.obs columns for tools like scETM.

## When NOT to use

- Input is already a pathways-by-genes matrix or other feature table rather than a cells-by-genes scRNA-seq count matrix
- Data is not single-cell transcriptomic (e.g. bulk RNA-seq or proteomics) and cannot be represented as an AnnData cells-by-genes object with batch annotations
- Batch and cell type metadata cannot be mapped to adata.obs columns and the downstream tool offers no batch_col/cell_type_col override

## Inputs

- scRNA-seq count matrix as an .h5ad file (e.g. MousePancreas.h5ad, HumanPancreas_aligned.h5ad)
- Batch/covariate annotation accessible via adata.obs (batch_indices, cell_types columns)

## Outputs

- AnnData object with cells-by-genes matrix and batch indices ready for model training
- Gene-aligned source/target AnnData pairs for transfer learning (via prepare_for_transfer)

## How to apply

Load each dataset with anndata.read_h5ad into a cells-by-genes AnnData object, ensuring batch information is stored in the 'batch_indices' column of adata.obs and cell type identity in 'cell_types' (scETM looks for these columns by default; use the batch_col and cell_type_col arguments if your data stores them elsewhere). For transfer learning across datasets, align the gene lists so source and target share the same homologous genes in the same order; if gene sets differ, use scETM's prepare_for_transfer function, which discards parameters tied to source-dataset-unique genes and randomly initializes those tied to target-dataset-unique genes. The number of unique batch indices (e.g. adata.obs.batch_indices.nunique()) directly parameterizes the model, and n_vars gives the gene count. Keep in mind that training proceeds by batched gradient descent, so dataset size interacts with epoch counts needed for convergence.

## Related tools

- **scETM** (Downstream consumer of the AnnData objects; requires cells-by-genes matrix with batch_indices in adata.obs and provides prepare_for_transfer for unaligned datasets) — github.com/hui2000ji/scETM
- **AnnData** (File format and in-memory container for the cells-by-genes matrix with obs-based annotations)

## Examples

```
import anndata; hp = anndata.read_h5ad('HumanPancreas_aligned.h5ad'); model = scETM(hp.n_vars, hp.obs.batch_indices.nunique(), enable_batch_bias=True)
```

## Evaluation signals

- adata.n_vars reflects the expected gene count and adata contains the expected number of cells after loading
- adata.obs.batch_indices exists and its nunique() value matches the number of batches to pass to the model
- For pre-aligned transfer: source and target AnnData have identical (same order) homologous gene lists
- After prepare_for_transfer, source-unique gene parameters are discarded and target-unique gene parameters are randomly initialized
- Model consumes the AnnData without KeyError on batch_indices/cell_types columns or explicit batch_col/cell_type_col overrides

## Limitations

- For unaligned datasets, prepare_for_transfer induces a significant parameter change, so the model usually requires fine-tuning on the target dataset before evaluation
- Gene order matters: pre-aligned transfer requires the same or homologous gene lists in the same order
- If batch/cell type information is not in 'batch_indices'/'cell_types' columns, batch_col and cell_type_col arguments must be passed explicitly to scETM functions

## Evidence

- [methods] scETM requires a cells-by-genes matrix `adata` as input, in the format of an AnnData object: "scETM requires a cells-by-genes matrix `adata` as input, in the format of an AnnData object"
- [readme] By default, scETM looks for batch information in the 'batch_indices' column of the `adata.obs` DataFrame: "By default, scETM looks for batch information in the 'batch_indices' column of the `adata.obs` DataFrame, and cell type identity in the 'cell_types' column"
- [readme] The `prepare_for_transfer` function discards the parameters tied to the source-dataset-unique genes and randomly initializes those tied to the target-dataset-unique genes.: "The `prepare_for_transfer` function discards the parameters tied to the source-dataset-unique genes and randomly initializes those tied to the target-dataset-unique genes."
- [readme] mp = anndata.read_h5ad("MousePancreas.h5ad"): "# Prepare the source dataset, Mouse Pancreas
mp = anndata.read_h5ad("MousePancreas.h5ad")"
