---
name: adversarial-variational-autoencoder-training
description: Use when integrating multi-batch scRNA-seq count matrices (Scanpy AnnData)
  and you need batch-corrected latent embeddings, especially with skewed cell-type
  distributions, nested batch effects, many batches, or developmental trajectories
  that must be conserved.
when_to_use_negative:
- Input is already batch-corrected or is not a raw/normalized count matrix from multiple
  batches — no batch effect exists to remove
- The number of batches is unknown and cannot be supplied, since the model assumes
  prior knowledge of the number of batches
- You need batch information treated unsupervised or to exploit hierarchical batch
  structure — the paper lists these as unimplemented future directions
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0091
tools:
- name: scDREAMER
  role: Adversarial VAE framework (encoder, decoder, discriminator, batch classifier)
    whose training procedure this skill implements
  repo: https://github.com/Zafar-Lab/scDREAMER
- name: Scanpy
  role: 'Preprocessing: AnnData import, mitochondrial QC filtering, normalize_total,
    log-transform, and highly_variable_genes (flavor=''seurat'') to produce the model
    input'
- name: ADAM optimizer
  role: Optimizer used for model training with batch size 128
provenance:
  source_task_ids:
  - task_001
  - task_005
  - task_007
  source_papers:
  - doi: 10.1038/s41467-023-43590-8
    title: scDREAMER for atlas-level integration of single-cell datasets using deep
      generative model paired with adversarial classifier
  source_content_hash: sha256:a1cef5ca16e8277882a5e1743ff501c8461a4e2da69447cb215db3230754ac30
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Zafar-Lab/scDREAMER
  tools:
  - ADAM optimizer
  - BBKNN
  - Harmony
  - INSCT
  - LIGER
  - Scanorama
  - Scanpy
  - Seurat
  - iMAP
  - scANVI
  - scDML
  - scDREAMER
  - scDREAMER-Sup
  - scGEN
  - scVI
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 18
---

# adversarial-variational-autoencoder-training

## Summary

Trains scDREAMER's adversarial variational autoencoder (encoder E, decoder D, discriminator D, batch classifier B) to produce batch-invariant latent embeddings of scRNA-seq data. The VAE maximizes the ELBO (KL terms scaled by beta plus ZINB reconstruction), while the batch classifier is trained adversarially with cross-entropy so the encoder generates embeddings that cannot be differentiated across batches.

## When to use

Use when integrating multi-batch scRNA-seq count matrices (Scanpy AnnData) and you need batch-corrected latent embeddings, especially with skewed cell-type distributions, nested batch effects, many batches, or developmental trajectories that must be conserved.

## When NOT to use

- Input is already batch-corrected or is not a raw/normalized count matrix from multiple batches — no batch effect exists to remove
- The number of batches is unknown and cannot be supplied, since the model assumes prior knowledge of the number of batches
- You need batch information treated unsupervised or to exploit hierarchical batch structure — the paper lists these as unimplemented future directions

## Inputs

- Raw count scRNA-seq expression matrix as a Scanpy AnnData object with per-cell batch labels
- Preprocessed top 2000 highly variable gene matrix (seurat flavor)
- Optional cell-type annotations for supervised/semi-supervised variants (scDREAMER-Sup)

## Outputs

- Batch-invariant latent embeddings per cell (e.g., latent_embeddings.npy)
- Decoder-corrected expression profiles
- Integrated embedding for downstream clustering, trajectory, and label-prediction analyses

## How to apply

Preprocess the AnnData by removing low-quality cells based on mitochondrial gene counts, normalizing with scanpy.pp.normalize_total and log-transforming, then select the top 2000 highly variable genes using scanpy.pp.highly_variable_genes with flavor='seurat'. Instantiate encoder E mapping (x, s) to (mu_z, sigma_z, mu_l, sigma_l), decoder D reconstructing expression from z and s, a discriminator, and a batch classifier B. Train encoder and decoder to maximize the ELBO (KL terms for z and l plus ZINB reconstruction likelihood with mean mu and dispersion theta), scaling the KL divergence term by a beta factor, using the ADAM optimizer with batch size 128. Adversarially train the discriminator with the Bhattacharyya loss L_B = -log sum P(x_i)*P(x_i_bar), and train the batch classifier with cross-entropy on batch labels so the encoder maximizes while the classifier minimizes, making embeddings indistinguishable across batches. Add a training-stability penalty by down-sampling each cell's UMIs to 80% and penalizing squared differences between the resulting latent representations z_i and z_hat_i. After convergence, extract encoder embeddings z and the decoder's corrected expression profiles.

## Related tools

- **scDREAMER** (Adversarial VAE framework (encoder, decoder, discriminator, batch classifier) whose training procedure this skill implements) — https://github.com/Zafar-Lab/scDREAMER
- **Scanpy** (Preprocessing: AnnData import, mitochondrial QC filtering, normalize_total, log-transform, and highly_variable_genes (flavor='seurat') to produce the model input)
- **ADAM optimizer** (Optimizer used for model training with batch size 128)

## Examples

```
git clone https://github.com/Zafar-Lab/scDREAMER.git; cd scDREAMER/Environments; conda env create -f scDREAMER.yml; conda activate scdreamer
```

## Evaluation signals

- latent_embeddings.npy exists and contains exactly one embedding per input cell
- The adversarially trained batch classifier fails to distinguish batches in the latent space (encoder wins the adversarial cross-entropy game)
- Batch-mixing metrics entering the composite batch-correction score (ASW(batch) scaled as 1 - abs(ASW), kBET with k=50, graph connectivity, PCR batch) are consistent with reported scDREAMER performance
- Biological conservation metrics (NMI, ARI, cell-type ASW, isolated label F1) computed from Louvain clusters at the resolution maximizing NMI remain high after correction
- Ablating the adversarial discriminator and batch classifier components should degrade batch mixing, confirming both components contribute

## Limitations

- The model assumes prior knowledge of the number of batches; unsupervised treatment of batch information is unexplored
- Hierarchical structure between different batch information is not yet utilized
- As a deep-learning-based method, scDREAMER may require fine-tuning of some parameters for extracting the best performance
- Multiomic dataset extension is not yet implemented (planned future work)

## Evidence

- [readme] scDREAMER is a single-cell data integration framework that employs a novel adversarial variational autoencoder: "scDREAMER is a single-cell data integration framework that employs a novel adversarial variational autoencoder"
- [methods] Top 2000 HVGs with seurat flavor: [evidence span withheld — anchor status: not_found]
- [methods] ADAM optimizer and batch size 128: [evidence span withheld — anchor status: not_found]
- [methods] 80% UMI down-sampling penalty: [evidence span withheld — anchor status: not_found]
- [abstract] Deep generative models with adversarial training: "we present scDREAMER ( https://github.com/Zafar-Lab/scDREAMER ), a data-integration framework that employs deep generative models and adversarial training"
- [methods] Batch-mixing evaluation metrics: "for the healthy heart atlas dataset, kBET could not be computed for some of the methods due to computational limitations in handling large number of cells and batches"
