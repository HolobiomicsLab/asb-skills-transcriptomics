---
name: single-cell-gan-generator-architecture
description: Use when building or modifying the generator of a single-cell GAN (scGAN/cscGAN)
  so that generated cells carry realistic library sizes, or when validating that a
  normalization layer is differentiable in TensorFlow.
when_to_use_negative:
- Input data is not single-cell RNA-seq count data to be generated/augmented (e.g.
  bulk RNA-seq, proteomics) — cscGAN application to other domains is only speculative
- The goal is manifold-based data generation rather than adversarial generation with
  library-size control — use SUGAR (diffusion-kernel-based generation) instead
- You need a trained model off-the-shelf with no generator modification; then use
  main.py --generate directly rather than reimplementing the architecture
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3474
tools:
- name: scGAN
  role: Base single-cell GAN model whose generator receives the LSN output layer
  repo: https://github.com/imsb-uke/scGAN
- name: cscGAN
  role: Conditional variant of scGAN in which the same generator architecture is conditioned
    to produce defined cell types
  repo: https://github.com/imsb-uke/scGAN
- name: TensorFlow
  role: Framework used to implement the LSN layer as a differentiable operation and
    to compute row-wise reduction sums for verification
- name: Python 3.5
  role: Runtime environment in which the LSN layer implementation is tested
- name: Docker
  role: Provides the reproducible environment with all dependencies for running the
    scGAN code
  repo: https://github.com/imsb-uke/scGAN
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-019-14018-z
    title: Realistic in silico generation and augmentation of single-cell RNA-seq
      data using generative adversarial networks
  source_content_hash: sha256:f69f550a5e02f57947bbf6ee96dedf271dbcb397393ce1101f0fe5e461145edb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/imsb-uke/scGAN
  tools:
  - Docker
  - Python
  - Python 3.5
  - TensorFlow
  - cscGAN
  - scGAN
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# single-cell-gan-generator-architecture

## Summary

Implements the scGAN/cscGAN generator architecture from Marouf et al., including the custom library-size normalization (LSN) output layer that rescales non-negative ReLU activations to a fixed per-cell library size, so generated scRNA-seq cells have realistic total read counts.

## When to use

Use when building or modifying the generator of a single-cell GAN (scGAN/cscGAN) so that generated cells carry realistic library sizes, or when validating that a normalization layer is differentiable in TensorFlow.

## When NOT to use

- Input data is not single-cell RNA-seq count data to be generated/augmented (e.g. bulk RNA-seq, proteomics) — cscGAN application to other domains is only speculative
- The goal is manifold-based data generation rather than adversarial generation with library-size control — use SUGAR (diffusion-kernel-based generation) instead
- You need a trained model off-the-shelf with no generator modification; then use main.py --generate directly rather than reimplementing the architecture

## Inputs

- Non-negative generator activation tensor (mimicking ReLU output) of shape (cells x genes)
- parameters. file defining model type (scGAN or cscGAN), layer sizes, normalization (batchnorm or layernorm), lambda, batch size, latent space dimension, and optimizer parameters
- Pre-processed training data in TFRecord format derived from .h5/.h5ad scRNA-seq inputs (e.g. Fresh 68k PBMCs Donor A)

## Outputs

- Differentiable TensorFlow LSN layer whose output rows each sum to phi = 20,000
- Generated single-cell RNA-seq expression vectors with realistic per-cell library sizes (saved via --generate to .h5ad)

## How to apply

Build the generator as a TensorFlow network whose final non-negative ReLU activations are passed through the LSN layer (equations 4-5): divide each cell's gene-expression vector by its row-wise sum and multiply by the target library size phi = 20,000, matching the total read counts of the training data. Implement LSN as a differentiable TensorFlow operation (Python 3.5) so gradients flow through the normalization during adversarial training. Set layer sizes, normalization type (batchnorm or layernorm), lambda, latent space dimension, and optimizer settings in the parameters. file, choosing scGAN or cscGAN via the model type flag. Feed a randomly initialized non-negative test tensor mimicking generator ReLU output through the layer and compute row-wise sums with TensorFlow reduction operations to verify every output row sums to phi within a relative error < 1e-6.

## Related tools

- **scGAN** (Base single-cell GAN model whose generator receives the LSN output layer) — https://github.com/imsb-uke/scGAN
- **cscGAN** (Conditional variant of scGAN in which the same generator architecture is conditioned to produce defined cell types) — https://github.com/imsb-uke/scGAN
- **TensorFlow** (Framework used to implement the LSN layer as a differentiable operation and to compute row-wise reduction sums for verification)
- **Python 3.5** (Runtime environment in which the LSN layer implementation is tested)
- **Docker** (Provides the reproducible environment with all dependencies for running the scGAN code) — https://github.com/imsb-uke/scGAN

## Examples

```
import tensorflow as tf; lsn_out = phi * gen_relu / tf.reduce_sum(gen_relu, axis=1, keepdims=True); row_sums = tf.reduce_sum(lsn_out, axis=1)  # assert allclose(row_sums, 20000)
```

## Evaluation signals

- Every row of the LSN layer output sums to the target library size phi = 20,000 within floating-point tolerance (relative error < 1e-6), verified with TensorFlow reduction operations
- The LSN operation is differentiable: gradients propagate through the normalization during generator training without errors
- Tensorboard loss curves evolve smoothly during training and checkpointing/resuming from the .-specified checkpoint works
- t-SNE validation plots computed on the whole validation set plus an equal number of generated cells show generated cells overlapping the real cell distribution
- Generated cells written via --generate load correctly in .h5ad and match the requested per-cluster counts given to --cells_no

## Limitations

- The mechanism enabling cscGAN to generate cells from very small subpopulations (e.g. 16 cells) is only speculated, not established
- Conditioning on side information other than cell type (e.g. case/control samples) is left untested for future work
- Code is tested with Python 3.5 and the packages in requirements.txt; other environments may miss essential dependencies

## Evidence

- [other] The LSN is a custom function added to the scGAN's generator so that the generator explicitly outputs generated cells with a total read count equal to that of the training data.: [evidence span withheld — anchor status: not_found]
- [other] Implement the LSN layer as a differentiable TensorFlow operation in Python 3.5, dividing each cell's gene-expression vector by its row sum and multiplying by phi = 20,000.: [evidence span withheld — anchor status: not_found]
- [other] verify numerically that every output row sums to the target library size phi = 20,000 within floating-point tolerance (relative error < 1e-6): [evidence span withheld — anchor status: not_found]
- [readme] The code was tested with Python 3.5 and the packages listed in requirements.txt.: "The code was tested with Python 3.5 and the packages listed in requirements.txt."
- [readme] The type of model (scGAN or cscGAN), the size of the different layers, the normalization layer used to condition the generation (batchnorm or layernorm), lambda, the batch size, dimension of the latent space: "The type of model (scGAN or cscGAN), the size of the different layers, the normalization layer used to condition the generation (batchnorm or layernorm)"
- [abstract] cscGAN learns non-linear gene-gene dependencies from complex, multiple cell type samples and uses this information to generate realistic cells of defined types: "cscGAN learns non-linear gene-gene dependencies from complex, multiple cell type samples and uses this information to generate realistic cells of defined types"
