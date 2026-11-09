---
name: five-fold-cross-validation-cell-subsampling
description: Use when you must quantify how a cross-modality translation model's performance
  degrades with limited paired training cells, or when testing whether data augmentation
  compensates for reduced sample size.
when_to_use_negative:
- When abundant paired training cells are available and the goal is not to study limited-sample
  behavior — a single train/test split suffices.
- When data are unpaired and no cross-modal correspondence exists for holdout evaluation
  — the subsampling design requires paired profiles to score held-out cells.
- When the target task is a downstream biological question rather than a benchmark
  of translation robustness to subsampling.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0102
tools:
- name: scButterfly
  role: Provides load_data, data_preprocessing, augmentation (cell_type_augmentation
    / MultiVI_augmentation), construct_model, train_model, and test_model used per
    fold and fraction for scButterfly-B, -T, and -C variants
  repo: https://github.com/BioX-NKU/scButterfly
- name: MultiVI
  role: scvi-tools joint multi-omics integration supplying cluster labels for scButterfly-C
    data augmentation
- name: Python 3.9
  role: Runtime environment (conda environment) for scButterfly pipeline
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-024-47418-x
    title: 'scButterfly: a versatile single-cell cross-modality translation method
      via dual-aligned variational autoencoders'
  source_content_hash: sha256:768bc471efd7e354e1bc7d07f952749a7bc32337fe1155eb20a57492cd355541
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BioX-NKU/scButterfly
  tools:
  - MultiVI
  - Python
  - Python 3.9
  - scButterfly
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# Five-fold cross-validation cell subsampling

## Summary

A benchmarking design that repeatedly holds out paired cells across five cross-validation folds while varying the training fraction (20%–100%) to test whether a method (here, scButterfly variants) retains translation accuracy as paired training cells become limited.

## When to use

Use when you must quantify how a cross-modality translation model's performance degrades with limited paired training cells, or when testing whether data augmentation compensates for reduced sample size.

## When NOT to use

- When abundant paired training cells are available and the goal is not to study limited-sample behavior — a single train/test split suffices.
- When data are unpaired and no cross-modal correspondence exists for holdout evaluation — the subsampling design requires paired profiles to score held-out cells.
- When the target task is a downstream biological question rather than a benchmark of translation robustness to subsampling.

## Inputs

- Paired scRNA-seq and scATAC-seq AnnData count matrices (raw counts, e.g., MCC dataset)
- Per-fold train_id / test_id cell-ID lists (training fractions 20%–100%, 5 folds)
- Cell-type labels (adata.obs 'cell_type') for scButterfly-T augmentation
- Leiden cluster labels from MultiVI integration for scButterfly-C augmentation
- chrom_list: sorted per-chromosome peak counts

## Outputs

- Translated held-out target-modality profiles from Butterfly.test_model (A2R_predict, R2A_predict)
- Per-fold, per-fraction clustering metrics from calculate_cluster.calculate_cluster_index
- t-SNE visualizations of translated profiles (draw_cluster.draw_tsne)
- Aggregated mean clustering metrics across folds and fractions (figure_2d_clustering_metrics.csv)

## How to apply

Load the paired scRNA–scATAC MCC dataset with Butterfly.load_data and normalize each modality with Butterfly.data_preprocessing. For each of five cross-validation folds, randomly select a training fraction from 20% to 100% of the paired cells and hold out the remaining paired cells for testing via train_id/test_id lists passed to load_data. For the augmented variants, call Butterfly.augmentation with 'cell_type_augmentation' (scButterfly-T) or 'MultiVI_augmentation' (scButterfly-C, Leiden + MultiVI labels), expanding training data to three times the subsampled size, while scButterfly-B uses aug_type=None. Build and train each model with Butterfly.construct_model (supplying a chrom_list of per-chromosome peak counts) and Butterfly.train_model, then predict held-out cells with Butterfly.test_model. Rationale: comparing augmented (T, C) against basic (B) across fractions reveals whether augmentation mitigates restricted sample size, with the three-fold augmentation ratio identified as optimal.

## Related tools

- **scButterfly** (Provides load_data, data_preprocessing, augmentation (cell_type_augmentation / MultiVI_augmentation), construct_model, train_model, and test_model used per fold and fraction for scButterfly-B, -T, and -C variants) — https://github.com/BioX-NKU/scButterfly
- **MultiVI** (scvi-tools joint multi-omics integration supplying cluster labels for scButterfly-C data augmentation)
- **Python 3.9** (Runtime environment (conda environment) for scButterfly pipeline)

## Examples

```
from scButterfly.butterfly import Butterfly; butterfly = Butterfly(); butterfly.load_data(RNA_data, ATAC_data, train_id, test_id); butterfly.data_preprocessing(); butterfly.augmentation('MultiVI_augmentation'); butterfly.construct_model(chrom_list); butterfly.train_model(); A2R_predict, R2A_predict = butterfly.test_model()
```

## Evaluation signals

- scButterfly-T and scButterfly-C clustering metrics exceed scButterfly-B at limited training-cell fractions, reproducing the Fig. 2d ordering
- The three-fold augmentation ratio (synthetic + original data = 3× subsampled size) is confirmed as optimal
- Aggregated mean clustering metrics are stable across the five cross-validation folds at each sampling fraction
- t-SNE visualizations of translated profiles preserve cluster structure consistent with true cell-type labels

## Limitations

- Augmentation increases training time; README notes cell_type_augmentation or MultiVI_augmentation 'will cause more training time used'
- If 'cell_type' is absent from adata.obs, scButterfly-T silently falls back to MultiVI_augmentation, changing the augmentation semantics
- Peaks must be sorted by chromosome and chrom_list supplied correctly, or construct_model will mis-handle the ATAC feature space

## Evidence

- [results] Both scButterfly-C and scButterfly-T improved over scButterfly-B, especially when the number of training cells was limited, demonstrating that data augmentation mitigates restricted sample size: [evidence span withheld — anchor status: not_found]
- [intro] An effective data augmentation scheme could improve translation performance, alleviate limited multi-omics cell numbers: "an effective data augmentation scheme may further improve the translation performance, alleviate the issues of limited number of cells measured by multi-omics protocols"
- [readme] train/validation/test split via load_data API: [evidence span withheld — anchor status: external_doc]
- [readme] augmentation API with aug_type options: [evidence span withheld — anchor status: external_doc]
