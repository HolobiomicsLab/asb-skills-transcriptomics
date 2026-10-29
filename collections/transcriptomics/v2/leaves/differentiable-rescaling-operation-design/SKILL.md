---
name: differentiable-rescaling-operation-design
description: Use when a generative model (e.g. scGAN/cscGAN generator) must explicitly
  output expression vectors with a fixed total read count, and the rescaling must
  remain differentiable for end-to-end backpropagation in TensorFlow.
when_to_use_negative:
- Input vectors contain negative values (LSN assumes non-negative generator/ReLU activations;
  row-sum rescaling is undefined for them)
- The analysis goal is library-size normalization of real count matrices during pre-processing
  rather than a trainable generator output layer (use standard scanpy/Seurat normalization
  instead)
- Downstream model expects unnormalized latent activations (e.g. debugging raw generator
  logits before the LSN layer)
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3474
tools:
- name: TensorFlow
  role: Framework in which the LSN layer is implemented as a differentiable operation
    (division by row sum, multiplication by phi) using reduction ops
  repo: https://github.com/imsb-uke/scGAN
- name: Python 3.5
  role: Language the LSN implementation and its numerical validation were tested with
  repo: https://github.com/imsb-uke/scGAN
- name: scGAN
  role: Generator architecture whose output layer the LSN rescaling operation augments
  repo: https://github.com/imsb-uke/scGAN
- name: cscGAN
  role: Conditional variant of the generator that inherits the LSN output layer
  repo: https://github.com/imsb-uke/scGAN
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-019-14018-z
    title: Realistic in silico generation and augmentation of single-cell RNA-seq
      data using generative adversarial networks
  source_content_hash: sha256:e4b8d923e85dacf975f488e7c5a365680132652538ebfff91c223d23bd3ed3c2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/imsb-uke/scGAN
  tools:
  - Python
  - Python 3.5
  - TensorFlow
  - cscGAN
  - scGAN
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# differentiable-rescaling-operation-design

## Summary

Design and validate a differentiable library-size normalization (LSN) layer that rescales a generator's non-negative ReLU activations so each generated scRNA-seq cell vector sums to a fixed library size phi (20,000 reads), implemented as TensorFlow operations.

## When to use

Use when a generative model (e.g. scGAN/cscGAN generator) must explicitly output expression vectors with a fixed total read count, and the rescaling must remain differentiable for end-to-end backpropagation in TensorFlow.

## When NOT to use

- Input vectors contain negative values (LSN assumes non-negative generator/ReLU activations; row-sum rescaling is undefined for them)
- The analysis goal is library-size normalization of real count matrices during pre-processing rather than a trainable generator output layer (use standard scanpy/Seurat normalization instead)
- Downstream model expects unnormalized latent activations (e.g. debugging raw generator logits before the LSN layer)

## Inputs

- Non-negative test tensor (e.g. randomly initialized) mimicking generator ReLU output, shape (cells x genes)
- Target library size phi (20,000 per cell)

## Outputs

- Row-normalized expression tensor where each cell's gene-expression vector sums exactly to phi = 20,000
- Row-wise sum verification results (per-row sums vs. target, relative error < 1e-6)

## How to apply

Read the LSN layer definition (equations 4-5) which rescales the non-negative ReLU activations of the generator to a fixed per-cell library size phi. Implement the layer as a differentiable TensorFlow operation in Python 3.5 by dividing each cell's gene-expression vector by its row sum and multiplying by phi = 20,000; using only differentiable TF reduction and division ops keeps gradients flowing through the generator during GAN training. Feed a randomly initialized non-negative test tensor mimicking generator ReLU output through the layer, then compute row-wise sums of the output with TensorFlow reduction operations. Validate numerically that every output row sums to phi = 20,000 within floating-point tolerance (relative error < 1e-6) and record the verification results. This layer exists so the generator explicitly emits cells whose total read count matches that of the training data.

## Related tools

- **TensorFlow** (Framework in which the LSN layer is implemented as a differentiable operation (division by row sum, multiplication by phi) using reduction ops) — https://github.com/imsb-uke/scGAN
- **Python 3.5** (Language the LSN implementation and its numerical validation were tested with) — https://github.com/imsb-uke/scGAN
- **scGAN** (Generator architecture whose output layer the LSN rescaling operation augments) — https://github.com/imsb-uke/scGAN
- **cscGAN** (Conditional variant of the generator that inherits the LSN output layer) — https://github.com/imsb-uke/scGAN

## Examples

```
lsn_out = lsn_layer(relu_activations, phi=20000); row_sums = tf.reduce_sum(lsn_out, axis=1); assert tf.reduce_max(tf.abs(row_sums - 20000)/20000) < 1e-6
```

## Evaluation signals

- Every output row of the LSN layer sums to phi = 20,000 within floating-point tolerance (relative error < 1e-6)
- Row-wise sums computed via tf.reduce_sum match the target library size for all cells in the test tensor
- The layer is composed solely of differentiable TensorFlow ops, so gradients propagate through it during backpropagation
- Output values remain non-negative (inherited from the ReLU activations being rescaled, not sign-flipped)

## Limitations

- The layer assumes non-negative inputs; it is designed to sit atop the generator's ReLU activations, not arbitrary real-valued tensors
- Validation depends on floating-point tolerance; row sums can deviate slightly from exactly 20,000 at machine precision
- The mechanism of downstream generative behavior is partially speculative — the paper only speculates why cscGAN can generate cells from very small subpopulations (e.g. 16 cells)

## Evidence

- [methods] The LSN is a custom function added to the scGAN's generator so that the generator explicitly outputs generated cells with a total read count equal to that of the training data.: [evidence span withheld — anchor status: not_found]
- [readme] The code was tested with Python 3.5 and the packages listed in requirements.txt.: "The code was tested with Python 3.5 and the packages listed in requirements.txt."
- [readme] This project contains the Tensorflow implementation and documentation for the training of the (c)scGAN models: "This project contains the Tensorflow implementation and documentation for the training of the (c)scGAN models described in our manuscript"
