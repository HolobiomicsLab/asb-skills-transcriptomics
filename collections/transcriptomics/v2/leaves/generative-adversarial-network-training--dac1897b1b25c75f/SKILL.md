---
name: generative-adversarial-network-training
description: Use when the goal is to train a GAN on single-cell RNA-seq data (.h5/.h5ad)
  so that generated cells are hard to distinguish from real cells by t-SNE embedding,
  marker-gene correlation, MMD, and classifier AUC.
when_to_use_negative:
- The task is trajectory inference or manifold visualization rather than cell generation
  — the companion repos (PAGA, SUGAR) serve other purposes and SUGAR is used here
  only as a comparison baseline for generation quality
- Data is not in .h5/.h5ad and there is no plan to convert mtx/tsv first — pre-processing
  expects these formats
- You need to condition generation on side information other than cell type (e.g.,
  case/control) — the article leaves this untested for future work
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3474
tools:
- name: scGAN
  role: Non-conditional single-cell GAN trained on PBMC scRNA-seq to generate realistic
    cells without cluster conditioning
  repo: https://github.com/imsb-uke/scGAN
- name: cscGAN
  role: Conditional variant generating cells of defined types/subpopulations on demand
    via conditioning normalization (batchnorm or layernorm)
  repo: https://github.com/imsb-uke/scGAN
- name: TensorFlow
  role: Deep-learning framework implementing and training the scGAN/cscGAN models,
    run inside the provided Docker image
  repo: https://github.com/imsb-uke/scGAN
- name: Docker
  role: Reproducible environment via dockerfile/Dockerfile or the ready-to-use scgan
    container image
  repo: https://github.com/imsb-uke/scGAN
- name: Tensorboard
  role: Monitors evolution of generator/discriminator losses through defined summary
    operations during or after training
  repo: https://github.com/imsb-uke/scGAN
- name: t-SNE
  role: Produces validation embeddings computed on the whole validation set plus an
    equal number of generated cells to check overlap
- name: SUGAR
  role: Geometry-based data-generation baseline against which scGAN MMD and AUC are
    compared
  repo: https://github.com/KrishnaswamyLab/SUGAR
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-019-14018-z
    title: Realistic in silico generation and augmentation of single-cell RNA-seq
      data using generative adversarial networks
  source_content_hash: sha256:baf58ddd898fee5bfe7fff82b00bc43e75d1cb94c9560da7bbd6deae4320206e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/imsb-uke/scGAN
  tools:
  - Docker
  - Python
  - SUGAR
  - TensorFlow
  - Tensorboard
  - cscGAN
  - scGAN
  - t-SNE
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# generative-adversarial-network-training

## Summary

Training a single-cell GAN (scGAN or cscGAN) with TensorFlow to generate realistic scRNA-seq cells, validated qualitatively by t-SNE overlap with real validation cells and quantitatively by MMD and AUC metrics against real-data and SUGAR baselines.

## When to use

Use when the goal is to train a GAN on single-cell RNA-seq data (.h5/.h5ad) so that generated cells are hard to distinguish from real cells by t-SNE embedding, marker-gene correlation, MMD, and classifier AUC.

## When NOT to use

- The task is trajectory inference or manifold visualization rather than cell generation — the companion repos (PAGA, SUGAR) serve other purposes and SUGAR is used here only as a comparison baseline for generation quality
- Data is not in .h5/.h5ad and there is no plan to convert mtx/tsv first — pre-processing expects these formats
- You need to condition generation on side information other than cell type (e.g., case/control) — the article leaves this untested for future work

## Inputs

- Single-cell RNA-seq count matrix in .h5 or .h5ad format (e.g., Fresh 68k PBMCs Donor A); mtx/tsv convertible via provided Jupyter notebook
- parameters. with model type, layer sizes, normalization layer, lambda, batch size, latent-space dimension, optimizer settings, filtering/split sizes, and clustering resolution
- Path to a model checkpoint (for resuming training)

## Outputs

- Trained scGAN/cscGAN model checkpoint
- TFRecord files of filtered/split pre-processed data
- Generated cells written to .h5ad (e.g., generated_cells.h5ad)
- Tensorboard loss summaries and t-SNE validation plots

## How to apply

