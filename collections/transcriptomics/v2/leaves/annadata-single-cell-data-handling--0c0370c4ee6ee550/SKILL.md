---
name: annadata-single-cell-data-handling
description: Use when scRNA-seq datasets must be loaded, annotated with batch/cell-type
  labels, or passed between methods (e.g. scETM, scVI, transfer learning) as cells-by-genes
  AnnData objects with 'batch_indices' and 'cell_types' obs columns.
when_to_use_negative:
- Data is not single-cell scRNA-seq (e.g. bulk RNA-seq or a non-cell-by-gene feature
  table), since the AnnData cells-by-genes layout and cell/batch obs annotations would
  not apply
- Correction of multiple categorical batch variables or continuous batch variables
  (e.g. subject age) is required without binning — scETM's current model only considers
  a single categorical batch variable
- Multi-omic integration (scRNA-seq with protein or scATAC-seq) is the goal — scETM
  does not yet implement this
tools:
- name: scETM
  role: Consumer of the AnnData input; requires a cells-by-genes AnnData with batch_indices/cell_types
    obs columns and writes embeddings back into obsm/varm/uns
  repo: github.com/hui2000ji/scETM
- name: anndata
  role: Python library used to read .h5ad files into AnnData objects and inspect n_vars,
    obs.batch_indices, obsm, varm, uns slots
  repo: https://anndata.readthedocs.io/en/latest/
- name: scvi-tools
  role: Downstream/baseline integration framework also built on top of PyTorch and
    AnnData, sharing the same cells-by-genes object conventions
  repo: https://github.com/scverse/scvi-tools
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-021-25534-2
    title: Learning interpretable cellular and gene signature embeddings from single-cell
      transcriptomic data
  source_content_hash: sha256:b20f7106f6b6b5023b7e27c36b850dec51da49a048afab6d6d1e0b6029ab7e25
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Harmony
  - LIGER
  - Scanorama
  - Seurat
  - anndata
  - scETM
  - scVAE-GM
  - scVI
  - scvi-tools
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# annadata-single-cell-data-handling

## Summary

Handles single-cell scRNA-seq data as cells-by-genes AnnData objects (.h5ad), carrying batch and cell-type metadata in adata.obs and storing learned cell/gene/topic embeddings in adata.obsm/varm/uns. This is the interchange format that scETM and its baselines (scVI, etc.) consume and write.

## When to use

Use when scRNA-seq datasets must be loaded, annotated with batch/cell-type labels, or passed between methods (e.g. scETM, scVI, transfer learning) as cells-by-genes AnnData objects with 'batch_indices' and 'cell_types' obs columns.

## When NOT to use

- Data is not single-cell scRNA-seq (e.g. bulk RNA-seq or a non-cell-by-gene feature table), since the AnnData cells-by-genes layout and cell/batch obs annotations would not apply
- Correction of multiple categorical batch variables or continuous batch variables (e.g. subject age) is required without binning — scETM's current model only considers a single categorical batch variable
- Multi-omic integration (scRNA-seq with protein or scATAC-seq) is the goal — scETM does not yet implement this

## Inputs

- Single-cell scRNA-seq dataset as an AnnData (.h5ad) cells-by-genes matrix, e.g. MousePancreas.h5ad or HumanPancreas_aligned.h5ad
- Batch labels in adata.obs['batch_indices'] (or another column passed via batch_col)
- Ground-truth cell type labels in adata.obs['cell_types'] (or another column passed via cell_type_col)

## Outputs

- AnnData annotated with scETM embeddings: obsm['delta'] (unnormalized cell embeddings), obsm['theta'] (normalized cell embeddings), varm['rho'] (gene embeddings), uns['alpha'] (topic embeddings)
- Evaluation artifacts (ARI/NMI clustering metrics and UMAP plots, e.g. figures/scETM_MP) computed against adata.obs['cell_types']

## How to apply

