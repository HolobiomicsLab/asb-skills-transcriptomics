---
name: deep-generative-model-training-gpu
description: Use when a multi-batch, multi-condition scRNA-seq count matrix (e.g.,
  6-patient GBM DMSO vs panobinostat) must be integrated with batch effect removal
  while preserving condition effects, and latent factors are needed for clustering/benchmarking.
when_to_use_negative:
- When each batch is measured under only one condition — there is not enough information
  to fully disentangle the batch and condition effect
- When only batch effect removal without condition modeling is needed and a simpler
  integration tool suffices
- When condition types are strongly correlated and cannot be treated as mutually independent;
  combine them into a single combined condition type first
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scDisInFact
  role: Conditional VAE framework whose scdisinfact model is instantiated with Ks
    and trained on GPU via train_model to learn disentangled shared/unshared-bio factors
  repo: https://github.com/ZhangLabGT/scDisInFact
- name: scINSIGHT
  role: Baseline matrix-factorization method producing common and condition-specific
    modules for comparison of ASW-batch and ARI scores
  repo: https://github.com/Vivianstats/scINSIGHT
- name: PyTorch
  role: Backend providing CUDA GPU device placement and tensor operations for inference/generative
    passes
- name: numpy
  role: Holds the input count matrix (ncells, ngenes) and the concatenated z_cs/z_ds
    latent factor arrays
- name: pandas
  role: Holds meta_cells with condition and batch columns matched to the count matrix
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-024-45227-w
    title: 'scDisInFact: disentangled learning for integration and prediction of multi-batch
      multi-condition single-cell RNA-sequencing data'
  source_content_hash: sha256:5f64c2686cd3c86e371bd84ff579c21691453f11e3af26d6a7a9ec05eda58960
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ZhangLabGT/scDisInFact
  tools:
  - PyTorch
  - numpy
  - pandas
  - scDisInFact
  - scINSIGHT
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 18
---

# deep-generative-model-training-gpu

## Summary

Train scDisInFact's conditional variational autoencoder on GPU to disentangle shared-bio, unshared-bio, and batch factors from multi-batch multi-condition scRNA-seq data, then extract latent factors (mu_c, mu_d) for downstream evaluation such as ASW-batch and ARI.

## When to use

Use when a multi-batch, multi-condition scRNA-seq count matrix (e.g., 6-patient GBM DMSO vs panobinostat) must be integrated with batch effect removal while preserving condition effects, and latent factors are needed for clustering/benchmarking.

## When NOT to use

- When each batch is measured under only one condition — there is not enough information to fully disentangle the batch and condition effect
- When only batch effect removal without condition modeling is needed and a simpler integration tool suffices
- When condition types are strongly correlated and cannot be treated as mutually independent; combine them into a single combined condition type first

## Inputs

- scRNA-seq count matrix as numpy.array of shape (ncells, ngenes)
- Cell metadata as pandas.dataframe of shape (ncells, xx) with columns for condition (e.g., drug treatment) and batch (e.g., patient of origin)
- Latent dimension list Ks (shared-bio factor dim plus one unshared-bio factor dim per condition type)
- CUDA-capable GPU device (torch.device('cuda:0'))

## Outputs

- Trained scDisInFact conditional VAE model with training losses from model.train_model(nepochs = 100)
- Concatenated shared-bio factors z_cs (mu_c across all training count matrices)
- Concatenated unshared-bio factors z_ds per condition type (e.g., z_ds_cond1, z_ds_cond2)
- Reconstructed means mu from model.generative for each training matrix

## How to apply

First build the model input with create_scdisinfact_dataset on the count matrix (numpy array, shape ncells x ngenes) and meta_cells (pandas dataframe), specifying condition_key (e.g., drug treatment) and batch_key (patient of origin). Instantiate scdisinfact with latent dimensions Ks, where the first element is the shared-bio factor dimension and one additional element is added per condition type for the unshared-bio factors, and set device to torch.device('cuda:0') when a GPU is available. Call model.train_model(nepochs = 100) followed by model.eval(). Then loop over all training count matrices, running model.inference on counts_norm and batch_ids to get mu_c and mu_d, and model.generative to reconstruct; concatenate mu_c across datasets into z_cs and per-condition mu_d into z_ds. These factors feed batch-mixing (ASW-batch) and cell-type clustering (ARI) evaluations against baselines such as scINSIGHT's common and condition-specific modules.

## Related tools

- **scDisInFact** (Conditional VAE framework whose scdisinfact model is instantiated with Ks and trained on GPU via train_model to learn disentangled shared/unshared-bio factors) — https://github.com/ZhangLabGT/scDisInFact
- **scINSIGHT** (Baseline matrix-factorization method producing common and condition-specific modules for comparison of ASW-batch and ARI scores) — https://github.com/Vivianstats/scINSIGHT
- **PyTorch** (Backend providing CUDA GPU device placement and tensor operations for inference/generative passes)
- **numpy** (Holds the input count matrix (ncells, ngenes) and the concatenated z_cs/z_ds latent factor arrays)
- **pandas** (Holds meta_cells with condition and batch columns matched to the count matrix)

## Examples

```
from scDisInFact import scdisinfact; device = torch.device('cuda:0'); model = scdisinfact(data_dict = data_dict, Ks = [8, 2], device = device); losses = model.train_model(nepochs = 100)
```

## Evaluation signals

- Training losses from model.train_model decrease over 100 epochs without divergence
- Concatenated z_cs has shape (total cells across all training matrices, Ks[0]); z_ds per condition have the unshared-bio dimension
- Shared-bio factors (z_cs) show batch mixing (ASW-batch) consistent with the reported Fig 4a result and high ARI against cell-type annotations
- Unshared-bio factors (z_ds) reflect condition structure (DMSO vs panobinostat), with scDisInFact outperforming scINSIGHT on both ARI (shared factors) and ASW-batch (unshared factors) on the GBM dataset
- model.eval() called after training so subsequent inference/generative passes run in evaluation mode

## Limitations

- When each batch of cells is measured under only one condition, there is not enough information to fully disentangle the batch and condition effect
- scDisInFact assumes condition types are mutually independent, which does not always hold for real-world datasets
- Condition effect is assumed independent of batch effect, batch effects are assumed consistent across conditions, and condition/batch effects are assumed additive in latent space, limiting modeling of higher-order effects

## Evidence

- [readme] Declare latent dimensions Ks and train on GPU: "device = torch.device("cuda:0" if torch.cuda.is_available() else "cpu")
model = scdisinfact(data_dict = data_dict, Ks = Ks, device = device)
losses = model.train_model(nepochs = 100)"
- [methods] Dataset construction from counts and metadata: "data_dict = create_scdisinfact_dataset(counts, meta_cells, condition_key = ["condition 1", "condition 2"], batch_key = "batch")"
- [readme] Extract latent factors via inference and generative passes: "dict_inf = model.inference(counts = dataset.counts_norm.to(model.device), batch_ids = dataset.batch_id[:,None].to(model.device), print_stat = True)"
- [abstract] Latent factors shared across batches and conditions: "scDisInFact learns latent factors that disentangle condition effect from batch effect, enabling it to simultaneously perform three tasks: batch effect removal, condition-associated key gene"
- [intro] Disentanglement of variation into shared and condition-specific factors: "It disentangles the variation within the multi-batch multi-condition dataset into latent factors encoding the biological information shared across all data matrices, condition-specific biological"
