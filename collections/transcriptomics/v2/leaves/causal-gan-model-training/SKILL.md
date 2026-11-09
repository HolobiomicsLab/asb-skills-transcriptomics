---
name: causal-gan-model-training
description: Use when a preprocessed, library-size-normalized (20,000) scRNA-seq train
  split and a formatted causal graph exist and a reference-based GAN must be trained
  to later generate realistic single-cell expression data.
when_to_use_negative:
- When the goal is prediction (e.g., forecasting expression changes) rather than generation
  — GRouNdGAN is a generative model and must be retrained for a new dataset.
- When a multi-layer GRN or TF-gene-phenotype causal graph must be imposed — the current
  version only supports bipartite GRNs.
- When unnormalized count data is required — the model generates expression values
  library-size normalized across the considered genes and TFs.
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: GRouNdGAN
  role: Causal implicit GAN trained with --train on the preprocessed data and causal
    graph; checkpoints saved as .pth
  repo: https://github.com/Emad-COMBINE-lab/GRouNdGAN
- name: GRNBoost2
  role: Infers the GRN from the preprocessed train set via --create_grn that is imposed
    on the GAN during training
- name: Docker
  role: Runs GRouNdGAN via the yazdanz/groundgan:4b98686 image for reproducible training
provenance:
  source_task_ids:
  - task_003
  - task_004
  - task_002
  source_papers:
  - doi: 10.1038/s41467-024-48516-6
    title: 'GRouNdGAN: GRN-guided simulation of single-cell RNA-seq data using causal
      generative adversarial networks'
  source_content_hash: sha256:7832e1f42c51b17f9eb435d46e1c77e9f2055f60d5f3bc9afca4156412781f2a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Emad-COMBINE-lab/GRouNdGAN
  tools:
  - Docker
  - GRNBoost2
  - GRouNdGAN
  - SPARSIM
  - TensorBoard
  - scDESIGN2
  - scGAN
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 14
---

# causal-gan-model-training

## Summary

Train the GRouNdGAN causal implicit GAN — a generator whose latent z is mapped through a user-provided causal graph (GRN) so TF-gene regulatory structure is imposed on simulated scRNA-seq — until checkpoints (.pth) are ready for in-silico cell generation.

## When to use

Use when a preprocessed, library-size-normalized (20,000) scRNA-seq train split and a formatted causal graph exist and a reference-based GAN must be trained to later generate realistic single-cell expression data.

## When NOT to use

- When the goal is prediction (e.g., forecasting expression changes) rather than generation — GRouNdGAN is a generative model and must be retrained for a new dataset.
- When a multi-layer GRN or TF-gene-phenotype causal graph must be imposed — the current version only supports bipartite GRNs.
- When unnormalized count data is required — the model generates expression values library-size normalized across the considered genes and TFs.

## Inputs

- Preprocessed train split of the reference scRNA-seq dataset (e.g., 10x Genomics PBMC data, library-size normalized to 20,000, 1000 highly variable genes)
- Formatted causal graph (pickled GRN) produced by --create_grn from GRNBoost2 with k = 15 top TFs per gene
- Homo_sapiens_TF.csv transcription factor list
- causal_gan.cfg configuration file (batch size 1024, critic iterations 5, latent dim 128, lambda 10, 1,000,000 max steps)

## Outputs

- .pth model checkpoints (generator and critic) that can be resumed or used for generation
- TensorBoard training logs/loss curves
- Checkpoint path recorded in [EXPERIMENT]/checkpoint of the config for downstream --generate runs

## How to apply

First run GRouNdGAN's --create_grn so GRNBoost2 infers a GRN on the preprocessed train set using the Homo_sapiens_TF.csv TF list, keeping the top k = 15 regulating TFs per gene and formatting it as the causal graph (or impose a custom GRN and skip --create_grn). Then start training with --train using causal_gan.cfg: batch size 1024, 5 critic iterations per generator iteration, latent dim 128, gradient penalty lambda 10, and 1,000,000 maximum steps on a CUDA GPU, monitoring loss curves with TensorBoard; training can be resumed from intermediate checkpoints. Once converged, populate the [EXPERIMENT]/checkpoint field with the path of the chosen .pth checkpoint so --generate can simulate cells (e.g., 1000–10,000 cells as simulated.h5ad). Note that GRouNdGAN is generative, not predictive: the model must be retrained from scratch for each new reference dataset.

## Related tools

- **GRouNdGAN** (Causal implicit GAN trained with --train on the preprocessed data and causal graph; checkpoints saved as .pth) — https://github.com/Emad-COMBINE-lab/GRouNdGAN
- **GRNBoost2** (Infers the GRN from the preprocessed train set via --create_grn that is imposed on the GAN during training)
- **Docker** (Runs GRouNdGAN via the yazdanz/groundgan:4b98686 image for reproducible training)

## Examples

```
docker pull yazdanz/groundgan:4b98686 && python groundgan --create_grn causal_gan.cfg && python groundgan --train causal_gan.cfg
```

## Evaluation signals

- TensorBoard loss curves show stable adversarial training without mode collapse; critic iterations (5) and gradient penalty lambda (10) remain as configured
- After generation, RF AUROC of real-vs-simulated classification is close to 0.5 (simulated cells indistinguishable from experimental test cells)
- miLISI of simulated data is close to 2 (e.g., 1.891 on PBMC-CTL, 1.90 on PBMC-All)
- Euclidean/cosine distances and MMD between mean profiles of simulated vs test cells are comparable to the real-real control computed between two halves of the test set (e.g., MMD 0.030 vs 0.062 control)
- Training resumes correctly from intermediate .pth checkpoints when interrupted

## Limitations

- The model must be retrained for each new dataset; it is generative, not predictive.
- When trained on a completely new dataset or datasets with significantly different numbers of genes and TFs, hyperparameters should be re-tuned using a validation set.
- Only bipartite GRNs are supported; multi-layer GRNs and TF-gene-phenotype causal graphs are not supported.
- Resemblance to the reference deteriorates if imposed TF-gene relationships differ significantly from reference data patterns.
- Generation of unnormalized count data is not supported in the current version.

## Evidence

- [methods] train command parameters: "--train          Start or resume model training"
- [methods] GRN inference before training: "--create_grn     Infer a GRN from preprocessed data using GRNBoost2 and appropriately format as causal graph"
- [methods] checkpoint configuration: "One training is done, populate the ``[EXPERIMENT]/checkpoint`` field with the path of the ``.pth`` checkpoint"
- [discussion] generative not predictive: [evidence span withheld — anchor status: not_found]
- [methods] docker image: "$ docker pull yazdanz/groundgan:4b98686"
