---
name: cell-type-disjoint-cross-validation-splitting
description: Use when benchmarking cross-modality translation methods (e.g., scRNA-seq
  to scATAC-seq) for generalization to cell types absent from training, mimicking
  the MB/MCC/MK/PBMC unseen-cell-type evaluation.
when_to_use_negative:
- When the goal is in-distribution translation accuracy on cell types seen during
  training — a random cell-level split, not a cell-type-disjoint split, is appropriate
- When the paired dataset has too few cell types to form disjoint folds, making the
  split ill-defined or folds unrepresentative
- When evaluating unpaired/diagonal translation or perturbation-response prediction,
  which use different training and evaluation protocols
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scButterfly
  role: Cross-modality translator under test; provides load_data, data_preprocessing,
    augmentation, construct_model, train_model and test_model APIs for the cell-type-disjoint
    folds
  repo: https://github.com/BioX-NKU/scButterfly
- name: BABEL
  role: Baseline autoencoder-based cross-modality translation method run on identical
    cell-type-disjoint splits
- name: Polarbear
  role: Baseline VAE-stitching cross-modality method run on identical cell-type-disjoint
    splits
- name: JAMIE
  role: Baseline joint-VAE multi-modal method run on identical cell-type-disjoint
    splits
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-024-47418-x
    title: 'scButterfly: a versatile single-cell cross-modality translation method
      via dual-aligned variational autoencoders'
  source_content_hash: sha256:2dcbd8d1940260a486274a4e88f07a78e283df28f2d75f70c26fa081b5ab8217
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
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# cell-type-disjoint-cross-validation-splitting

## Summary

A cross-validation protocol for evaluating single-cell cross-modality translation to novel cell types: partition cell types into folds so training and test sets share no cell type, then measure whether translated profiles preserve cellular heterogeneity on unseen cell types.

## When to use

Use when benchmarking cross-modality translation methods (e.g., scRNA-seq to scATAC-seq) for generalization to cell types absent from training, mimicking the MB/MCC/MK/PBMC unseen-cell-type evaluation.

## When NOT to use

- When the goal is in-distribution translation accuracy on cell types seen during training — a random cell-level split, not a cell-type-disjoint split, is appropriate
- When the paired dataset has too few cell types to form disjoint folds, making the split ill-defined or folds unrepresentative
- When evaluating unpaired/diagonal translation or perturbation-response prediction, which use different training and evaluation protocols

## Inputs

- Paired single-batch scRNA-seq/scATAC-seq AnnData objects with raw count matrices and cell-type labels (e.g., MB, MCC, MK, PBMC datasets)
- Cell-type annotation per cell (adata.obs['cell_type']) used to define the disjoint folds
- train_id / test_id / validation_id cell ID lists per fold

## Outputs

- Per-fold, per-method translated profiles for held-out unseen cell types in both directions (A2R_predict, R2A_predict)
- Clustering index metrics comparing translated profiles to ground-truth labels, aggregated across folds (e.g., clustering_metrics_comparison.csv)
- Comparison figure of clustering metrics for scButterfly-B/T/C vs. BABEL, Polarbear, JAMIE (e.g., fig3a_clustering_comparison.png)

## How to apply

Partition the paired dataset's cell types into folds (e.g., three folds for MB, MCC, MK and PBMC) such that each fold's training and test sets share no cell type. For each dataset-fold pair, train scButterfly-B (no augmentation), scButterfly-T (cell_type_augmentation) and scButterfly-C (MultiVI_augmentation) via Butterfly.construct_model and Butterfly.train_model, then translate held-out cells with Butterfly.test_model. Run baselines BABEL, Polarbear and JAMIE on the identical cell-type-disjoint splits so all methods face the same unseen cell types. Compute the clustering index of translated profiles against ground-truth labels with calculate_cluster.calculate_cluster_index, aggregate across folds, and compare all three scButterfly variants against the three baselines per direction of translation (A2R and R2A).

## Related tools

- **scButterfly** (Cross-modality translator under test; provides load_data, data_preprocessing, augmentation, construct_model, train_model and test_model APIs for the cell-type-disjoint folds) — https://github.com/BioX-NKU/scButterfly
- **BABEL** (Baseline autoencoder-based cross-modality translation method run on identical cell-type-disjoint splits)
- **Polarbear** (Baseline VAE-stitching cross-modality method run on identical cell-type-disjoint splits)
- **JAMIE** (Baseline joint-VAE multi-modal method run on identical cell-type-disjoint splits)

## Examples

```
from scButterfly.butterfly import Butterfly; butterfly = Butterfly(); butterfly.load_data(RNA_data, ATAC_data, train_id, test_id, validation_id); butterfly.data_preprocessing(); butterfly.augmentation('cell_type_augmentation'); butterfly.construct_model(chrom_list); butterfly.train_model(); A2R_predict, R2A_predict = butterfly.test_model()
```

## Evaluation signals

- All three scButterfly variants (B, T, C) attain higher clustering metrics than BABEL, Polarbear and JAMIE on unseen cell types in both translation directions, matching Fig. 3a
- Each fold's training and test cell-type sets are verified disjoint (no shared cell type between train_id and test_id)
- Clustering index is computed with calculate_cluster.calculate_cluster_index against ground-truth labels for every method, dataset and fold
- Metrics aggregate consistently across the three folds before method-level comparison
- Identical cell-type-disjoint splits are used for scButterfly and all three baselines

## Limitations

- Performance for cross-cell-type (and cross-organ) translation is flagged by the authors as needing improvement via advanced machine learning techniques with adaptive and interpretable model parameters
- Using scButterfly-T requires cell_type labels in adata.obs; if absent, it automatically falls back to MultiVI augmentation (scButterfly-C), which costs additional training time

## Evidence

- [results] Novel cell types benchmark protocol: [evidence span withheld — anchor status: not_found]
- [intro] Split rationale: "there is a prevalent challenging scenario where the single-modality data to be translated has inter-sample variations (e.g., batch effects) and even different biological contexts (e.g., novel cell"
- [methods] Preprocessing APIs: [evidence span withheld — anchor status: not_found]
- [readme] Augmentation modes from README: [evidence span withheld — anchor status: not_found]
