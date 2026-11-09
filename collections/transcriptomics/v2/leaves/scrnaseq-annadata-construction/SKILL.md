---
name: scrnaseq-annadata-construction
description: Use when raw scRNA-seq studies must be combined into a single AnnData
  with explicit batch (source study) and cell-type annotation columns before training
  scETM or running batch-integration benchmarks.
when_to_use_negative:
- The input is already an AnnData with populated 'batch_indices' and 'cell_types'
  columns — no reconstruction needed
- The downstream tool requires a different container (e.g. Seurat object or LIGER's
  input format) rather than the AnnData contract described here
- More than one categorical batch variable must be corrected simultaneously — scETM
  currently handles only a single categorical batch variable
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scETM
  role: 'Defines the AnnData input contract: requires cells-by-genes matrix with ''batch_indices''
    and ''cell_types'' obs columns, and consumes adata.n_vars and batch count at model
    initialization'
  repo: github.com/hui2000ji/scETM
- name: AnnData
  role: Container format used to assemble and store the cells-by-genes matrix plus
    batch and cell-type metadata (read via anndata.read_h5ad)
  repo: https://anndata.readthedocs.io/en/latest/
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-021-25534-2
    title: Learning interpretable cellular and gene signature embeddings from single-cell
      transcriptomic data
  source_content_hash: sha256:95397ce03a8646a01049e667e49355ac7df8feb6865c2fb46736a7d624404ba5
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - AnnData
  - Harmony
  - LIGER
  - Scanorama
  - Seurat
  - UnsupervisedTrainer
  - scETM
  - scVAE-GM
  - scVI
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# scrnaseq-annadata-construction

## Summary

Assemble scRNA-seq count data (e.g. Mouse Retina Macosko GSE63476 and Shekhar GSE81905) into a cells-by-genes AnnData object whose obs frame carries 'batch_indices' and 'cell_types', the exact input format scETM and the benchmarked integration methods (LIGER, Harmony, Scanorama, Seurat, scVI, scVAE-GM) require.

## When to use

Use when raw scRNA-seq studies must be combined into a single AnnData with explicit batch (source study) and cell-type annotation columns before training scETM or running batch-integration benchmarks.

## When NOT to use

- The input is already an AnnData with populated 'batch_indices' and 'cell_types' columns — no reconstruction needed
- The downstream tool requires a different container (e.g. Seurat object or LIGER's input format) rather than the AnnData contract described here
- More than one categorical batch variable must be corrected simultaneously — scETM currently handles only a single categorical batch variable

## Inputs

- scRNA-seq count matrices from multiple source studies (e.g. GSE63476 Macosko, GSE81905 Shekhar)
- Per-cell batch labels (source study identifier)
- Per-cell gold-standard cell type annotations (including the 669 annotated Shekhar doublelets)

## Outputs

- Single cells-by-genes AnnData object with adata.obs['batch_indices'] and adata.obs['cell_types']
- AnnData ready for scETM training (UnsupervisedTrainer) or downstream benchmark scripts

## How to apply

Build a cells-by-genes AnnData object per the scETM input contract: 'scETM requires a cells-by-genes matrix adata as input, in the format of an AnnData object'. Assign the source study (e.g. Macosko vs Shekhar) to the 'batch_indices' column of adata.obs and the retinal cell identity annotations to 'cell_types', since by default scETM looks for batch information in 'batch_indices' and cell type identity in 'cell_types'. For the Mouse Retina two-study batch test, retain the 669 annotated Shekhar doublelets in the object so that batch overcorrection metrics (ARI/ASW/kBET) can later be recomputed with and without them. Verify that adata.obs.batch_indices.nunique() returns the expected number of batches (2) and that n_vars matches the retained gene count; these values feed the scETM constructor (scETM(adata.n_vars, adata.obs.batch_indices.nunique(), enable_batch_bias=True)). If batch and cell type metadata live in other columns, pass them via the batch_col and cell_type_col arguments instead of renaming.

## Related tools

- **scETM** (Defines the AnnData input contract: requires cells-by-genes matrix with 'batch_indices' and 'cell_types' obs columns, and consumes adata.n_vars and batch count at model initialization) — github.com/hui2000ji/scETM
- **AnnData** (Container format used to assemble and store the cells-by-genes matrix plus batch and cell-type metadata (read via anndata.read_h5ad)) — https://anndata.readthedocs.io/en/latest/

## Examples

```
import anndata
mp = anndata.read_h5ad("MousePancreas.h5ad")
```

## Evaluation signals

- adata is a valid AnnData object that round-trips through anndata.read_h5ad without losing obs metadata
- adata.obs contains non-null 'batch_indices' and 'cell_types' columns and adata.obs.batch_indices.nunique() equals 2 for the Macosko/Shekhar combination
- The 669 annotated Shekhar doublelets are present in the object so metrics can be recomputed with and without them
- scETM initialization scETM(adata.n_vars, adata.obs.batch_indices.nunique(), enable_batch_bias=True) succeeds without shape errors
- Downstream evaluation reproduces Supplementary Table 7 values (scETM ARI 0.859, ASW 0.2873, kBET 0.0656) within tolerance

## Limitations

- scETM's current model handles only a single categorical batch variable; multi-batch-variable correction is not supported
- Continuous/numerical batch variables (e.g. subject age) are not directly modeled and must be converted into categorical ranges
- If batch and cell type information are stored under different column names, the columns must be renamed or batch_col/cell_type_col must be supplied, otherwise scETM silently fails to find them

## Evidence

- [methods] scETM AnnData input contract: "scETM requires a cells-by-genes matrix `adata` as input, in the format of an AnnData object"
- [readme] Default batch and cell type columns: "By default, scETM looks for batch information in the 'batch_indices' column of the `adata.obs` DataFrame, and cell type identity in the 'cell_types' column."
- [readme] Alternative column arguments: "If your data stores the batch and cell type information in different columns, pass them to the `batch_col` and `cell_type_col` arguments"
- [readme] Model initialization consumes AnnData dimensions: "model = scETM(mp.n_vars, mp.obs.batch_indices.nunique(), enable_batch_bias=True)"
