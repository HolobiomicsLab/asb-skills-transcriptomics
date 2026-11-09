---
name: scdisinfact-dataset-construction
description: Use when you have a raw gene-expression count matrix (numpy array, ncells
  x ngenes) plus a meta_cells dataframe containing batch IDs and one or more condition-type
  columns, and need to feed them into scDisInFact training.
when_to_use_negative:
- Each batch of cells is measured under only one condition — the paper states there
  is not enough information to fully disentangle batch and condition effect in this
  case.
- The input is already an integrated/normalized embedding rather than a raw count
  matrix with matching per-cell metadata columns.
- You only have a single-condition single-batch dataset; the disentanglement objective
  (shared vs. unshared bio factors) has nothing to separate.
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scDisInFact
  role: Provides create_scdisinfact_dataset and the scdisinfact conditional VAE model
    class used to build and train on the constructed dataset
  repo: https://github.com/ZhangLabGT/scDisInFact
- name: numpy
  role: Holds the input counts matrix as a numpy.array of shape (ncells, ngenes)
- name: pandas
  role: Holds meta_cells dataframe with batch and condition columns aligned to counts
    rows
- name: PyTorch
  role: Provides the device selection and no_grad inference loop used after training
    the constructed dataset's model
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-024-45227-w
    title: 'scDisInFact: disentangled learning for integration and prediction of multi-batch
      multi-condition single-cell RNA-sequencing data'
  source_content_hash: sha256:f2701f2a201a8ed7aafe3a3037df2e33a7c628fe8e221d920972b9267271929d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ZhangLabGT/scDisInFact
  tools:
  - PyTorch
  - numpy
  - pandas
  - scDisInFact
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6428571428571429
n_claims: 14
---

# scdisinfact-dataset-construction

## Summary

Constructs a scdisinfact dataset object from a raw count matrix and per-cell metadata so scDisInFact can disentangle shared-bio factors, condition-specific unshared-bio factors, and batch effects in multi-batch multi-condition scRNA-seq data.

## When to use

Use when you have a raw gene-expression count matrix (numpy array, ncells x ngenes) plus a meta_cells dataframe containing batch IDs and one or more condition-type columns, and need to feed them into scDisInFact training.

## When NOT to use

- Each batch of cells is measured under only one condition — the paper states there is not enough information to fully disentangle batch and condition effect in this case.
- The input is already an integrated/normalized embedding rather than a raw count matrix with matching per-cell metadata columns.
- You only have a single-condition single-batch dataset; the disentanglement objective (shared vs. unshared bio factors) has nothing to separate.

## Inputs

- counts: numpy.array of shape (ncells, ngenes) with raw gene expression
- meta_cells: pandas.dataframe of shape (ncells, xx) with batch ID column and condition-type columns
- condition_key: list of condition column names, e.g. ["condition 1", "condition 2"]
- batch_key: name of the batch column, e.g. "batch"
- Ks: latent dimension list, e.g. [8, 2, 2]

## Outputs

- data_dict returned by create_scdisinfact_dataset containing per-matrix datasets with counts_norm and batch_id
- trained scdisinfact model with shared-bio (mu_c) and unshared-bio (mu_d) latent factors extractable via model.inference

## How to apply

Load the counts as a numpy.array of shape (ncells, ngenes) and the meta_cells as a pandas dataframe, ensuring meta_cells has one column per condition type (e.g. 'condition 1' and 'condition 2') and a 'batch' column whose cells align row-wise with counts. Call create_scdisinfact_dataset with condition_key set to the list of condition-type column names and batch_key set to the batch column name; with two condition types this yields a data_dict that maps to three latent blocks (Ks=[8,2,2]: one shared-bio factor block plus one unshared-bio factor block per condition type). Instantiate the scdisinfact model on torch.device('cuda:0' if available else 'cpu') and train with model.train_model(nepochs=100). The rationale is that scDisInFact's conditional VAE needs the dataset explicitly partitioned by condition type and batch so its encoders can be guided to extract only condition-relevant variation.

## Related tools

- **scDisInFact** (Provides create_scdisinfact_dataset and the scdisinfact conditional VAE model class used to build and train on the constructed dataset) — https://github.com/ZhangLabGT/scDisInFact
- **numpy** (Holds the input counts matrix as a numpy.array of shape (ncells, ngenes))
- **pandas** (Holds meta_cells dataframe with batch and condition columns aligned to counts rows)
- **PyTorch** (Provides the device selection and no_grad inference loop used after training the constructed dataset's model)

## Examples

```
from scDisInFact import scdisinfact, create_scdisinfact_dataset; data_dict = create_scdisinfact_dataset(counts, meta_cells, condition_key=["condition 1", "condition 2"], batch_key="batch"); model = scdisinfact(data_dict=data_dict, Ks=[8,2,2], device=torch.device("cuda:0" if torch.cuda.is_available() else "cpu")); losses = model.train_model(nepochs=100)
```

## Evaluation signals

- meta_cells row count equals counts row count and meta_cells contains the batch column plus one column per declared condition_key
- data_dict['datasets'] contains one entry per batch-condition combination, each exposing counts_norm and batch_id tensors
- len(Ks) equals 1 + number of condition types (e.g. [8, 2, 2] for two condition types); training losses from model.train_model(nepochs=100) decrease monotonically overall
- After inference under torch.no_grad, mu_c has the shared-bio latent dimension (8 in the demo) and mu_d yields one tensor per condition type with the corresponding unshared dimension (2 each)
- np.concatenate of z_cs across all training matrices yields shape (total_cells, shared_dim) with no NaNs

## Limitations

- When each batch is measured under only one condition, there is not enough information to fully disentangle the batch and condition effects.
- scDisInFact assumes condition types are mutually independent, which does not always hold for real-world datasets; for highly correlated condition types users should create a combined condition type instead.
- The method assumes condition effect is independent of batch effect, batch effects are consistent across conditions, and effects are additive in latent space, limiting modeling of higher-order effects.

## Evidence

- [methods] create_scdisinfact_dataset call: "data_dict = create_scdisinfact_dataset(counts, meta_cells, condition_key = ["condition 1", "condition 2"], batch_key = "batch")"
- [readme] metadata alignment requirement: "`meta_cells` must include columns corresponding to the batch ID and condition of each cells, and the cells in `meta_cells` should match `counts`."
- [readme] latent dimension layout: "we have two condition types, so there are three element corresponding to 
# shared-bio factor, unshared-bio factor for condition 1, unshared-bio factor for condition 2"
- [abstract] disentanglement objective: "scDisInFact learns latent factors that disentangle condition effect from batch effect"
