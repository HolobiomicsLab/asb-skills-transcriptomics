---
name: scrna-seq-batch-effect-correction
description: Use when a Scanpy AnnData object with raw scRNA-seq counts plus per-cell
  batch labels shows cells separating by batch rather than by cell type (e.g. skewed
  cell types across batches, nested batch effects, or >10 batches), and an unsupervised
  integrated embedding is needed.
when_to_use_negative:
- Data contains only one batch (no batch labels to adversarially confuse) — integration
  is undefined
- Cell type annotations are available and a supervised integration is desired — use
  scDREAMER-Sup, which adds an additional variational autoencoder and cell type classifier
- The task is multiomic integration (e.g. paired RNA + ATAC) — the article states
  extension to multiomic datasets is planned future work, not yet implemented
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scDREAMER
  role: Core adversarial VAE framework whose encoder/batch-classifier minimax training
    produces the batch-invariant latent embeddings
  repo: https://github.com/Zafar-Lab/scDREAMER
- name: Scanpy
  role: 'Preprocessing: AnnData import, normalization (scanly.pp.normalize_total),
    log transformation, and top-2000 highly variable gene selection (seurat flavor)'
- name: ADAM optimizer
  role: Optimizer used to train the scDREAMER networks with batch size 128
- name: INSCT
  role: Alternative batch-aware triplet-network integrator used as a comparison method
    for runtime benchmarking on subsampled datasets
  repo: https://github.com/lkmklsmn/insct
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-023-43590-8
    title: scDREAMER for atlas-level integration of single-cell datasets using deep
      generative model paired with adversarial classifier
  source_content_hash: sha256:5f7fc08ed50d4a278554f32973a1ee7dcdfc62a384c93b0c40f68dd43199fb14
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Zafar-Lab/scDREAMER
  tools:
  - ADAM optimizer
  - INSCT
  - Scanpy
  - scDREAMER
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# scRNA-seq-batch-effect-correction

## Summary

Integrate multiple scRNA-seq batches into a shared latent space using an adversarial variational autoencoder (scDREAMER), in which a batch classifier is trained adversarially against the encoder so that the learned embeddings are batch-invariant while conserving biological variation.

## When to use

Use when a Scanpy AnnData object with raw scRNA-seq counts plus per-cell batch labels shows cells separating by batch rather than by cell type (e.g. skewed cell types across batches, nested batch effects, or >10 batches), and an unsupervised integrated embedding is needed.

## When NOT to use

- Data contains only one batch (no batch labels to adversarially confuse) — integration is undefined
- Cell type annotations are available and a supervised integration is desired — use scDREAMER-Sup, which adds an additional variational autoencoder and cell type classifier
- The task is multiomic integration (e.g. paired RNA + ATAC) — the article states extension to multiomic datasets is planned future work, not yet implemented

## Inputs

- Raw count scRNA-seq expression matrix as a Scanpy AnnData object
- Per-cell batch label vector (e.g. adata.obs['batch'])
- Preprocessed feature matrix restricted to top 2000 highly variable genes (seurat flavor)

## Outputs

- latent_embeddings.npy containing one batch-corrected embedding per input cell
- Decoder-corrected expression profiles for all cells
- Trained encoder, decoder, discriminator, and batch classifier networks

## How to apply

Import raw count matrices as a Scanpy AnnData object, remove low-quality cells based on mitochondrial gene counts, normalize with scanpy.pp.normalize_total, log-transform, and select the top 2000 highly variable genes with scanly.pp.highly_variable_genes (flavor='seurat') as input to scDREAMER. Instantiate encoder E mapping (x, s) to (μz, σz, μl, σl), decoder D reconstructing expression from z and s, a discriminator, and a batch classifier; train encoder and decoder by maximizing the ELBO (KL terms for z and l plus ZINB reconstruction likelihood with mean μ and dispersion θ, KL scaled by β) using the ADAM optimizer with batch size 128. Adversarially train the discriminator against the autoencoder with the Bhattacharyya loss LB = −log Σ P(xi)·P(xi¯), and train the batch classifier with cross-entropy loss on batch labels so the encoder maximizes the loss and the classifier minimizes it, producing embeddings the classifier cannot assign to a batch. Add the training-stability penalty by down-sampling each cell's UMIs to 80% and penalizing squared differences between the resulting latent representations zi and z^i. After convergence, extract encoder embeddings z for all cells (and optionally decoder-corrected profiles) and save them as latent_embeddings.npy.

## Related tools

- **scDREAMER** (Core adversarial VAE framework whose encoder/batch-classifier minimax training produces the batch-invariant latent embeddings) — https://github.com/Zafar-Lab/scDREAMER
- **Scanpy** (Preprocessing: AnnData import, normalization (scanly.pp.normalize_total), log transformation, and top-2000 highly variable gene selection (seurat flavor))
- **ADAM optimizer** (Optimizer used to train the scDREAMER networks with batch size 128)
- **INSCT** (Alternative batch-aware triplet-network integrator used as a comparison method for runtime benchmarking on subsampled datasets) — https://github.com/lkmklsmn/insct

## Examples

```
git clone https://github.com/Zafar-Lab/scDREAMER.git; cd scDREAMER/Environments; conda env create -f scDREAMER.yml; conda activate scdreamer
```

## Evaluation signals

- latent_embeddings.npy exists and contains exactly one embedding per input cell
- The adversarially trained batch classifier fails to distinguish batches from the latent embeddings (encoder wins the minimax game)
- Batch-mixing metrics ASW(batch), kBET, and graph connectivity are consistent with the paper's composite batch correction score and reported performance
- Biological conservation verified via Louvain clustering with resolution chosen to maximize NMI against known labels
- Ablation check: removing the discriminator and/or batch classifier components should measurably degrade batch mixing, confirming both adversarial components contribute

## Limitations

- The model assumes prior knowledge of the number of batches; unsupervised treatment of batch information is unexplored
- As a deep learning-based method, scDREAMER may require fine-tuning of some parameters for extracting the best performance
- kBET could not be computed for some methods on the healthy heart atlas dataset due to computational limitations with large numbers of cells and batches
- Hierarchical structure between different batch information is not yet utilized, and multiomic extension is future work

## Evidence

- [results] encoder maximizes a cross-entropy loss so as to generate embeddings that the batch classifier cannot differentiate between batches: [evidence span withheld — anchor status: not_found]
- [methods] HVG selection parameters: [evidence span withheld — anchor status: not_found]
- [methods] training optimizer and batch size: "For training our model, we have adopted ADAM optimizer41."
- [methods] UMI down-sampling stability penalty: [evidence span withheld — anchor status: not_found]
- [readme] installation via conda environment: "Creating conda environment using ./ENVIRONMENTS/scDREAMER.yml will install all the dependent packages and libraries."
- [abstract] scope of integration challenges handled: "scDREAMER can overcome critical challenges including skewed cell type distribution among batches, nested batch-effects, large number of batches and conservation of development trajectory across"
