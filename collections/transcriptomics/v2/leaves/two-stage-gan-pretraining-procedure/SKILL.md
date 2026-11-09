---
name: two-stage-gan-pretraining-procedure
description: Use when building or retraining a CausalGAN-style simulator for scRNA-seq
  where a masked causal controller must first be pretrained as a non-conditional WGAN-GP
  generator before the GRN-structured model is trained end-to-end.
when_to_use_negative:
- You need a predictive model for an existing trained checkpoint on a new dataset
  - GRouNdGAN is generative, not predictive, and must be retrained for a new dataset
- Your GRN is multi-layer or includes TF-gene-phenotype edges - the current version
  only supports imposing a bipartite GRN
- You need unnormalized count output - the model only generates library-size normalized
  expression values
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3512
- http://edamontology.org/topic_3308
tools:
- name: GRouNdGAN
  role: 'Implements both training stages: causal controller pretraining and main causal-GAN
    training with masked target generators, critic, labeler/anti-labeler, and LSN
    layer'
  repo: github.com/Emad-COMBINE-lab/GRouNdGAN
- name: CausalGAN
  role: Architectural basis that GRouNdGAN builds on for its causal controller / target
    generator decomposition
- name: scGAN
  role: Non-conditional scRNA-seq GAN formulation used to pretrain the causal controller
    (WGAN-GP generator) in stage 1
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-024-48516-6
    title: 'GRouNdGAN: GRN-guided simulation of single-cell RNA-seq data using causal
      generative adversarial networks'
  source_content_hash: sha256:9407c689ada9a2526ce047c1f796095101dde546f6bebc1acf5bbc871c08ea2c
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CausalGAN
  - GRouNdGAN
  - scGAN
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3512
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# two-stage GAN pretraining procedure

## Summary

A two-stage training scheme in which GRouNdGAN first pretrains a causal controller as a non-conditional scRNA-seq GAN (WGAN-GP generator), then trains the full causal GAN (target generators, critic, labeler/anti-labeler) so that a user-defined causal GRN governs TF-gene relationships in simulated scRNA-seq data.

## When to use

Use when building or retraining a CausalGAN-style simulator for scRNA-seq where a masked causal controller must first be pretrained as a non-conditional WGAN-GP generator before the GRN-structured model is trained end-to-end.

## When NOT to use

- You need a predictive model for an existing trained checkpoint on a new dataset - GRouNdGAN is generative, not predictive, and must be retrained for a new dataset
- Your GRN is multi-layer or includes TF-gene-phenotype edges - the current version only supports imposing a bipartite GRN
- You need unnormalized count output - the model only generates library-size normalized expression values

## Inputs

- pickled causal graph (nested dict: gene index -> set of regulating TF indices) defining the imposed bipartite GRN
- preprocessed reference scRNA-seq dataset (library-size normalized) for GAN training
- config file with controller pretraining settings (200000 max steps, batch 128, LR 0.0001->0.00001) and main causal GAN settings (1000000 max steps, batch 1024, GP lambda 10, LR 0.001->0.0001)

## Outputs

- pretrained causal controller checkpoint (non-conditional scGAN generator, layers 256-512-1024, latent dim 128)
- fully trained GRouNdGAN checkpoint step_1000000.pth saved under checkpoints/
- TensorBoard summaries (every 10000 steps) and real-vs-simulated t-SNE plots (every 10000 steps)

## How to apply

Stage 1: pretrain the causal controller as a non-conditional scRNA-seq GAN with generator layers 256-512-1024, latent dim 128, gradient-penalty lambda 10, batch size 128, critic iterations 5, Adam betas 0.5/0.9, and learning rate 0.0001 decaying to 0.00001, for up to 200000 steps. Stage 2: instantiate the masked causal generator (one generator per target gene; noise per gene = 1, depth per gene = 3, width per gene = 2; connectivity fixed by the pickled causal graph, a nested dict of gene index -> set of regulating TF indices), a Wasserstein critic with layers 1024-512-256, a labeler/anti-labeler with layers 2000-2000-2000, and a library-size normalization layer (library size = 20000) as the output. Train the main causal GAN for up to 1000000 steps with batch size 1024, critic iterations 5, GP lambda 10, betas 0.5/0.9, and generator learning rate decayed from 0.001 to 0.0001. Each target generator receives only a noise value and the simulated expression of its regulating TFs from the input GRN, which is what makes the user-defined GRN govern the regulatory structure. Log with TensorBoard (summary frequency 10000), plot real-vs-simulated t-SNE every 10000 steps, and checkpoint every 100000 steps under checkpoints/. Rationale: pretraining the controller first stabilizes the implicit latent-to-expression mapping that the causal controller must provide to downstream target generators, and the hyperparameters may need re-tuning on a validation set for datasets with very different gene/TF counts.

## Related tools

- **GRouNdGAN** (Implements both training stages: causal controller pretraining and main causal-GAN training with masked target generators, critic, labeler/anti-labeler, and LSN layer) — github.com/Emad-COMBINE-lab/GRouNdGAN
- **CausalGAN** (Architectural basis that GRouNdGAN builds on for its causal controller / target generator decomposition)
- **scGAN** (Non-conditional scRNA-seq GAN formulation used to pretrain the causal controller (WGAN-GP generator) in stage 1)

## Examples

```
git clone https://github.com/Emad-COMBINE-lab/GRouNdGAN.git
```

## Evaluation signals

- Stage-1 pretraining completes the 200000-step schedule with the controller's WGAN-GP loss converging (lambda 10, betas 0.5/0.9)
- Main training finishes with 'Finished training' and the final checkpoint step_1000000.pth is saved to the checkpoints/ directory
- TensorBoard summaries appear at summary frequency 10000 without diverging critic/generator losses
- Real-vs-simulated t-SNE plots, regenerated every 10000 steps, show simulated cells overlapping the reference distribution
- Generated expression values are library-size normalized to the configured library size of 20000

## Limitations

- Hyperparameters may need re-tuning via a validation set when training on a completely new dataset or datasets with significantly different numbers of genes and TFs
- Resemblance to the reference dataset deteriorates if imposed TF-gene relationships differ significantly from patterns in the reference data
- Only a bipartite GRN can be imposed; multi-layer GRNs and TF-gene-phenotype causal graphs are unsupported in the current version

## Evidence

- [intro] GRouNdGAN (GRN-guided in silico simulation of single-cell RNA-seq data using Causal generative adversarial networks) is a causal implicit generative model: "GRouNdGAN (GRN-guided in silico simulation of single-cell RNA-seq data using Causal generative adversarial networks) is a causal implicit generative model"
- [intro] a causal implicit generative model for reference-based GRN-guided simulation of scRNA-seq data inspired by CausalGAN31: "a causal implicit generative model for reference-based GRN-guided simulation of scRNA-seq data inspired by CausalGAN31"
- [methods] $ git clone https://github.com/Emad-COMBINE-lab/GRouNdGAN.git: "$ git clone https://github.com/Emad-COMBINE-lab/GRouNdGAN.git"
- [methods] You can start training the model using the following command::: "You can start training the model using the following command::"
- [readme] _**GR**N-guided in silic**o** sim**u**lation of single-cell R**N**A-seq **d**ata using Causal **G**enerative **A**dversarial **N**etworks_: "_**GR**N-guided in silic**o** sim**u**lation of single-cell R**N**A-seq **d**ata using Causal **G**enerative **A**dversarial **N**etworks_"
