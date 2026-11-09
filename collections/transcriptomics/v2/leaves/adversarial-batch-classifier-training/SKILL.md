---
name: adversarial-batch-classifier-training
description: 'Use when integrating multiple scRNA-seq batches whose embeddings show
  batch separation: per-cell batch labels are available and a VAE latent space must
  be made batch-invariant without erasing biological variation.'
when_to_use_negative:
- Batch identity is unknown and cannot be annotated — the model assumes prior knowledge
  of the number of batches; unsupervised treatment of batch information is unexplored
- The goal is comparing against MNN-based methods (BBKNN, Scanorama, Seurat v3) rather
  than adversarial deep-learning integration — different tool family with different
  memory profiles
- Extremely large batches plus datasets where kBET could not be computed due to computational
  limitations handling large numbers of cells and batches
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scDREAMER
  role: Framework whose encoder and adversarially trained batch classifier produce
    batch-invariant cellular embeddings
  repo: https://github.com/Zafar-Lab/scDREAMER
- name: ADAM optimizer
  role: Optimizer used for training the encoder, decoder, discriminator, and batch
    classifier at batch size 128
- name: Scanpy
  role: Preprocessing of raw counts into AnnData with normalization, log-transform,
    and top-2000 HVG selection before adversarial training
provenance:
  source_task_ids:
  - task_001
  - task_002
  source_papers:
  - doi: 10.1038/s41467-023-43590-8
    title: scDREAMER for atlas-level integration of single-cell datasets using deep
      generative model paired with adversarial classifier
  source_content_hash: sha256:e79aab9b3a32f82b077a3e1c48722cfed5faa7ef6b5a17792ad241674bd502ad
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Zafar-Lab/scDREAMER
  tools:
  - ADAM optimizer
  - Scanpy
  - scDREAMER
  - scDREAMER-Sup
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# adversarial-batch-classifier-training

## Summary

Adversarially train a feed-forward batch classifier against the scDREAMER encoder (encoder maximizes cross-entropy, classifier minimizes it) so latent embeddings become indistinguishable across batches, enabling batch-invariant single-cell integration.

## When to use

Use when integrating multiple scRNA-seq batches whose embeddings show batch separation: per-cell batch labels are available and a VAE latent space must be made batch-invariant without erasing biological variation.

## When NOT to use

- Batch identity is unknown and cannot be annotated — the model assumes prior knowledge of the number of batches; unsupervised treatment of batch information is unexplored
- The goal is comparing against MNN-based methods (BBKNN, Scanorama, Seurat v3) rather than adversarial deep-learning integration — different tool family with different memory profiles
- Extremely large batches plus datasets where kBET could not be computed due to computational limitations handling large numbers of cells and batches

## Inputs

- Preprocessed Scanpy AnnData expression matrix (normalized, log-transformed, top 2000 highly variable genes, seurat flavor)
- Per-cell batch label vector (number of batches assumed known)

## Outputs

- Batch-invariant latent embeddings z for all cells (latent_embeddings.npy)
- Trained batch classifier B whose failure to distinguish batches certifies batch mixing

## How to apply

Add a multi-layer dense batch classifier B as a fourth network alongside encoder E, decoder D, and discriminator D. Train with the ADAM optimizer at batch size 128: B minimizes cross-entropy loss on per-cell batch labels while the encoder maximizes that same cross-entropy, so the embeddings it produces cannot be differentiated between batches. This adversarial game runs on top of ELBO maximization (KL terms for z and l scaled by β, ZINB reconstruction with dispersion θ) and the Bhattacharyya discriminator loss LB = −log Σ P(xi)·P(xi¯). Add the stability penalty from down-sampling each cell's UMIs to 80% and penalizing squared differences between the resulting latent representations zi and z^i. Optionally run an ablation (remove the batch classifier, keep the discriminator) to quantify its contribution to batch mixing.

## Related tools

- **scDREAMER** (Framework whose encoder and adversarially trained batch classifier produce batch-invariant cellular embeddings) — https://github.com/Zafar-Lab/scDREAMER
- **ADAM optimizer** (Optimizer used for training the encoder, decoder, discriminator, and batch classifier at batch size 128)
- **Scanpy** (Preprocessing of raw counts into AnnData with normalization, log-transform, and top-2000 HVG selection before adversarial training)

## Examples

```
git clone https://github.com/Zafar-Lab/scDREAMER.git && cd scDREAMER/Environments && conda env create -f scDREAMER.yml && conda activate scdreamer
```

## Evaluation signals

- latent_embeddings.npy exists and contains one embedding per input cell
- The adversarially trained batch classifier fails to distinguish batches (cross-entropy approaches chance level)
- Downstream batch-mixing metrics ASW(batch), kBET, and graph connectivity contribute to a composite batch correction score consistent with reported performance
- Louvain clustering (resolution maximizing NMI) confirms biological conservation alongside batch mixing
- Ablation study removing the batch classifier shows measurably worse batch mixing, confirming its contribution

## Limitations

- Model assumes prior knowledge of the number of batches; unsupervised treatment of batch information is unexplored
- As a deep learning-based method it may require fine-tuning of some parameters for extracting the best performance
- kBET could not be computed for some methods on the healthy heart atlas dataset due to computational limitations in handling large number of cells and batches

## Evidence

- [methods] the encoder maximizes a cross-entropy loss so as to generate embeddings that the batch classifier cannot differentiate between batches, yielding batch-invariant latent embeddings: [evidence span withheld — anchor status: not_found]
- [methods] For training our model, we have adopted ADAM optimizer41. ... The batch size of 128 is used while training the model.: [evidence span withheld — anchor status: not_found]
- [results] We performed ablation studies to examine the contribution of the two adversarial components of scDREAMER—adversarial discriminator and batch classifier: "We performed ablation studies to examine the contribution of the two adversarial components of scDREAMER—adversarial discriminator and batch classifier"
- [discussion] Our current model assumes prior knowledge of the number of batches. An important future direction would be to explore the unsupervised treatment of batch information: "Our current model assumes prior knowledge of the number of batches. An important future direction would be to explore the unsupervised treatment of batch information"
- [readme] a batch classifier neural network for the removal of batch effects: "a batch classifier neural network for the removal of batch effects"
