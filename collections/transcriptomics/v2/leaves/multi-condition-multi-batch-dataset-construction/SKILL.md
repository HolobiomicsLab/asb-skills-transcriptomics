---
name: multi-condition-multi-batch-dataset-construction
description: Use when you have a scRNA-seq count matrix plus cell-level metadata with
  >=1 condition columns and a batch/study ID column, and you need to train scDisInFact
  or run cross-batch perturbation prediction.
when_to_use_negative:
- Each batch is measured under only one condition — the paper notes there is then
  not enough information to fully disentangle batch and condition effects
- Only a single condition type exists and no batch structure needs modeling — task-specific
  perturbation methods like scGen/scPreGAN may suffice
- Condition types are strongly correlated and cannot be assumed mutually independent
  — combine them into one condition type rather than passing them as separate condition_keys
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scDisInFact
  role: Provides create_scdisinfact_dataset which assembles counts, condition keys,
    and batch key into the scdisinfact dataset object consumed by model training and
    predict_counts
  repo: github.com/ZhangLabGT/scDisInFact
- name: numpy
  role: Count matrix input must be a numpy.array of shape (ncells, ngenes)
- name: pandas
  role: Cell metadata input must be a pandas.dataframe with condition and batch columns
    matching counts rows
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-024-45227-w
    title: 'scDisInFact: disentangled learning for integration and prediction of multi-batch
      multi-condition single-cell RNA-sequencing data'
  source_content_hash: sha256:4b852c48182b22c3689ce3b06545f95394b1e3546b0b603e9b929aabe1c7de6b
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - numpy
  - pandas
  - scDisInFact
  - scGen
  - scPreGAN
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 14
---

# multi-condition-multi-batch-dataset-construction

## Summary

Constructs a scdisinfact dataset object from a count matrix and cell metadata for multi-batch, multi-condition scRNA-seq integration, explicitly encoding multiple condition types (e.g., age and disease severity) and a batch key so batch and condition effects can be disentangled downstream.

## When to use

Use when you have a scRNA-seq count matrix plus cell-level metadata with >=1 condition columns and a batch/study ID column, and you need to train scDisInFact or run cross-batch perturbation prediction.

## When NOT to use

- Each batch is measured under only one condition — the paper notes there is then not enough information to fully disentangle batch and condition effects
- Only a single condition type exists and no batch structure needs modeling — task-specific perturbation methods like scGen/scPreGAN may suffice
- Condition types are strongly correlated and cannot be assumed mutually independent — combine them into one condition type rather than passing them as separate condition_keys

## Inputs

- numpy.array count matrix of shape (ncells, ngenes)
- pandas.DataFrame meta_cells of shape (ncells, xx) with one column per condition type and one batch ID column
- condition_key: list of metadata column names (one per condition type)
- batch_key: metadata column name holding batch/study ID

## Outputs

- data_dict: a scdisinfact dataset object containing per-batch count matrices grouped by condition combination, ready for scdisinfact model training

## How to apply

Format the count matrix as a numpy.array of shape (ncells, ngenes) and the metadata as a pandas.DataFrame of shape (ncells, xx) whose rows match the counts order and include columns for each condition type and the batch ID. Call create_scdisinfact_dataset(counts, meta_cells, condition_key=[...], batch_key=...) passing one column name per condition type (e.g., ["condition 1", "condition 2"] or age and disease severity in the COVID-19 case) and the single batch column. The number of condition types determines the latent dimensions later: Ks = [shared-bio dim, one unshared-bio dim per condition type], e.g., [8, 2, 2] for two condition types. Because meta_cells must align row-wise with counts, verify cell order before construction; for highly correlated condition types, combine them into a single combined condition column (e.g., stim1_stim2) and treat that as one condition type instead.

## Related tools

- **scDisInFact** (Provides create_scdisinfact_dataset which assembles counts, condition keys, and batch key into the scdisinfact dataset object consumed by model training and predict_counts) — github.com/ZhangLabGT/scDisInFact
- **numpy** (Count matrix input must be a numpy.array of shape (ncells, ngenes))
- **pandas** (Cell metadata input must be a pandas.dataframe with condition and batch columns matching counts rows)

## Examples

```
from scDisInFact import scdisinfact, create_scdisinfact_dataset; data_dict = create_scdisinfact_dataset(counts, meta_cells, condition_key=["condition 1", "condition 2"], batch_key="batch")
```

## Evaluation signals

- data_dict is returned without error and contains one dataset per (condition-combination, batch) group defined by the condition and batch columns
- Row counts of counts and meta_cells match (ncells identical), satisfying the README requirement that 'the cells in meta_cells should match counts'
- Number of condition keys supplied equals the number of unshared-bio latent dimensions plus one shared-bio dimension in the downstream Ks argument (e.g., 2 condition types -> Ks = [8, 2, 2])
- Each batch in the constructed dataset appears under multiple conditions; any batch measured under a single condition flags the disentanglement limitation

## Limitations

- When each batch is measured under only one condition, there is not enough information to fully disentangle batch and condition effect
- scDisInFact assumes condition types are mutually independent, which does not always hold; highly correlated condition types should be combined into a single condition type
- The framework assumes condition effect is independent of batch effect and batch effects are consistent across conditions
- Condition and batch effects are assumed additive in latent space, limiting modeling of higher-order effects

## Evidence

- [methods] data_dict = create_scdisinfact_dataset(counts, meta_cells, condition_key = ["condition 1", "condition 2"], batch_key = "batch"): "data_dict = create_scdisinfact_dataset(counts, meta_cells, condition_key = ["condition 1", "condition 2"], batch_key = "batch")"
- [readme] meta_cells must include columns corresponding to the batch ID and condition of each cells, and the cells in meta_cells should match counts: "**Note**: `meta_cells` must include columns corresponding to the batch ID and condition of each cells, and the cells in `meta_cells` should match `counts`."
- [discussion] the users can create a combined condition type (stim1_stim2, ctrl1_stim2, stim1_ctrl2, ctrl1_ctrl2) and train scDisInFact on the combined condition type instead: "the users can create a combined condition type (stim1_stim2, ctrl1_stim2, stim1_ctrl2, ctrl1_ctrl2) and train scDisInFact on the combined condition type instead."
- [discussion] when each batch of cells is measured under only one condition, there is not enough information to fully disentangle the batch and condition effect: "when each batch of cells is measured under only one condition, there is not enough information to fully disentangle the batch and condition effect."
