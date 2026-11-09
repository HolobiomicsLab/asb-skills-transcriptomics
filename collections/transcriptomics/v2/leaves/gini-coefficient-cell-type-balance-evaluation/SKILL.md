---
name: gini-coefficient-cell-type-balance-evaluation
description: Use when comparing subsampling/sketching methods on scRNA-seq data and
  you need a single numeric score of how well each method preserves cell-type balance,
  especially capture of rare cell types.
when_to_use_negative:
- The input is the full (unsketched) dataset and no subsampling comparison is being
  made — Gini of the sketch is a subsampling-quality metric, not a general cluster-balance
  measure
- Cell-type annotations are absent or unreliable in adata.obs, since the Gini computation
  requires per-cell type labels
tools:
- name: scValue
  role: Sketching method whose cell-type balance is scored with the Gini coefficient
    (sketch_size=0.1, cell_type_key='cell_type', strategy='FB')
  repo: https://github.com/LHBCB/scvalue
- name: Scanpy
  role: Preprocessing of .h5ad datasets (top 3,000 HVGs, 50 PCs into adata.obsm['X_pca'])
    before sketching and Gini evaluation
- name: GeoSketch
  role: Comparator box-partitioning sketching method evaluated by Gini coefficient
- name: Sphetcher
  role: Comparator sphere-covering sketching method evaluated by Gini coefficient
- name: Hopper
  role: Comparator sketching method evaluated by Gini coefficient
- name: scSampler
  role: Comparator maximin-distance sketching method evaluated by Gini coefficient
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bib/bbaf279
    title: 'scValue: value-based subsampling of large-scale single-cell transcriptomic
      data for machine and deep learning tasks'
  source_content_hash: sha256:b6b2bb807ff7635d6d3c45c161fa75aa124a41bf4d1ccbb86ed3ad1a6dab3f5f
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
  - KH
  - Scanpy
  - Seurat
  - Sphetcher
  - scSampler
  - scValue
  - scikit-learn
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.625
n_claims: 16
---

# gini-coefficient-cell-type-balance-evaluation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Quantifies how evenly a scRNA-seq subsample ('sketch') represents cell types by computing the Gini coefficient over cell-type proportions in the sketch; used in the scValue benchmark to score the balance of rare-cell-type representation across sketching methods.

## When to use

Use when comparing subsampling/sketching methods on scRNA-seq data and you need a single numeric score of how well each method preserves cell-type balance, especially capture of rare cell types.

## When NOT to use

- The input is the full (unsketched) dataset and no subsampling comparison is being made — Gini of the sketch is a subsampling-quality metric, not a general cluster-balance measure
- Cell-type annotations are absent or unreliable in adata.obs, since the Gini computation requires per-cell type labels

## Inputs

- Sketch AnnData (.h5ad) or cell-type label vector for a subsample of a scRNA-seq dataset (e.g. 10% sketch of mTC, 89,429 cells)
- Full-dataset cell-type annotations in adata.obs['cell_type'] to compute sketch cell-type proportions

## Outputs

- Per-dataset Gini coefficient over cell-type proportions for each sketching method
- Mean ± SD Gini rank across 16 datasets (written to e.g. method_rankings_table4.csv)

## How to apply

For each dataset, generate a sketch (e.g. 10% of cells) with the method under test, then compute the Gini coefficient over the resulting cell-type proportions: a lower Gini means more balanced representation, which matters because rare cell types are easily lost by distance- or uniform-based sketching. In the scValue benchmark this is done for scValue (sketch_size=0.1, use_rep='X_pca', cell_type_key='cell_type', type_weighting='inv_cv', strategy='FB') and the six comparators (Uniform, GeoSketch, Sphetcher, Hopper, KH, scSampler). Rank all methods per dataset on the Gini metric and average ranks across the 16 public scRNA-seq datasets to obtain mean ± SD rank summaries (scValue achieved Gini rank 2.1 ± 1.5, the best among the seven methods). The Gini score should be reported alongside computation time and Hausdorff distance to capture efficiency, balance, and distributional fidelity separately.

## Related tools

- **scValue** (Sketching method whose cell-type balance is scored with the Gini coefficient (sketch_size=0.1, cell_type_key='cell_type', strategy='FB')) — https://github.com/LHBCB/scvalue
- **Scanpy** (Preprocessing of .h5ad datasets (top 3,000 HVGs, 50 PCs into adata.obsm['X_pca']) before sketching and Gini evaluation)
- **GeoSketch** (Comparator box-partitioning sketching method evaluated by Gini coefficient)
- **Sphetcher** (Comparator sphere-covering sketching method evaluated by Gini coefficient)
- **Hopper** (Comparator sketching method evaluated by Gini coefficient)
- **scSampler** (Comparator maximin-distance sketching method evaluated by Gini coefficient)

## Examples

```
sketch = scv.value(); gini = gini_coefficient(sketch.obs['cell_type'].value_counts(normalize=True).values)
```

## Evaluation signals

- Gini coefficient is computable for every sketch and lies in [0, 1], with lower values indicating more even cell-type proportions
- Per-method Gini ranks averaged over the 16 datasets should be reproducible against Table 4 values (scValue Gini rank 2.1 ± 1.5, the best of the seven methods)
- Sketch cell-type proportions are derived from the same cell_type_key labels as the full data, so per-type counts in the sketch must sum to the sketch size (e.g. 10% of cells)
- Results are written per dataset to method_rankings_table4.csv, enabling diff-comparison against reported rank summaries

## Limitations

- Gini only captures cell-type proportion balance, not distributional fidelity; it must be paired with Hausdorff distance (scValue Hausdorff rank 5.3 ± 1.1) for a full quality picture
- Accession numbers/repository identifiers for the 16 benchmark datasets are not provided, and package versions for comparator tools (GeoSketch, Sphetcher, Hopper, KH, scSampler) are unspecified, complicating exact reproduction

## Evidence

- [other] Benchmark workflow with Gini: [evidence span withheld — anchor status: not_found]
