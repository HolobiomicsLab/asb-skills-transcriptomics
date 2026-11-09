---
name: label-transfer-agreement-visualization
description: Use when comparing label transfer from a large reference (e.g. the 428k-cell
  human Gut scRNA-seq dataset) to a query (e.g. immune T-cell populations) across
  multiple subsampling methods, and you need a per-cell-type agreement dot plot against
  the full-reference baseline.
when_to_use_negative:
- When only a single reference is available and no full-reference transfer baseline
  can be computed to compare sketches against
- When the query dataset's cell types are absent from the reference label vocabulary
  — transferred annotations cannot be meaningfully compared per population
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: CellTypist
  role: Trains annotation models on each sketch and the full reference, and transfers
    labels to the immune query dataset
- name: scValue
  role: Produces value-based sketches of the reference to be compared via label-transfer
    agreement
  repo: https://github.com/LHBCB/scvalue
- name: Scanpy
  role: Loads the .h5ad reference, selects highly variable genes, computes PCA in
    adata.obsm['X_pca'], and performs Uniform subsampling
- name: scikit-learn
  role: Supports agreement computation between transferred and full-reference annotations
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bib/bbaf279
    title: 'scValue: value-based subsampling of large-scale single-cell transcriptomic
      data for machine and deep learning tasks'
  source_content_hash: sha256:747f61dbe84bf03a2776371ff860a7cdd8aeb50769eb7e629afa109601ef77d8
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
  - CellTypist
  - GeoSketch
  - Hopper
  - Scanpy
  - Seurat
  - Sphetcher
  - scSampler
  - scValue
  - scikit-learn
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# label-transfer-agreement-visualization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Trains CellTypist models on subsampled ('sketched') and full scRNA-seq references, transfers labels to a query dataset, and renders per-population agreement with the full-reference transfer as a dot plot — used to judge which subsampling method (e.g. scValue) most closely reproduces full-data annotation behavior.

## When to use

Use when comparing label transfer from a large reference (e.g. the 428k-cell human Gut scRNA-seq dataset) to a query (e.g. immune T-cell populations) across multiple subsampling methods, and you need a per-cell-type agreement dot plot against the full-reference baseline.

## When NOT to use

- When only a single reference is available and no full-reference transfer baseline can be computed to compare sketches against
- When the query dataset's cell types are absent from the reference label vocabulary — transferred annotations cannot be meaningfully compared per population

## Inputs

- Full reference scRNA-seq dataset in .h5ad format (e.g. 428k-cell human gut reference, log-normalised, with cell_type labels in adata.obs['cell_type'])
- Query dataset (immune dataset with nine T-cell populations)
- Sketches produced per subsampling method (SCValue, Uniform, GeoSketch, Sphetcher, Hopper, scSampler)

## Outputs

- CellTypist models trained on each sketch and on the full reference
- Transferred labels for the immune query T-cell populations per model
- Per-population agreement scores vs. the full-reference transfer for each of the nine T-cell populations
- Dot plot figure4_label_transfer_agreement.png reproducing Figure 4

## How to apply

Load the full reference (e.g. the 428k-cell human gut dataset) with Scanpy, select highly variable genes, and compute PCA stored in adata.obsm['X_pca']. Generate a sketch per method — SCValue with sketch_size as an integer cell count, use_rep='X_pca', cell_type_key='cell_type', type_weighting='inv_cv', strategy='FB', n_trees=100, seed=42 — plus 10% sketches from baselines (Uniform via Scanpy/Seurat, GeoSketch, Sphetcher, Hopper, scSampler). Train a CellTypist model on each sketch and on the full reference, then transfer labels from every model to the immune query dataset. Compute per-population agreement with the full-reference transfer for each of the nine T-cell populations and render the per-method agreement patterns as a dot plot reproducing Figure 4, saved as figure4_label_transfer_agreement.png. The decision point is which sketch yields agreement patterns closest to the full-reference transfer.

## Related tools

- **CellTypist** (Trains annotation models on each sketch and the full reference, and transfers labels to the immune query dataset)
- **scValue** (Produces value-based sketches of the reference to be compared via label-transfer agreement) — https://github.com/LHBCB/scvalue
- **Scanpy** (Loads the .h5ad reference, selects highly variable genes, computes PCA in adata.obsm['X_pca'], and performs Uniform subsampling)
- **scikit-learn** (Supports agreement computation between transferred and full-reference annotations)

## Examples

```
from scvalue import SCValue; scv = SCValue(adata=adata, sketch_size=0.1, use_rep='X_pca', cell_type_key='cell_type', type_weighting='inv_cv', strategy='FB', n_trees=100, seed=42); adata_sub = scv.value()
```

## Evaluation signals

- The dot plot must show scValue (together with Uniform) yielding agreement closest to the full-reference transfer across all nine T-cell populations
- Agreement is computed per T-cell population against the full 428k-cell-reference CellTypist transfer, not against original annotations
- The output file figure4_label_transfer_agreement.png must reproduce the paper's Figure 4 per-method agreement patterns
- Every sketch and the full reference must yield a trained CellTypist model that successfully transfers labels to all nine query populations

## Limitations

- Agreement is measured relative to the full-reference transfer, so the quality of the full-reference model bounds the interpretation of all comparisons
- Uniform subsampling can match scValue on this task; agreement-based evaluation alone may not distinguish them
- Per-population agreement depends on the nine T-cell populations present in the immune query; populations absent from the reference cannot be assessed

## Evidence

- [other] Transfer labels from each model to the immune query dataset and compute per-population agreement with the full-reference transfer for each of the nine T-cell populations.: [evidence span withheld — anchor status: not_found]
- [other] Dot plot validation: [evidence span withheld — anchor status: not_found]
