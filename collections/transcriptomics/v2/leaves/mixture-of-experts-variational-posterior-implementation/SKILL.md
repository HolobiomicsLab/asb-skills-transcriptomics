---
name: mixture-of-experts-variational-posterior-implementation
description: Use when you need to infer a joint latent representation from paired
  single-cell multiomics raw count matrices (e.g. CITE-seq transcriptome + surface
  protein counts) with modality-specific encoders and a mixture-of-experts posterior.
when_to_use_negative:
- Input data are already normalized values rather than raw counts — scMM is designed
  for end-to-end learning by modeling raw count data with different probability distributions
  (e.g. negative binomial likelihoods).
- The task is generic multimodal VAE modeling on non-single-cell paired data (e.g.
  MNIST-SVHN images or CUB image-caption pairs) — use the upstream iffsid/mmvae repository
  directly instead.
- Only a single modality is available with no prospect of joint inference — the MoE
  posterior provides no benefit over a unimodal VAE.
edam_operation: http://edamontology.org/operation_0314
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scMM
  role: PyTorch implementation of the mixture-of-experts multimodal VAE being reconstructed,
    including data loading for RNA-seq/CITE-seq folders and a Colab tutorial
  repo: https://github.com/kodaim1115/scMM
- name: MMVAE
  role: Upstream repository from which scMM's mixture-of-experts VAE code (encoder/decoder/posterior
    factorization) was adopted
  repo: https://github.com/iffsid/mmvae
- name: Google Colab
  role: Hosted GPU environment used to run the scMM tutorial training (colab_tutorial.ipynb)
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.crmeth.2021.100071
    title: A mixture-of-experts deep generative model for integrated analysis of single-cell
      multiomics data
  source_content_hash: sha256:4341ef32eced4fcb7af64c4bae5720ecde17281121b46694d450550775b19d78
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/kodaim1115/scMM
  repo_url: https://github.com/kodaim1115/scMM
  tools:
  - Google Colab
  - MMVAE
  - R
  - scMM
  edam_operation: http://edamontology.org/operation_0314
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# mixture-of-experts-variational-posterior-implementation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Implements scMM's mixture-of-experts multimodal VAE, in which each modality's encoder estimates its own recognition posterior and the experts are mixed with equal weights to infer a joint latent representation from single-cell multiomics counts. Rationale: MoE factorization lets scMM handle missing/complex modalities while achieving end-to-end learning on raw counts via modality-specific likelihoods.

## When to use

Use when you need to infer a joint latent representation from paired single-cell multiomics raw count matrices (e.g. CITE-seq transcriptome + surface protein counts) with modality-specific encoders and a mixture-of-experts posterior.

## When NOT to use

- Input data are already normalized values rather than raw counts — scMM is designed for end-to-end learning by modeling raw count data with different probability distributions (e.g. negative binomial likelihoods).
- The task is generic multimodal VAE modeling on non-single-cell paired data (e.g. MNIST-SVHN images or CUB image-caption pairs) — use the upstream iffsid/mmvae repository directly instead.
- Only a single modality is available with no prospect of joint inference — the MoE posterior provides no benefit over a unimodal VAE.

## Inputs

- RNA count matrix (RNA_count.mtx) with gene.tsv and barcode.tsv feature/barcode files under a data/BMNC/RNA-seq folder
- Surface-protein count matrix (Protein_count.mtx) with protein.tsv and barcode.tsv under a data/BMNC/CITE-seq folder
- Raw single-cell count data for CITE-seq BMNC, randomly subsampled to 15,000 cells with the top 5,000 variable genes for the transcriptome
- ATAC-seq peak-count matrix (peak.tsv features) if using chromatin accessibility as a modality

## Outputs

- Trained model parameters saved to scmm_model.pt
- Joint latent representations for all cells saved to latent_embeddings.tsv
- Monotonically decreasing ELBO training curve

## How to apply

Adopt the encoder/decoder architecture from the scMM PyTorch code, which itself was adopted from the MMVAE repository. Implement one modality-specific encoder per modality (e.g. transcriptome and surface-protein count matrices from the 15,000-cell BMNC CITE-seq subsample), then combine the expert recognition distributions into a mixture-of-experts-factorized variational joint posterior with equal mixing weights across modalities, so each expert's posterior contributes equally regardless of modality size. Configure the prior as a Laplace distribution with learnable per-dimension scale values to encourage disentangled latent dimensions. Define modality-specific decoders with negative binomial likelihoods so the model learns end-to-end directly on raw counts without preprocessing to normalized values. Train the full model on the subsample (GPU optional via the provided Colab notebook), then save parameters to scmm_model.pt and the joint latents to latent_embeddings.tsv.

## Related tools

- **scMM** (PyTorch implementation of the mixture-of-experts multimodal VAE being reconstructed, including data loading for RNA-seq/CITE-seq folders and a Colab tutorial) — https://github.com/kodaim1115/scMM
- **MMVAE** (Upstream repository from which scMM's mixture-of-experts VAE code (encoder/decoder/posterior factorization) was adopted) — https://github.com/iffsid/mmvae
- **Google Colab** (Hosted GPU environment used to run the scMM tutorial training (colab_tutorial.ipynb))

## Evaluation signals

- scmm_model.pt and latent_embeddings.tsv exist and are loadable
- Training ELBO decreases monotonically toward convergence over epochs with no NaN losses
- Each modality's encoder yields its own posterior and equal-weight MoE mixing produces a single joint latent per cell rather than modality-specific latents
- Learned Laplace prior scale parameters are per-dimension and finite, supporting disentangled latent dimensions
- Downstream analysis in R/tutorial.R runs on the emitted latent embeddings

## Limitations

- Equal mixing weights mean a very low-quality or sparse modality contributes equally to the joint posterior, which can dilute signal from the stronger modality.
- Negative binomial likelihoods require raw counts; feeding normalized expression bypasses the end-to-end count modeling scMM was designed for.
- The tutorial data are a small 15,000-cell subsample of BMNC CITE-seq with 5,000 variable genes; scaling behavior to much larger or highly imbalanced datasets is not established in the provided context.
- The MoE posterior implementation depends on code adopted from the MMVAE repository (GPL-3.0) inside scMM (MIT), so licensing provenance of adapted components should be checked on reuse.

## Evidence
