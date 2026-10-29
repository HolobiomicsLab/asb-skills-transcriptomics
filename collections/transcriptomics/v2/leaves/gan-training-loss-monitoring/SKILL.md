---
name: gan-training-loss-monitoring
description: Use when training an scGAN/cscGAN model on .h5/.h5ad scRNA-seq data in
  TensorFlow and you need to verify stable adversarial convergence (via loss curves)
  before generating cells or accepting t-SNE validation plots.
when_to_use_negative:
- Input data has not yet been pre-processed into TFRecord format via the '--process'
  flag — monitoring presumes an active training run
- The model is not adversarial (e.g. a VAE baseline); generator/discriminator loss
  curves do not apply
- You only need final generation quality metrics (MMD, classifier AUC) and training
  already converged — direct evaluation of generated cells is more informative
edam_operation: http://edamontology.org/operation_0564
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3474
tools:
- name: Tensorboard
  role: Visualizes the evolution of generator/discriminator losses and other summary
    quantities during (c)scGAN training
  repo: https://github.com/imsb-uke/scGAN
- name: scGAN
  role: The (conditional) single-cell GAN whose training losses are monitored; implemented
    in TensorFlow
  repo: https://github.com/imsb-uke/scGAN
- name: TensorFlow
  role: Framework providing the summary operations logged to Tensorboard during training
  repo: https://github.com/imsb-uke/scGAN
- name: Docker
  role: Provides the containerized environment (scgan:latest image) in which training
    and Tensorboard run reproducibly
  repo: https://github.com/imsb-uke/scGAN
provenance:
  source_task_ids:
  - task_001
  - task_007
  source_papers:
  - doi: 10.1038/s41467-019-14018-z
    title: Realistic in silico generation and augmentation of single-cell RNA-seq
      data using generative adversarial networks
  source_content_hash: sha256:5e15460b109ecaf2a3bb1d65136c88898f6e9afe3333b94f9a335b6cf4ac5f16
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/imsb-uke/scGAN
  tools:
  - Docker
  - Python
  - TensorFlow
  - Tensorboard
  - cscGAN
  - scGAN
  edam_operation: http://edamontology.org/operation_0564
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# GAN training loss monitoring with Tensorboard

## Summary

Monitors the evolution of generator and discriminator losses during (c)scGAN training on scRNA-seq data using Tensorboard summary operations, confirming that GAN training converges stably before cells are generated or validated.

## When to use

Use when training an scGAN/cscGAN model on .h5/.h5ad scRNA-seq data in TensorFlow and you need to verify stable adversarial convergence (via loss curves) before generating cells or accepting t-SNE validation plots.

## When NOT to use

- Input data has not yet been pre-processed into TFRecord format via the '--process' flag — monitoring presumes an active training run
- The model is not adversarial (e.g. a VAE baseline); generator/discriminator loss curves do not apply
- You only need final generation quality metrics (MMD, classifier AUC) and training already converged — direct evaluation of generated cells is more informative

## Inputs

- parameters. file with logging frequencies, model hyperparameters (model type scGAN/cscGAN, layer sizes, batchnorm/layernorm, lambda, batch size, latent-space dimension, optimizer settings) and optional checkpoint path
- TFRecord-format preprocessed scRNA-seq training data (from .h5/.h5ad input)
- Tensorboard event logs written by the training run

## Outputs

- Tensorboard loss curves and summary scalars for the scGAN/cscGAN generator and discriminator
- Periodic t-SNE validation plots (whole validation set vs. an equal number of generated cells)
- Model checkpoints saved at the configured save frequency

## How to apply

After launching training with 'python main.py --param parameters. --train' (or jointly with '--process'), define in parameters. the logging frequencies controlling how often values are written for Tensorboard, how often t-SNE validation plots are produced, how often losses are printed to stdout, and how often the model checkpoint is saved. Open Tensorboard during or after training to inspect the summary operations defined on the most important model quantities, chiefly the generator and discriminator losses. Treat steadily evolving, non-diverging loss curves as evidence that the adversarial training is stable; unstable or diverging losses indicate hyperparameter problems (e.g. learning rate, batch size, lambda, latent-space dimension) that should be fixed in parameters. before proceeding. Training can be resumed from a checkpoint path specified in the . file if it was interrupted. Only once losses have stabilized should you accept the qualitative validation criterion: the t-SNE embedding of generated cells (computed on the whole validation set plus an equal number of generated cells) overlapping the validation-set cells.

## Related tools

- **Tensorboard** (Visualizes the evolution of generator/discriminator losses and other summary quantities during (c)scGAN training) — https://github.com/imsb-uke/scGAN
- **scGAN** (The (conditional) single-cell GAN whose training losses are monitored; implemented in TensorFlow) — https://github.com/imsb-uke/scGAN
- **TensorFlow** (Framework providing the summary operations logged to Tensorboard during training) — https://github.com/imsb-uke/scGAN
- **Docker** (Provides the containerized environment (scgan:latest image) in which training and Tensorboard run reproducibly) — https://github.com/imsb-uke/scGAN

## Examples

```
python main.py --param parameters. --process --train  # then: tensorboard --logdir path/to/logs
```

## Evaluation signals

- Tensorboard shows generator and discriminator losses evolving without divergence or collapse (stable GAN training)
- Periodic non-conditional t-SNE plots show generated cells overlapping the validation-set cells, matching the paper's reference t-SNE figure
- Logging, plotting, stdout-display, and checkpoint-save frequencies in parameters. produced the expected number of events for the configured number of training steps
- Checkpoints exist at the configured save frequency, allowing training to be resumed from the path given in the . file

## Limitations

- Loss curves alone are not an acceptance criterion — the paper judges generation quality qualitatively by t-SNE overlap of generated and validation cells, and quantitatively elsewhere by MMD and classifier AUC
- Each model must be trained on a single GPU; monitoring does not replace retraining when hyperparameters (learning rate, batch size, lambda, latent dimension, normalization layer) are misconfigured
- Tensorboard requires the summary operations defined in the codebase; only the 'most important quantities in the model' are logged, not arbitrary internal tensors

## Evidence

- [readme] After or during the training, you can use the Tensorboard to monitor the losses (for instance) evolve.: "After or during the training, you can use the Tensorboard to monitor the losses (for instance) evolve."
- [readme] We have defined some summary operations on the most important quantities in the model.: "We have defined some summary operations on the most important quantities in the model."
- [readme] Also, the frequencies (how often the values are logged for the Tensorboard, the t-SNE validation plots are plotted, the loss values are displayed on the standard output, the model is saved) are defined there.: "Also, the frequencies (how often the values are logged for the Tensorboard, the t-SNE validation plots are plotted, the loss values are displayed on the standard output, the model is saved) are"
- [readme] Note that you can also resume the training by specifying the path to a checkpoint in the `.` file.: "Note that you can also resume the training by specifying the path to a checkpoint in the `.` file."
- [other] Validation: acceptance is judged by the t-SNE embedding of generated cells overlapping the validation-set cells (qualitative criterion per the paper's reference non-conditional t-SNE plot), with Tensorboard loss curves confirming stable GAN training.: [evidence span withheld — anchor status: not_found]