Load each dataset with anndata.read_h5ad (e.g. 'MousePancreas.h5ad') so it is a cells-by-genes AnnData matrix, and confirm scETM's expected metadata: batch information in adata.obs['batch_indices'] and cell-type identity in adata.obs['cell_types']; if stored elsewhere, pass batch_col and cell_type_col arguments instead of renaming. Instantiate models using the AnnData's own dimensions (mp.n_vars for genes, mp.obs.batch_indices.nunique() for batches), so the model is sized directly from the object. After training with UnsupervisedTrainer, call model.get_all_embeddings_and_nll(adata) to write unnormalized cell embeddings to obsm['delta'], normalized cell embeddings to obsm['theta'], gene embeddings to varm['rho'], and topic embeddings to uns['alpha'], then evaluate with evaluate(mp, embedding_key='delta'). For transfer between datasets with different gene sets, use prepare_for_transfer, which discards parameters tied to source-dataset-unique genes and randomly initializes those tied to target-dataset-unique genes; pre-aligned datasets must share the same or homologous gene lists in the same order.

## Related tools

- **scETM** (Consumer of the AnnData input; requires a cells-by-genes AnnData with batch_indices/cell_types obs columns and writes embeddings back into obsm/varm/uns) — github.com/hui2000ji/scETM
- **anndata** (Python library used to read .h5ad files into AnnData objects and inspect n_vars, obs.batch_indices, obsm, varm, uns slots) — https://anndata.readthedocs.io/en/latest/
- **scvi-tools** (Downstream/baseline integration framework also built on top of PyTorch and AnnData, sharing the same cells-by-genes object conventions) — https://github.com/scverse/scvi-tools

## Examples

```
import anndata; mp = anndata.read_h5ad("MousePancreas.h5ad"); model = scETM(mp.n_vars, mp.obs.batch_indices.nunique(), enable_batch_bias=True)
```

## Evaluation signals

- File loads successfully via anndata.read_h5ad and adata.n_vars equals the expected gene count, while adata.obs.batch_indices.nunique() returns the expected number of batches used at model initialization
- After model.get_all_embeddings_and_nll, adata.obsm['delta'] and adata.obsm['theta'] each contain one embedding row per cell, varm['rho'] has gene embeddings, and uns['alpha'] has topic embeddings
- evaluate(mp, embedding_key='delta') runs without error and computes ARI/NMI against adata.obs['cell_types'] (e.g. reproduced scETM ARI values 0.946/0.943/0.761/0.996/0.717/0.859 on MP/HP/TM/AD/MDD/MR)
- For transfer learning, the source and target AnnData have identical/homologous gene lists in the same order (pre-aligned case), or prepare_for_transfer was invoked before fine-tuning (unaligned case)

## Limitations

- scETM's current model handles only a single categorical batch variable; multi-batch-variable correction is future work, and numerical batch variables such as subject age must be converted into categorical ranges
- For unaligned datasets, prepare_for_transfer discards source-unique-gene parameters and randomly initializes target-unique-gene parameters, so the model usually requires fine-tuning before evaluation on the target dataset
- For p-scETM with trainable_gene_emb_dim=0, the gene set is restricted to the intersection of scRNA-seq genes and the pathways-by-genes matrix from pathDIP4 (keeping only pathways with more than 5 genes)

## Evidence

- [methods] scETM requires a cells-by-genes matrix `adata` as input, in the format of an AnnData object: "scETM requires a cells-by-genes matrix `adata` as input, in the format of an AnnData object"
- [readme] batch and cell type metadata location: "By default, scETM looks for batch information in the 'batch_indices' column of the `adata.obs` DataFrame, and cell type identity in the 'cell_types' column."
- [readme] embedding slot layout: "Unnormalized cell embeddings will be stored at mp.obsm['delta'], normalized cell embeddings at mp.obsm['theta'], gene embeddings at mp.varm['rho'], topic embeddings at mp.uns['alpha']."
- [methods] model sizing from AnnData dimensions: "model = scETM(mp.n_vars, mp.obs.batch_indices.nunique(), enable_batch_bias=True)"
- [methods] transfer to unaligned datasets: "The `prepare_for_transfer` function discards the parameters tied to the source-dataset-unique genes and randomly initializes those tied to the target-dataset-unique genes."
- [discussion] single batch variable limitation: "as our current model only considers a single categorical batch variable, we can extend it to correct for multiple categorical batch variables"
