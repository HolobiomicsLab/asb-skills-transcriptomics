---
name: tensorflow-gan-training-workflow
description: Use when a scRNA-seq subpopulation is too sparse for reliable downstream
  analysis (e.g. classifier F1 degradation after downsampling) and synthetic cell
  generation with a GAN is needed.
when_to_use_negative:
- When data is still in raw mtx/tsv format and has not been converted to .h5/.h5ad
  — convert first with the provided Jupyter notebook.
- When the goal is conditional generation on side information other than cell type
  (e.g. case/control samples), which the authors left untested for future work.
- When you need a benchmarked best generator for quality comparison and your task
  is not augmentation — the paper reports cscGAN outperforms current methods, but
  other generators like SUGAR may suit manifold-based generation goals.
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3474
tools:
- name: cscGAN
  role: Conditional single-cell GAN model trained to generate realistic cells of defined
    types on demand
  repo: github.com/imsb-uke/scGAN
- name: TensorFlow
  role: Deep-learning backend implementing and training the (c)scGAN models
  repo: github.com/imsb-uke/scGAN
- name: Docker
  role: Containerized environment with all dependencies; image built via `docker build
    . -t scgan:latest -f dockerfile/Dockerfile`
  repo: github.com/imsb-uke/scGAN
- name: Jupyter
  role: Notebook provided to convert mtx/tsv data to .h5 or .h5ad before training
  repo: github.com/imsb-uke/scGAN
- name: scGAN
  role: Unconditional variant selectable in parameters. via the model type parameter
  repo: github.com/imsb-uke/scGAN
- name: Tensorboard
  role: Monitors how training losses evolve via summary operations
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-019-14018-z
    title: Realistic in silico generation and augmentation of single-cell RNA-seq
      data using generative adversarial networks
  source_content_hash: sha256:c8e16cbbc564995731e34c1fde6b88154536bf5cda06ccff9cecbfff1dfedc76
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Docker
  - Jupyter
  - Python
  - TensorFlow
  - Tensorboard
  - cscGAN
  - scGAN
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.46153846153846156
n_claims: 13
---

# tensorflow-gan-training-workflow

## Summary

Train conditional single-cell GAN (cscGAN) models with TensorFlow to generate realistic scRNA-seq cells of defined types, enabling augmentation of sparse subpopulations (e.g. downsampled PBMC cluster 2) for downstream classifier robustness.

## When to use

Use when a scRNA-seq subpopulation is too sparse for reliable downstream analysis (e.g. classifier F1 degradation after downsampling) and synthetic cell generation with a GAN is needed.

## When NOT to use

- When data is still in raw mtx/tsv format and has not been converted to .h5/.h5ad — convert first with the provided Jupyter notebook.
- When the goal is conditional generation on side information other than cell type (e.g. case/control samples), which the authors left untested for future work.
- When you need a benchmarked best generator for quality comparison and your task is not augmentation — the paper reports cscGAN outperforms current methods, but other generators like SUGAR may suit manifold-based generation goals.

## Inputs

- scRNA-seq count matrix in .h5 or .h5ad format (e.g. Fresh 68k PBMCs Donor A dataset)
- mtx/tsv data convertible via the provided Jupyter notebook
- parameters. file with model type, architecture, optimizer, and I/O paths

## Outputs

- Trained scGAN/cscGAN model checkpoint
- Tensorboard loss logs and validation t-SNE plots
- Generated cells written to .h5ad (or .h5) via --save_path
- Augmented training sets used to compute F1 scores (f1_scores.csv)

## How to apply

Convert raw mtx/tsv data to .h5/.h5ad using the provided Jupyter notebook, then define all (hyper-)parameters (model type scGAN or cscGAN, layer sizes, normalization layer, lambda, batch size, latent-space dimension, optimizer settings) in a parameters. file. Run pre-processing with `python main.py --param parameters. --process` to filter cells and split the data (train/validation/test) into TFRecord format; set the test-set size to 0 unless needed. Train with `--train` (optionally resume from a checkpoint path specified in the . file), monitoring losses in Tensorboard and checking the validation-set t-SNE plots where each embedding uses the whole validation set plus an equal number of generated cells. Once trained, generate cells per cluster index with `--generate`, `--cells_no`, `--model_path`, and `--save_path` (e.g. an .h5ad output), repeating across random seeds. The rationale is that cscGAN conditioning produces specific cell types on demand so that augmenting downsampled populations (e.g. cluster 2 of the Fresh 68k PBMCs Donor A dataset at retention rates down to 0.5%) restores Random Forest F1 scores.

## Related tools

- **cscGAN** (Conditional single-cell GAN model trained to generate realistic cells of defined types on demand) — github.com/imsb-uke/scGAN
- **TensorFlow** (Deep-learning backend implementing and training the (c)scGAN models) — github.com/imsb-uke/scGAN
- **Docker** (Containerized environment with all dependencies; image built via `docker build . -t scgan:latest -f dockerfile/Dockerfile`) — github.com/imsb-uke/scGAN
- **Jupyter** (Notebook provided to convert mtx/tsv data to .h5 or .h5ad before training) — github.com/imsb-uke/scGAN
- **scGAN** (Unconditional variant selectable in parameters. via the model type parameter) — github.com/imsb-uke/scGAN
- **Tensorboard** (Monitors how training losses evolve via summary operations)

## Examples

```
python main.py --param parameters. --train && python main.py --param parameters. --generate --cells_no 0 1000 --model_path path/to/my/model --save_path cluster2_cells.h5ad
```

## Evaluation signals

- Validation t-SNE plots: each embedding computed on the whole validation set plus an equal number of generated cells should show generated cells overlapping their target clusters.
- Augmentation benchmark: mean Random Forest F1 at 0.5% downsampling of cluster 2 should approach ~0.93 for cscGAN-augmented vs 0.45 (downsampled) and 0.6 (upsampled), within seed-level variability over five random seeds.
- Tensorboard loss curves for generator and discriminator evolve stably across logged training steps.
- Generated file integrity: --generate with --cells_no writes the requested number of cells (one int per cluster index) to the --save_path .h5ad file.

## Limitations

- The mechanism enabling cscGAN to generate cells from very small subpopulations (e.g. 16 cells) is only speculated, not established.
- Conditioning on side information other than cell type (e.g. case/control samples) is left untested for future work.
- Applications beyond classification (clustering, cell type detection, data denoising) are proposed but not demonstrated in the paper.

## Evidence

- [abstract] we propose the use of conditional single-cell generative adversarial neural networks (cscGAN) for the realistic generation of single-cell RNA-seq data: "we propose the use of conditional single-cell generative adversarial neural networks (cscGAN) for the realistic generation of single-cell RNA-seq data"
- [readme] preprocessing into TFRecord: "For performance issues, the data will be stored, after pre-processing and splitting, in TFRecord format, which in turn will be used for the training of the model."
- [readme] type of model (scGAN or cscGAN): "The type of model (scGAN or cscGAN), the size of the different layers, the normalization layer used to condition the generation (batchnorm or layernorm), lambda, the batch size, dimension of the"
- [readme] generate cells and write them to a file: "To generate cells and write them to a file, you can use the `--generate` flag. In that case, also use the `--cells_no` flag with a list of integers corresponding to the number of cells"
- [abstract] augmentation improves classifier robustness: "Augmenting sparse cell populations with cscGAN generated cells improves downstream analyses such as the detection of marker genes, the robustness and reliability of classifiers"
