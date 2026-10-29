---
name: method-comparison-visualization
description: Use when multiple cross-modality translation methods have been run on
  identical data splits and you must aggregate their per-fold metrics (e.g., calculate_cluster_index
  outputs) into a comparative table and figure to test whether a proposed method outperforms
  baselines.
when_to_use_negative:
- Only one method was run and there is no baseline comparison to visualize — a single-method
  report suffices
- Evaluation metric is not comparable across methods because methods were run on different
  data splits or different preprocessing (comparisons must use identical cell-type-disjoint
  splits)
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0091
tools:
- name: scButterfly
  role: Source of calculate_cluster.calculate_cluster_index for computing clustering
    indices and draw_cluster.draw_tsne for visualization; provides the scButterfly-B/T/C
    variants being compared
  repo: https://github.com/BioX-NKU/scButterfly
- name: BABEL
  role: Baseline cross-modality translation method included in the comparison plot
- name: Polarbear
  role: Baseline cross-modality translation method included in the comparison plot
- name: JAMIE
  role: Baseline cross-modality translation method included in the comparison plot
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-024-47418-x
    title: 'scButterfly: a versatile single-cell cross-modality translation method
      via dual-aligned variational autoencoders'
  source_content_hash: sha256:5b5e934d665828538fc95026984af8f7a1b7c6aa451127825d3d579231883ce7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BioX-NKU/scButterfly
  tools:
  - BABEL
  - JAMIE
  - Polarbear
  - Python
  - scButterfly
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# method-comparison-visualization

## Summary

Aggregates a quantitative performance metric (e.g., clustering index) across methods, datasets, and cross-validation folds, then renders a head-to-head comparison plot and table in the style of scButterfly's Fig. 3a, where scButterfly-B/T/C are benchmarked against BABEL, Polarbear and JAMIE on unseen cell types.

## When to use

Use when multiple cross-modality translation methods have been run on identical data splits and you must aggregate their per-fold metrics (e.g., calculate_cluster_index outputs) into a comparative table and figure to test whether a proposed method outperforms baselines.

## When NOT to use

- Only one method was run and there is no baseline comparison to visualize — a single-method report suffices
- Evaluation metric is not comparable across methods because methods were run on different data splits or different preprocessing (comparisons must use identical cell-type-disjoint splits)

## Inputs

- Per-method, per-dataset, per-fold clustering index values from calculate_cluster.calculate_cluster_index computed on translated profiles vs. ground-truth cell-type labels
- Cell-type-disjoint three-fold cross-validation splits of the paired scRNA-seq/scATAC-seq datasets (MB, MCC, MK, PBMC)
- Ground-truth cell-type labels for held-out (unseen) cell-type cells

## Outputs

- clustering_metrics_comparison.csv — aggregated clustering metrics per method, dataset and fold
- fig3a_clustering_comparison.png — Fig. 3a-style grouped comparison of clustering metrics for scButterfly-B/T/C vs. BABEL, Polarbear, JAMIE
- Optional t-SNE plots of translated profiles via draw_cluster.draw_tsne

## How to apply

After running every method (scButterfly-B, scButterfly-T with cell_type_augmentation, scButterfly-C with MultiVI_augmentation, plus baselines BABEL, Polarbear and JAMIE) on the same cell-type-disjoint folds of the MB, MCC, MK and PBMC paired scRNA-seq/scATAC-seq datasets, compute the clustering index of each translated profile against ground-truth labels using calculate_cluster.calculate_cluster_index. Aggregate the metrics across the three folds per dataset-method pair, and write the aggregated table to clustering_metrics_comparison.csv. Draw a Fig. 3a-style comparison plot (clustering metrics per method, grouped by dataset and translation direction) saved as fig3a_clustering_comparison.png. Validate by confirming that all three scButterfly variants attain higher clustering metrics than BABEL, Polarbear and JAMIE in both RNA-to-ATAC and ATAC-to-RNA directions on unseen cell types, matching the reported ordering. Optionally complement the quantitative plot with draw_cluster.draw_tsne visualizations of translated profiles to inspect preservation of cellular heterogeneity.

## Related tools

- **scButterfly** (Source of calculate_cluster.calculate_cluster_index for computing clustering indices and draw_cluster.draw_tsne for visualization; provides the scButterfly-B/T/C variants being compared) — https://github.com/BioX-NKU/scButterfly
- **BABEL** (Baseline cross-modality translation method included in the comparison plot)
- **Polarbear** (Baseline cross-modality translation method included in the comparison plot)
- **JAMIE** (Baseline cross-modality translation method included in the comparison plot)

## Examples

```
from scButterfly.calculate_cluster import calculate_cluster_index; metrics = calculate_cluster_index(predict_data, true_data, cell_type, method='Leiden')
```

## Evaluation signals

- clustering_metrics_comparison.csv contains one row per method-dataset-fold with clustering index values from calculate_cluster_index
- Aggregated comparison shows all three scButterfly variants (B, T, C) with higher clustering metrics than BABEL, Polarbear and JAMIE on unseen cell types in both translation directions
- Figure renders metrics grouped by dataset (MB, MCC, MK, PBMC) and direction (RNA→ATAC and ATAC→RNA), mirroring Fig. 3a
- All methods were evaluated on identical cell-type-disjoint folds (train/test sets share no cell type within a fold)

## Limitations

- Performance for cross-cell-type and cross-organ translation is flagged as needing improvement via advanced machine learning techniques with adaptive and interpretable model parameters
- Cell-type-label augmentation (scButterfly-T) requires 'cell_type' in adata.obs; if absent, the package automatically falls back to MultiVI_augmentation
- Clustering metrics depend on the cell-type-disjoint split design; results are specific to generalization on unseen cell types and may not reflect in-distribution performance

## Evidence

- [methods] Clustering index computation via calculate_cluster_index: "calculate_cluster.calculate_cluster_index"
- [abstract] scButterfly superiority over baselines in preserving heterogeneity: "we provide compelling evidence of scButterfly’s superiority over baseline methods in preserving cellular heterogeneity while translating datasets of various contexts"
- [readme] Predicting and evaluation API returns translated profiles: [evidence span withheld — anchor status: external_doc]
- [intro] Challenging scenario of translating data with novel cell types motivates the comparison: "there is a prevalent challenging scenario where the single-modality data to be translated has inter-sample variations (e.g., batch effects) and even different biological contexts"
- [readme] Augmentation variants distinguish scButterfly-B, -T and -C in the comparison: [evidence span withheld — anchor status: not_found]
