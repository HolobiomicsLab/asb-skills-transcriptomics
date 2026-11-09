---
name: cross-study-label-harmonization-with-cellhint
description: Use when evaluating whether a subsampled scRNA-seq sketch (e.g. 10% scValue,
  Uniform, GeoSketch, Sphetcher, Hopper, KH, scSampler) preserves cross-study cell-type
  subtype relationships, via CellHint harmonization trees built from .h5ad inputs
  with X_pca representations.
when_to_use_negative:
- The goal is direct cell-type annotation accuracy (e.g. CellTypist, scANVI, ACTINN
  cell-type-assignment tasks) rather than preserving inter-study subtype relationship
  structure — the paper evaluates those as separate benchmark tasks
- The dataset is small enough to run CellHint on all cells, making sketching unnecessary
  and the tree-agreement comparison moot
- Cell type labels lack the fine-grained T-cell subtype structure needed for a meaningful
  subtype relationship tree (e.g. only a handful of coarse labels)
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: CellHint
  role: Performs the cross-study label harmonization that builds the T-cell subtype
    relationship tree for each sketch and for the full dataset
- name: scValue
  role: Generates the 10% sketch under test by ranking cells with out-of-bag random-forest
    data value and selecting top-DV cells within 0.1-wide DV bins
  repo: https://github.com/LHBCB/scvalue
- name: Scanpy
  role: Loads the .h5ad dataset and computes the top 3,000 HVGs and 50 PCs into adata.obsm['X_pca']
    used for sketching
- name: GeoSketch
  role: Baseline subsampling method whose harmonization tree agreement is compared
    against scValue's
- name: Sphetcher
  role: Baseline subsampling method whose harmonization tree agreement is compared
    against scValue's
- name: Hopper
  role: Baseline subsampling method whose harmonization tree agreement is compared
    against scValue's
- name: TreeHopper
  role: Partition-based Hopper variant included in the toolset for subsampling comparisons
- name: scSampler
  role: Baseline maximin-distance subsampling method whose harmonization tree agreement
    is compared against scValue's
- name: scikit-learn
  role: Underlies the random forest that computes per-cell out-of-bag data values
    in scValue
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bib/bbaf279
    title: 'scValue: value-based subsampling of large-scale single-cell transcriptomic
      data for machine and deep learning tasks'
  source_content_hash: sha256:7800b738660bf6b71ea01e37d54dd17bd627f749162cdcf84dd471783a680e12
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
  - CellHint
  - GeoSketch
  - Hopper
  - Scanpy
  - Sphetcher
  - TreeHopper
  - scSampler
  - scValue
  - scikit-learn
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# Cross-study label harmonization with CellHint

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Runs CellHint cross-study label harmonization on scRNA-seq sketches (e.g. a 10% scValue sketch of the 200k-cell human Spleen dataset) to build T-cell subtype relationship trees, then compares each method's tree against the full-data reference tree to judge how well a subsampling method preserves fine-grained cell-type structure.

## When to use

Use when evaluating whether a subsampled scRNA-seq sketch (e.g. 10% scValue, Uniform, GeoSketch, Sphetcher, Hopper, KH, scSampler) preserves cross-study cell-type subtype relationships, via CellHint harmonization trees built from .h5ad inputs with X_pca representations.

## When NOT to use

- The goal is direct cell-type annotation accuracy (e.g. CellTypist, scANVI, ACTINN cell-type-assignment tasks) rather than preserving inter-study subtype relationship structure — the paper evaluates those as separate benchmark tasks
- The dataset is small enough to run CellHint on all cells, making sketching unnecessary and the tree-agreement comparison moot
- Cell type labels lack the fine-grained T-cell subtype structure needed for a meaningful subtype relationship tree (e.g. only a handful of coarse labels)

## Inputs

- Large scRNA-seq dataset in .h5ad format (e.g. 200k-cell human Spleen) with cell type labels in adata.obs['cell_type']
- Preprocessed AnnData with top 3,000 highly variable genes and 50 PCs stored in adata.obsm['X_pca']
- Sketches (10% subsets) from scValue and six baselines: Uniform, GeoSketch, Sphetcher, Hopper, KH, and scSampler

## Outputs

