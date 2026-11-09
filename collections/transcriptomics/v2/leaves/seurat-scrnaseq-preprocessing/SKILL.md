---
name: seurat-scrnaseq-preprocessing
description: Use when raw scRNA-seq count data (e.g. Tabula Muris thymus) must be
  normalized, HVG-selected, scaled, and embedded via PCA before trajectory or DEG
  analysis downstream.
when_to_use_negative:
- Input is already a normalized/processed AnnData (.h5ad) or Seurat object with PCA
  computed — skip to downstream steps like hs.haystack(adata, basis="pca")
- The goal is multi-condition comparison (e.g. wild-type vs knockout) — singleCellHaystack-based
  workflows downstream of this preprocessing do not easily support comparisons between
  multiple conditions
- Cluster-oriented differential expression between predefined groups is the goal —
  cluster-based DEG approaches integrate results more straightforwardly than trajectory/haystack-style
  analyses
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: Seurat
  role: Primary R framework for loading counts, normalization, HVG selection, scaling,
    PCA, and cell cluster subsetting prior to trajectory/DEG analysis
  repo: https://satijalab.org/seurat/
- name: satijalab/seurat-wrappers
  role: Provides Seurat-to-monocle3 conversion and trajectory vignette ('Calculating
    Trajectories with Monocle 3 and Seurat') enabling pseudotime inference on the
    preprocessed Seurat object
  repo: https://github.com/satijalab/seurat-wrappers
- name: singleCellHaystack
  role: Downstream consumer of the preprocessed expression matrix and PCA space; predicts
    differentially active features along 1D trajectories or in higher-dimensional
    latent spaces
  repo: https://github.com/alexisvdb/singleCellHaystack
- name: monocle3
  role: Receives the converted Seurat object to run learn_graph and order_cells for
    pseudotime inference from DN to SP thymocytes
  repo: https://cole-trapnell-lab.github.io/monocle3
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41598-023-38965-2
    title: A universal tool for predicting differentially active features in single-cell
      and spatial genomics data
  source_content_hash: sha256:848bbb83f5e68f993e353d5190a336a2c1a15022fa82cee0593e6b74471d0486
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - Seurat
  - monocle3
  - satijalab/seurat-wrappers
  - singleCellHaystack
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# seurat-scrnaseq-preprocessing

## Summary

Standard Seurat scRNA-seq preprocessing workflow — loading a count matrix, normalization, highly variable gene identification, scaling, and PCA — that produces the processed expression matrix and PCA latent space consumed by downstream tools such as monocle3 pseudotime inference and singleCellHaystack differential activity prediction.

## When to use

Use when raw scRNA-seq count data (e.g. Tabula Muris thymus) must be normalized, HVG-selected, scaled, and embedded via PCA before trajectory or DEG analysis downstream.

## When NOT to use

- Input is already a normalized/processed AnnData (.h5ad) or Seurat object with PCA computed — skip to downstream steps like hs.haystack(adata, basis="pca")
- The goal is multi-condition comparison (e.g. wild-type vs knockout) — singleCellHaystack-based workflows downstream of this preprocessing do not easily support comparisons between multiple conditions
- Cluster-oriented differential expression between predefined groups is the goal — cluster-based DEG approaches integrate results more straightforwardly than trajectory/haystack-style analyses

## Inputs

- scRNA-seq raw count matrix (e.g. Tabula Muris thymus) loadable into a Seurat object
- Cell-type cluster annotations for subsetting cell populations (e.g. DN vs CD4+/CD8+ SP T cells)

## Outputs

- Normalized, scaled Seurat object with highly variable genes identified and PCA embedding computed
- Processed expression matrix suitable as input to singleCellHaystack or monocle3 (via Seurat-to-monocle3 conversion)

## How to apply

Load the scRNA-seq count matrix into a Seurat object, then normalize counts, identify highly variable genes, scale the data, and run PCA (steps 1 of the Tabula Muris thymus workflow). Subset the cell populations of interest (e.g. double-negative and CD4+/CD8+ single-positive T cell clusters) on the processed object before trajectory analysis. The resulting PCA embedding and scaled expression matrix serve as inputs to subsequent steps: Seurat-to-monocle3 conversion for pseudotime (available via satijalab/seurat-wrappers Monocle 3 integration) and singleCellHaystack, which accepts the processed matrix with 1D pseudotime or higher-dimensional latent space coordinates. Preprocessing quality determines downstream behavior — singleCellHaystack v1.0+ models continuous activity levels on the expression matrix and uses the PCA basis as its default coordinate input.

## Related tools

- **Seurat** (Primary R framework for loading counts, normalization, HVG selection, scaling, PCA, and cell cluster subsetting prior to trajectory/DEG analysis) — https://satijalab.org/seurat/
- **satijalab/seurat-wrappers** (Provides Seurat-to-monocle3 conversion and trajectory vignette ('Calculating Trajectories with Monocle 3 and Seurat') enabling pseudotime inference on the preprocessed Seurat object) — https://github.com/satijalab/seurat-wrappers
- **singleCellHaystack** (Downstream consumer of the preprocessed expression matrix and PCA space; predicts differentially active features along 1D trajectories or in higher-dimensional latent spaces) — https://github.com/alexisvdb/singleCellHaystack
- **monocle3** (Receives the converted Seurat object to run learn_graph and order_cells for pseudotime inference from DN to SP thymocytes) — https://cole-trapnell-lab.github.io/monocle3

## Examples

```
seurat_obj <- CreateSeuratObject(counts); seurat_obj <- NormalizeData(seurat_obj) %>% FindVariableFeatures() %>% ScaleData() %>% RunPCA()
```

## Evaluation signals

- The Seurat object contains normalized counts, a set of highly variable genes, scaled data, and a valid PCA reduction before any downstream step is run
- Cell subsetting retains only the intended populations (DN and CD4+/CD8+ SP T cell clusters) and the converted object preserves cell and gene metadata for monocle3 learn_graph/order_cells
- The processed expression matrix is accepted by singleCellHaystack without error and yields ranked features via top_features (e.g. top 1000 DEGs extractable with top_features)
- Downstream pseudotime ordering proceeds from DN to SP states, confirming preprocessing did not corrupt cluster identity or trajectory structure

## Limitations

- Preprocessing choices (HVG selection, scaling) feed directly into the PCA basis that singleCellHaystack and monocle3 consume; a poor PCA space will bias downstream DEG and pseudotime results
- singleCellHaystack results downstream are not straightforward to incorporate into cluster-oriented scRNA-seq analyses, so this preprocessing pipeline is best suited to trajectory/spatial rather than cluster-comparison questions
- singleCellHaystack cannot easily conduct comparisons between multiple conditions (e.g. wild-type and knockout), constraining the experimental designs this preprocessing-to-DEG workflow supports

## Evidence

- [other] Seurat preprocessing for trajectory DEG analysis: [evidence span withheld — anchor status: not_found]
- [readme] Monocle 3 integration via SeuratWrappers: [evidence span withheld — anchor status: external_doc]
- [methods] singleCellHaystack consumes the PCA basis: "res = hs.haystack(adata, basis="pca")"
- [abstract] continuous-feature support in the downstream tool: "singleCellHaystack now accepts continuous features that can be RNA or protein expression, chromatin accessibility or module scores from single-cell, spatial and even bulk genomics data"
