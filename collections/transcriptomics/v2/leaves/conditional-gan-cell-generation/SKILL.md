---
name: conditional-gan-cell-generation
description: Use when a scRNA-seq dataset contains sparse subpopulations (e.g. a downsampled
  cluster) and you need type-specific synthetic cells to augment data for classification
  or marker analyses, rather than unconditional generation.
when_to_use_negative:
- You only need unconditional realistic cell generation without control over the produced
  cell type — use plain scGAN instead of the conditional model.
- Your data cannot be provided as .h5/.h5ad and cannot be converted (the pipeline's
  pre-processing requires these formats).
- The goal is manifold-based density-equalized data generation without type conditioning
  — a tool like SUGAR, not cscGAN, would be the appropriate generator.
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3474
tools:
- name: cscGAN
  role: Conditional GAN model used to generate realistic cells of defined types or
    subpopulations on demand
  repo: https://github.com/imsb-uke/scGAN
- name: TensorFlow
  role: Deep-learning framework in which the scGAN/cscGAN models are implemented and
    trained
  repo: https://github.com/imsb-uke/scGAN
- name: Docker
  role: Provides a reproducible environment with all dependencies; image built with
    docker build . -t scgan:latest -f dockerfile/Dockerfile
  repo: https://github.com/imsb-uke/scGAN
- name: Jupyter
  role: Notebook provided to convert mtx/tsv data to .h5 or .h5ad before pre-processing
  repo: https://github.com/imsb-uke/scGAN
- name: Tensorboard
  role: Monitors evolution of model losses during training
  repo: https://github.com/imsb-uke/scGAN
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-019-14018-z
    title: Realistic in silico generation and augmentation of single-cell RNA-seq
      data using generative adversarial networks
  source_content_hash: sha256:63e7913a0ef6b6ad057e2820398e375598481df2f6f84beff707cdf95b5d78a0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/imsb-uke/scGAN
  tools:
  - Docker
  - Jupyter
  - Python
  - TensorFlow
  - Tensorboard
  - cscGAN
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# conditional-gan-cell-generation

## Summary

Generates realistic single-cell RNA-seq cells of a defined cell type or subpopulation on demand using a conditional single-cell GAN (cscGAN), implemented in the imsb-uke/scGAN TensorFlow codebase. It enables augmentation of sparse cell populations to improve downstream classifier robustness and marker gene detection.

## When to use

Use when a scRNA-seq dataset contains sparse subpopulations (e.g. a downsampled cluster) and you need type-specific synthetic cells to augment data for classification or marker analyses, rather than unconditional generation.

## When NOT to use

- You only need unconditional realistic cell generation without control over the produced cell type — use plain scGAN instead of the conditional model.
- Your data cannot be provided as .h5/.h5ad and cannot be converted (the pipeline's pre-processing requires these formats).
- The goal is manifold-based density-equalized data generation without type conditioning — a tool like SUGAR, not cscGAN, would be the appropriate generator.

## Inputs

- scRNA-seq count matrix in .h5 or .h5ad format (e.g. Fresh 68k PBMCs (Donor A) dataset)
- mtx/tsv data convertible to .h5/.h5ad via the provided Jupyter notebook
- parameters. file with filtering parameters, dataset path, train/validation/test split sizes, clustering resolution, model type (scGAN or cscGAN), layer sizes, normalization layer, lambda, batch size, latent space dimension, and optimizer settings

## Outputs

- TFRecord-format pre-processed and split data
- trained cscGAN model checkpoints
- generated cells of a specified cluster written to a file (e.g. .h5ad) via --save_path
- Tensorboard loss summaries and t-SNE validation plots

## How to apply

Place scRNA-seq data in .h5 or .h5ad format (convert mtx/tsv with the provided Jupyter notebook), define filtering parameters, train/validation/test split sizes, clustering resolution, and model type cscGAN in the parameters. file, then run pre-processing with `python main.py --param parameters. --process` to produce TFRecord files for training. Train the conditional model with `python main.py --param parameters. --train`, monitoring losses with Tensorboard and validating realism via t-SNE plots computed on the validation set alongside an equal number of generated cells. Generate cells of a chosen cluster with the `--generate` flag, passing `--cells_no` as one integer per cluster index, plus `--model_path` and `--save_path` (e.g. an .h5ad output). The conditioning lets the generator produce specific cell types on demand, which is what distinguishes cscGAN from unconditional scGAN; a key decision point is choosing the model type (scGAN vs cscGAN) and normalization layer (batchnorm or layernorm) in the parameter file. Correctness is judged by comparing generated cells to real cells of the same type (t-SNE overlap, downstream F1 scores), not by training loss alone.

## Related tools

- **cscGAN** (Conditional GAN model used to generate realistic cells of defined types or subpopulations on demand) — https://github.com/imsb-uke/scGAN
- **TensorFlow** (Deep-learning framework in which the scGAN/cscGAN models are implemented and trained) — https://github.com/imsb-uke/scGAN
- **Docker** (Provides a reproducible environment with all dependencies; image built with docker build . -t scgan:latest -f dockerfile/Dockerfile) — https://github.com/imsb-uke/scGAN
- **Jupyter** (Notebook provided to convert mtx/tsv data to .h5 or .h5ad before pre-processing) — https://github.com/imsb-uke/scGAN
- **Tensorboard** (Monitors evolution of model losses during training) — https://github.com/imsb-uke/scGAN

## Examples

```
python main.py --param parameters. --generate --cells_no 1000 500 0 200 --model_path path/to/my/model --save_path where_to_save.h5ad
```

## Evaluation signals

- t-SNE plots of the validation set overlaid with an equal number of generated cells should show realistic overlap between real and synthetic cells of the same type
- Downstream Random Forest F1 scores after augmentation should recover toward the undownsampled baseline (paper: mean F1 0.93 for cscGAN-augmented cluster 2 at 0.5% downsampling vs 0.45 downsampled and 0.6 upsampled), validated across five random seeds
- Generation requires per-cluster integer counts via --cells_no; the number of generated cells must match the requested list (one int per cluster index)
- Training should be resumable from a specified checkpoint path in the . file, confirming checkpoint saving frequencies were configured correctly
- The generator should scale sublinearly in the number of cells when generating representations of complex multi-type datasets

## Limitations

- The mechanism enabling cscGAN to generate cells from very small subpopulations (e.g. 16 cells) is only speculated, not established.
- Conditioning on side information other than cell type (e.g. case/control samples) is untested and left for future work.
- Applications beyond classification (clustering, cell type detection, data denoising) are proposed but not demonstrated; extension to other data types (proteomics, whole genomes, imaging) is speculative.

## Evidence

- [abstract] conditional single-cell generative adversarial neural networks produce specific cell types on demand: "we propose the use of conditional single-cell generative adversarial neural networks (cscGAN) for the realistic generation of single-cell RNA-seq data"
- [readme] Generation flag usage in the scGAN README: "To generate cells and write them to a file, you can use the `--generate` flag. In that case, also use the `--cells_no` flag with a list of integers"
- [methods] Pre-processing and TFRecord conversion: "For performance issues, the data will be stored, after pre-processing and splitting, in TFRecord format, which in turn will be used for the training of the model."
- [methods] t-SNE validation of generated cells: "Note that the validation set is used to produce the t-SNE plots (each embedding is computed using the whole validation set and an equal number of generated cells)."
- [abstract] cscGAN learns non-linear dependencies and generates realistic cells of defined types: "cscGAN learns non-linear gene-gene dependencies from complex, multiple cell type samples and uses this information to generate realistic cells of defined types"
