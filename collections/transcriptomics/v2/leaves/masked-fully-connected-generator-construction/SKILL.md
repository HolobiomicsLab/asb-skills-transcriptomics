---
name: masked-fully-connected-generator-construction
description: Use when building the GRouNdGAN/CausalGAN generator stack — instantiating
  per-target-gene masked generators from a pickled causal graph and pretraining the
  controller before the 1,000,000-step Wasserstein GAN training run.
when_to_use_negative:
- The GRN you want to impose is multi-layer (TF-gene-phenotype causal graph) — the
  current version only supports bipartite GRNs
- You need unnormalized count data — the model currently only generates library-size
  normalized expression values
- You need a predictive model of knockout outcomes for a new dataset rather than a
  generative model — GRouNdGAN must be retrained for new data and is not predictive
edam_operation: http://edamontology.org/operation_0242
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: GRouNdGAN
  role: Implements the masked causal generator, critic, labeler/anti-labeler, and
    library-size normalization layer used in this skill
  repo: https://github.com/Emad-COMBINE-lab/GRouNdGAN
- name: CausalGAN
  role: 'Architectural basis: GRouNdGAN''s controller/target-generator design builds
    on the causal GAN architecture of CausalGAN'
- name: GRNBoost2
  role: Infers the GRN from preprocessed data that is formatted as the causal graph
    masking generator connectivity
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-024-48516-6
    title: 'GRouNdGAN: GRN-guided simulation of single-cell RNA-seq data using causal
      generative adversarial networks'
  source_content_hash: sha256:0409e461c0186b8b0bfd2c74e8bab09c5d5d3fbf5254e85235461f9a43ff771c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Emad-COMBINE-lab/GRouNdGAN
  tools:
  - CausalGAN
  - GRNBoost2
  - GRouNdGAN
  - scGAN
  edam_operation: http://edamontology.org/operation_0242
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# Masked fully-connected generator construction

## Summary

Constructs the causal core of GRouNdGAN: a causal controller plus one fully connected target generator per target gene, whose input connectivity is masked by a user-provided causal GRN so simulated scRNA-seq expression obeys the imposed TF-gene regulatory structure.

## When to use

Use when building the GRouNdGAN/CausalGAN generator stack — instantiating per-target-gene masked generators from a pickled causal graph and pretraining the controller before the 1,000,000-step Wasserstein GAN training run.

## When NOT to use

- The GRN you want to impose is multi-layer (TF-gene-phenotype causal graph) — the current version only supports bipartite GRNs
- You need unnormalized count data — the model currently only generates library-size normalized expression values
- You need a predictive model of knockout outcomes for a new dataset rather than a generative model — GRouNdGAN must be retrained for new data and is not predictive

## Inputs

- Preprocessed reference scRNA-seq dataset (output of the --preprocess step)
- Pickled causal graph (nested dict of gene index -> set of regulating TF indices), from --create_grn (GRNBoost2) or a custom imposed GRN
- Configuration file with generator/critic/labeler architecture and training hyperparameters

## Outputs

- Pretrained causal controller and trained GRouNdGAN checkpoints (.pth files, e.g. step_1000000.pth in checkpoints/)
- Library-size normalized (library size = 20000) simulated scRNA-seq expression values
- TensorBoard training summaries and real-vs-simulated t-SNE plots every 10000 steps

## How to apply

Instantiate the masked causal generator as one generator per target gene with noise per gene = 1, depth per gene = 3, width per gene = 2, and connectivity fixed by the pickled causal graph (nested dict of gene index -> set of regulating TF indices), so each target generator receives only a noise value and the simulated expression of its regulating TFs. Pretrain the causal controller as a non-conditional scRNA-seq GAN (generator layers 256 512 1024, latent dim 128, lambda 10) for up to 200000 steps, batch size 128, 5 critic iterations, Adam betas 0.5/0.9, learning rates 0.0001 decaying to 0.00001. Instantiate the Wasserstein critic with layers 1024 512 256 and the labeler/anti-labeler with layers 2000 2000 2000, and append a library-size normalization layer (library size = 20000) as the model output. Train the main causal GAN for up to 1000000 steps, batch size 1024, 5 critic iterations, gradient-penalty lambda 10, betas 0.5/0.9, generator learning rate decayed from 0.001 to 0.0001. Log with TensorBoard every 10000 steps and checkpoint to checkpoints/ every 100000 steps. The rationale is that masking enforces the user-defined bipartite GRN directly in the generator architecture rather than as a soft penalty.

## Related tools

- **GRouNdGAN** (Implements the masked causal generator, critic, labeler/anti-labeler, and library-size normalization layer used in this skill) — https://github.com/Emad-COMBINE-lab/GRouNdGAN
- **CausalGAN** (Architectural basis: GRouNdGAN's controller/target-generator design builds on the causal GAN architecture of CausalGAN)
- **GRNBoost2** (Infers the GRN from preprocessed data that is formatted as the causal graph masking generator connectivity)

## Examples

```
git clone https://github.com/Emad-COMBINE-lab/GRouNdGAN.git && docker pull yazdanz/groundgan:4b98686
```

## Evaluation signals

- Training terminates with the message 'Finished training'
- Final checkpoint step_1000000.pth is saved under the checkpoints/ directory
- TensorBoard summaries and real-vs-simulated t-SNE plots appear at the 10000-step summary/plot frequency
- Each target generator's inputs match exactly the noise value plus the regulating TF indices in the pickled causal graph (mask correctness check)
- Simulated output is library-size normalized to 20000 across the considered genes and TFs

## Limitations

- Current version only imposes bipartite GRNs; multi-layer GRNs and TF-gene-phenotype causal graphs are unsupported and planned for future versions
- Hyperparameters may need re-tuning on a validation set for new datasets or datasets with very different gene/TF counts
- Resemblance to the reference dataset deteriorates if the imposed TF-gene relationships differ significantly from patterns consistent with the reference data
- GRouNdGAN is generative, not predictive; the model must be retrained for a new dataset

## Evidence

- [abstract] Causal controller and target generators: "We introduce GRouNdGAN, a gene regulatory network (GRN)-guided reference-based causal implicit generative model for simulating single-cell RNA-seq data"
- [methods] Masked generator instantiation parameters: [evidence span withheld — anchor status: not_found]
- [methods] Critic, labeler, and normalization layer: [evidence span withheld — anchor status: not_found]
- [discussion] Bipartite-only restriction: "The current version of GRouNdGAN only allows imposing a bipartite GRN and does not support a multi-layer GRN, an assumption that we plan to relax"
- [readme] Model full name: "**GR**N-guided in silic**o** sim**u**lation of single-cell R**N**A-seq **d**ata using Causal **G**enerative **A**dversarial **N**etworks"
