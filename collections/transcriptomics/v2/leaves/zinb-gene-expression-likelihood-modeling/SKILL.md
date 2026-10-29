---
name: zinb-gene-expression-likelihood-modeling
description: Use when building or reimplementing a VAE-based single-cell integration
  model (e.g., scDREAMER) where the decoder must reconstruct raw UMI counts with a
  ZINB likelihood and KL-regularized latent terms.
when_to_use_negative:
- Data are already batch-corrected embeddings or feature tables rather than raw/normalized
  count matrices — the ZINB likelihood presumes UMI count structure
- Cell type labels are available and a supervised integration (scDREAMER-Sup with
  its additional variational autoencoder and cell type classifier) is intended instead
  of the unsupervised ZINB-VAE
- The task requires supervised or semi-supervised integration where cell type annotations
  should drive the latent space
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2269
tools:
- name: scDREAMER
  role: Framework whose adversarial variational autoencoder implements the ZINB likelihood
    (μ, θ) within the ELBO maximization
  repo: https://github.com/Zafar-Lab/scDREAMER
- name: Scanpy
  role: Preprocessing pipeline (AnnData import, normalize_total, log transform, top-2000
    HVG selection with seurat flavor) that produces the model input
- name: ADAM optimizer
  role: Optimizer used for ELBO maximization with batch size 128
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-023-43590-8
    title: scDREAMER for atlas-level integration of single-cell datasets using deep
      generative model paired with adversarial classifier
  source_content_hash: sha256:f82c1163e4e9497c469297d544f02e80f82d565681cc2e503953981da1d25074
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Zafar-Lab/scDREAMER
  tools:
  - ADAM optimizer
  - Scanpy
  - scDREAMER
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# zinb-gene-expression-likelihood-modeling

## Summary

Model scRNA-seq count data with a ZINB reconstruction likelihood (mean μ, dispersion θ) inside a variational autoencoder, so the decoder's generative likelihood properly accounts for dropout and overdispersion while training the encoder to maximize the ELBO.

## When to use

Use when building or reimplementing a VAE-based single-cell integration model (e.g., scDREAMER) where the decoder must reconstruct raw UMI counts with a ZINB likelihood and KL-regularized latent terms.

## When NOT to use

- Data are already batch-corrected embeddings or feature tables rather than raw/normalized count matrices — the ZINB likelihood presumes UMI count structure
- Cell type labels are available and a supervised integration (scDREAMER-Sup with its additional variational autoencoder and cell type classifier) is intended instead of the unsupervised ZINB-VAE
- The task requires supervised or semi-supervised integration where cell type annotations should drive the latent space

## Inputs

- Raw count expression matrix as a Scanpy AnnData object, subset to top 2000 highly variable genes (seurat flavor) after scanly.pp.normalize_total and log transformation
- Per-cell batch information labels
- Latent dimensionality and KL scaling factor β hyperparameters

## Outputs

- Trained encoder embeddings z for all cells (saved as latent_embeddings.npy, one embedding per input cell)
- Decoder's batch-corrected expression profiles
- ZINB parameters μ (mean) and θ (dispersion) per gene per cell from the decoder

## How to apply

Instantiate a multi-layer dense decoder D that reconstructs expression from the latent biological state z and batch covariate s, outputting the ZINB mean μ and dispersion θ, and train encoder E and decoder D by maximizing the ELBO — the KL divergence terms for z and l (scaled by factor β) plus the ZINB reconstruction likelihood. Use the ADAM optimizer with a batch size of 128, feeding Scanpy AnnData objects preprocessed to the top 2000 highly variable genes (scanpy.pp.highly_variable_genes, flavor 'seurat') after normalize_total and log transformation. The ZINB likelihood is chosen because scRNA-seq counts are zero-inflated and overdispersed; the decoder's corrected expression profiles can additionally be extracted as an output alongside the latent embeddings. Rationale comes from scDREAMER's architecture, where E maps (x, s) to (μz, σz, μl, σl) and reconstruction quality under the ZINB likelihood anchors the latent space before adversarial batch correction (Bhattacharyya loss on the discriminator, cross-entropy on the batch classifier).

## Related tools

- **scDREAMER** (Framework whose adversarial variational autoencoder implements the ZINB likelihood (μ, θ) within the ELBO maximization) — https://github.com/Zafar-Lab/scDREAMER
- **Scanpy** (Preprocessing pipeline (AnnData import, normalize_total, log transform, top-2000 HVG selection with seurat flavor) that produces the model input)
- **ADAM optimizer** (Optimizer used for ELBO maximization with batch size 128)

## Examples

```
conda env create -f scDREAMER.yml; conda activate scdreamer; python train_scdreamer.py --adata processed.h5ad --batch_key batch --batch_size 128
```

## Evaluation signals

- latent_embeddings.npy exists and contains exactly one embedding per input cell
- Training maximizes the ELBO with both KL terms (z and l, scaled by β) and ZINB reconstruction likelihood converging without divergence under ADAM (batch size 128)
- Downstream batch-mixing metrics (ASW(batch), kBET, graph connectivity) entering the composite batch correction score are consistent with reported scDREAMER performance
- Biological conservation is preserved: Louvain clustering at resolution maximizing NMI reproduces expected cell types and rare cell types are identified
- Decoder's corrected expression profiles retain gene-level structure suitable for downstream marker gene analysis

## Limitations

- The model assumes prior knowledge of the number of batches; unsupervised treatment of batch information is unexplored
- As a deep learning-based method, scDREAMER may require fine-tuning of some parameters (e.g., β, latent dimensions) for extracting the best performance
- A training-stability penalty requiring UMI down-sampling to 80% per cell adds compute; hierarchical structure between different batch information is not yet utilized

## Evidence

- [other] Instantiate the four multi-layer dense networks: encoder E mapping (x, s) to (μz, σz, μl, σl), decoder D reconstructing expression from z and s: [evidence span withheld — anchor status: not_found]
- [other] Train encoder and decoder by maximizing the ELBO (KL terms for z and l plus ZINB reconstruction likelihood with mean μ and dispersion θ), scaling the KL divergence term with factor β: [evidence span withheld — anchor status: not_found]
- [methods] we selected the top 2000 highly variable genes using the function “scanpy.pp.highly_variable_genes” (with flavor parameter as “seurat”): [evidence span withheld — anchor status: not_found]
- [methods] For training our model, we have adopted ADAM optimizer41. ... The batch size of 128 is used while training the model.: [evidence span withheld — anchor status: not_found]
- [readme] scDREAMER is a single-cell data integration framework that employs a novel adversarial variational autoencoder for learning lower-dimensional cellular embeddings: "scDREAMER is a single-cell data integration framework that employs a novel adversarial variational autoencoder for learning lower-dimensional cellular embeddings and a batch classifier neural network"
