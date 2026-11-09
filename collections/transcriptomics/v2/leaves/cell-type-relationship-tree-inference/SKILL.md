---
name: cell-type-relationship-tree-inference
description: Use when you need to assess whether a subsampled ('sketched') single-cell
  dataset preserves cross-study cell-type subtype relationships, by harmonizing labels
  with CellHint and comparing the inferred relationship tree against a full-data reference
  tree.
when_to_use_negative:
- The analysis goal is cell-type annotation accuracy (e.g. CellTypist/scANVI tasks)
  rather than cross-study label harmonization and subtype relationship structure
- No cell type labels are available in adata.obs for the dataset or the sketches,
  since CellHint harmonization and scValue's cell_type_key both require annotations
- The dataset is small enough to run CellHint directly on all cells, making sketch-based
  tree comparison unnecessary
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scValue
  role: Value-based subsampling method whose sketch is used to build the CellHint
    tree being compared against the full-data reference
  repo: https://github.com/LHBCB/scvalue
- name: Scanpy
  role: Reads the .h5ad dataset, selects top 3,000 highly variable genes, and computes
    50 PCs into adata.obsm['X_pca'] before sketching
- name: GeoSketch
  role: Baseline sketching method whose CellHint tree agreement is compared against
    scValue's
- name: Sphetcher
  role: Baseline sketching method whose CellHint tree agreement is compared against
    scValue's
- name: Hopper
  role: Baseline sketching method whose CellHint tree agreement is compared against
    scValue's
- name: TreeHopper
  role: Partition-based Hopper variant listed among the compared subsampling tools
- name: scSampler
  role: Baseline maximin-distance sketching method whose CellHint tree agreement is
    compared against scValue's
- name: scikit-learn
  role: Provides the random forest (scikit-learn 1.5.2) used by scValue for out-of-bag
    data value estimation
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bib/bbaf279
    title: 'scValue: value-based subsampling of large-scale single-cell transcriptomic
      data for machine and deep learning tasks'
  source_content_hash: sha256:40f24583fb44af530967934c76541c5c1af6ab945873227454e2d94c8701e5d6
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/LHBCB/scvalue
  repo_url: https://github.com/LHBCB/scvalue
  tools:
  - GeoSketch
  - Hopper
  - Scanpy
  - Sphetcher
  - TreeHopper
  - scSampler
  - scValue
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# cell-type-relationship-tree-inference

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Builds a cross-study cell-type subtype relationship tree with CellHint on a scRNA-seq dataset (e.g. the 200k-cell human Spleen dataset) and compares trees derived from subsampled sketches against the full-data reference tree, here to show that a 10% scValue sketch best preserves fine-grained T-cell subtype relationships.

## When to use

Use when you need to assess whether a subsampled ('sketched') single-cell dataset preserves cross-study cell-type subtype relationships, by harmonizing labels with CellHint and comparing the inferred relationship tree against a full-data reference tree.

## When NOT to use

- The analysis goal is cell-type annotation accuracy (e.g. CellTypist/scANVI tasks) rather than cross-study label harmonization and subtype relationship structure
- No cell type labels are available in adata.obs for the dataset or the sketches, since CellHint harmonization and scValue's cell_type_key both require annotations
- The dataset is small enough to run CellHint directly on all cells, making sketch-based tree comparison unnecessary

## Inputs

- Full scRNA-seq AnnData .h5ad file (e.g. 200k-cell human Spleen dataset, log-normalised) with cell type annotations in adata.obs['cell_type']
- 10% cell sketches of the same dataset produced by scValue and six baselines (Uniform, GeoSketch, Sphetcher, Hopper, KH, scSampler)

## Outputs

- T-cell subtype relationship tree from CellHint label harmonization for each sketch and for the full dataset
- Per-method tree-agreement comparison table (spleen_cellhint_tree_comparison.csv)
- Plotted subtype trees (Figure 5) for qualitative comparison against the full-data reference tree

## How to apply

Load the scRNA-seq .h5ad (e.g. the 200k-cell human Spleen dataset) with scanpy's sc.read_h5ad, select the top 3,000 highly variable genes, and compute 50 PCs into adata.obsm['X_pca']. Generate a 10% sketch with SCValue (sketch_size=0.1, use_rep='X_pca', cell_type_key='cell_type', n_trees=100, type_weighting='inv_cv', strategy='FB', seed=42), which ranks cells by out-of-bag random-forest data value and picks top-DV cells within 0.1-wide bins. Build matched 10% sketches with the six baselines (Uniform, GeoSketch, Sphetcher, Hopper, KH, scSampler). Run CellHint cross-study label harmonization separately on each sketch and on the full data to build the T-cell subtype relationship tree for each. Compare each sketch's tree against the full-data reference tree, write per-method agreement to a CSV (spleen_cellhint_tree_comparison.csv), and plot the inferred trees; validation requires scValue's tree to agree most closely with the full-data tree among all seven methods.

## Related tools

- **scValue** (Value-based subsampling method whose sketch is used to build the CellHint tree being compared against the full-data reference) — https://github.com/LHBCB/scvalue
- **Scanpy** (Reads the .h5ad dataset, selects top 3,000 highly variable genes, and computes 50 PCs into adata.obsm['X_pca'] before sketching)
- **GeoSketch** (Baseline sketching method whose CellHint tree agreement is compared against scValue's)
- **Sphetcher** (Baseline sketching method whose CellHint tree agreement is compared against scValue's)
- **Hopper** (Baseline sketching method whose CellHint tree agreement is compared against scValue's)
- **TreeHopper** (Partition-based Hopper variant listed among the compared subsampling tools)
- **scSampler** (Baseline maximin-distance sketching method whose CellHint tree agreement is compared against scValue's)
- **scikit-learn** (Provides the random forest (scikit-learn 1.5.2) used by scValue for out-of-bag data value estimation)

## Examples

```
import scanpy as sc; from scvalue import SCValue; adata = sc.read_h5ad('spleen.h5ad'); sc.pp.highly_variable_genes(adata, n_top_genes=3000); sc.pp.pca(adata, n_comps=50); scv = SCValue(adata=adata, sketch_size=0.1, use_rep='X_pca', cell_type_key='cell_type', n_trees=100, type_weighting='inv_cv', strategy='FB', seed=42); adata_sub = scv.value()
```

## Evaluation signals

- The spleen_cellhint_tree_comparison.csv table must show scValue's harmonization tree agreeing most closely with the full-data tree among all seven subsampling methods
- Plotted trees should qualitatively match the reported spleen-dataset result (Figure 5), with scValue's tree most closely reproducing full-data T-cell subtype relationships
- PCA representation must exist in adata.obsm['X_pca'] with 50 components computed on the top 3,000 HVGs before SCValue is instantiated
- Each sketch must contain ~10% of cells (sketch_size=0.1) and retain per-cell DV values in obs (adata.obs['dv'])
- Identical SCValue parameters (n_trees=100, type_weighting='inv_cv', strategy='FB', seed=42) should reproduce the same sketch given the fixed seed

## Limitations

- Tool versions for benchmarked baselines (CellHint, GeoSketch, Sphetcher, Hopper, KH, scSampler) were not specified in the study, complicating exact reproduction
- No accession numbers or repository identifiers were provided for the 16 benchmark datasets, so the Spleen .h5ad must be sourced separately
- scValue's DV quality depends on random-forest stability; tuning hyperparameters (e.g. increasing n_trees, adjusting max depth) may be needed in noisy conditions

## Evidence

- [abstract] cross-study label harmonization case study: [evidence span withheld — anchor status: not_found]