Configure model type (scGAN non-conditional or cscGAN), layer sizes, conditioning normalization layer (batchnorm or layernorm), lambda, batch size, latent-space dimension, and optimizer parameters (number of steps, learning rate, algorithm) in a parameters. file. Pre-process the .h5/.h5ad input (e.g., Fresh 68k PBMCs Donor A) and split into train/validation/test TFRecord files with 'python main.py --param parameters. --process', then train with '--train' (flags can be run jointly) on a single GPU, monitoring generator/discriminator losses via Tensorboard summary operations; training can be resumed from a checkpoint path specified in the parameter file. Generate cells with '--generate --cells_no <ints per cluster> --model_path ... --save_path generated_cells.h5ad'. Validate by computing non-conditional t-SNE embeddings on the whole validation set plus an equal number of generated cells and confirming the generated cells overlap the real validation cells, as in the paper's reference non-conditional t-SNE plot; quantify realism by comparing MMD and AUC of a classifier distinguishing real from generated cells against the real-data floor and SUGAR baseline.

## Related tools

- **scGAN** (Non-conditional single-cell GAN trained on PBMC scRNA-seq to generate realistic cells without cluster conditioning) — https://github.com/imsb-uke/scGAN
- **cscGAN** (Conditional variant generating cells of defined types/subpopulations on demand via conditioning normalization (batchnorm or layernorm)) — https://github.com/imsb-uke/scGAN
- **TensorFlow** (Deep-learning framework implementing and training the scGAN/cscGAN models, run inside the provided Docker image) — https://github.com/imsb-uke/scGAN
- **Docker** (Reproducible environment via dockerfile/Dockerfile or the ready-to-use scgan container image) — https://github.com/imsb-uke/scGAN
- **Tensorboard** (Monitors evolution of generator/discriminator losses through defined summary operations during or after training) — https://github.com/imsb-uke/scGAN
- **t-SNE** (Produces validation embeddings computed on the whole validation set plus an equal number of generated cells to check overlap)
- **SUGAR** (Geometry-based data-generation baseline against which scGAN MMD and AUC are compared) — https://github.com/KrishnaswamyLab/SUGAR

## Examples

```
python main.py --param parameters. --process --train && python main.py --param parameters. --generate --cells_no 1000 500 0 200 --model_path path/to/my/model --save_path where_to_save.h5ad
```

## Evaluation signals

- t-SNE embedding of generated cells overlaps the real validation-set cells (qualitative criterion per the paper's reference non-conditional t-SNE figure)
- MMD between generated and real cells approaches the real-data floor (MMD ≈ 0.03) rather than remaining high (scGAN ≈ 0.87, SUGAR ≈ high)
- AUC of the classifier distinguishing real from generated cells stays near chance (real-data AUC ≈ 0.52) instead of the scGAN value of ≈ 0.65
- Tensorboard loss curves confirm stable GAN training without mode collapse or divergence
- Marker-gene correlation of generated cells matches the real clusters used for per-cluster cell-count generation

## Limitations

- Acceptance is primarily qualitative: the t-SNE overlap criterion is judged against the paper's reference non-conditional t-SNE plot
- The mechanism enabling cscGAN to generate cells from very small subpopulations (e.g., 16 cells) is only speculated, not established
- Conditioning on side information other than cell type and applications beyond classification (clustering, denoising, cell-type detection) are left untested or proposed only
- Each model must be trained on a single GPU; test set is not used by default (size can be set to 0)

## Evidence

- [methods] To run the pre-processing, run the main.py script with the --process flag: "To run the pre-processing, run the main.py script with the --process flag"
- [readme] Validation via t-SNE with equal numbers of generated cells: "Note that the validation set is used to produce the t-SNE plots (each embedding is computed using the whole validation set and an equal number of generated cells)."
- [readme] Generation command with cells_no, model_path, save_path flags: "To generate cells and write them to a file, you can use the `--generate` flag. In that case, also use the `--cells_no` flag with a list of integers corresponding to the number of cells"
- [abstract] cscGAN learns non-linear gene-gene dependencies and generates realistic defined cell types: "cscGAN learns non-linear gene-gene dependencies from complex, multiple cell type samples and uses this information to generate realistic cells of defined types"
- [discussion] Small-subpopulation generation mechanism only speculated: "It may be surprising or even suspicious that our cscGAN is able to learn to generate cells coming from very small subpopulations (e.g. 16 cells) so well. We speculate that"
