---
name: single-cell-disentangled-latent-model-training
description: Use when a multi-batch scRNA-seq dataset contains cells measured under
  multiple biological conditions and you need to jointly remove batch effect while
  preserving condition effect in latent factors.
when_to_use_negative:
- Each batch of cells is measured under only one condition — the paper notes there
  is not enough information to fully disentangle batch and condition effects in that
  setting.
- The task is single-condition scRNA-seq integration with only batch effects and no
  condition design (e.g. plain batch correction with scVI/BBKNN).
- Condition types are strongly correlated, requiring a combined condition type (e.g.
  stim1_stim2, ctrl1_stim2, stim1_ctrl2, ctrl1_ctrl2) rather than independent condition
  encodings.
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scDisInFact
  role: Conditional VAE framework whose scdisinfact model, create_scdisinfact_dataset,
    train_model, and inference functions implement the disentangled latent training
    loop
  repo: https://github.com/ZhangLabGT/scDisInFact
- name: PyTorch
  role: Deep learning backend used to instantiate the encoders/decoders on CUDA/CPU
    and run no-grad inference to extract mu_c and mu_d
- name: numpy
  role: Holds the (ncells, ngenes) count matrices and concatenates extracted z_c /
    z_d factor arrays across training matrices
- name: pandas
  role: Holds meta_cells with batch and condition columns used to build the scdisinfact
    dataset
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-024-45227-w
    title: 'scDisInFact: disentangled learning for integration and prediction of multi-batch
      multi-condition single-cell RNA-sequencing data'
  source_content_hash: sha256:9ca2e1f080391ba17d1fc3b0340801a2f1ac50dbc2c331c24c396b9d9dba3658
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ZhangLabGT/scDisInFact
  tools:
  - PyTorch
  - numpy
  - pandas
  - scDisInFact
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# single-cell-disentangled-latent-model-training

## Summary

Train scDisInFact, a conditional variational autoencoder that disentangles multi-batch multi-condition scRNA-seq data into shared-bio factors, condition-specific unshared-bio factors, and batch factors, enabling batch effect removal, key gene detection, and perturbation prediction.

## When to use

Use when a multi-batch scRNA-seq dataset contains cells measured under multiple biological conditions and you need to jointly remove batch effect while preserving condition effect in latent factors.

## When NOT to use

- Each batch of cells is measured under only one condition — the paper notes there is not enough information to fully disentangle batch and condition effects in that setting.
- The task is single-condition scRNA-seq integration with only batch effects and no condition design (e.g. plain batch correction with scVI/BBKNN).
- Condition types are strongly correlated, requiring a combined condition type (e.g. stim1_stim2, ctrl1_stim2, stim1_ctrl2, ctrl1_ctrl2) rather than independent condition encodings.

## Inputs

- counts: numpy.array of shape (ncells, ngenes)
- meta_cells: pandas.dataframe of shape (ncells, xx) with batch ID and condition columns (e.g. 'condition 1', 'condition 2', 'batch')
- Ks: list of latent dimensions, e.g. [8, 2, 2]
- device: torch device (cuda:0 or cpu)

## Outputs

- Trained scdisinfact model checkpoint (conditional VAE)
- losses returned by model.train_model(nepochs = 100)
- z_c: concatenated shared-bio factor matrix (batch- and condition-effect removed)
- z_d: per-condition-type concatenated unshared-bio factor matrices (e.g. z_ds_cond1, z_ds_cond2)

## How to apply

Load the counts matrix (numpy.array of shape (ncells, ngenes)) and meta_cells dataframe, then build the dataset with create_scdisinfact_dataset using condition_key = ["condition 1", "condition 2"] and batch_key = "batch"; meta_cells must contain columns matching those keys and align row-wise with counts. Instantiate the scdisinfact model with latent dimensions Ks = [8, 2, 2] — one shared-bio block plus one unshared-bio block per condition type — on torch.device("cuda:0" if available else "cpu"). Train with model.train_model(nepochs = 100), which optimizes a combined objective with alternating updates between the inference and generative modules; unshared encoders are regularized by a Gaussian mixture prior and a jointly trained linear classifier that predicts cell conditions, forcing them to encode only condition-relevant variation. After training, call model.eval() and run model.inference under torch.no_grad() on each training count matrix to extract mu_c (shared-bio factors) and mu_d (per-condition unshared-bio factors), then concatenate z_c and z_d across all training matrices.

## Related tools

- **scDisInFact** (Conditional VAE framework whose scdisinfact model, create_scdisinfact_dataset, train_model, and inference functions implement the disentangled latent training loop) — https://github.com/ZhangLabGT/scDisInFact
- **PyTorch** (Deep learning backend used to instantiate the encoders/decoders on CUDA/CPU and run no-grad inference to extract mu_c and mu_d)
- **numpy** (Holds the (ncells, ngenes) count matrices and concatenates extracted z_c / z_d factor arrays across training matrices)
- **pandas** (Holds meta_cells with batch and condition columns used to build the scdisinfact dataset)

## Examples

```
from scDisInFact import scdisinfact, create_scdisinfact_dataset; data_dict = create_scdisinfact_dataset(counts, meta_cells, condition_key=["condition 1", "condition 2"], batch_key="batch"); model = scdisinfact(data_dict=data_dict, Ks=[8,2,2], device=torch.device("cuda:0" if torch.cuda.is_available() else "cpu")); losses = model.train_model(nepochs=100)
```

## Evaluation signals

- train_model returns a decreasing losses sequence over nepochs = 100 without divergence or NaNs.
- model.eval() after training confirms the model is switched to inference mode before factor extraction.
- For each training dataset, model.inference returns dict_inf with 'mu_c' and 'mu_d' entries, and model.generative consumes z_c/mu_c and z_d/mu_d with batch_ids to produce decoded mu.
- After np.concatenate over data_dict['datasets'], z_cs has ncells_total rows with dimension Ks[0] (8), and z_ds_cond1 / z_ds_cond2 each have dimension 2 per the Ks = [8, 2, 2] spec.
- Shared-bio factors z_c show no systematic separation by batch or condition (they are removed of batch and condition effect), while unshared factors separate by condition.

## Limitations

- When each batch of cells is measured under only one condition, there is not enough information to fully disentangle the batch and condition effect.
- scDisInFact assumes condition types are mutually independent, which does not always hold for real-world datasets.
- It assumes condition effect is independent of batch effect, batch effects are consistent across conditions, and condition/batch effects are additive in latent space, limiting modeling of higher-order effects.
- Training the full demo takes around 15 minutes on an Nvidia A40 GPU, and installation takes around 10 minutes.

## Evidence

- [methods] Create scdisinfact dataset from count matrix and meta data: "data_dict = create_scdisinfact_dataset(counts, meta_cells, condition_key = ["condition 1", "condition 2"], batch_key = "batch")"
- [methods] Train scdisinfact model: "losses = model.train_model(nepochs = 100)"
- [methods] Extract shared-bio factor and unshared-bio factors: "Extract shared-bio factor and unshared-bio factors. Shared-bio factor is removed of batch and condition effect."
- [readme] README usage and latent dimensions: "we have two condition types, so there are three element corresponding to shared-bio factor, unshared-bio factor for condition 1, unshared-bio factor for condition 2"
- [abstract] Disentanglement objective: "scDisInFact learns latent factors that disentangle condition effect from batch effect"
- [discussion] Single-condition limitation: "when each batch of cells is measured under only one condition, there is not enough information to fully disentangle the batch and condition effect."
