---
name: scgan-generative-model-training
description: Use when you need to train a non-conditional GAN (scGAN) on single-cell
  RNA-seq count data to generate realistic synthetic cells for augmentation or fidelity
  studies, and the data is available in .h5/.h5ad format.
when_to_use_negative:
- You need cell-type-specific generation on demand — use the conditional variant (cscGAN,
  --model type cscGAN) instead of the non-conditional scGAN.
- Your data is in mtx/tsv format — first convert it to .h5/.h5ad with the provided
  Jupyter notebook before preprocessing; the pipeline does not ingest mtx/tsv directly.
- You want geometry/manifold-based data generation rather than adversarial training
  — tools like SUGAR use diffusion-based generation and follow a different workflow.
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scGAN
  role: Non-conditional GAN model type selected in parameters. and trained with the
    --train flag to generate realistic scRNA-seq cells
  repo: https://github.com/imsb-uke/scGAN
- name: TensorFlow
  role: Deep learning framework in which the scGAN models are implemented and trained
    (run inside the provided Docker image)
  repo: https://github.com/imsb-uke/scGAN
- name: Docker
  role: Provides the reproducible working environment with all dependencies; image
    built with docker build . -t scgan:latest -f dockerfile/Dockerfile
  repo: https://github.com/imsb-uke/scGAN
- name: Tensorboard
  role: Monitors evolution of the model losses and summary quantities during training
  repo: https://github.com/imsb-uke/scGAN
- name: t-SNE
  role: Embedding used to produce validation plots comparing the whole validation
    set with an equal number of generated cells
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-019-14018-z
    title: Realistic in silico generation and augmentation of single-cell RNA-seq
      data using generative adversarial networks
  source_content_hash: sha256:265a6bfed88d110bb829b46a145104e0f3d25d0d07d1a75f7543d242d563c0c5
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
  - t-SNE
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# scgan-generative-model-training

## Summary

Train a single-cell generative adversarial network (scGAN) on preprocessed scRNA-seq data (.h5/.h5ad converted to TFRecord) to generate realistic in silico single-cell RNA-seq data, with training monitored via Tensorboard and non-conditional t-SNE validation plots.

## When to use

Use when you need to train a non-conditional GAN (scGAN) on single-cell RNA-seq count data to generate realistic synthetic cells for augmentation or fidelity studies, and the data is available in .h5/.h5ad format.

## When NOT to use

- You need cell-type-specific generation on demand — use the conditional variant (cscGAN, --model type cscGAN) instead of the non-conditional scGAN.
- Your data is in mtx/tsv format — first convert it to .h5/.h5ad with the provided Jupyter notebook before preprocessing; the pipeline does not ingest mtx/tsv directly.
- You want geometry/manifold-based data generation rather than adversarial training — tools like SUGAR use diffusion-based generation and follow a different workflow.

## Inputs

- single-cell RNA-seq data in .h5 or .h5ad format (e.g. Fresh 68k PBMCs Donor A dataset)
- parameters. file with dataset paths, GPU ids, experiment name, and model hyperparameters (model type, layer sizes, normalization layer, lambda, batch size, latent space dimension, optimizer settings)

## Outputs

- trained scGAN model checkpoints
- TFRecord-formatted preprocessed training/validation data
- Tensorboard loss logs
- non-conditional t-SNE validation plots comparing validation set and generated cells
- generated cells written to .h5ad file via the --generate flag

## How to apply

Create a parameters. file specifying the dataset path, output directories, GPU identifiers, custom experiment name, and model hyperparameters (model type scGAN, layer sizes, normalization layer batchnorm or layernorm, lambda, batch size, latent space dimension, optimizer settings such as number of steps, learning rate, and algorithm). Run preprocessing with 'python main.py --param parameters. --process' to filter the data, split it into training/validation/test sets (test set size can be 0 since it is unused), and store it in TFRecord format. Train with 'python main.py --param parameters. --train' on a single GPU using TensorFlow, optionally resuming from a checkpoint path given in the . file. During or after training, monitor losses with Tensorboard and inspect the t-SNE validation plots computed on the whole validation set versus an equal number of generated cells. Convergence is judged by stable logged losses and visual agreement of the non-conditional t-SNE embedding of generated cells with the validation set embedding.

## Related tools

- **scGAN** (Non-conditional GAN model type selected in parameters. and trained with the --train flag to generate realistic scRNA-seq cells) — https://github.com/imsb-uke/scGAN
- **TensorFlow** (Deep learning framework in which the scGAN models are implemented and trained (run inside the provided Docker image)) — https://github.com/imsb-uke/scGAN
- **Docker** (Provides the reproducible working environment with all dependencies; image built with docker build . -t scgan:latest -f dockerfile/Dockerfile) — https://github.com/imsb-uke/scGAN
- **Tensorboard** (Monitors evolution of the model losses and summary quantities during training) — https://github.com/imsb-uke/scGAN
- **t-SNE** (Embedding used to produce validation plots comparing the whole validation set with an equal number of generated cells)

## Examples

```
python main.py --param parameters. --train
```

## Evaluation signals

- Tensorboard shows stable/converged logged losses and summary quantities over training steps
- The non-conditional t-SNE embedding of an equal number of generated cells visually matches the embedding of the whole validation set (compare with reference non-cond_t-SNE.jpg)
- Preprocessing completes with data split into training/validation/test sets and stored in TFRecord format with the sizes specified in parameters.
- Training resumes correctly from a specified checkpoint path when the . parameter file points to it
- The --generate flag writes cells to the specified .h5ad save path with the requested number of cells per cluster index

## Limitations

- Each model must be trained on a single GPU; multi-GPU training of one model is not described.
- The test set is not used by the pipeline; set its size to 0 unless running custom experiments.
- The t-SNE validation check is visual/non-quantitative, and generated-cell quality is judged by embedding agreement rather than a numeric fidelity threshold.
- Data in mtx/tsv format must first be converted via the provided Jupyter notebook; the pipeline only accepts .h5/.h5ad.

## Evidence

- [readme] All the parameters regarding the training should be defined in the `.` parameter file.: "The type of model (scGAN or cscGAN), the size of the different layers, the normalization layer used to condition the generation (batchnorm or layernorm), lambda, the batch size, dimension of the"
- [readme] Preprocessing converts filtered, split data into TFRecord format for training.: "For performance issues, the data will be stored, after pre-processing and splitting, in TFRecord format, which in turn will be used for the training of the model."
- [readme] Tensorboard is used to monitor losses during or after training.: "After or during the training, you can use the Tensorboard to monitor the losses (for instance) evolve."
- [methods] Validation set is used for t-SNE plots; test set is unused.: "Note that the validation set is used to produce the t-SNE plots (each embedding is computed using the whole validation set and an equal number of generated cells). The test set is not used."
- [readme] Docker image provides the reproducible environment with all dependencies.: "To allow you to reproduce our results easily, we also provide a Dockerfile that contains a working environment containing all the dependencies."