- T-cell subtype relationship tree from CellHint harmonization for each of the seven sketches and the full dataset
- Per-method tree-agreement comparison table written to spleen_cellhint_tree_comparison.csv
- Plotted inferred trees (Figure 5) showing which sketch's tree most closely matches the full-data reference tree

## How to apply

Load the large scRNA-seq dataset (e.g. the 200k-cell human Spleen .h5ad) with scanpy's sc.read_h5ad, select the top 3,000 highly variable genes with sc.pp.highly_variable_genes, and compute 50 PCs into adata.obsm['X_pca']. Generate the sketch under test (e.g. SCValue with sketch_size=0.1, use_rep='X_pca', cell_type_key='cell_type', n_trees=100, type_weighting='inv_cv', strategy='FB', seed=42, which ranks cells by out-of-bag random-forest data value and selects top-DV cells within 0.1-wide DV bins) plus matched 10% sketches with Uniform, GeoSketch, Sphetcher, Hopper, KH, and scSampler. Run CellHint cross-study label harmonization separately on each of the seven sketches and on the full dataset to build the T-cell subtype relationship tree for each. Compare each sketch's subtype tree against the full-data reference tree, write the per-method tree-agreement comparison to a CSV (e.g. spleen_cellhint_tree_comparison.csv), and plot the inferred trees (Figure 5 in the paper) to verify that scValue's tree most closely reproduces the full-data T-cell subtype relationships.

## Related tools

- **CellHint** (Performs the cross-study label harmonization that builds the T-cell subtype relationship tree for each sketch and for the full dataset)
- **scValue** (Generates the 10% sketch under test by ranking cells with out-of-bag random-forest data value and selecting top-DV cells within 0.1-wide DV bins) — https://github.com/LHBCB/scvalue
- **Scanpy** (Loads the .h5ad dataset and computes the top 3,000 HVGs and 50 PCs into adata.obsm['X_pca'] used for sketching)
- **GeoSketch** (Baseline subsampling method whose harmonization tree agreement is compared against scValue's)
- **Sphetcher** (Baseline subsampling method whose harmonization tree agreement is compared against scValue's)
- **Hopper** (Baseline subsampling method whose harmonization tree agreement is compared against scValue's)
- **TreeHopper** (Partition-based Hopper variant included in the toolset for subsampling comparisons)
- **scSampler** (Baseline maximin-distance subsampling method whose harmonization tree agreement is compared against scValue's)
- **scikit-learn** (Underlies the random forest that computes per-cell out-of-bag data values in scValue)

## Examples

```
from scvalue import SCValue; scv = SCValue(adata=adata, sketch_size=0.1, use_rep='X_pca', cell_type_key='cell_type', n_trees=100, type_weighting='inv_cv', strategy='FB', seed=42); adata_sub = scv.value()
```

## Evaluation signals

- The comparison table (spleen_cellhint_tree_comparison.csv) must show scValue's harmonization tree agreeing most closely with the full-data reference tree among all seven subsampling methods
- Plotted subtype trees (Figure 5) must qualitatively show scValue's tree reproducing the full-data T-cell subtype relationships better than the six baselines
- DV outputs must be well-formed: dv stored in adata.obs['dv'], scv.dv a DataFrame of cell type + DV, and dv_values.csv / dv_summary.csv produced when write_dv=True
- Sketch size must be controlled: sketch_size=0.1 yields 10% of cells (float in (0,1]) with reproducibility from seed=42
- Preprocessing invariant: adata.obsm['X_pca'] must contain 50 PCs computed on the top 3,000 HVGs before sketching

## Limitations

- No accession numbers or repository identifiers were provided for the Spleen or other benchmark datasets, so dataset provenance must be resolved separately
- Tool versions for benchmarked software such as CellHint were not specified; only the scValue environment (Python 3.10, scikit-learn 1.5.2, Scanpy 1.10.2, AnnData 0.10.8) is documented
- scValue's random-forest OOB data value estimates may need hyperparameter tuning (more trees, adjusted max depth) to stabilize in noisy conditions, and use of 'X' with sparse matrices may trigger internal densification

## Evidence

- [full_text] Workflow step 4: [evidence span withheld — anchor status: not_found]
- [full_text] Validation criterion: [evidence span withheld — anchor status: not_found]
