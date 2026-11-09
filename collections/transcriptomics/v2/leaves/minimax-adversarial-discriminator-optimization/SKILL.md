---
name: minimax-adversarial-discriminator-optimization
description: 'Use when integrating scRNA-seq batches with scDREAMER and batch-mixing
  must be achieved adversarially: the discriminator''s Bhattacharyya minimax objective
  and encoder-vs-batch-classifier game drive batch-invariant latent embeddings.'
when_to_use_negative:
- The number of batches is unknown and cannot be supplied — the model assumes prior
  knowledge of the number of batches, and unsupervised treatment of batch information
  is unexplored
- Cell type annotations are available and supervised integration is the goal — scDREAMER-Sup
  with its cell type classifier is the appropriate variant
- The input is not a single-batch-resolved scRNA-seq count matrix (e.g., multiomic
  data — this extension is not yet implemented)
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3308
tools:
- name: scDREAMER
  role: Framework containing the adversarial variational autoencoder, discriminator
    (Bhattacharyya loss), and batch classifier trained in the minimax game
  repo: https://github.com/Zafar-Lab/scDREAMER
- name: Scanpy
  role: Prepares the AnnData input (normalize_total, log-transform, highly_variable_genes
    with seurat flavor) on which the adversarial training runs
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-023-43590-8
    title: scDREAMER for atlas-level integration of single-cell datasets using deep
      generative model paired with adversarial classifier
  source_content_hash: sha256:fc391252e175999b4882e4de6b9d6015ae7ab61df9d9aead7c648c8df492a510
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Zafar-Lab/scDREAMER
  tools:
  - ADAM optimizer
  - Scanpy
  - scDREAMER
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# minimax-adversarial-discriminator-optimization

## Summary

Trains the adversarial components of scDREAMER — a discriminator distinguishing real vs. reconstructed expression profiles via Bhattacharyya loss, and a batch classifier played against the encoder — so the latent embeddings become batch-invariant while preserving biological variation.

## When to use

Use when integrating scRNA-seq batches with scDREAMER and batch-mixing must be achieved adversarially: the discriminator's Bhattacharyya minimax objective and encoder-vs-batch-classifier game drive batch-invariant latent embeddings.

## When NOT to use

- The number of batches is unknown and cannot be supplied — the model assumes prior knowledge of the number of batches, and unsupervised treatment of batch information is unexplored
- Cell type annotations are available and supervised integration is the goal — scDREAMER-Sup with its cell type classifier is the appropriate variant
- The input is not a single-batch-resolved scRNA-seq count matrix (e.g., multiomic data — this extension is not yet implemented)

## Inputs

- Preprocessed Scanpy AnnData expression matrix (raw counts imported as AnnData, normalized with scanpy.pp.normalize_total, log-transformed, top 2000 highly variable genes, seurat flavor)
- Per-cell batch label vector (scDREAMER assumes prior knowledge of the number of batches)

## Outputs

- Batch-invariant latent embeddings (z) saved as latent_embeddings.npy, one vector per input cell
- Decoder-corrected expression profiles for all cells
- Trained discriminator and batch classifier networks used for validating batch invariance

## How to apply

Feed the preprocessed AnnData (top 2000 highly variable genes, seurat flavor) plus per-cell batch labels into four multi-layer dense networks: encoder E mapping (x, s) to (μz, σz, μl, σl), decoder D, discriminator D, and batch classifier B. First maximize the ELBO (KL terms scaled by β plus ZINB reconstruction likelihood with mean μ and dispersion θ) using ADAM with batch size 128. Then run the minimax game: the discriminator is trained against the autoencoder with the Bhattacharyya loss LB = −log Σ P(xi)·P(xī), while the encoder maximizes the cross-entropy loss the batch classifier minimizes, so the classifier cannot differentiate latent embeddings by batch. Stabilize training by down-sampling each cell's UMIs to 80% and penalizing squared differences between the resulting latent representations zi and z^i. The two adversarial components are separable: ablation studies in the paper remove the discriminator and batch classifier independently to quantify each contribution.

## Related tools

- **scDREAMER** (Framework containing the adversarial variational autoencoder, discriminator (Bhattacharyya loss), and batch classifier trained in the minimax game) — https://github.com/Zafar-Lab/scDREAMER
- **Scanpy** (Prepares the AnnData input (normalize_total, log-transform, highly_variable_genes with seurat flavor) on which the adversarial training runs)

## Examples

```
git clone https://github.com/Zafar-Lab/scDREAMER.git; cd scDREAMER/Environments; conda env create -f scDREAMER.yml; conda activate scdreamer
```

## Evaluation signals

- latent_embeddings.npy exists and contains exactly one embedding per input cell
- The adversarially trained batch classifier fails to distinguish batches in the latent space (encoder has maximized its cross-entropy loss, indicating batch-invariant embeddings)
- Downstream batch-mixing metrics — ASW(batch), kBET, graph connectivity — contributing to the paper's composite batch correction score are consistent with reported scDREAMER performance
- Ablation sanity check: removing either the adversarial discriminator or the batch classifier measurably degrades batch mixing relative to the full minimax model
- Training-stability penalty is active: latent representations zi and z^i (from 80%-downsampled UMIs) remain close under the squared-difference penalty

## Limitations

- The model assumes prior knowledge of the number of batches; unsupervised treatment of batch information is not supported
- As a deep learning-based method, scDREAMER may require fine-tuning of some parameters for best performance
- kBET could not be computed for some methods on the healthy heart atlas dataset due to computational limitations in handling large numbers of cells and batches

## Evidence

- [abstract] scDREAMER employs deep generative models and adversarial training: "we present scDREAMER ( https://github.com/Zafar-Lab/scDREAMER ), a data-integration framework that employs deep generative models and adversarial training"
- [readme] scDREAMER uses an adversarial VAE plus batch classifier for batch removal: "employs a novel adversarial variational autoencoder for learning lower-dimensional cellular embeddings and a batch classifier neural network for the removal of batch effects"
- [methods] ADAM optimizer and batch size 128 for training: [evidence span withheld — anchor status: not_found]
- [methods] 80% UMI down-sampling ratio for the stability penalty: [evidence span withheld — anchor status: not_found]
- [results] Ablation of the two adversarial components validates their contribution: "We performed ablation studies to examine the contribution of the two adversarial components of scDREAMER—adversarial discriminator and batch classifier"
- [discussion] Prior knowledge of the number of batches is required: "Our current model assumes prior knowledge of the number of batches. An important future direction would be to explore the unsupervised treatment of batch information"
