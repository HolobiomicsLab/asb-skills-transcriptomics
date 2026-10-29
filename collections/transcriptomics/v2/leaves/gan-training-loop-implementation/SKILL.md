---
name: gan-training-loop-implementation
description: Use when training a GAN-based single-cell modality alignment model (e.g.
  scMMGAN) and you need to structure the alternating generator/discriminator optimization
  session loop with periodic loss monitoring.
when_to_use_negative:
- When the goal is to train a GAN on very many input modalities — the number of required
  generators grows quadratically, forcing small or separately trained networks
- When the data columns are far outside roughly [-10, 10] and preprocessing cannot
  bring them there — the README states best results require this range
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3474
tools:
- name: scMMGAN
  role: The GAN model whose generator/discriminator alternating training loop is implemented
  repo: github.com/KrishnaswamyLab/scMMGAN
- name: TensorFlow
  role: Framework for graph construction, sess.run session loop, train_op_G and train_op_D
    optimization ops
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.patter.2022.100577
    title: Single-cell multi-modal GAN reveals spatial patterns in single-cell data
      from triple-negative breast cancer
  source_content_hash: sha256:e61778408a41cdb59459fcdd986a5af1b48b2c6cd439163bd29049c2f9c2d8a5
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/KrishnaswamyLab/scMMGAN
  tools:
  - CMAE
  - Chromium 10X
  - CycleGAN
  - TensorFlow
  - scMMGAN
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# gan-training-loop-implementation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Implements the scMMGAN TensorFlow training loop in which the generator (train_op_G) and discriminator (train_op_D) are alternately optimized each iteration, with periodic loss sanity checks and extraction of cross-domain mappings (output_fake, output_fake2).

## When to use

Use when training a GAN-based single-cell modality alignment model (e.g. scMMGAN) and you need to structure the alternating generator/discriminator optimization session loop with periodic loss monitoring.

## When NOT to use

- When the goal is to train a GAN on very many input modalities — the number of required generators grows quadratically, forcing small or separately trained networks
- When the data columns are far outside roughly [-10, 10] and preprocessing cannot bring them there — the README states best results require this range

## Inputs

- Preprocessed single-cell datasets x1 and x2 as numpy arrays (e.g. MAGIC-corrected, zero-centered/unit-scaled, reduced to 50 PCs) with columns mostly in [-10, 10]
- Hyperparameters: TRAINING_STEPS, batch_size, nfilt, learning_rate, lambda_cycle, lambda_correspondence, add_noise, use_bn

## Outputs

- output_fake: generated data mapping x2 into the first domain
- output_fake2: generated data mapping x1 into the second domain
- Periodic sanity-check loss reports every 100 training steps

## How to apply

Build the TensorFlow graph (variable definitions, loss functions, and ops) before training; the sess.run call prepares the code without starting training. In the while-loop, each iteration first optimizes the generator with train_op_G and then the discriminator with train_op_D. Report sanity-check loss values every 100 steps using a training_counter variable, and optionally plot or run additional checks intermittently. Use a learning rate of 0.0001 and batch size 256 (for the TNBC xenograft task), with lambda_correspondence=10 and lambda_cycle=1. Stop training when the generated output is stable across iterations; afterwards read output_fake2 (mapping x1 into the second domain) and output_fake (mapping x2 into the first domain), inverting any preprocessing scaling to return data to its original form.

## Related tools

- **scMMGAN** (The GAN model whose generator/discriminator alternating training loop is implemented) — github.com/KrishnaswamyLab/scMMGAN
- **TensorFlow** (Framework for graph construction, sess.run session loop, train_op_G and train_op_D optimization ops)

## Examples

```
for i in range(TRAINING_STEPS): sess.run(train_op_G); sess.run(train_op_D)
```

## Evaluation signals

- Sanity-check losses reported every 100 steps should show the generator and discriminator losses evolving without the model deteriorating from effective to ineffective
- Generated output should be stable across training iterations before stopping (per README: further training is unnecessary once stable)
- output_fake and output_fake2 should map x2→domain 1 and x1→domain 2 respectively, with any preprocessing scaling inverted
- For the TNBC xenograft task, resulting mappings should yield per-sample MSE ~0.87–1.03 for scMMGAN vs ~1.6–2.1 for CycleGAN/CMAE baselines
- Altered batch_size and nfilt upward on larger computational resources should generally improve results per the README guidance

## Limitations

- GANs are difficult to train because adversarial losses can lead to instability; the model can deteriorate from effective to ineffective quickly across training iterations
- GANs often suffer from mode collapse because they are not penalized by distribution-level losses; early stopping and the geometric loss only informally mitigate these effects
- Changing learning rates has rarely been effective observationally, so tuning should focus on lambda_cycle, lambda_correspondence, batch_size, and nfilt
- Early stopping is used as regularization, but instability and mode collapse may still be present in some contexts

## Evidence
