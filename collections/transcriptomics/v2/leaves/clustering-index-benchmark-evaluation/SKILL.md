---
name: clustering-index-benchmark-evaluation
description: Use when you have cross-modality translated profiles (e.g., A2R/R2A predictions
  from scButterfly test_model or baselines) on cell-type-disjoint test folds and need
  to quantify heterogeneity preservation via clustering indices.
when_to_use_negative:
- Translated profiles lack ground-truth cell-type labels, so no clustering index against
  true labels can be computed
- The test cells' cell types were already seen in training — this skill's evaluation
  targets generalization to unseen cell types, not standard random splits
- Input is raw count scRNA-seq/scATAC-seq data still needing preprocessing (load_data,
  RNA/ATAC_data_preprocessing) rather than translated profiles to score
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3473
- http://edamontology.org/topic_0203
tools:
- name: scButterfly
  role: Source of translated profiles (Butterfly.test_model returning A2R_predict/R2A_predict)
    and of the calculate_cluster.calculate_cluster_index evaluation utility
  repo: https://github.com/BioX-NKU/scButterfly
- name: BABEL
  role: Baseline cross-modality translation method scored on identical cell-type-disjoint
    splits
- name: Polarbear
  role: Baseline cross-modality translation method scored on identical cell-type-disjoint
    splits
- name: JAMIE
  role: Baseline cross-modality translation method scored on identical cell-type-disjoint
    splits
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-024-47418-x
    title: 'scButterfly: a versatile single-cell cross-modality translation method
      via dual-aligned variational autoencoders'
  source_content_hash: sha256:4078ae2c3afd5936ad33de1fa2c1ecf37d5f6bfb63e355479bf2cb09a41fc638
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
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3473
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# clustering-index-benchmark-evaluation

## Summary

Scores cross-modality translated single-cell profiles (scRNA↔scATAC) by computing clustering indices against ground-truth cell-type labels, to test whether translation preserves cellular heterogeneity — particularly for cell types unseen during training — in benchmark comparisons like scButterfly vs. BABEL, Polarbear and JAMIE (Fig. 3a).

## When to use

Use when you have cross-modality translated profiles (e.g., A2R/R2A predictions from scButterfly test_model or baselines) on cell-type-disjoint test folds and need to quantify heterogeneity preservation via clustering indices.

## When NOT to use

- Translated profiles lack ground-truth cell-type labels, so no clustering index against true labels can be computed
- The test cells' cell types were already seen in training — this skill's evaluation targets generalization to unseen cell types, not standard random splits
- Input is raw count scRNA-seq/scATAC-seq data still needing preprocessing (load_data, RNA/ATAC_data_preprocessing) rather than translated profiles to score

## Inputs

- Translated single-modality profiles per method and direction (A2R_predict, R2A_predict matrices from Butterfly.test_model)
- Ground-truth cell-type labels for held-out cells
- Cell-type-disjoint cross-validation fold definitions (3 folds over cell types) shared by all methods
- Baseline translation outputs from BABEL, Polarbear and JAMIE on the identical splits
- Datasets: MB, MCC, MK, PBMC paired scRNA-seq/scATAC-seq (single-batch)

## Outputs

- Per-method/dataset/fold clustering index values against ground-truth labels
- Aggregated cross-fold comparison table clustering_metrics_comparison.csv
- Fig. 3a-style comparison plot fig3a_clustering_comparison.png

## How to apply

Partition cell types into three cross-validation folds so training and test sets share no cell type (cell-type-disjoint splitting), run each translator (scButterfly-B/-T/-C, BABEL, Polarbear, JAMIE) on the identical splits, and translate held-out cells in both directions using Butterfly.test_model, which returns A2R_predict and R2A_predict. Compute the clustering index of every translated profile against the ground-truth cell-type labels with calculate_cluster.calculate_cluster_index for each method, dataset (MB, MCC, MK, PBMC) and fold. Aggregate metrics across the three folds, then build the Fig. 3a-style comparison (clustering metrics per method, three scButterfly variants vs. three baselines), saving clustering_metrics_comparison.csv and fig3a_clustering_comparison.png. Validate that all scButterfly variants attain higher clustering metrics than the baselines on unseen cell types, matching the reported ordering; note that scButterfly-T uses cell-type labels (cell_type in adata.obs, falling back to MultiVI augmentation) while scButterfly-C uses MultiVI-integrated cluster labels.

## Related tools

- **scButterfly** (Source of translated profiles (Butterfly.test_model returning A2R_predict/R2A_predict) and of the calculate_cluster.calculate_cluster_index evaluation utility) — https://github.com/BioX-NKU/scButterfly
- **BABEL** (Baseline cross-modality translation method scored on identical cell-type-disjoint splits)
- **Polarbear** (Baseline cross-modality translation method scored on identical cell-type-disjoint splits)
- **JAMIE** (Baseline cross-modality translation method scored on identical cell-type-disjoint splits)

## Examples

```
A2R_predict, R2A_predict = butterfly.test_model(); from scButterfly.calculate_cluster import calculate_cluster_index; ci = calculate_cluster_index(label_true, label_pred)
```

## Evaluation signals

- Clustering index is computed with calculate_cluster.calculate_cluster_index for every method, dataset (MB, MCC, MK, PBMC) and fold, in both RNA→ATAC and ATAC→RNA directions
- Metrics are aggregated across the three cell-type-disjoint folds, and the expected ordering is: all three scButterfly variants > BABEL, Polarbear and JAMIE on unseen cell types (Fig. 3a)
- Comparison artifacts exist: clustering_metrics_comparison.csv (table) and fig3a_clustering_comparison.png (per-method clustering metrics plot)
- Sanity check that baseline methods ran on the identical cell-type-disjoint splits as scButterfly variants
- scButterfly-T should have fallen back from cell_type_augmentation to MultiVI_augmentation when 'cell_type' is absent from adata.obs, matching README behavior

## Limitations

- Performance for cross-cell-type (and cross-organ) translation is flagged as needing improvement via advanced machine learning techniques with adaptive and interpretable model parameters
- scButterfly-T depends on 'cell_type' labels in adata.obs; if absent it automatically transfers to MultiVI_augmentation, which changes the variant being benchmarked (T vs. C)
- Evaluation on unseen cell types relies on cell-type-disjoint three-fold cross-validation; results on four single-batch datasets (MB, MCC, MK, PBMC) may not transfer to multi-batch or other tissue contexts

## Evidence

- [results] In three-fold cross-validation split by cell type (unseen cell types) on the four single-batch datasets (MB, MCC, MK, PBMC), all three scButterfly variants outperformed BABEL, Polarbear and JAMIE: [evidence span withheld — anchor status: not_found]
- [abstract] we provide compelling evidence of scButterfly’s superiority over baseline methods in preserving cellular heterogeneity while translating datasets of various contexts: "we provide compelling evidence of scButterfly’s superiority over baseline methods in preserving cellular heterogeneity while translating datasets of various contexts"
- [readme] A series of evaluating method also be integrated in this function, you could get these evaluation using parameters:: [evidence span withheld — anchor status: external_doc]
- [intro] there is a prevalent challenging scenario where the single-modality data to be translated has inter-sample variations (e.g., batch effects) and even different biological contexts: "there is a prevalent challenging scenario where the single-modality data to be translated has inter-sample variations (e.g., batch effects) and even different biological contexts"
