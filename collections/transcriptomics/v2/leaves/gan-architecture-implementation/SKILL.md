---
name: gan-architecture-implementation
description: Use when simulating single-cell RNA-seq data that must honor a user-provided
  causal gene regulatory network, or when implementing a CausalGAN-derived generator
  with masked connectivity from a pickled causal graph.
when_to_use_negative:
- Your GRN is multi-layer or includes TF-gene-phenotype edges — the current GRouNdGAN
  version only supports bipartite GRNs
- You need unnormalized count data — the model only generates library-size-normalized
  expression values
- You want to predict knockout outcomes from a fixed model — GRouNdGAN is generative,
  not predictive, and must be retrained for a new dataset
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: GRouNdGAN
  role: 'Causal GAN implementation being reconstructed: causal controller, masked
    target generators, critic, labeler/anti-labeler, LSN layer'
  repo: https://github.com/Emad-COMBINE-lab/GRouNdGAN
- name: CausalGAN
  role: Architectural inspiration for the causal controller / target generator design
- name: scGAN
  role: Non-conditional scRNA-seq GAN used as the pre-training stage for the causal
    controller
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-024-48516-6
    title: 'GRouNdGAN: GRN-guided simulation of single-cell RNA-seq data using causal
      generative adversarial networks'
  source_content_hash: sha256:358b2a4109332b69e3d7ffa81080aafd24d9ad8af35f855e2d260de327b50087
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Emad-COMBINE-lab/GRouNdGAN
  tools:
  - CausalGAN
  - GRouNdGAN
  - scGAN
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# GAN architecture implementation

## Summary

Reconstruct a two-step causal GAN (GRouNdGAN-style) in which a pre-trained causal controller feeds masked per-target-gene generators constrained by a user-defined GRN, trained with Wasserstein critic, gradient penalty, and labeler/anti-labeler heads so simulated scRNA-seq respects TF-gene regulation.

## When to use

Use when simulating single-cell RNA-seq data that must honor a user-provided causal gene regulatory network, or when implementing a CausalGAN-derived generator with masked connectivity from a pickled causal graph.

## When NOT to use

- Your GRN is multi-layer or includes TF-gene-phenotype edges — the current GRouNdGAN version only supports bipartite GRNs
- You need unnormalized count data — the model only generates library-size-normalized expression values
- You want to predict knockout outcomes from a fixed model — GRouNdGAN is generative, not predictive, and must be retrained for a new dataset

## Inputs

- Pickled causal graph (nested dict of gene index -> set of regulating TF indices) from GRNBoost2 or a custom GRN
- Preprocessed reference scRNA-seq dataset for GAN training
- Configuration file specifying generator/critic/labeler layer sizes, latent dim, batch sizes, learning rates, lambda, and checkpoint paths

## Outputs

- Trained GRouNdGAN model checkpoints (step_1000000.pth under checkpoints/)
- Causal controller pre-trained as a WGAN-GP generator
- TensorBoard training summaries and real-vs-simulated t-SNE plots
- Generator capable of library-size-normalized (library size 20000) simulated scRNA-seq expression

## How to apply

Instantiate the masked causal generator with one generator per target gene (noise per gene = 1, depth per gene = 3, width per gene = 2), where each generator receives only a noise value plus the simulated expression of its regulating TFs as fixed by the pickled causal graph (nested dict of gene index -> set of regulating TF indices). Add a Wasserstein critic (layers 1024 512 256), labeler and anti-labeler (layers 2000 2000 2000), and a library-size normalization output layer (library size = 20000). Pretrain the causal controller as a non-conditional scRNA-seq GAN (generator layers 256 512 1024, latent dim 128, lambda 10) for max 200000 steps, batch size 128, critic iterations 5, Adam betas 0.5/0.9, learning rate 0.0001 decaying to 0.00001. Then train the main causal GAN for max 1000000 steps, batch size 1024, critic iterations 5, gradient-penalty lambda 10, betas 0.5/0.9, generator LR decayed 0.001 -> 0.0001, logging to TensorBoard (summary frequency 10000), t-SNE real-vs-simulated plots (frequency 10000), and checkpoints saved every 100000 steps.

## Related tools

- **GRouNdGAN** (Causal GAN implementation being reconstructed: causal controller, masked target generators, critic, labeler/anti-labeler, LSN layer) — https://github.com/Emad-COMBINE-lab/GRouNdGAN
- **CausalGAN** (Architectural inspiration for the causal controller / target generator design)
- **scGAN** (Non-conditional scRNA-seq GAN used as the pre-training stage for the causal controller)

## Examples

```
docker pull yazdanz/groundgan:4b98686
```

## Evaluation signals

- Training completes with 'Finished training' and the final checkpoint step_1000000.pth is written to checkpoints/
- Real-vs-simulated t-SNE plots (logged every 10000 steps) show overlap of simulated cells with the reference dataset
- Critic and generator losses converge without mode collapse at gradient-penalty lambda = 10 and critic iterations = 5
- Simulated data passes the --evaluate data-quality checks and preserves gene identities and TF-gene regulatory relationships from the imposed GRN
- The generated expression is library-size normalized to 20000 across the considered genes and TFs

## Limitations

- Resemblance to the reference dataset deteriorates if the imposed TF-gene relationships differ significantly from reference data patterns
- Only bipartite GRNs are supported; multi-layer GRNs and TF-gene-phenotype causal graphs are planned for future versions
- Hyperparameters may need re-tuning on a validation set for new datasets or datasets with very different gene/TF counts
- The causal graph underlying an observational dataset is not necessarily unique; verifying the imposed GRN is the most probable one is out of scope
- The model only generates library-size normalized expression; unnormalized count generation is planned for future iterations

## Evidence

- [intro] causal architecture built on CausalGAN: "a causal implicit generative model for reference-based GRN-guided simulation of scRNA-seq data inspired by CausalGAN31"
- [methods] training command entry point: "--train          Start or resume model training"
- [readme] README expansion of the tool name: "_**GR**N-guided in silic**o** sim**u**lation of single-cell R**N**A-seq **d**ata using Causal **G**enerative **A**dversarial **N**etworks_"
- [abstract] model purpose from abstract: "We introduce GRouNdGAN, a gene regulatory network (GRN)-guided reference-based causal implicit generative model for simulating single-cell RNA-seq data"
