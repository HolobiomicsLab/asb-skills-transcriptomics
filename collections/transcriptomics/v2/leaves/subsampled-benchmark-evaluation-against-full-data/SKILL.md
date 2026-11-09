---
name: subsampled-benchmark-evaluation-against-full-data
description: Use when a scRNA-seq dataset is too large to analyse directly (tens of
  thousands to millions of cells) and you must decide which subsampling method (scValue,
  Uniform, GeoSketch, Sphetcher, Hopper, KH, scSampler) preserves downstream-task
  structure, judged against a full-data reference.
when_to_use_negative:
- The dataset is small enough to process in full — sketches and full-data comparison
  add cost with no benefit.
- The AnnData lacks cell type labels for the chosen cell_type_key, so scValue's DV
  computation and per-type sketch allocation cannot run as specified.
- You only need a quick uniform subset for visualization, not a method comparison
  against a full-data reference outcome.
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scValue
  role: 'Primary sketching method: ranks cells by out-of-bag random-forest data value
    and selects top-DV cells within 0.1-wide DV bins (FB strategy) for the evaluated
    sketch'
  repo: https://github.com/LHBCB/scvalue
- name: CellHint
  role: Downstream cross-study label harmonization task used to build the T-cell subtype
    relationship tree from each sketch and the full dataset
- name: Scanpy
  role: 'Preprocessing: reads the .h5ad, selects top 3,000 highly variable genes,
    and computes 50 PCs into adata.obsm[''X_pca''] used by scValue'
- name: GeoSketch
  role: Baseline sketching method (equal-sized boxes over expression space) included
    in the seven-method comparison
- name: Sphetcher
  role: Baseline sketching method (small-radius spheres covering cells) included in
    the seven-method comparison
- name: Hopper
  role: Baseline sketching method with theoretical guarantees for maintaining dataset
    structure, included in the seven-method comparison
- name: scSampler
  role: Baseline sketching method using maximin distance design, included in the seven-method
    comparison
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bib/bbaf279
    title: 'scValue: value-based subsampling of large-scale single-cell transcriptomic
      data for machine and deep learning tasks'
  source_content_hash: sha256:6f6ae5106ab2d964417fdcecd90ae5a829c08805556f649444deb8b54bc75fb2
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
yes_rate: 0.6666666666666666
n_claims: 12
---

# subsampled-benchmark-evaluation-against-full-data

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmarks single-cell subsampling ('sketching') methods by running the same downstream task on each sketch and comparing outputs against the full-dataset reference — e.g. running CellHint cross-study label harmonization on 10% sketches of a 200k-cell Spleen scRNA-seq .h5ad and scoring subtype-tree agreement.

## When to use

Use when a scRNA-seq dataset is too large to analyse directly (tens of thousands to millions of cells) and you must decide which subsampling method (scValue, Uniform, GeoSketch, Sphetcher, Hopper, KH, scSampler) best preserves downstream-task structure, judged against a full-data reference.

## When NOT to use

- The dataset is small enough to process in full — sketches and full-data comparison add cost with no benefit.
- The AnnData lacks cell type labels for the chosen cell_type_key, so scValue's DV computation and per-type sketch allocation cannot run as specified.
- You only need a quick uniform subset for visualization, not a method comparison against a full-data reference outcome.

## Inputs

- Large scRNA-seq dataset in AnnData .h5ad format with cell type labels (e.g. adata.obs['cell_type']), preprocessed to X_pca over ~3,000 HVGs
- Subsampling methods to benchmark (scValue, Uniform, GeoSketch, Sphetcher, Hopper, KH, scSampler)
- Downstream task pipeline (e.g. CellHint cross-study label harmonization) runnable on both sketches and full data

## Outputs

- One sketch (AnnData subset) per subsampling method at the same fraction/size
- Downstream-task output per sketch plus full-data reference (e.g. CellHint T-cell subtype relationship trees; Figure 5)
- Per-method agreement table (e.g. spleen_cellhint_tree_comparison.csv)
- Optional per-cell data values: adata.obs['dv'], dv_values.csv, dv_summary.csv when write_dv=True

## How to apply

Preprocess the full .h5ad with Scanpy (top 3,000 highly variable genes, 50 PCs stored in adata.obsm['X_pca']), then generate matched sketches — here 10% of a ~200k-cell human Spleen dataset — with each method under identical settings (e.g. SCValue with sketch_size=0.1, use_rep='X_pca', cell_type_key='cell_type', n_trees=100, type_weighting='inv_cv', strategy='FB', seed=42). Run the downstream task independently on each sketch and on the full dataset; for this task CellHint builds a cross-study T-cell subtype relationship tree from each. Quantify agreement of each sketch's output against the full-data reference (tree agreement for CellHint; Gini coefficient and Hausdorff distance for general sketch quality per the article's 16-dataset evaluation) and tabulate per-method results. Rationale: the article shows few empirical studies test how sketching methods affect downstream ML/DL tasks, and scValue — which ranks cells by out-of-bag random-forest 'data value' — most closely matched full-data performance.

## Related tools

- **scValue** (Primary sketching method: ranks cells by out-of-bag random-forest data value and selects top-DV cells within 0.1-wide DV bins (FB strategy) for the evaluated sketch) — https://github.com/LHBCB/scvalue
- **CellHint** (Downstream cross-study label harmonization task used to build the T-cell subtype relationship tree from each sketch and the full dataset)
- **Scanpy** (Preprocessing: reads the .h5ad, selects top 3,000 highly variable genes, and computes 50 PCs into adata.obsm['X_pca'] used by scValue)
- **GeoSketch** (Baseline sketching method (equal-sized boxes over expression space) included in the seven-method comparison)
- **Sphetcher** (Baseline sketching method (small-radius spheres covering cells) included in the seven-method comparison)
- **Hopper** (Baseline sketching method with theoretical guarantees for maintaining dataset structure, included in the seven-method comparison)
- **scSampler** (Baseline sketching method using maximin distance design, included in the seven-method comparison)

## Examples

```
from scvalue import SCValue; adata_sub = SCValue(adata=adata, sketch_size=0.1, use_rep='X_pca', cell_type_key='cell_type', n_trees=100, type_weighting='inv_cv', strategy='FB', seed=42).value()
```

## Evaluation signals

- The per-method comparison table (e.g. spleen_cellhint_tree_comparison.csv) shows scValue's harmonization tree agreeing most closely with the full-data reference tree among all seven methods.
- Plotted subtype trees qualitatively reproduce the full-data T-cell subtype relationships (Figure 5 in the paper).
- All sketches use identical size (e.g. 10%) and shared preprocessing (top 3,000 HVGs, 50 PCs in X_pca) so comparisons are like-for-like.
- scValue run with write_dv=True yields dv_values.csv and dv_summary.csv, and adata.obs['dv'] is populated for the full dataset.
- General sketch-quality metrics (Gini coefficient, Hausdorff distance) and computational time are recorded alongside downstream-task agreement, as in the article's 16-dataset evaluation.

## Limitations

- scValue's OOB-based DV estimates can be noisy; the article suggests tuning random forest hyperparameters (more trees, adjusted max depth) to stabilize OOB estimates in noisy conditions.
- scValue requires a cell_type_key with cell type labels; sketches are allocated per type via type_weighting, so results depend on label quality.
- Using use_rep='X' may densify sparse matrices internally, and tool versions for baseline methods (CellTypist, CellHint, GeoSketch, KH, etc.) were not specified in the study.
- Benchmark conclusions are task- and dataset-dependent (e.g. CellHint trees on the Spleen dataset); a method that wins here may not win on every downstream task.

## Evidence
