---
name: tensorflow-graph-session-execution
description: Use when implementing or reproducing a multi-loss GAN (adversarial +
  cycle + correspondence) in TensorFlow 1.x-style graph/session mode where generator
  and discriminator must be optimized in a strict per-iteration order with logged
  diagnostics.
when_to_use_negative:
- You need eager execution / interactive debugging rather than a pre-built static
  graph executed via sess.run
- You are aligning more than two modalities — the number of required generators grows
  quadratically, so the networks would have to be made small or trained separately
- You only need distributional alignment without point-wise geometry preservation;
  the correspondence/geometry loss machinery (and its choice of point distance) is
  not plug-and-play
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_3308
tools:
- name: scMMGAN
  role: The model whose graph (losses, train_op_G, train_op_D, training loop) is constructed
    and executed in TensorFlow; repo hosts the implementation script
  repo: github.com/KrishnaswamyLab/scMMGAN
- name: TensorFlow
  role: Framework used for graph construction and sess.run-based session execution
    of generator-then-discriminator optimization
- name: MAGIC
  role: Manifold-smoothing dropout correction applied upstream so the x1/x2 tensors
    fed into the graph are denoised before network training
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.patter.2022.100577
    title: Single-cell multi-modal GAN reveals spatial patterns in single-cell data
      from triple-negative breast cancer
  source_content_hash: sha256:02c725679123e4c6a3d118913d93fda51bbd22c921dc498cb7a472e9ba39a098
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/KrishnaswamyLab/scMMGAN
  tools:
  - MAGIC
  - TensorFlow
  - scMMGAN
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# tensorflow-graph-session-execution

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Structures scMMGAN training as a TensorFlow graph built in advance (losses, optimizers) and executed inside a tf.Session, so that generator optimization (train_op_G) runs before discriminator optimization (train_op_D) each iteration with periodic loss sanity checks.

## When to use

Use when implementing or reproducing a multi-loss GAN (adversarial + cycle + correspondence) in TensorFlow 1.x-style graph/session mode where generator and discriminator must be optimized in a strict per-iteration order with logged diagnostics.

## When NOT to use

- You need eager execution / interactive debugging rather than a pre-built static graph executed via sess.run
- You are aligning more than two modalities — the number of required generators grows quadratically, so the networks would have to be made small or trained separately
- You only need distributional alignment without point-wise geometry preservation; the correspondence/geometry loss machinery (and its choice of point distance) is not plug-and-play

## Inputs

- Preprocessed modality matrices x1 and x2 (dropout-corrected with MAGIC, zero-centered/unit-scaled, reduced to 50 principal components, columns mostly in [-10, 10])
- Hyperparameters: TRAINING_STEPS, batch_size (256), nfilt, learning_rate, lambda_cycle, lambda_correspondence (=10), add_noise, use_bn
- Diffusion transition probability matrices Px1 and PG(x1) and their first 10-20 eigenvectors for the correspondence loss

## Outputs

- Trained generator mappings: output_fake2 (x1 mapped into the second domain) and output_fake (x2 mapped into the first domain)
- Per-step loss values logged every 100 training steps via training_counter for sanity checks

## How to apply

Define the full model in the TensorFlow graph: preprocess x1/x2 minibatches of size 256 (MAGIC-corrected, zero-centered, unit-scaled, 50 principal components), then build the correspondence loss Lc as the L2 distance between rescaled diffusion coordinates of x1 and G(x1), with eigenvector sign-correlation check (flip phi_G(x) when correlation < epsilon_r = -0.5) inside the graph. Set the total generator loss LG1 = lambda_r*Lr + lambda_d*Ld + lambda_c*Lc with lambda_r = lambda_d = 1 and lambda_c = 10, and run lines through the sess.run call (line 421 in the scMMGAN script) to prepare the graph without starting training. In the training while loop, execute one iteration per step: sess.run(train_op_G) first to optimize the generator, then sess.run(train_op_D) to optimize the discriminator. Use the training_counter variable to recompute helpful loss values every 100 steps for sanity checks; TRAINING_STEPS can be chosen based on when generated output is stable across training iterations.

## Related tools

- **scMMGAN** (The model whose graph (losses, train_op_G, train_op_D, training loop) is constructed and executed in TensorFlow; repo hosts the implementation script) — github.com/KrishnaswamyLab/scMMGAN
- **TensorFlow** (Framework used for graph construction and sess.run-based session execution of generator-then-discriminator optimization)
- **MAGIC** (Manifold-smoothing dropout correction applied upstream so the x1/x2 tensors fed into the graph are denoised before network training)

## Examples

```
for step in range(TRAINING_STEPS): sess.run(train_op_G, feed_dict={x1: bx1, x2: bx2}); sess.run(train_op_D, feed_dict={x1: bx1, x2: bx2})
```

## Evaluation signals

- correspondence_loss.py executes within the TensorFlow session on real minibatches (script_runs) without errors through the sess.run call at line 421
- Reported loss values are finite, non-negative, and consistent with lambda_c = 10 weighting of the L2 eigenvector-distance term
- Per-iteration order is verified: train_op_G executes before train_op_D in each while-loop step
- Loss values logged every 100 steps via training_counter show decreasing/stable behavior; training stops when generated output is stable across iterations
- output_fake2 contains generated data of mapping x1 into the second domain, with any input scaling inverted on output

## Limitations

- GANs are difficult to train because of adversarial losses, which can lead to instability; the model can deteriorate from effective to ineffective quickly across training iterations
- Mode collapse can occur because points are not penalized by distribution-level losses to match the entire distribution; early stopping and the geometric loss only informally mitigate these effects
- Changing learning rates has rarely been effective observationally; batch_size and nfilt should be increased based on computational resources
- The geometry-based loss is not completely plug-and-play: a choice of distance between data points is still required

## Evidence

- [methods] set the total generator loss with lambda_c = 10: [evidence span withheld — anchor status: not_found]
