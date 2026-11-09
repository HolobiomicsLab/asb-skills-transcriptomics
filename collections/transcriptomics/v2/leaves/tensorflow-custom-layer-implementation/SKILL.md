---
name: tensorflow-custom-layer-implementation
description: Use when a generative model for scRNA-seq must output expression vectors
  constrained to a target per-cell library size, and the architecture requires a custom
  differentiable layer rather than an off-the-shelf normalization op.
when_to_use_negative:
- Input expression vectors are already library-size normalized or the downstream analysis
  does not require a fixed total read count per generated cell
- The model is not built in TensorFlow (e.g. a Matlab or other-framework implementation
  such as SUGAR) and the op cannot be made differentiable in the training graph
tools:
- name: TensorFlow
  role: Framework in which the LSN layer is implemented as a differentiable operation
    (division by row sum, multiplication by phi)
  repo: https://github.com/imsb-uke/scGAN
- name: scGAN
  role: Generator architecture into which the custom LSN output layer is inserted
  repo: https://github.com/imsb-uke/scGAN
- name: Python 3.5
  role: Language version in which the custom layer code is written and tested
  repo: https://github.com/imsb-uke/scGAN
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-019-14018-z
    title: Realistic in silico generation and augmentation of single-cell RNA-seq
      data using generative adversarial networks
  source_content_hash: sha256:330918daaf074fe91e1f5c53dff79be1e6f1db81e2f54aa85db1398fd16ac7d8
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
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# tensorflow-custom-layer-implementation

## Summary

Implements a custom, differentiable TensorFlow layer — here the library-size normalization (LSN) output layer of the scGAN/cscGAN generator — so the generator explicitly emits cells with a fixed total read count (phi = 20,000), matching the training data.

## When to use

Use when a generative model for scRNA-seq must output expression vectors constrained to a target per-cell library size, and the architecture requires a custom differentiable layer rather than an off-the-shelf normalization op.

## When NOT to use

- Input expression vectors are already library-size normalized or the downstream analysis does not require a fixed total read count per generated cell
- The model is not built in TensorFlow (e.g. a Matlab or other-framework implementation such as SUGAR) and the op cannot be made differentiable in the training graph

## Inputs

- Non-negative tensor mimicking generator ReLU output (cells x genes)
- Target per-cell library size phi = 20,000
- scGAN generator architecture definition in the imsb-uke/scGAN TensorFlow codebase

## Outputs

- Row-normalized generated cell-expression tensor with every row summing to phi = 20,000
- Numerical verification record of row-wise sums (relative error < 1e-6)

## How to apply

Read the LSN layer definition (equations 4-5), which rescales the non-negative ReLU activations of the generator to a fixed per-cell library size phi. Implement the layer as a differentiable TensorFlow operation in Python 3.5 by dividing each cell's gene-expression vector by its row sum and multiplying by phi = 20,000. Feed a randomly initialized non-negative test tensor (mimicking generator ReLU output) through the layer and compute row-wise sums with TensorFlow reduction operations. Validate numerically that every output row sums to phi = 20,000 within floating-point tolerance (relative error < 1e-6). The layer must remain differentiable so gradients flow through the generator during (c)scGAN adversarial training.

## Related tools

- **TensorFlow** (Framework in which the LSN layer is implemented as a differentiable operation (division by row sum, multiplication by phi)) — https://github.com/imsb-uke/scGAN
- **scGAN** (Generator architecture into which the custom LSN output layer is inserted) — https://github.com/imsb-uke/scGAN
- **Python 3.5** (Language version in which the custom layer code is written and tested) — https://github.com/imsb-uke/scGAN

## Examples

```
lsn = tf.multiply(tf.div(x, tf.reduce_sum(x, reduction_indices=1, keep_dims=True)), 20000.0); sums = tf.reduce_sum(lsn, reduction_indices=1)
```

## Evaluation signals

- Row-wise sums of the layer output equal phi = 20,000 within floating-point tolerance (relative error < 1e-6)
- Output remains non-negative (layer applied to non-negative ReLU activations)
- Gradients propagate through the layer (operation is differentiable with respect to its input tensor)
- Test with a randomly initialized non-negative tensor of arbitrary shape reproduces exact row sums, confirming shape-independence

## Limitations

- The layer assumes non-negative (ReLU) generator activations; applying it to signed inputs would break the library-size semantics
- The reference implementation targets Python 3.5 with the pinned TensorFlow version listed in requirements.txt; other versions are untested
- Equation 4-5 rescaling fixes output to a single constant phi = 20,000; it does not preserve per-cell variability of library sizes

## Evidence

- [full_text] The LSN is a custom function added to the scGAN's generator so that the generator explicitly outputs generated cells with a total read count equal to that of the training data.: [evidence span withheld — anchor status: not_found]
- [methods] The code was tested with Python 3.5 and the packages listed in requirements.txt.: "The code was tested with Python 3.5 and the packages listed in requirements.txt."
- [readme] This project contains the Tensorflow implementation and documentation for the training of the (c)scGAN models: "This project contains the Tensorflow implementation and documentation for the training of the (c)scGAN models described in our manuscript"
