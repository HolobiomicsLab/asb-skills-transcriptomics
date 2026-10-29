---
name: cross-dataset-shared-gene-merging
description: Use when two or more scRNA-seq AnnData objects with partially non-overlapping
  (or homologous but differently ordered) gene lists must be merged into a single
  cells-by-genes matrix before integration, transfer, or runtime benchmarking.
when_to_use_negative:
- Datasets are already pre-aligned with identical gene lists in the same order — just
  load them directly (e.g., HumanPancreas_aligned.h5ad)
- Gene sets differ substantially and a soft transfer is preferred — use scETM's prepare_for_transfer
  to discard source-unique-gene parameters and fine-tune, rather than hard intersecting
  genes
- Only one dataset is involved — no cross-dataset gene merging is needed
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: scETM
  role: Downstream topic model trained on the merged shared-gene AnnData for the AD+MDD
    scalability benchmark
  repo: github.com/hui2000ji/scETM
- name: anndata
  role: Reads the .h5ad datasets and performs the cells-by-genes matrix subsetting/merge
  repo: https://anndata.readthedocs.io/en/latest/
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-021-25534-2
    title: Learning interpretable cellular and gene signature embeddings from single-cell
      transcriptomic data
  source_content_hash: sha256:e1dc4d3673c2a8c11cabba984086f89764bb72eff60cc28902821213391630e2
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
  - scVI-LD
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 14
---

# cross-dataset-shared-gene-merging

## Summary

Merges multiple scRNA-seq AnnData objects (e.g., AD and MDD cohorts) by keeping only shared/homologous genes, then restricts to the top 3000 most variable genes to build a common feature space for cross-dataset integration and scalability benchmarking against scETM and competing methods.

## When to use

Use when two or more scRNA-seq AnnData objects with partially non-overlapping (or homologous but differently ordered) gene lists must be merged into a single cells-by-genes matrix before integration, transfer, or runtime benchmarking.

## When NOT to use

- Datasets are already pre-aligned with identical gene lists in the same order — just load them directly (e.g., HumanPancreas_aligned.h5ad)
- Gene sets differ substantially and a soft transfer is preferred — use scETM's prepare_for_transfer to discard source-unique-gene parameters and fine-tune, rather than hard intersecting genes
- Only one dataset is involved — no cross-dataset gene merging is needed

## Inputs

- AD scRNA-seq AnnData (.h5ad) object
- MDD scRNA-seq AnnData (.h5ad) object
- Homologous gene mapping (when gene symbols differ across datasets)
- Batch and cell type annotations in adata.obs ('batch_indices', 'cell_types')

## Outputs

- Merged AnnData object restricted to the shared/homologous gene intersection
- Feature-space-restricted matrix of the 3000 most variable genes
- Subsampled AnnData objects (14k, 28k, 70k, 148,247 cells) for scalability benchmarking

## How to apply

Load each dataset as an AnnData cells-by-genes matrix (adata.obs carries batch and cell type annotations such as 'batch_indices' and 'cell_types'). Intersect the gene lists across datasets, keeping only shared or homologous genes; when homologous lists are used (e.g., across species or orthologues), the gene order must be aligned across objects. Restrict the merged object to the 3000 most variable genes as the feature space, then subsample cells without replacement to the target sizes (14,000 / 28,000 / 70,000 / 148,247 for the AD+MDD scalability benchmark). The rationale is that models like scETM, and competitors like Harmony, Scanorama, Seurat, LIGER, scVI, scVI-LD and scVAE-GM all assume a common gene axis; alternatively, for unaligned cross-dataset transfer, scETM's prepare_for_transfer discards parameters tied to source-dataset-unique genes instead of hard-filtering.

## Related tools

- **scETM** (Downstream topic model trained on the merged shared-gene AnnData for the AD+MDD scalability benchmark) — github.com/hui2000ji/scETM
- **anndata** (Reads the .h5ad datasets and performs the cells-by-genes matrix subsetting/merge) — https://anndata.readthedocs.io/en/latest/

## Examples

```
shared = ad.var_names.intersection(mdd.var_names); merged = anndata.concat([ad[:, shared], mdd[:, shared]], join='inner')
```

## Evaluation signals

- Gene dimension of the merged object equals the size of the shared/homologous gene intersection across all input datasets, and gene order is identical across concatenated objects
- Exactly 3000 most variable genes are retained as the feature space
- Subsample sizes match the benchmark design (14,000 / 28,000 / 70,000 / 148,247 cells) and cells are drawn without replacement
- Batch ('batch_indices') and cell type ('cell_types') annotations survive the merge so ARI can be computed against annotated labels
- Downstream model (scETM) initializes and trains on the merged object without gene-mismatch errors

## Limitations

- Hard gene intersection discards source-dataset-unique genes entirely, which loses signal that prepare_for_transfer-based soft transfer would retain
- Homologous (not identical) gene lists require correct ordering/alignment; misordered genes silently corrupt the merged matrix
- scETM's current model handles only a single categorical batch variable, so merged batch designs must encode batch as one categorical column

## Evidence

- [other] Load the AD and MDD scRNA-seq AnnData objects and merge them keeping only shared/homologous genes, then select the 3000 most variable genes as the feature space.: [evidence span withheld — anchor status: not_found]
- [readme] Pre-aligned datasets have the same or homologous gene lists and gene order must be aligned: "the source and target dataset are pre-aligned, meaning they have the same / homologous gene **lists** (note that the order of the genes would have to be aligned)"
- [readme] scETM requires a cells-by-genes AnnData matrix as input: "scETM requires a cells-by-genes matrix `adata` as input, in the format of an AnnData object"
- [readme] prepare_for_transfer discards parameters tied to source-dataset-unique genes for unaligned gene sets: "The `prepare_for_transfer` function discards the parameters tied to the source-dataset-unique genes and randomly initializes those tied to the target-dataset-unique genes"
- [abstract] scETM is scalable to over 10^6 cells, motivating the merged large-scale benchmark: "scETM is scalable to over 10 6 cells and confers remarkable cross-tissue and cross-species zero-shot transfer-learning performance."
